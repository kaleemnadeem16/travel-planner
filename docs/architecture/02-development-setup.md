# Development Environment Setup

This guide provides comprehensive instructions for setting up the Travel Planner development environment using FastAPI + LangGraph architecture.

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
git --version          # Should show git version
node --version         # Should show v18.0+
python --version       # Should show 3.9+
docker --version       # Should show 20.10+
```

## 🚀 Quick Setup Options

### Option 1: Docker Development (Recommended)

**Best for**: Complete isolation, consistent environment across team members

```bash
# Clone the repository
git clone https://github.com/kaleemnadeem16/travel-planner.git
cd travel-planner

# Start all services with Docker Compose
docker-compose -f docker-compose.dev.yml up -d

# Access the application
# Frontend: http://localhost:3000
# Backend API: http://localhost:8000
# API Docs: http://localhost:8000/docs
# PostgreSQL: localhost:5432
# Redis: localhost:6379
# Qdrant: http://localhost:6333
```

### Option 2: Local Development

**Best for**: Direct debugging, faster development cycles

```bash
# Clone and setup
git clone https://github.com/kaleemnadeem16/travel-planner.git
cd travel-planner

# Backend setup
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt

# Frontend setup
cd ../frontend
npm install

# Database setup (using Docker)
docker run -d --name travel-db -p 5432:5432 -e POSTGRES_PASSWORD=devpass postgres:15
docker run -d --name travel-qdrant -p 6333:6333 qdrant/qdrant
docker run -d --name travel-redis -p 6379:6379 redis:7-alpine

# Start development servers
# Terminal 1: Backend
cd backend && uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Terminal 2: Frontend
cd frontend && npm run dev

# Terminal 3: Celery Worker
cd backend && celery -A travel_planner.celery worker --loglevel=info
```

## 📁 Project Structure

```
travel-planner/
├── backend/                 # FastAPI + LangGraph backend
│   ├── app/
│   │   ├── agents/         # LangGraph agent definitions
│   │   ├── api/            # FastAPI routes
│   │   ├── core/           # Configuration and utilities
│   │   ├── models/         # SQLAlchemy models
│   │   ├── schemas/        # Pydantic schemas
│   │   └── services/       # Business logic
│   ├── tests/              # Backend tests
│   ├── requirements.txt    # Python dependencies
│   └── main.py            # FastAPI application entry point
├── frontend/               # React frontend
│   ├── src/
│   │   ├── components/    # React components
│   │   ├── pages/         # Page components
│   │   ├── hooks/         # Custom React hooks
│   │   ├── services/      # API clients
│   │   └── utils/         # Utility functions
│   ├── package.json       # Node.js dependencies
│   └── vite.config.js     # Vite configuration
├── docs/                  # Documentation
├── docker-compose.yml     # Production Docker setup
├── docker-compose.dev.yml # Development Docker setup
└── README.md             # Project overview
```

## ⚙️ Environment Configuration

### Backend Environment Variables (.env)

```bash
# Core Application
ENVIRONMENT=development
DEBUG=true
SECRET_KEY=your-super-secret-key-here
ALLOWED_HOSTS=localhost,127.0.0.1

# Database Configuration
DATABASE_URL=postgresql+asyncpg://postgres:devpass@localhost:5432/travel_planner
REDIS_URL=redis://localhost:6379/0

# Vector Database
QDRANT_URL=http://localhost:6333
QDRANT_API_KEY=your-qdrant-api-key

# AI/ML Services
OPENAI_API_KEY=your-openai-api-key
ANTHROPIC_API_KEY=your-anthropic-api-key

# External Services
AMADEUS_API_KEY=your-amadeus-api-key
AMADEUS_API_SECRET=your-amadeus-api-secret
GOOGLE_MAPS_API_KEY=your-google-maps-api-key
OPENWEATHER_API_KEY=your-openweather-api-key

# Monitoring
LANGSMITH_API_KEY=your-langsmith-api-key
LANGSMITH_PROJECT=travel-planner-dev

# Security
JWT_SECRET_KEY=your-jwt-secret-key
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Celery Configuration
CELERY_BROKER_URL=redis://localhost:6379/1
CELERY_RESULT_BACKEND=redis://localhost:6379/2

# CORS Settings
FRONTEND_URL=http://localhost:3000
```

### Frontend Environment Variables (.env.local)

```bash
# API Configuration
VITE_API_BASE_URL=http://localhost:8000
VITE_WS_BASE_URL=ws://localhost:8000

# Feature Flags
VITE_ENABLE_DEBUG=true
VITE_ENABLE_MOCK_DATA=false

# External Services
VITE_GOOGLE_MAPS_API_KEY=your-google-maps-api-key
```

## 🛠️ Development Commands

### Backend Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run development server
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Run tests
pytest tests/ -v

# Code formatting
black app/
isort app/

# Type checking
mypy app/

# Database migrations
alembic upgrade head
alembic revision --autogenerate -m "Migration message"

# Start Celery worker
celery -A travel_planner.celery worker --loglevel=info

# Start Celery beat scheduler
celery -A travel_planner.celery beat --loglevel=info
```

### Frontend Commands

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build

# Run tests
npm run test

# Code formatting
npm run format

# Linting
npm run lint

# Type checking
npm run type-check
```

### Docker Commands

```bash
# Development environment
docker-compose -f docker-compose.dev.yml up -d
docker-compose -f docker-compose.dev.yml down

# Production environment
docker-compose up -d
docker-compose down

# Rebuild services
docker-compose build --no-cache

# View logs
docker-compose logs -f backend
docker-compose logs -f frontend

# Access container shell
docker-compose exec backend bash
docker-compose exec frontend sh
```

## 🔍 Development URLs

| Service | URL | Description |
|---------|-----|-------------|
| Frontend | http://localhost:3000 | React development server |
| Backend API | http://localhost:8000 | FastAPI application |
| API Docs | http://localhost:8000/docs | Swagger UI documentation |
| Redoc | http://localhost:8000/redoc | Alternative API documentation |
| PostgreSQL | localhost:5432 | Database connection |
| Redis | localhost:6379 | Cache and message broker |
| Qdrant | http://localhost:6333 | Vector database dashboard |
| Celery Flower | http://localhost:5555 | Task monitoring |

## 🧪 Testing Setup

### Backend Testing

```bash
# Install test dependencies
pip install pytest pytest-asyncio pytest-cov httpx

# Run all tests
pytest

# Run with coverage
pytest --cov=app tests/

# Run specific test file
pytest tests/test_agents.py

# Run tests in watch mode
ptw
```

### Frontend Testing

```bash
# Install test dependencies
npm install --save-dev @testing-library/react vitest

# Run tests
npm run test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

## 🐛 Debugging Configuration

### VS Code Debug Configuration (.vscode/launch.json)

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "FastAPI Server",
      "type": "python",
      "request": "launch",
      "program": "${workspaceFolder}/backend/main.py",
      "args": [],
      "console": "integratedTerminal",
      "env": {
        "PYTHONPATH": "${workspaceFolder}/backend"
      },
      "cwd": "${workspaceFolder}/backend"
    },
    {
      "name": "React App",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/frontend/node_modules/.bin/vite",
      "args": ["dev"],
      "cwd": "${workspaceFolder}/frontend",
      "console": "integratedTerminal"
    }
  ]
}
```

### PyCharm Configuration

1. **Python Interpreter**: Select the virtual environment Python interpreter
2. **Run Configuration**: 
   - Script path: `main.py`
   - Parameters: `--reload --host 0.0.0.0 --port 8000`
   - Working directory: `backend/`
   - Environment variables: Load from `.env` file

## 📊 Performance Monitoring

### Local Development Monitoring

```python
# Add to main.py for development profiling
import time
from fastapi import Request

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

### Database Query Monitoring

```python
# SQLAlchemy logging for development
import logging

logging.basicConfig()
logging.getLogger('sqlalchemy.engine').setLevel(logging.INFO)
```

## 🔧 Troubleshooting

### Common Issues

#### Backend Not Starting

```bash
# Check Python version
python --version

# Verify virtual environment
which python

# Check dependencies
pip list

# Verify environment variables
python -c "import os; print(os.getenv('DATABASE_URL'))"
```

#### Database Connection Issues

```bash
# Check PostgreSQL status
docker ps | grep postgres

# Test database connection
python -c "import asyncpg; print('PostgreSQL available')"

# Reset database
docker-compose down -v
docker-compose up -d postgres
```

#### Frontend Build Issues

```bash
# Clear cache
npm cache clean --force

# Remove node_modules
rm -rf node_modules package-lock.json
npm install

# Check Node version
node --version
```

### Performance Issues

- **Slow API responses**: Check database queries and add indexes
- **High memory usage**: Monitor agent memory consumption
- **WebSocket disconnections**: Check network stability and connection limits

## 📚 Additional Resources

### Documentation Links
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [React Documentation](https://react.dev/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Qdrant Documentation](https://qdrant.tech/documentation/)

### Internal Documentation
- [Backend Implementation Guide](../backend/README.md)
- [Agent System Design](../agents/README.md)
- [Database Schema](../backend/02-database-schema.md)
- [Deployment Guide](../deployment/README.md)
- [Monitoring System](../backend/03-monitoring-system.md)

---

**Last Updated**: September 4, 2025  
**Technology Stack**: FastAPI + LangGraph + PostgreSQL + Qdrant  
**Target Deployment**: Oracle Cloud ARM64 (Ampere A1)