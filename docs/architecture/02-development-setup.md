# Development Environment Setup

This guide provides comprehensive instructions for setting up the Travel Planner development environment. Multiple setup options are provided to accommodate different preferences and system configurations.

## 🔧 Prerequisites

### System Requirements

#### Minimum Requirements
- **OS**: Windows 10+, macOS 10.15+, or Linux (Ubuntu 18.04+)
- **RAM**: 8GB minimum, 16GB recommended
- **Storage**: 10GB free space
- **CPU**: Dual-core processor minimum
- **Network**: Stable internet connection for API integrations

#### Required Software
- **Git**: Version control system
- **Node.js**: v18.0+ (LTS recommended)
- **Python**: v3.9+ (3.11 recommended)
- **Docker**: v20.10+ (for containerized development)
- **PostgreSQL**: v13+ (if not using Docker)

### Pre-Installation Checklist

```bash
# Verify installations
git --version
node --version
npm --version
python --version
docker --version
psql --version
```

Expected output should show version numbers for all tools.

## 🚀 Setup Options

### Option 1: Docker Development Environment (Recommended)

This is the fastest way to get started with minimal system configuration.

#### Step 1: Clone Repository
```bash
git clone https://github.com/your-username/travel-planner.git
cd travel-planner
```

#### Step 2: Environment Configuration
```bash
# Copy environment template
cp .env.example .env.development

# Edit environment variables
# Windows
notepad .env.development
# macOS/Linux
nano .env.development
```

#### Step 3: Docker Setup
```bash
# Build and start all services
docker-compose -f docker-compose.dev.yml up --build

# Run in background
docker-compose -f docker-compose.dev.yml up -d

# View logs
docker-compose -f docker-compose.dev.yml logs -f
```

#### Step 4: Database Setup
```bash
# Run migrations
docker-compose exec backend python manage.py migrate

# Create superuser
docker-compose exec backend python manage.py createsuperuser

# Load sample data (optional)
docker-compose exec backend python manage.py loaddata fixtures/sample_data.json
```

#### Step 5: Verify Installation
- Frontend: http://localhost:3000
- Backend API: http://localhost:8000/api
- Django Admin: http://localhost:8000/admin
- Database: localhost:5432

### Option 2: Native Development Environment

For developers who prefer running services directly on their system.

#### Step 1: Python Environment Setup
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

# Upgrade pip
pip install --upgrade pip

# Install dependencies
cd backend
pip install -r requirements.dev.txt
```

#### Step 2: Node.js Environment Setup
```bash
cd frontend
npm install

# Install global tools (optional)
npm install -g @storybook/cli
npm install -g eslint
```

#### Step 3: Database Setup
```bash
# Install PostgreSQL (if not already installed)
# Windows: Download from postgresql.org
# macOS: brew install postgresql
# Ubuntu: sudo apt install postgresql postgresql-contrib

# Create database
sudo -u postgres psql
CREATE DATABASE travel_planner_dev;
CREATE USER travel_dev WITH ENCRYPTED PASSWORD 'dev_password';
GRANT ALL PRIVILEGES ON DATABASE travel_planner_dev TO travel_dev;
ALTER USER travel_dev CREATEDB;
\q
```

#### Step 4: Redis Setup (Optional)
```bash
# Install Redis
# Windows: Use Redis for Windows or WSL
# macOS: brew install redis
# Ubuntu: sudo apt install redis-server

# Start Redis
# macOS/Linux: redis-server
# Ubuntu service: sudo systemctl start redis-server
```

#### Step 5: Environment Configuration
```bash
# Backend environment
cd backend
cp .env.example .env.development

# Edit .env.development with your database credentials
DATABASE_URL=postgresql://travel_dev:dev_password@localhost:5432/travel_planner_dev
REDIS_URL=redis://localhost:6379/0
```

#### Step 6: Run Development Servers
```bash
# Terminal 1: Backend
cd backend
python manage.py migrate
python manage.py runserver

# Terminal 2: Frontend
cd frontend
npm start

# Terminal 3: Redis (if running locally)
redis-server
```

### Option 3: VS Code Dev Containers

For VS Code users who want a consistent development environment.

#### Step 1: Install Extensions
```json
{
  "recommendations": [
    "ms-vscode-remote.remote-containers",
    "ms-python.python",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next"
  ]
}
```

#### Step 2: Dev Container Configuration
The project includes `.devcontainer/devcontainer.json`:

```json
{
  "name": "Travel Planner Dev",
  "dockerComposeFile": ["../docker-compose.dev.yml"],
  "service": "backend",
  "workspaceFolder": "/workspace",
  "settings": {
    "python.defaultInterpreterPath": "/usr/local/bin/python",
    "python.linting.enabled": true,
    "python.linting.pylintEnabled": true
  },
  "extensions": [
    "ms-python.python",
    "ms-python.vscode-pylance",
    "bradlc.vscode-tailwindcss"
  ],
  "forwardPorts": [3000, 8000, 5432, 6379],
  "postCreateCommand": "pip install -r requirements.dev.txt && python manage.py migrate"
}
```

#### Step 3: Open in Container
1. Open VS Code
2. Install Remote-Containers extension
3. Press `Ctrl+Shift+P` → "Remote-Containers: Open Folder in Container"
4. Select the project folder

## ⚙️ IDE Configuration

### VS Code Setup

#### Recommended Extensions
```json
{
  "recommendations": [
    "ms-python.python",
    "ms-python.vscode-pylance",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "ms-vscode-remote.remote-containers",
    "ms-vscode.vscode-json",
    "redhat.vscode-yaml",
    "ms-azuretools.vscode-docker"
  ]
}
```

#### Workspace Settings
```json
{
  "python.defaultInterpreterPath": "./backend/venv/bin/python",
  "python.linting.enabled": true,
  "python.linting.pylintEnabled": true,
  "python.linting.flake8Enabled": true,
  "python.formatting.provider": "black",
  "typescript.preferences.quoteStyle": "single",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  "files.exclude": {
    "**/__pycache__": true,
    "**/node_modules": true,
    "**/.git": true
  }
}
```

#### Debug Configuration
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Django",
      "type": "python",
      "request": "launch",
      "program": "${workspaceFolder}/backend/manage.py",
      "args": ["runserver"],
      "django": true,
      "cwd": "${workspaceFolder}/backend"
    },
    {
      "name": "React",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceFolder}/frontend",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["start"]
    }
  ]
}
```

### PyCharm Setup

#### Project Configuration
1. Open PyCharm
2. File → Open → Select project root directory
3. Configure Python interpreter:
   - File → Settings → Project → Python Interpreter
   - Select virtual environment from `backend/venv`

#### Run Configurations
```xml
<!-- Django Server -->
<configuration name="Django Server" type="Python">
  <module name="travel-planner" />
  <option name="INTERPRETER_OPTIONS" value="" />
  <option name="PARENT_ENVS" value="true" />
  <envs>
    <env name="DJANGO_SETTINGS_MODULE" value="travel_planner.settings.development" />
  </envs>
  <option name="SDK_HOME" value="$PROJECT_DIR$/backend/venv/bin/python" />
  <option name="WORKING_DIRECTORY" value="$PROJECT_DIR$/backend" />
  <option name="IS_MODULE_SDK" value="false" />
  <option name="ADD_CONTENT_ROOTS" value="true" />
  <option name="ADD_SOURCE_ROOTS" value="true" />
  <option name="SCRIPT_NAME" value="manage.py" />
  <option name="PARAMETERS" value="runserver" />
</configuration>
```

## 🔑 Environment Variables

### Development Environment Variables

#### Backend (.env.development)
```bash
# Django Settings
DEBUG=True
SECRET_KEY=your-development-secret-key
DJANGO_SETTINGS_MODULE=travel_planner.settings.development

# Database Configuration
DATABASE_URL=postgresql://travel_dev:dev_password@localhost:5432/travel_planner_dev
# For Docker: postgresql://travel_user:travel_password@db:5432/travel_planner

# Redis Configuration
REDIS_URL=redis://localhost:6379/0
# For Docker: redis://redis:6379/0

# External API Keys (Development)
OPENWEATHER_API_KEY=your-openweather-dev-key
OPENROUTESERVICE_API_KEY=your-openrouteservice-dev-key
FOURSQUARE_API_KEY=your-foursquare-dev-key

# OAuth Configuration (Development)
GOOGLE_OAUTH2_CLIENT_ID=your-google-dev-client-id
GOOGLE_OAUTH2_CLIENT_SECRET=your-google-dev-client-secret

# Email Configuration (Development)
EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend
EMAIL_HOST=smtp.mailtrap.io
EMAIL_PORT=2525
EMAIL_HOST_USER=your-mailtrap-user
EMAIL_HOST_PASSWORD=your-mailtrap-password

# Logging
LOG_LEVEL=DEBUG

# CORS Settings
CORS_ALLOW_ALL_ORIGINS=True
```

#### Frontend (.env.development)
```bash
# API Configuration
REACT_APP_API_URL=http://localhost:8000/api
REACT_APP_WS_URL=ws://localhost:8000/ws

# External Services
REACT_APP_MAPBOX_TOKEN=your-mapbox-dev-token
REACT_APP_GOOGLE_MAPS_API_KEY=your-google-maps-dev-key

# OAuth Configuration
REACT_APP_GOOGLE_CLIENT_ID=your-google-dev-client-id

# Feature Flags
REACT_APP_ENABLE_PWA=true
REACT_APP_ENABLE_OFFLINE=true
REACT_APP_ENABLE_ANALYTICS=false

# Development Settings
REACT_APP_LOG_LEVEL=debug
GENERATE_SOURCEMAP=true
```

### API Key Setup Guide

#### OpenWeatherMap API
1. Visit https://openweathermap.org/api
2. Sign up for free account
3. Generate API key (free tier: 1000 calls/day)
4. Add to `.env.development`

#### OpenRouteService API
1. Visit https://openrouteservice.org/
2. Create free account
3. Generate API key (free tier: 2000 requests/day)
4. Add to `.env.development`

#### Google OAuth Setup
1. Visit Google Cloud Console
2. Create new project or select existing
3. Enable Google+ API
4. Create OAuth 2.0 credentials
5. Add authorized origins and redirect URIs
6. Add client ID and secret to environment

## 🐳 Docker Configuration

### Development Docker Compose

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: travel_planner
      POSTGRES_USER: travel_user
      POSTGRES_PASSWORD: travel_password
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data
      - ./scripts/init-dev-db.sql:/docker-entrypoint-initdb.d/init-db.sql
    ports:
      - "5432:5432"
    networks:
      - travel-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_dev_data:/data
    networks:
      - travel-network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./backend:/app
      - backend_static:/app/staticfiles
    ports:
      - "8000:8000"
    environment:
      - DEBUG=True
      - DATABASE_URL=postgresql://travel_user:travel_password@db:5432/travel_planner
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - db
      - redis
    networks:
      - travel-network

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    command: npm start
    volumes:
      - ./frontend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:8000/api
      - CHOKIDAR_USEPOLLING=true
    depends_on:
      - backend
    networks:
      - travel-network

volumes:
  postgres_dev_data:
  redis_dev_data:
  backend_static:

networks:
  travel-network:
    driver: bridge
```

### Development Dockerfiles

#### Backend Dockerfile (Dockerfile.dev)
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.dev.txt .
RUN pip install --no-cache-dir -r requirements.dev.txt

# Copy application code
COPY . .

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser
RUN chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8000

# Development command (will be overridden in compose)
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

#### Frontend Dockerfile (Dockerfile.dev)
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Development command (will be overridden in compose)
CMD ["npm", "start"]
```

## 🧪 Testing Setup

### Backend Testing
```bash
# Install test dependencies
pip install -r requirements.test.txt

# Run tests
python manage.py test

# Run with coverage
coverage run --source='.' manage.py test
coverage report
coverage html
```

### Frontend Testing
```bash
# Run unit tests
npm test

# Run with coverage
npm test -- --coverage

# Run E2E tests
npm run test:e2e
```

### Test Database Setup
```python
# settings/test.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': ':memory:',
    }
}

# Disable migrations for faster tests
class DisableMigrations:
    def __contains__(self, item):
        return True
    
    def __getitem__(self, item):
        return None

MIGRATION_MODULES = DisableMigrations()
```

## 🛠️ Development Tools

### Code Quality Tools

#### Python (Backend)
```bash
# Install development tools
pip install black isort flake8 mypy pylint

# Format code
black .
isort .

# Lint code
flake8 .
pylint **/*.py

# Type checking
mypy .
```

#### JavaScript/TypeScript (Frontend)
```bash
# Install development tools
npm install --save-dev eslint prettier @typescript-eslint/parser

# Format code
npm run format

# Lint code
npm run lint

# Type checking
npm run type-check
```

### Pre-commit Hooks
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.0.1
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files

  - repo: https://github.com/psf/black
    rev: 21.9b0
    hooks:
      - id: black

  - repo: https://github.com/pycqa/isort
    rev: 5.9.3
    hooks:
      - id: isort

  - repo: https://github.com/pycqa/flake8
    rev: 3.9.2
    hooks:
      - id: flake8
```

## 🚦 Verification Steps

### Health Check Script
```bash
#!/bin/bash
# scripts/health-check-dev.sh

echo "🔍 Checking development environment..."

# Check if services are running
check_service() {
    local service_name=$1
    local url=$2
    local expected_status=$3
    
    echo "Checking $service_name..."
    status=$(curl -s -o /dev/null -w "%{http_code}" $url)
    
    if [ "$status" = "$expected_status" ]; then
        echo "✅ $service_name is healthy"
    else
        echo "❌ $service_name is not responding (got $status, expected $expected_status)"
        return 1
    fi
}

# Check frontend
check_service "Frontend" "http://localhost:3000" "200"

# Check backend API
check_service "Backend API" "http://localhost:8000/api/health/" "200"

# Check database connection
echo "Checking database connection..."
if docker-compose exec -T backend python manage.py check --database default; then
    echo "✅ Database connection is healthy"
else
    echo "❌ Database connection failed"
    exit 1
fi

echo "🎉 All services are healthy!"
```

### Development Checklist

- [ ] Repository cloned successfully
- [ ] Environment variables configured
- [ ] Database running and accessible
- [ ] Backend server running on port 8000
- [ ] Frontend server running on port 3000
- [ ] Redis server running (if using cache)
- [ ] API endpoints responding correctly
- [ ] Database migrations applied
- [ ] Sample data loaded (optional)
- [ ] Tests passing
- [ ] Code quality tools configured
- [ ] IDE/editor configured with proper extensions

## 🐛 Common Issues and Solutions

### Port Already in Use
```bash
# Find process using port
# Windows
netstat -ano | findstr :8000
# macOS/Linux
lsof -i :8000

# Kill process
# Windows
taskkill /PID <PID> /F
# macOS/Linux
kill -9 <PID>
```

### Database Connection Issues
```bash
# Check PostgreSQL status
# Windows: Check Services app
# macOS: brew services list | grep postgresql
# Linux: sudo systemctl status postgresql

# Reset database
docker-compose down -v
docker-compose up db
```

### Node Module Issues
```bash
# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Python Environment Issues
```bash
# Recreate virtual environment
rm -rf venv
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -r requirements.dev.txt
```

## 📚 Additional Resources

### Documentation Links
- [Django Documentation](https://docs.djangoproject.com/)
- [React Documentation](https://reactjs.org/docs/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Documentation](https://docs.docker.com/)

### Development Guides
- [Django Best Practices](../backend/06-best-practices.md)
- [React Development Guidelines](../frontend/05-development-guidelines.md)
- [API Design Standards](../backend/05-api-reference.md)
- [Testing Strategies](../backend/04-testing.md)

---

**Last Updated**: August 16, 2025  
**Version**: 1.0.0  
**Next Review**: August 30, 2025