# Environment Setup & Configuration Guide

## 🎯 Complete Environment Configuration

This guide provides comprehensive environment setup for the Travel Planner multi-agent system, including development, staging, and Oracle Cloud ARM production environments.

## 📋 Environment Template (.env)

### Complete .env Configuration

```bash
# =====================================================
# TRAVEL PLANNER MULTI-AGENT SYSTEM CONFIGURATION
# =====================================================

# ===========================================
# ENVIRONMENT & DEPLOYMENT
# ===========================================
ENVIRONMENT=development  # development, staging, production
DEBUG=true
APP_VERSION=1.0.0
DEPLOYMENT_ID=local-dev
LOG_LEVEL=DEBUG

# ===========================================
# DATABASE CONFIGURATION
# ===========================================

# PostgreSQL Primary Database
DATABASE_URL=postgresql://travel_user:travel_pass@localhost:5432/travel_planner
DB_HOST=localhost
DB_PORT=5432
DB_NAME=travel_planner
DB_USER=travel_user
DB_PASSWORD=travel_pass
DB_POOL_SIZE=20
DB_MAX_OVERFLOW=30
DB_POOL_RECYCLE=3600

# Redis Cache & Message Queue
REDIS_URL=redis://localhost:6379/0
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
REDIS_DB=0
REDIS_MAX_CONNECTIONS=50

# Qdrant Vector Database
QDRANT_URL=http://localhost:6333
QDRANT_API_KEY=
QDRANT_COLLECTION_SIZE=1536  # OpenAI embedding size

# ===========================================
# AI MODEL PROVIDER CONFIGURATION
# ===========================================

# OpenAI Configuration
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_ORG_ID=
OPENAI_MAX_RETRIES=3
OPENAI_TIMEOUT=30

# Anthropic Configuration
ANTHROPIC_API_KEY=your_anthropic_api_key_here
ANTHROPIC_MAX_RETRIES=3
ANTHROPIC_TIMEOUT=30

# Azure OpenAI Configuration
AZURE_OPENAI_API_KEY=your_azure_openai_key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
AZURE_OPENAI_API_VERSION=2024-02-15-preview

# Google AI Configuration (Optional)
GOOGLE_API_KEY=your_google_ai_key
GOOGLE_PROJECT_ID=your_project_id

# ===========================================
# AGENT MODEL CONFIGURATION
# ===========================================

# Default Model Settings by Tier
DEFAULT_MODEL_TIER1=gpt-5
DEFAULT_MODEL_TIER2=gpt-5-mini
DEFAULT_MODEL_TIER3=gpt-5-nano
DEFAULT_PROVIDER=openai

# Default Token Limits by Tier
DEFAULT_TOKENS_TIER1=8192
DEFAULT_TOKENS_TIER2=6144
DEFAULT_TOKENS_TIER3=4096

# Default Temperature Settings
DEFAULT_TEMPERATURE_TIER1=0.7
DEFAULT_TEMPERATURE_TIER2=0.7
DEFAULT_TEMPERATURE_TIER3=0.3

# ===========================================
# AGENT-SPECIFIC MODEL OVERRIDES
# ===========================================

# Planning Agent (Tier 1 - Complex reasoning)
PLANNING_AGENT_MODEL=gpt-5
PLANNING_AGENT_PROVIDER=openai
PLANNING_AGENT_MAX_TOKENS=8192
PLANNING_AGENT_TEMPERATURE=0.7
PLANNING_AGENT_TIMEOUT=45
PLANNING_AGENT_MAX_RETRIES=3

# Transport Agent (Tier 1 - Complex logistics)
TRANSPORT_AGENT_MODEL=gpt-5
TRANSPORT_AGENT_PROVIDER=openai
TRANSPORT_AGENT_MAX_TOKENS=8192
TRANSPORT_AGENT_TEMPERATURE=0.6
TRANSPORT_AGENT_TIMEOUT=30
TRANSPORT_AGENT_MAX_RETRIES=3

# Location Agent (Tier 2 - Research & recommendations)
LOCATION_AGENT_MODEL=gpt-5-mini
LOCATION_AGENT_PROVIDER=openai
LOCATION_AGENT_MAX_TOKENS=6144
LOCATION_AGENT_TEMPERATURE=0.7
LOCATION_AGENT_TIMEOUT=25
LOCATION_AGENT_MAX_RETRIES=3

# Accommodation Agent (Tier 2 - Search & filtering)
ACCOMMODATION_AGENT_MODEL=gpt-5-mini
ACCOMMODATION_AGENT_PROVIDER=openai
ACCOMMODATION_AGENT_MAX_TOKENS=6144
ACCOMMODATION_AGENT_TEMPERATURE=0.7
ACCOMMODATION_AGENT_TIMEOUT=25
ACCOMMODATION_AGENT_MAX_RETRIES=3

# Activity Agent (Tier 2 - Creative recommendations)
ACTIVITY_AGENT_MODEL=gpt-5-mini
ACTIVITY_AGENT_PROVIDER=openai
ACTIVITY_AGENT_MAX_TOKENS=6144
ACTIVITY_AGENT_TEMPERATURE=0.8
ACTIVITY_AGENT_TIMEOUT=25
ACTIVITY_AGENT_MAX_RETRIES=3

# Budget Agent (Tier 3 - Calculations & optimization)
BUDGET_AGENT_MODEL=gpt-5-nano
BUDGET_AGENT_PROVIDER=openai
BUDGET_AGENT_MAX_TOKENS=4096
BUDGET_AGENT_TEMPERATURE=0.3
BUDGET_AGENT_TIMEOUT=15
BUDGET_AGENT_MAX_RETRIES=2

# Weather Agent (Tier 3 - Data retrieval & formatting)
WEATHER_AGENT_MODEL=gpt-5-nano
WEATHER_AGENT_PROVIDER=openai
WEATHER_AGENT_MAX_TOKENS=2048
WEATHER_AGENT_TEMPERATURE=0.1
WEATHER_AGENT_TIMEOUT=10
WEATHER_AGENT_MAX_RETRIES=2

# ===========================================
# COST MANAGEMENT & OPTIMIZATION
# ===========================================

# Cost Limits by Subscription Tier
FREE_TIER_DAILY_LIMIT=1.00
FREE_TIER_REQUEST_LIMIT=0.10
BASIC_TIER_DAILY_LIMIT=10.00
BASIC_TIER_REQUEST_LIMIT=0.50
PREMIUM_TIER_DAILY_LIMIT=50.00
PREMIUM_TIER_REQUEST_LIMIT=2.00

# Cost Optimization Settings
COST_OPTIMIZATION_ENABLED=true
AUTO_MODEL_SWITCHING=true
COST_ALERT_THRESHOLD=0.80  # Alert at 80% of limit
AGGRESSIVE_COST_OPTIMIZATION=false

# ===========================================
# AGENT SYSTEM CONFIGURATION
# ===========================================

# Orchestrator Settings
MAX_CONCURRENT_AGENTS=10
AGENT_TIMEOUT_SECONDS=30
ORCHESTRATOR_RETRY_ATTEMPTS=3
CIRCUIT_BREAKER_ENABLED=true
CIRCUIT_BREAKER_THRESHOLD=5
CIRCUIT_BREAKER_TIMEOUT=300

# Celery Worker Configuration
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0
CELERY_WORKER_CONCURRENCY=4
CELERY_TASK_SERIALIZER=json
CELERY_ACCEPT_CONTENT=["json"]
CELERY_TIMEZONE=UTC
CELERY_ENABLE_UTC=true

# Agent Pool Configuration
PLANNING_AGENT_WORKERS=2
TRANSPORT_AGENT_WORKERS=2
LOCATION_AGENT_WORKERS=3
ACCOMMODATION_AGENT_WORKERS=3
ACTIVITY_AGENT_WORKERS=3
BUDGET_AGENT_WORKERS=1
WEATHER_AGENT_WORKERS=1

# ===========================================
# COMMUNICATION & COORDINATION
# ===========================================

# Message Queue Settings
MESSAGE_QUEUE_TYPE=redis
MESSAGE_RETENTION_HOURS=24
MAX_MESSAGE_SIZE_KB=64
COMPRESSION_ENABLED=true
ENCRYPTION_ENABLED=true

# WebSocket Configuration
WS_ENABLED=true
WS_MAX_CONNECTIONS=1000
WS_HEARTBEAT_INTERVAL=30
WS_MESSAGE_BUFFER_SIZE=100

# ===========================================
# CONTEXT MANAGEMENT
# ===========================================

# Context Storage & Management
CONTEXT_COMPRESSION_ENABLED=true
LAZY_LOADING_ENABLED=true
SHARED_CONTEXT_POOL_ENABLED=true
CONTEXT_TTL_SECONDS=3600
MAX_CONTEXT_SIZE_MB=10

# Vector Storage Configuration
VECTOR_CACHE_ENABLED=true
VECTOR_CACHE_TTL_HOURS=24
SIMILARITY_THRESHOLD=0.8
MAX_SIMILAR_RESULTS=5

# ===========================================
# MONITORING & OBSERVABILITY
# ===========================================

# LangSmith Configuration
LANGSMITH_API_KEY=your_langsmith_api_key
LANGSMITH_PROJECT=travel-planner-agents
LANGSMITH_ENABLED=true
LANGSMITH_TRACING_ENABLED=true

# Prometheus Metrics
PROMETHEUS_ENABLED=true
PROMETHEUS_PORT=8001
PROMETHEUS_METRICS_PATH=/metrics

# Application Monitoring
METRICS_ENABLED=true
DETAILED_LOGGING=false
COST_TRACKING_ENABLED=true
PERFORMANCE_ALERTS_ENABLED=true
USER_ANALYTICS_ENABLED=true

# Health Check Configuration
HEALTH_CHECK_ENABLED=true
HEALTH_CHECK_INTERVAL=30
HEALTH_CHECK_TIMEOUT=10

# ===========================================
# SECURITY & AUTHENTICATION
# ===========================================

# JWT Configuration
JWT_SECRET_KEY=your_super_secret_jwt_key_here_change_in_production
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=60
JWT_REFRESH_TOKEN_EXPIRE_DAYS=7

# API Security
API_RATE_LIMIT_ENABLED=true
API_RATE_LIMIT_REQUESTS=100
API_RATE_LIMIT_WINDOW=3600  # 1 hour
CORS_ENABLED=true
CORS_ORIGINS=["http://localhost:3000", "https://yourdomain.com"]

# Password Security
PASSWORD_MIN_LENGTH=8
PASSWORD_REQUIRE_SPECIAL_CHARS=true
PASSWORD_REQUIRE_NUMBERS=true
PASSWORD_REQUIRE_UPPERCASE=true

# ===========================================
# EXTERNAL API CONFIGURATION
# ===========================================

# Travel & Location APIs
AMADEUS_API_KEY=your_amadeus_api_key
AMADEUS_SECRET=your_amadeus_secret
BOOKING_COM_API_KEY=your_booking_com_key
GOOGLE_PLACES_API_KEY=your_google_places_key
GOOGLE_MAPS_API_KEY=your_google_maps_key

# Weather APIs
OPENWEATHER_API_KEY=your_openweather_key
WEATHERAPI_KEY=your_weatherapi_key

# Currency & Exchange
EXCHANGE_RATE_API_KEY=your_exchange_rate_key
DEFAULT_CURRENCY=USD

# ===========================================
# FRONTEND INTEGRATION
# ===========================================

# Frontend URLs
FRONTEND_URL=http://localhost:3000
FRONTEND_CORS_ORIGINS=["http://localhost:3000"]

# WebSocket Configuration
WS_AGENT_COORDINATION_URL=ws://localhost:8000/ws/agent-coordination
WS_RECONNECT_ATTEMPTS=5
WS_HEARTBEAT_INTERVAL=30000

# UI Feature Flags
GRAPH_VISUALIZATION_ENABLED=true
REAL_TIME_UPDATES_ENABLED=true
VOICE_INTERFACE_ENABLED=false
MOBILE_OPTIMIZED=true
OFFLINE_MODE_ENABLED=false

# ===========================================
# ORACLE CLOUD ARM64 OPTIMIZATION
# ===========================================

# Resource Optimization for Ampere A1
OPTIMIZE_FOR_ARM64=true
CPU_COUNT=4
MEMORY_LIMIT_GB=24
MEMORY_OPTIMIZATION=true

# Container Resource Limits
API_MEMORY_LIMIT=4GB
API_CPU_LIMIT=2.0
WORKER_MEMORY_LIMIT=1GB
WORKER_CPU_LIMIT=0.5
DB_MEMORY_LIMIT=4GB
DB_CPU_LIMIT=1.0

# Performance Tuning
GUNICORN_WORKERS=4
GUNICORN_WORKER_CLASS=uvicorn.workers.UvicornWorker
GUNICORN_MAX_REQUESTS=1000
GUNICORN_MAX_REQUESTS_JITTER=100

# ===========================================
# BACKUP & RECOVERY
# ===========================================

# Database Backup
DB_BACKUP_ENABLED=true
DB_BACKUP_INTERVAL_HOURS=6
DB_BACKUP_RETENTION_DAYS=30
DB_BACKUP_STORAGE=local  # local, s3, gcs

# Data Retention
AGENT_LOGS_RETENTION_DAYS=30
COST_DATA_RETENTION_DAYS=365
USER_SESSION_RETENTION_DAYS=90
VECTOR_DATA_RETENTION_DAYS=90

# ===========================================
# DEVELOPMENT & TESTING
# ===========================================

# Development Mode Settings
DEV_MODE=true
AUTO_RELOAD=true
HOT_RELOAD_ENABLED=true

# Testing Configuration
TEST_DATABASE_URL=postgresql://test_user:test_pass@localhost:5432/travel_planner_test
TEST_REDIS_URL=redis://localhost:6379/1
RUN_TESTS_ON_STARTUP=false

# Mock/Stub Configuration
MOCK_EXTERNAL_APIS=false
MOCK_AI_RESPONSES=false
STUB_AGENT_RESPONSES=false

# ===========================================
# LOGGING CONFIGURATION
# ===========================================

# Log Levels and Output
LOG_LEVEL=INFO  # DEBUG, INFO, WARNING, ERROR, CRITICAL
LOG_FORMAT=json  # json, text
LOG_TO_FILE=true
LOG_TO_CONSOLE=true
LOG_FILE_PATH=/var/log/travel-planner/app.log
LOG_MAX_SIZE_MB=100
LOG_BACKUP_COUNT=5

# Component-Specific Logging
AGENT_LOG_LEVEL=INFO
DB_LOG_LEVEL=WARNING
EXTERNAL_API_LOG_LEVEL=INFO
COST_TRACKING_LOG_LEVEL=DEBUG

# ===========================================
# FEATURE FLAGS
# ===========================================

# Experimental Features
ENABLE_VOICE_PLANNING=false
ENABLE_IMAGE_GENERATION=false
ENABLE_REAL_TIME_COLLABORATION=false
ENABLE_PLAN_SHARING=true
ENABLE_PLAN_TEMPLATES=true

# Beta Features
ENABLE_SMART_NOTIFICATIONS=true
ENABLE_PREDICTIVE_PRICING=false
ENABLE_AI_TRAVEL_ASSISTANT=true
ENABLE_CARBON_FOOTPRINT_TRACKING=false

# ===========================================
# ANALYTICS & BUSINESS INTELLIGENCE
# ===========================================

# User Analytics
TRACK_USER_BEHAVIOR=true
TRACK_PLAN_PERFORMANCE=true
TRACK_AGENT_EFFICIENCY=true
TRACK_COST_OPTIMIZATION=true

# Business Metrics
REVENUE_TRACKING_ENABLED=true
CONVERSION_TRACKING_ENABLED=true
RETENTION_ANALYSIS_ENABLED=true
SATISFACTION_SURVEYS_ENABLED=true

# Privacy Settings
ANONYMIZE_USER_DATA=true
GDPR_COMPLIANCE=true
DATA_EXPORT_ENABLED=true
DATA_DELETION_ENABLED=true
```

## 🔧 Environment-Specific Configurations

### Development Environment (.env.development)

```bash
# Development-specific overrides
ENVIRONMENT=development
DEBUG=true
LOG_LEVEL=DEBUG

# Use local services
DATABASE_URL=postgresql://dev_user:dev_pass@localhost:5432/travel_planner_dev
REDIS_URL=redis://localhost:6379/0
QDRANT_URL=http://localhost:6333

# Relaxed cost limits for testing
FREE_TIER_DAILY_LIMIT=10.00
FREE_TIER_REQUEST_LIMIT=1.00

# Development optimizations
MOCK_EXTERNAL_APIS=true
COST_OPTIMIZATION_ENABLED=false
METRICS_ENABLED=false
LANGSMITH_ENABLED=false

# Resource limits for development
MAX_CONCURRENT_AGENTS=5
CELERY_WORKER_CONCURRENCY=2
GUNICORN_WORKERS=2
```

### Staging Environment (.env.staging)

```bash
# Staging-specific overrides
ENVIRONMENT=staging
DEBUG=false
LOG_LEVEL=INFO

# Staging database
DATABASE_URL=postgresql://staging_user:staging_pass@staging-db:5432/travel_planner_staging
REDIS_URL=redis://staging-redis:6379/0
QDRANT_URL=http://staging-qdrant:6333

# Production-like cost limits
FREE_TIER_DAILY_LIMIT=2.00
FREE_TIER_REQUEST_LIMIT=0.20

# Full monitoring enabled
METRICS_ENABLED=true
LANGSMITH_ENABLED=true
COST_TRACKING_ENABLED=true

# Moderate resource allocation
MAX_CONCURRENT_AGENTS=8
CELERY_WORKER_CONCURRENCY=3
GUNICORN_WORKERS=3
```

### Production Environment (.env.production)

```bash
# Production configuration
ENVIRONMENT=production
DEBUG=false
LOG_LEVEL=WARNING

# Production databases with SSL
DATABASE_URL=postgresql://prod_user:prod_pass@prod-db:5432/travel_planner?sslmode=require
REDIS_URL=redis://prod-redis:6379/0
QDRANT_URL=http://prod-qdrant:6333

# Production cost limits
FREE_TIER_DAILY_LIMIT=1.00
FREE_TIER_REQUEST_LIMIT=0.10
BASIC_TIER_DAILY_LIMIT=10.00
BASIC_TIER_REQUEST_LIMIT=0.50
PREMIUM_TIER_DAILY_LIMIT=50.00
PREMIUM_TIER_REQUEST_LIMIT=2.00

# Full production optimizations
COST_OPTIMIZATION_ENABLED=true
AUTO_MODEL_SWITCHING=true
AGGRESSIVE_COST_OPTIMIZATION=true

# Full monitoring suite
METRICS_ENABLED=true
LANGSMITH_ENABLED=true
COST_TRACKING_ENABLED=true
PERFORMANCE_ALERTS_ENABLED=true

# Production resource allocation (Oracle Cloud ARM64)
MAX_CONCURRENT_AGENTS=10
CELERY_WORKER_CONCURRENCY=4
GUNICORN_WORKERS=4
API_MEMORY_LIMIT=4GB
API_CPU_LIMIT=2.0

# Security hardening
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=30
API_RATE_LIMIT_REQUESTS=50
API_RATE_LIMIT_WINDOW=3600
CORS_ORIGINS=["https://yourdomain.com"]

# Production logging
LOG_TO_FILE=true
LOG_FILE_PATH=/var/log/travel-planner/app.log
LOG_MAX_SIZE_MB=100
LOG_BACKUP_COUNT=10
```

## 🚀 Quick Setup Scripts

### Development Setup Script

```bash
#!/bin/bash
# scripts/setup-development.sh

echo "🚀 Setting up Travel Planner Development Environment..."

# Check prerequisites
command -v python3.11 >/dev/null 2>&1 || { echo "❌ Python 3.11+ required"; exit 1; }
command -v docker >/dev/null 2>&1 || { echo "❌ Docker required"; exit 1; }
command -v node >/dev/null 2>&1 || { echo "❌ Node.js 18+ required"; exit 1; }

# Create project directory structure
mkdir -p backend/{app,tests,migrations}
mkdir -p frontend/src
mkdir -p monitoring/{prometheus,grafana}
mkdir -p scripts

# Setup Python virtual environment
echo "📦 Creating Python virtual environment..."
python3.11 -m venv venv
source venv/bin/activate

# Install Python dependencies
echo "📦 Installing Python dependencies..."
cd backend
pip install --upgrade pip
pip install -r requirements.txt

# Setup environment file
echo "⚙️ Setting up environment configuration..."
cd ..
cp .env.example .env
echo "✏️ Please update .env file with your API keys"

# Start development services
echo "🐳 Starting development services..."
docker-compose -f docker-compose.dev.yml up -d postgres redis qdrant

# Wait for services to be ready
echo "⏳ Waiting for services to be ready..."
sleep 15

# Run database migrations
echo "🗄️ Running database migrations..."
cd backend
alembic upgrade head

# Install frontend dependencies
echo "🎨 Setting up frontend..."
cd ../frontend
npm install

echo "✅ Development environment setup complete!"
echo ""
echo "🎯 Next steps:"
echo "1. Update .env file with your API keys"
echo "2. Run 'npm run dev' in frontend/ directory"
echo "3. Run 'uvicorn app.main:app --reload' in backend/ directory"
echo "4. Access application at http://localhost:3000"
echo "5. Access API docs at http://localhost:8000/docs"
```

### Oracle Cloud ARM64 Production Setup

```bash
#!/bin/bash
# scripts/setup-oracle-cloud-production.sh

echo "🏭 Setting up Travel Planner on Oracle Cloud ARM64..."

# System optimization for Ampere A1
echo "⚙️ Optimizing system for ARM64..."
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-plugin nginx certbot htop

# Docker optimization for ARM64
echo "🐳 Configuring Docker for ARM64..."
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

# Create Docker daemon configuration optimized for ARM64
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  }
}
EOF

sudo systemctl restart docker

# System resource optimization for 24GB RAM / 4 CPU
echo "🎛️ Optimizing system resources..."
sudo sysctl -w vm.swappiness=10
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w net.core.somaxconn=65535

# Make optimizations permanent
sudo tee -a /etc/sysctl.conf > /dev/null <<EOF
# Travel Planner ARM64 Optimizations
vm.swappiness=10
vm.max_map_count=262144
net.core.somaxconn=65535
net.core.netdev_max_backlog=5000
net.ipv4.tcp_max_syn_backlog=8192
EOF

# Setup application environment
echo "📁 Setting up application..."
git clone https://github.com/yourusername/travel-planner.git
cd travel-planner

# Setup production environment
cp .env.example .env.production
echo "✏️ Please update .env.production with production values"

# Build ARM64 optimized containers
echo "🏗️ Building ARM64 optimized containers..."
docker compose -f docker-compose.arm64.yml build --parallel

# Setup SSL certificates
echo "🔒 Setting up SSL certificates..."
sudo certbot --nginx -d yourdomain.com

# Start production services
echo "🚀 Starting production services..."
docker compose -f docker-compose.arm64.yml up -d

# Setup monitoring
echo "📊 Setting up monitoring..."
docker compose -f docker-compose.arm64.yml logs -f &

echo "✅ Oracle Cloud ARM64 production setup complete!"
echo ""
echo "🎯 Access your application:"
echo "- Main App: https://yourdomain.com"
echo "- API Docs: https://yourdomain.com/docs"
echo "- Grafana: https://yourdomain.com:3000"
echo "- Prometheus: https://yourdomain.com:9090"
```

## 🔐 Environment Security Checklist

### Development Security
- [ ] Never commit real API keys to version control
- [ ] Use `.env.example` template with placeholder values
- [ ] Use different databases for dev/staging/production
- [ ] Enable debug mode only in development
- [ ] Use weak passwords only in development

### Staging Security
- [ ] Use production-like security settings
- [ ] Test SSL/TLS configuration
- [ ] Validate API rate limiting
- [ ] Test authentication flows
- [ ] Verify CORS configuration

### Production Security
- [ ] Strong JWT secret keys (64+ characters)
- [ ] Production API keys from secure sources
- [ ] SSL/TLS certificates configured
- [ ] Database connections encrypted
- [ ] API rate limiting enabled
- [ ] CORS restricted to production domains
- [ ] Log sensitive data excluded
- [ ] Regular security updates scheduled

## 🧪 Testing Environment Configurations

### Unit Testing (.env.test)

```bash
# Test environment configuration
ENVIRONMENT=test
DEBUG=true
LOG_LEVEL=ERROR

# Test databases
DATABASE_URL=postgresql://test:test@localhost:5432/travel_planner_test
REDIS_URL=redis://localhost:6379/1

# Mock all external services
MOCK_EXTERNAL_APIS=true
MOCK_AI_RESPONSES=true
STUB_AGENT_RESPONSES=true

# Disable unnecessary features
METRICS_ENABLED=false
LANGSMITH_ENABLED=false
COST_TRACKING_ENABLED=false

# Fast test execution
AGENT_TIMEOUT_SECONDS=5
MAX_CONCURRENT_AGENTS=2
```

This comprehensive environment configuration provides:

✅ **Complete Configuration** - All necessary environment variables covered  
✅ **Environment-Specific** - Separate configs for dev/staging/production  
✅ **Oracle Cloud Optimized** - ARM64 and Ampere A1 specific settings  
✅ **Security Focused** - Proper secrets management and security settings  
✅ **Cost Optimized** - Granular cost controls and optimization settings  
✅ **Monitoring Ready** - Full observability and monitoring configuration  
✅ **Agent Tuned** - Optimal settings for each agent type and workload