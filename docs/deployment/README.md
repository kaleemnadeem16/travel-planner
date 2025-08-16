# Deployment & Infrastructure

## Deployment Strategy

### Development Environment
- **Local Development**: Docker Compose for easy setup
- **Frontend**: React development server (localhost:3000)
- **Backend**: Django/Flask development server (localhost:8000)
- **Database**: PostgreSQL in Docker container
- **Cache**: Redis in Docker container (optional)

### Production Environment Options

#### Option 1: Free Tier Cloud Deployment
- **Frontend**: Vercel, Netlify, or GitHub Pages
- **Backend**: Railway, Render, or Heroku free tier
- **Database**: PostgreSQL on Railway, Render, or Supabase
- **Static Files**: Cloudinary or AWS S3 free tier

#### Option 2: VPS Deployment
- **Server**: DigitalOcean Droplet, Linode, or Vultr VPS
- **Web Server**: Nginx as reverse proxy
- **Application Server**: Gunicorn for Django, uWSGI for Flask
- **Database**: PostgreSQL on same server
- **SSL**: Let's Encrypt certificates

#### Option 3: Container Deployment
- **Platform**: DigitalOcean App Platform, Render, or Fly.io
- **Containerization**: Docker containers
- **Orchestration**: Docker Compose or Kubernetes (for scaling)

## Docker Setup

### Development Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: travel_planner
      POSTGRES_USER: travel_user
      POSTGRES_PASSWORD: travel_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data

  backend:
    build: ./backend
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./backend:/app
    ports:
      - "8000:8000"
    environment:
      - DEBUG=True
      - DATABASE_URL=postgresql://travel_user:travel_password@db:5432/travel_planner
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis

  frontend:
    build: ./frontend
    command: npm start
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:8000/api
    depends_on:
      - backend

volumes:
  postgres_data:
  redis_data:
```

### Backend Dockerfile

```dockerfile
# backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser
RUN chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8000

# Run application
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "travel_planner.wsgi:application"]
```

### Frontend Dockerfile

```dockerfile
# frontend/Dockerfile
FROM node:18-alpine as build

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built application
COPY --from=build /app/build /usr/share/nginx/html

# Copy Nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Frontend Nginx Configuration

```nginx
# frontend/nginx.conf
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;

        # Handle React Router
        location / {
            try_files $uri $uri/ /index.html;
        }

        # API proxy (for development)
        location /api/ {
            proxy_pass http://backend:8000/api/;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        # Static files caching
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }
}
```

## Production Deployment

### Backend Production Configuration

```python
# backend/settings/production.py
import os
from .base import *

DEBUG = False
ALLOWED_HOSTS = [
    'your-domain.com',
    'www.your-domain.com',
    'api.your-domain.com',
]

# Database configuration
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DB_NAME'),
        'USER': os.getenv('DB_USER'),
        'PASSWORD': os.getenv('DB_PASSWORD'),
        'HOST': os.getenv('DB_HOST'),
        'PORT': os.getenv('DB_PORT', '5432'),
        'OPTIONS': {
            'sslmode': 'require',
        },
    }
}

# Redis configuration
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': os.getenv('REDIS_URL'),
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

# Security settings
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True

# Static files (use Whitenoise or CDN)
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'

# CORS settings
CORS_ALLOWED_ORIGINS = [
    "https://your-domain.com",
    "https://www.your-domain.com",
]

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': '/var/log/travel_planner/django.log',
            'formatter': 'verbose',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },
    'root': {
        'handlers': ['file', 'console'],
        'level': 'INFO',
    },
}
```

### Environment Variables

```bash
# .env.production
DEBUG=False
SECRET_KEY=your-secret-key-here
DATABASE_URL=postgresql://user:password@host:port/dbname
REDIS_URL=redis://host:port/0

# External API keys
OPENWEATHER_API_KEY=your-openweather-key
OPENROUTESERVICE_API_KEY=your-openrouteservice-key
FOURSQUARE_API_KEY=your-foursquare-key

# OAuth credentials
GOOGLE_OAUTH2_CLIENT_ID=your-google-client-id
GOOGLE_OAUTH2_CLIENT_SECRET=your-google-client-secret

# Email configuration (for notifications)
EMAIL_HOST=smtp.sendgrid.net
EMAIL_HOST_USER=apikey
EMAIL_HOST_PASSWORD=your-sendgrid-api-key
EMAIL_PORT=587
EMAIL_USE_TLS=True
```

### Nginx Production Configuration

```nginx
# /etc/nginx/sites-available/travel-planner
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
    ssl_prefer_server_ciphers off;

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/m;
    limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

    # Frontend (React build)
    location / {
        root /var/www/travel-planner/frontend/build;
        try_files $uri $uri/ /index.html;
        
        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # Backend API
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # CORS headers
        add_header Access-Control-Allow-Origin "https://your-domain.com";
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
        add_header Access-Control-Allow-Headers "Accept, Authorization, Content-Type, X-CSRF-Token";
    }

    # Auth endpoints with stricter rate limiting
    location /api/auth/ {
        limit_req zone=login burst=5 nodelay;
        
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Django admin (optional, secure access)
    location /admin/ {
        allow 192.168.1.0/24;  # Your IP range
        deny all;
        
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## Database Management

### PostgreSQL Production Setup

```bash
# Install PostgreSQL
sudo apt update
sudo apt install postgresql postgresql-contrib

# Create database and user
sudo -u postgres psql
CREATE DATABASE travel_planner;
CREATE USER travel_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE travel_planner TO travel_user;
ALTER USER travel_user CREATEDB;
\q

# Configure PostgreSQL
sudo nano /etc/postgresql/15/main/postgresql.conf
# Set: shared_preload_libraries = 'pg_stat_statements'

sudo nano /etc/postgresql/15/main/pg_hba.conf
# Add: host travel_planner travel_user 127.0.0.1/32 md5

sudo systemctl restart postgresql
```

### Database Backup Strategy

```bash
#!/bin/bash
# backup-db.sh

DB_NAME="travel_planner"
DB_USER="travel_user"
BACKUP_DIR="/var/backups/travel-planner"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Full database backup
pg_dump -U $DB_USER -h localhost $DB_NAME | gzip > $BACKUP_DIR/full_backup_$TIMESTAMP.sql.gz

# Keep only last 7 days of backups
find $BACKUP_DIR -name "full_backup_*.sql.gz" -mtime +7 -delete

# Upload to cloud storage (optional)
# aws s3 cp $BACKUP_DIR/full_backup_$TIMESTAMP.sql.gz s3://your-backup-bucket/database/
```

### Database Migration Strategy

```bash
# Production deployment script
#!/bin/bash

# Backup database before migration
./backup-db.sh

# Run migrations
cd /var/www/travel-planner/backend
source venv/bin/activate
python manage.py migrate --settings=travel_planner.settings.production

# Collect static files
python manage.py collectstatic --noinput --settings=travel_planner.settings.production

# Restart application
sudo systemctl restart gunicorn
sudo systemctl restart nginx
```

## Monitoring & Logging

### Application Monitoring

```python
# monitoring/health_check.py
from django.http import JsonResponse
from django.db import connection
from django.core.cache import cache
import redis

def health_check(request):
    """Health check endpoint for monitoring"""
    status = {
        'status': 'healthy',
        'database': 'ok',
        'cache': 'ok',
        'external_apis': 'ok'
    }
    
    # Database check
    try:
        with connection.cursor() as cursor:
            cursor.execute("SELECT 1")
    except Exception as e:
        status['database'] = f'error: {str(e)}'
        status['status'] = 'unhealthy'
    
    # Cache check
    try:
        cache.set('health_check', 'ok', 10)
        if cache.get('health_check') != 'ok':
            raise Exception('Cache write/read failed')
    except Exception as e:
        status['cache'] = f'error: {str(e)}'
        status['status'] = 'unhealthy'
    
    return JsonResponse(status)
```

### Log Management

```bash
# logrotate configuration
# /etc/logrotate.d/travel-planner

/var/log/travel_planner/*.log {
    daily
    missingok
    rotate 30
    compress
    notifempty
    create 644 www-data www-data
    postrotate
        systemctl reload nginx
    endscript
}
```

### Monitoring with Prometheus (Optional)

```yaml
# docker-compose.monitoring.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  grafana_data:
```

## Deployment Scripts

### Automated Deployment Script

```bash
#!/bin/bash
# deploy.sh

set -e

PROJECT_DIR="/var/www/travel-planner"
BACKEND_DIR="$PROJECT_DIR/backend"
FRONTEND_DIR="$PROJECT_DIR/frontend"

echo "Starting deployment..."

# Pull latest code
cd $PROJECT_DIR
git pull origin main

# Backend deployment
echo "Deploying backend..."
cd $BACKEND_DIR
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate --settings=travel_planner.settings.production
python manage.py collectstatic --noinput --settings=travel_planner.settings.production

# Frontend deployment
echo "Deploying frontend..."
cd $FRONTEND_DIR
npm ci --production
npm run build

# Copy built frontend to nginx directory
sudo cp -r build/* /var/www/travel-planner/frontend/

# Restart services
echo "Restarting services..."
sudo systemctl restart gunicorn
sudo systemctl restart nginx

echo "Deployment completed successfully!"
```

### Systemd Service Configuration

```ini
# /etc/systemd/system/travel-planner.service
[Unit]
Description=Travel Planner Django Application
After=network.target

[Service]
Type=notify
User=www-data
Group=www-data
WorkingDirectory=/var/www/travel-planner/backend
ExecStart=/var/www/travel-planner/backend/venv/bin/gunicorn \
    --bind 127.0.0.1:8000 \
    --workers 3 \
    --timeout 120 \
    --access-logfile /var/log/travel_planner/access.log \
    --error-logfile /var/log/travel_planner/error.log \
    travel_planner.wsgi:application
ExecReload=/bin/kill -s HUP $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## SSL/TLS Setup

### Let's Encrypt Certificate

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Auto-renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

## TODO: Implementation Tasks

- [ ] Set up Docker development environment
- [ ] Configure production server (VPS or cloud)
- [ ] Set up PostgreSQL database
- [ ] Configure Nginx reverse proxy
- [ ] Obtain SSL certificates
- [ ] Set up database backup system
- [ ] Configure monitoring and logging
- [ ] Create deployment scripts
- [ ] Set up CI/CD pipeline (GitHub Actions)
- [ ] Configure domain name and DNS
- [ ] Set up error tracking (Sentry)
- [ ] Implement health checks
- [ ] Configure auto-scaling (if using cloud platform)