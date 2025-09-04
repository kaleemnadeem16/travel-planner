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
# plans/urls.py
urlpatterns = [
    path('plans/', PlanListCreateView.as_view(), name='plan-list-create'),
    path('plans/<int:pk>/', PlanDetailView.as_view(), name='plan-detail'),
    path('plans/<int:pk>/share/', PlanShareView.as_view(), name='plan-share'),
]
```

## Data Models

### User Model (Custom)

```python
# authentication/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    email = models.EmailField(unique=True)
    first_name = models.CharField(max_length=30, blank=True)
    last_name = models.CharField(max_length=30, blank=True)
    date_joined = models.DateTimeField(auto_now_add=True)
    is_verified = models.BooleanField(default=False)
    
    # OAuth fields
    google_id = models.CharField(max_length=100, blank=True, null=True)
    facebook_id = models.CharField(max_length=100, blank=True, null=True)
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']
    
    def __str__(self):
        return self.email
```

### Plan Model

```python
# plans/models.py
from django.db import models
from django.contrib.auth import get_user_model

User = get_user_model()

class Plan(models.Model):
    PLAN_STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='plans')
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    destination = models.CharField(max_length=100)
    start_date = models.DateField(null=True, blank=True)
    end_date = models.DateField(null=True, blank=True)
    budget = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)
    status = models.CharField(max_length=20, choices=PLAN_STATUS_CHOICES, default='draft')
    
    # JSON field for flexible plan data
    plan_data = models.JSONField(default=dict, blank=True)
    
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-updated_at']
        indexes = [
            models.Index(fields=['user', '-updated_at']),
        ]
    
    def __str__(self):
        return f"{self.title} - {self.user.email}"
```

## Serializers

### Authentication Serializers

```python
# authentication/serializers.py
from rest_framework import serializers
from django.contrib.auth import authenticate
from .models import CustomUser

class UserRegistrationSerializer(serializers.ModelSerializer):
    password = serializers.CharField(write_only=True, min_length=8)
    password_confirm = serializers.CharField(write_only=True)
    
    class Meta:
        model = CustomUser
        fields = ('email', 'username', 'first_name', 'last_name', 'password', 'password_confirm')
    
    def validate(self, data):
        if data['password'] != data['password_confirm']:
            raise serializers.ValidationError("Passwords don't match")
        return data
    
    def create(self, validated_data):
        validated_data.pop('password_confirm')
        user = CustomUser.objects.create_user(**validated_data)
        return user

class UserLoginSerializer(serializers.Serializer):
    email = serializers.EmailField()
    password = serializers.CharField()
    
    def validate(self, data):
        email = data.get('email')
        password = data.get('password')
        
        if email and password:
            user = authenticate(username=email, password=password)
            if not user:
                raise serializers.ValidationError('Invalid credentials')
            if not user.is_active:
                raise serializers.ValidationError('User account is disabled')
            data['user'] = user
        return data
```

### Plan Serializers

```python
# plans/serializers.py
from rest_framework import serializers
from .models import Plan

class PlanSerializer(serializers.ModelSerializer):
    user = serializers.StringRelatedField(read_only=True)
    
    class Meta:
        model = Plan
        fields = '__all__'
        read_only_fields = ('user', 'created_at', 'updated_at')
    
    def validate(self, data):
        if data.get('start_date') and data.get('end_date'):
            if data['start_date'] > data['end_date']:
                raise serializers.ValidationError("Start date must be before end date")
        return data
```

## Views

### Authentication Views

```python
# authentication/views.py
from rest_framework import status, generics
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from rest_framework_simplejwt.tokens import RefreshToken
from .serializers import UserRegistrationSerializer, UserLoginSerializer

class SignupView(generics.CreateAPIView):
    serializer_class = UserRegistrationSerializer
    
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = serializer.save()
        
        # Generate JWT tokens
        refresh = RefreshToken.for_user(user)
        
        return Response({
            'user': {
                'id': user.id,
                'email': user.email,
                'username': user.username,
            },
            'tokens': {
                'refresh': str(refresh),
                'access': str(refresh.access_token),
            }
        }, status=status.HTTP_201_CREATED)

class LoginView(generics.GenericAPIView):
    serializer_class = UserLoginSerializer
    
    def post(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = serializer.validated_data['user']
        
        refresh = RefreshToken.for_user(user)
        
        return Response({
            'user': {
                'id': user.id,
                'email': user.email,
                'username': user.username,
            },
            'tokens': {
                'refresh': str(refresh),
                'access': str(refresh.access_token),
            }
        })
```

### Plan Views

```python
# plans/views.py
from rest_framework import generics, permissions
from rest_framework.response import Response
from .models import Plan
from .serializers import PlanSerializer

class PlanListCreateView(generics.ListCreateAPIView):
    serializer_class = PlanSerializer
    permission_classes = [permissions.IsAuthenticated]
    
    def get_queryset(self):
        return Plan.objects.filter(user=self.request.user)
    
    def perform_create(self, serializer):
        serializer.save(user=self.request.user)

class PlanDetailView(generics.RetrieveUpdateDestroyAPIView):
    serializer_class = PlanSerializer
    permission_classes = [permissions.IsAuthenticated]
    
    def get_queryset(self):
        return Plan.objects.filter(user=self.request.user)
```

## Rate Limiting

### Redis-based Rate Limiting

```python
# core/middleware.py
import redis
from django.http import JsonResponse
from django.utils.deprecation import MiddlewareMixin

class RateLimitMiddleware(MiddlewareMixin):
    def __init__(self, get_response):
        self.get_response = get_response
        self.redis_client = redis.Redis(host='localhost', port=6379, db=0)
    
    def process_request(self, request):
        if request.user.is_authenticated:
            return None  # No rate limiting for authenticated users
        
        ip = self.get_client_ip(request)
        key = f"rate_limit:{ip}"
        
        current_count = self.redis_client.get(key)
        if current_count is None:
            self.redis_client.setex(key, 30 * 24 * 60 * 60, 1)  # 30 days
        else:
            current_count = int(current_count)
            if current_count >= 10:  # 10 requests per month
                return JsonResponse({
                    'error': 'Rate limit exceeded. Please sign up for continued access.'
                }, status=429)
            self.redis_client.incr(key)
    
    def get_client_ip(self, request):
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip
```

## Security Considerations

### Settings Configuration

```python
# settings.py
import os
from datetime import timedelta

# JWT Configuration
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
}

# Password Validation
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {'min_length': 8}
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

# CORS Configuration
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",  # React development server
    "https://your-domain.com",  # Production frontend
]

# Security Headers
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'
```

## Social OAuth Integration

### Google OAuth Setup

```python
# Install: pip install social-auth-app-django

# settings.py
INSTALLED_APPS = [
    # ... other apps
    'social_django',
]

AUTHENTICATION_BACKENDS = [
    'social_core.backends.google.GoogleOAuth2',
    'django.contrib.auth.backends.ModelBackend',
]

SOCIAL_AUTH_GOOGLE_OAUTH2_KEY = os.getenv('GOOGLE_OAUTH2_KEY')
SOCIAL_AUTH_GOOGLE_OAUTH2_SECRET = os.getenv('GOOGLE_OAUTH2_SECRET')
```

## Testing

### Unit Tests Example

```python
# tests/test_authentication.py
from django.test import TestCase
from django.contrib.auth import get_user_model
from rest_framework.test import APITestCase

User = get_user_model()

class AuthenticationTestCase(APITestCase):
    def test_user_registration(self):
        data = {
            'email': 'test@example.com',
            'username': 'testuser',
            'password': 'testpass123',
            'password_confirm': 'testpass123'
        }
        response = self.client.post('/api/auth/signup/', data)
        self.assertEqual(response.status_code, 201)
        self.assertTrue(User.objects.filter(email='test@example.com').exists())
```

## TODO: Implementation Tasks

- [ ] Set up Django project with DRF
- [ ] Configure custom user model
- [ ] Implement JWT authentication
- [ ] Create plan CRUD operations
- [ ] Add input validation and serializers
- [ ] Implement rate limiting
- [ ] Set up social OAuth
- [ ] Add comprehensive error handling
- [ ] Write unit and integration tests
- [ ] Configure logging and monitoring
- [ ] Set up database migrations
- [ ] Implement permissions and authorization