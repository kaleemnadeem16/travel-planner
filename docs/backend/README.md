# Backend Development Guide (Python)

## Technology Stack

- **Framework**: Django REST Framework (recommended) or Flask/FastAPI
- **Database ORM**: Django ORM or SQLAlchemy
- **Authentication**: JWT tokens + OAuth2
- **Password Hashing**: bcrypt or Argon2
- **Validation**: Django serializers or Pydantic
- **Rate Limiting**: Redis or in-memory counters

## Framework Comparison

### Django REST Framework (Recommended)
**Pros:**
- Built-in user model and admin interface
- Comprehensive authentication system
- Serializers for data validation
- Extensive middleware support
- Rich ecosystem and documentation

**Cons:**
- Heavier footprint
- More opinionated structure

### Flask/FastAPI Alternative
**Pros:**
- Lightweight and flexible
- Fast development for simple APIs
- FastAPI has automatic API documentation

**Cons:**
- More manual setup required
- Authentication implementation from scratch

## Project Structure (Django)

```
backend/
├── travel_planner/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── apps/
│   ├── authentication/
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   ├── plans/
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   └── core/
│       ├── permissions.py
│       ├── middleware.py
│       └── utils.py
├── requirements.txt
└── manage.py
```

## API Endpoints

### Authentication Endpoints

```python
# authentication/urls.py
urlpatterns = [
    path('signup/', SignupView.as_view(), name='signup'),
    path('login/', LoginView.as_view(), name='login'),
    path('logout/', LogoutView.as_view(), name='logout'),
    path('user/', UserProfileView.as_view(), name='user-profile'),
    path('token/refresh/', TokenRefreshView.as_view(), name='token-refresh'),
    
    # Social OAuth
    path('login/google/', GoogleOAuth2LoginView.as_view(), name='google-login'),
    path('login/facebook/', FacebookOAuth2LoginView.as_view(), name='facebook-login'),
]
```

### Plan Management Endpoints

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