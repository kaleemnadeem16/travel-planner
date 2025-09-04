# Backend Development Guide (Python)

## 🎯 Current Architecture: FastAPI + LangGraph

This guide covers the implementation of our **FastAPI + LangGraph** backend architecture, designed for scalable multi-agent travel planning with comprehensive monitoring and Oracle Cloud ARM deployment.

> **📋 For detailed architecture decisions, see:** [`01-architecture-decisions.md`](./01-architecture-decisions.md)

## 🏗️ Core Technology Stack

### **Primary Framework Stack**
- **API Framework**: FastAPI 0.104+ (async-first, high performance)
- **Agent Framework**: LangGraph (state management, workflow orchestration)
- **Model Integration**: Universal provider abstraction (OpenAI, Anthropic, Azure, etc.)
- **Database**: PostgreSQL 15+ (primary) + Qdrant (vector storage)
- **Caching**: Redis 7+ (sessions, queues, agent communication)
- **Task Queue**: Celery (background agent processing)
- **Monitoring**: LangSmith + Prometheus + Grafana
- **Authentication**: JWT tokens with FastAPI Security
- **Validation**: Pydantic v2 (type safety throughout)

### **Why FastAPI + LangGraph?**
✅ **Native async/await** - Perfect for concurrent agent execution  
✅ **Automatic API docs** - OpenAPI/Swagger out of the box  
✅ **Type safety** - Pydantic integration prevents runtime errors  
✅ **High performance** - Comparable to Node.js/Go  
✅ **LangGraph state management** - Purpose-built for multi-agent workflows  
✅ **Provider flexibility** - Easy switching between AI providers  
✅ **Production ready** - WebSocket support, monitoring, health checks

## 📁 Project Structure (FastAPI + LangGraph)

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI application entry point
│   ├── core/
│   │   ├── config.py           # Environment configuration
│   │   ├── database.py         # Database connections
│   │   ├── security.py         # Authentication & JWT
│   │   ├── model_providers.py  # Universal AI provider abstraction
│   │   ├── monitoring.py       # LangSmith & metrics integration
│   │   └── vector_store.py     # Qdrant vector database
│   ├── agents/
│   │   ├── base_agent.py       # Abstract agent base class
│   │   ├── planning_agent.py   # Trip planning agent
│   │   ├── location_agent.py   # Location research agent
│   │   ├── transport_agent.py  # Transportation agent
│   │   ├── accommodation_agent.py
│   │   ├── activity_agent.py
│   │   ├── budget_agent.py
│   │   └── weather_agent.py
│   ├── orchestrator/
│   │   ├── graph_workflow.py   # LangGraph workflow definition
│   │   ├── state_manager.py    # Workflow state management
│   │   ├── coordinator.py      # Agent coordination logic
│   │   └── tasks.py           # Celery background tasks
│   ├── api/
│   │   ├── routes/
│   │   │   ├── auth.py         # Authentication endpoints
│   │   │   ├── plans.py        # Travel plan CRUD
│   │   │   ├── agents.py       # Agent interaction endpoints
│   │   │   ├── monitoring.py   # Cost & performance monitoring
│   │   │   └── websocket.py    # Real-time agent communication
│   │   ├── dependencies.py     # FastAPI dependency injection
│   │   ├── middleware.py       # Custom middleware
│   │   └── exceptions.py       # Error handling
│   ├── models/
│   │   ├── database.py         # SQLAlchemy models
│   │   ├── schemas.py          # Pydantic schemas
│   │   └── enums.py            # Enums and constants
│   ├── services/
│   │   ├── auth_service.py     # Authentication logic
│   │   ├── plan_service.py     # Plan management
│   │   ├── cost_service.py     # Cost tracking & optimization
│   │   └── vector_service.py   # Vector search & RAG
│   └── utils/
│       ├── helpers.py          # Utility functions
│       ├── validators.py       # Custom validators
│       └── formatters.py       # Data formatting
├── tests/
│   ├── unit/              # Unit tests for agents & services
│   ├── integration/       # Integration tests
│   └── load/              # Load testing
├── migrations/           # Database migrations (Alembic)
├── requirements.txt      # Python dependencies
├── Dockerfile.arm64      # ARM64 optimized container
├── docker-compose.yml    # Development environment
└── .env.example          # Environment template
```

## 🚀 API Endpoints (FastAPI)

### Authentication Endpoints

```python
# app/api/routes/auth.py
from fastapi import APIRouter, Depends, HTTPException
from fastapi.security import HTTPBearer

router = APIRouter(prefix="/auth", tags=["authentication"])
security = HTTPBearer()

@router.post("/signup", response_model=UserResponse)
async def signup(user_data: UserCreate):
    """Register new user with email/password"""
    pass

@router.post("/login", response_model=TokenResponse)
async def login(credentials: UserLogin):
    """Authenticate user and return JWT tokens"""
    pass

@router.post("/refresh", response_model=TokenResponse)
async def refresh_token(current_user: User = Depends(get_current_user)):
    """Refresh access token"""
    pass

@router.get("/me", response_model=UserResponse)
async def get_current_user_profile(current_user: User = Depends(get_current_user)):
    """Get current user profile"""
    pass

# Social OAuth endpoints
@router.get("/oauth/google")
async def google_oauth():
    """Initiate Google OAuth flow"""
    pass
```

### Travel Plan Management

```python
# app/api/routes/plans.py
from fastapi import APIRouter, Depends, HTTPException, Query
from typing import List, Optional
from app.models.schemas import PlanCreate, PlanResponse, PlanUpdate
from app.services.plan_service import PlanService
from app.api.dependencies import get_current_user

router = APIRouter(prefix="/plans", tags=["plans"])

@router.post("/", response_model=PlanResponse)
async def create_plan(
    plan_data: PlanCreate,
    current_user: User = Depends(get_current_user)
):
    """Create a new travel plan"""
    return await PlanService.create_plan(plan_data, current_user.id)

@router.get("/", response_model=List[PlanResponse])
async def get_user_plans(
    skip: int = Query(0, ge=0),
    limit: int = Query(10, ge=1, le=100),
    current_user: User = Depends(get_current_user)
):
    """Get user's travel plans with pagination"""
    return await PlanService.get_user_plans(current_user.id, skip, limit)

@router.get("/{plan_id}", response_model=PlanResponse)
async def get_plan(
    plan_id: str,
    current_user: User = Depends(get_current_user)
):
    """Get specific travel plan"""
    return await PlanService.get_plan(plan_id, current_user.id)

@router.put("/{plan_id}", response_model=PlanResponse)
async def update_plan(
    plan_id: str,
    plan_data: PlanUpdate,
    current_user: User = Depends(get_current_user)
):
    """Update travel plan"""
    return await PlanService.update_plan(plan_id, plan_data, current_user.id)

@router.delete("/{plan_id}")
async def delete_plan(
    plan_id: str,
    current_user: User = Depends(get_current_user)
):
    """Delete travel plan"""
    await PlanService.delete_plan(plan_id, current_user.id)
    return {"message": "Plan deleted successfully"}

@router.post("/{plan_id}/share")
async def share_plan(
    plan_id: str,
    current_user: User = Depends(get_current_user)
):
    """Share travel plan"""
    return await PlanService.share_plan(plan_id, current_user.id)
```

### Agent Interaction Endpoints

```python
# app/api/routes/agents.py
from fastapi import APIRouter, Depends, BackgroundTasks
from typing import Dict, Any
from app.models.schemas import AgentTaskRequest, AgentTaskResponse
from app.orchestrator.coordinator import AgentCoordinator
from app.api.dependencies import get_current_user

router = APIRouter(prefix="/agents", tags=["agents"])

@router.post("/tasks", response_model=AgentTaskResponse)
async def create_agent_task(
    task_request: AgentTaskRequest,
    background_tasks: BackgroundTasks,
    current_user: User = Depends(get_current_user)
):
    """Create and execute agent task"""
    return await AgentCoordinator.execute_task(task_request, current_user.id)

@router.get("/tasks/{task_id}", response_model=AgentTaskResponse)
async def get_agent_task(
    task_id: str,
    current_user: User = Depends(get_current_user)
):
    """Get agent task status and results"""
    return await AgentCoordinator.get_task_status(task_id, current_user.id)

@router.websocket("/ws/{plan_id}")
async def websocket_agent_updates(
    websocket: WebSocket,
    plan_id: str,
    current_user: User = Depends(get_current_user_ws)
):
    """Real-time agent progress updates via WebSocket"""
    await AgentCoordinator.handle_websocket_connection(websocket, plan_id, current_user.id)
```

## 🗄️ Data Models (SQLAlchemy)

### User Model

```python
# app/models/database.py
from sqlalchemy import Column, String, Boolean, DateTime, Text, DECIMAL, Integer
from sqlalchemy.dialects.postgresql import UUID, JSONB
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.sql import func
import uuid

Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    email = Column(String(255), unique=True, nullable=False, index=True)
    username = Column(String(100), unique=True, nullable=False, index=True)
    password_hash = Column(String(255), nullable=False)
    
    # Personal information
    first_name = Column(String(100))
    last_name = Column(String(100))
    phone_number = Column(String(20))
    
    # User preferences and profile
    profile_data = Column(JSONB, default={})
    travel_preferences = Column(JSONB, default={})
    
    # Subscription and status
    subscription_tier = Column(String(50), default='free')
    is_active = Column(Boolean, default=True)
    is_verified = Column(Boolean, default=False)
    
    # Timestamps
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    last_login = Column(DateTime(timezone=True))
    
    def __repr__(self):
        return f"<User {self.email}>"

class TravelPlan(Base):
    __tablename__ = "travel_plans"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    user_id = Column(UUID(as_uuid=True), ForeignKey('users.id'), nullable=False)
    
    # Plan details
    title = Column(String(255), nullable=False)
    description = Column(Text)
    destinations = Column(JSONB, default=[])
    start_date = Column(Date)
    end_date = Column(Date)
    
    # Budget and travelers
    budget_total = Column(DECIMAL(12, 2))
    budget_currency = Column(String(3), default='USD')
    traveler_count = Column(Integer, default=1)
    traveler_details = Column(JSONB, default={})
    
    # Preferences and generated data
    preferences = Column(JSONB, default={})
    constraints = Column(JSONB, default={})
    plan_data = Column(JSONB, default={})
    agent_results = Column(JSONB, default={})
    
    # Version control
    version = Column(Integer, default=1)
    parent_plan_id = Column(UUID(as_uuid=True), ForeignKey('travel_plans.id'))
    is_active = Column(Boolean, default=True)
    
    # Status and metrics
    status = Column(String(50), default='draft')
    completion_percentage = Column(Integer, default=0)
    total_cost_usd = Column(DECIMAL(10, 6), default=0.00)
    
    # Timestamps
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    completed_at = Column(DateTime(timezone=True))
    
    def __repr__(self):
        return f"<TravelPlan {self.title}>"
```

### Pydantic Schemas

```python
# app/models/schemas.py
from pydantic import BaseModel, EmailStr, validator
from typing import Optional, List, Dict, Any
from datetime import date, datetime
from enum import Enum

class SubscriptionTier(str, Enum):
    FREE = "free"
    BASIC = "basic"
    PREMIUM = "premium"

class PlanStatus(str, Enum):
    DRAFT = "draft"
    PROCESSING = "processing"
    COMPLETED = "completed"
    ERROR = "error"
    CANCELLED = "cancelled"

# User Schemas
class UserBase(BaseModel):
    email: EmailStr
    username: str
    first_name: Optional[str] = None
    last_name: Optional[str] = None
    phone_number: Optional[str] = None

class UserCreate(UserBase):
    password: str
    password_confirm: str
    
    @validator('password_confirm')
    def passwords_match(cls, v, values):
        if 'password' in values and v != values['password']:
            raise ValueError('Passwords do not match')
        return v

class UserResponse(UserBase):
    id: str
    subscription_tier: SubscriptionTier
    is_active: bool
    is_verified: bool
    created_at: datetime
    last_login: Optional[datetime] = None
    
    class Config:
        from_attributes = True

class UserLogin(BaseModel):
    email: EmailStr
    password: str

class TokenResponse(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"
    expires_in: int

# Travel Plan Schemas
class PlanBase(BaseModel):
    title: str
    description: Optional[str] = None
    destinations: List[str] = []
    start_date: Optional[date] = None
    end_date: Optional[date] = None
    budget_total: Optional[float] = None
    budget_currency: str = "USD"
    traveler_count: int = 1
    preferences: Dict[str, Any] = {}
    constraints: Dict[str, Any] = {}

class PlanCreate(PlanBase):
    pass

class PlanUpdate(BaseModel):
    title: Optional[str] = None
    description: Optional[str] = None
    destinations: Optional[List[str]] = None
    start_date: Optional[date] = None
    end_date: Optional[date] = None
    budget_total: Optional[float] = None
    traveler_count: Optional[int] = None
    preferences: Optional[Dict[str, Any]] = None
    constraints: Optional[Dict[str, Any]] = None

class PlanResponse(PlanBase):
    id: str
    user_id: str
    version: int
    status: PlanStatus
    completion_percentage: int
    total_cost_usd: float
    plan_data: Dict[str, Any] = {}
    agent_results: Dict[str, Any] = {}
    created_at: datetime
    updated_at: datetime
    completed_at: Optional[datetime] = None
    
    class Config:
        from_attributes = True

# Agent Schemas
class AgentTaskRequest(BaseModel):
    agent_type: str
    task_data: Dict[str, Any]
    context_data: Optional[Dict[str, Any]] = {}
    priority: int = 5

class AgentTaskResponse(BaseModel):
    task_id: str
    agent_type: str
    status: str
    result_data: Optional[Dict[str, Any]] = None
    error_data: Optional[Dict[str, Any]] = None
    execution_time_ms: Optional[int] = None
    cost_usd: Optional[float] = None
    created_at: datetime
```

## 🔐 Authentication & Security

### FastAPI Security Implementation

```python
# app/core/security.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from app.core.config import settings
from app.models.database import User
from app.services.user_service import UserService

# Password hashing
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# JWT bearer scheme
security = HTTPBearer()

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)
    return encoded_jwt

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    
    try:
        payload = jwt.decode(credentials.credentials, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        user_id: str = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    
    user = await UserService.get_user_by_id(user_id)
    if user is None:
        raise credentials_exception
    
    return user
```

## 📊 Rate Limiting & Middleware

### FastAPI Rate Limiting

```python
# app/api/middleware.py
import time
import redis
from fastapi import Request, HTTPException, status
from fastapi.middleware.base import BaseHTTPMiddleware
from app.core.config import settings

class RateLimitMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, redis_url: str = "redis://localhost:6379"):
        super().__init__(app)
        self.redis_client = redis.from_url(redis_url)
    
    async def dispatch(self, request: Request, call_next):
        # Skip rate limiting for authenticated users
        if hasattr(request.state, 'user') and request.state.user:
            return await call_next(request)
        
        # Rate limit by IP for anonymous users
        client_ip = self.get_client_ip(request)
        key = f"rate_limit:{client_ip}"
        
        current = self.redis_client.get(key)
        if current is None:
            self.redis_client.setex(key, 3600, 1)  # 1 hour window
        else:
            current = int(current)
            if current >= 100:  # 100 requests per hour for anonymous users
                raise HTTPException(
                    status_code=status.HTTP_429_TOO_MANY_REQUESTS,
                    detail="Rate limit exceeded. Please sign up for higher limits."
                )
            self.redis_client.incr(key)
        
        response = await call_next(request)
        return response
    
    def get_client_ip(self, request: Request) -> str:
        forwarded = request.headers.get("X-Forwarded-For")
        if forwarded:
            return forwarded.split(",")[0].strip()
        return request.client.host
```

## 🧪 Testing Framework

### FastAPI Testing Setup

```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.core.database import get_db
from app.models.database import Base

# Test database setup
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

@pytest.fixture
def db():
    Base.metadata.create_all(bind=engine)
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()
        Base.metadata.drop_all(bind=engine)

@pytest.fixture
def client(db):
    def override_get_db():
        try:
            yield db
        finally:
            db.close()
    
    app.dependency_overrides[get_db] = override_get_db
    yield TestClient(app)
    app.dependency_overrides.clear()

# tests/test_auth.py
def test_user_registration(client):
    response = client.post("/auth/signup", json={
        "email": "test@example.com",
        "username": "testuser",
        "password": "testpass123",
        "password_confirm": "testpass123"
    })
    assert response.status_code == 201
    assert "access_token" in response.json()

def test_user_login(client):
    # First register a user
    client.post("/auth/signup", json={
        "email": "test@example.com",
        "username": "testuser", 
        "password": "testpass123",
        "password_confirm": "testpass123"
    })
    
    # Then login
    response = client.post("/auth/login", json={
        "email": "test@example.com",
        "password": "testpass123"
    })
    assert response.status_code == 200
    assert "access_token" in response.json()
```

## 🎯 Implementation Roadmap

### Phase 1: Core Infrastructure (Week 1-2)
- [ ] Set up FastAPI project structure
- [ ] Configure SQLAlchemy with PostgreSQL
- [ ] Implement Pydantic schemas and models
- [ ] Set up Alembic database migrations
- [ ] Configure Redis connection and Celery
- [ ] Implement basic health checks

### Phase 2: Authentication & User Management (Week 2-3)
- [ ] Implement JWT authentication with FastAPI Security
- [ ] Create user registration and login endpoints
- [ ] Add password hashing and validation
- [ ] Implement user profile management
- [ ] Set up OAuth2 social authentication
- [ ] Add rate limiting middleware

### Phase 3: Agent System (Week 3-5)
- [ ] Implement base agent class with provider abstraction
- [ ] Create individual agent implementations
- [ ] Set up LangGraph workflow orchestration
- [ ] Implement Celery task queue for agents
- [ ] Add agent communication and state management
- [ ] Integrate LangSmith monitoring

### Phase 4: Travel Plan Management (Week 4-6)
- [ ] Implement travel plan CRUD operations
- [ ] Add plan versioning and history
- [ ] Integrate Qdrant vector database
- [ ] Implement plan similarity search
- [ ] Add real-time WebSocket updates
- [ ] Create plan sharing functionality

### Phase 5: Cost Tracking & Optimization (Week 5-7)
- [ ] Implement comprehensive cost tracking
- [ ] Add cost optimization suggestions
- [ ] Create cost alert system
- [ ] Build cost analytics dashboard
- [ ] Implement usage limits by subscription tier
- [ ] Add billing and payment integration

### Phase 6: Monitoring & Deployment (Week 6-8)
- [ ] Set up Prometheus metrics collection
- [ ] Configure Grafana dashboards
- [ ] Implement comprehensive logging
- [ ] Create ARM64 optimized Docker containers
- [ ] Deploy to Oracle Cloud infrastructure
- [ ] Set up CI/CD pipeline

### Phase 7: Testing & Optimization (Week 7-9)
- [ ] Write comprehensive unit tests
- [ ] Implement integration tests
- [ ] Add load testing for agent system
- [ ] Performance optimization and tuning
- [ ] Security audit and hardening
- [ ] Documentation completion

### Phase 8: Production Launch (Week 9-10)
- [ ] Production deployment and monitoring
- [ ] User acceptance testing
- [ ] Performance monitoring and optimization
- [ ] Bug fixes and stability improvements
- [ ] Feature enhancement based on feedback
- [ ] Scale planning and optimization