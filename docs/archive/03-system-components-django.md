# System Components Architecture

This document provides detailed information about the Travel Planner application's system components, their responsibilities, interactions, and implementation patterns.

## 🏗️ Component Overview

The Travel Planner application follows a modular architecture with clearly defined component boundaries and responsibilities. Each component is designed to be loosely coupled, highly cohesive, and independently deployable.

### High-Level Component Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        EXTERNAL SERVICES                            │
├─────────────────────────────────────────────────────────────────────┤
│  OpenWeatherMap  │  OpenStreetMap  │  Foursquare  │  Google OAuth   │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                             External API Gateway
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                        PRESENTATION LAYER                           │
├─────────────────────────────────────────────────────────────────────┤
│                         React Frontend                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │    Auth     │  │    Plans    │  │   Profile   │  │   Shared    │ │
│  │ Components  │  │ Components  │  │ Components  │  │ Components  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │    State    │  │   Routing   │  │     API     │  │     UI      │ │
│  │ Management  │  │   System    │  │   Client    │  │ Framework   │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                              HTTP/HTTPS REST API
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                        APPLICATION LAYER                            │
├─────────────────────────────────────────────────────────────────────┤
│                        Python Backend                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │ Auth Service│  │Plan Service │  │ User Service│  │ External    │ │
│  │             │  │             │  │             │  │ API Service │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │ Middleware  │  │   API       │  │   Task      │  │   Cache     │ │
│  │   Layer     │  │ Endpoints   │  │   Queue     │  │   Layer     │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                              Database Queries
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                          DATA LAYER                                 │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────┐    ┌─────────────────────────────┐ │
│  │        PostgreSQL           │    │          Redis              │ │
│  │    ┌─────────────────┐      │    │   ┌─────────────────────┐   │ │
│  │    │ User Management │      │    │   │  Session Storage    │   │ │
│  │    │ Plan Storage    │      │    │   │  Rate Limiting      │   │ │
│  │    │ Activity Data   │      │    │   │  API Cache          │   │ │
│  │    │ System Config   │      │    │   │  Background Jobs    │   │ │
│  │    └─────────────────┘      │    │   └─────────────────────┘   │ │
│  └─────────────────────────────┘    └─────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

## 🎨 Frontend Components

### React Application Structure

#### Core Application Components

##### 1. App Component (Root)
**File**: `src/App.tsx`  
**Responsibilities**:
- Application root and global state initialization
- Route configuration and navigation
- Global error boundary and fallback UI
- Theme and context providers setup

```typescript
// App.tsx
import React from 'react';
import { BrowserRouter as Router } from 'react-router-dom';
import { AuthProvider } from './contexts/AuthContext';
import { ThemeProvider } from './contexts/ThemeContext';
import { AppRoutes } from './routes/AppRoutes';
import { ErrorBoundary } from './components/common/ErrorBoundary';
import { GlobalStyles } from './styles/GlobalStyles';

const App: React.FC = () => {
  return (
    <ErrorBoundary>
      <ThemeProvider>
        <AuthProvider>
          <Router>
            <GlobalStyles />
            <AppRoutes />
          </Router>
        </AuthProvider>
      </ThemeProvider>
    </ErrorBoundary>
  );
};

export default App;
```

##### 2. Authentication Components
**Directory**: `src/components/auth/`  
**Components**:
- `LoginForm.tsx` - User login interface
- `SignupForm.tsx` - User registration interface
- `PasswordResetForm.tsx` - Password reset functionality
- `SocialLoginButton.tsx` - OAuth social login buttons
- `ProtectedRoute.tsx` - Route protection wrapper

```typescript
// LoginForm.tsx
interface LoginFormProps {
  onSuccess: () => void;
  onError: (error: string) => void;
}

export const LoginForm: React.FC<LoginFormProps> = ({ onSuccess, onError }) => {
  const { login } = useAuth();
  const [formData, setFormData] = useState({ email: '', password: '' });
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsLoading(true);
    
    try {
      await login(formData);
      onSuccess();
    } catch (error) {
      onError(error.message);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="login-form">
      {/* Form implementation */}
    </form>
  );
};
```

##### 3. Plan Management Components
**Directory**: `src/components/plans/`  
**Components**:
- `PlanList.tsx` - Display user's travel plans
- `PlanCard.tsx` - Individual plan preview card
- `PlanForm.tsx` - Create/edit plan form
- `PlanDetails.tsx` - Detailed plan view
- `ActivityForm.tsx` - Add/edit activities
- `PlanShare.tsx` - Plan sharing functionality

```typescript
// PlanList.tsx
interface PlanListProps {
  userId: string;
  searchTerm?: string;
  sortBy?: 'date' | 'name' | 'updated';
}

export const PlanList: React.FC<PlanListProps> = ({ userId, searchTerm, sortBy = 'updated' }) => {
  const { plans, loading, error, fetchPlans } = usePlans(userId);
  const [filteredPlans, setFilteredPlans] = useState<Plan[]>([]);

  useEffect(() => {
    fetchPlans();
  }, [userId]);

  useEffect(() => {
    let filtered = plans;
    
    if (searchTerm) {
      filtered = plans.filter(plan => 
        plan.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
        plan.destination.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }
    
    filtered = sortPlans(filtered, sortBy);
    setFilteredPlans(filtered);
  }, [plans, searchTerm, sortBy]);

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} />;

  return (
    <div className="plan-list">
      {filteredPlans.map(plan => (
        <PlanCard key={plan.id} plan={plan} />
      ))}
    </div>
  );
};
```

#### State Management Components

##### 1. Context Providers
**Directory**: `src/contexts/`

```typescript
// AuthContext.tsx
interface AuthContextType {
  user: User | null;
  token: string | null;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
  signup: (userData: SignupData) => Promise<void>;
  isAuthenticated: boolean;
  isLoading: boolean;
}

export const AuthContext = createContext<AuthContextType | null>(null);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(localStorage.getItem('token'));
  const [isLoading, setIsLoading] = useState(false);

  const login = async (credentials: LoginCredentials) => {
    setIsLoading(true);
    try {
      const response = await authService.login(credentials);
      setUser(response.user);
      setToken(response.token);
      localStorage.setItem('token', response.token);
    } finally {
      setIsLoading(false);
    }
  };

  const logout = () => {
    setUser(null);
    setToken(null);
    localStorage.removeItem('token');
  };

  const value = {
    user,
    token,
    login,
    logout,
    signup,
    isAuthenticated: !!token,
    isLoading
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

##### 2. Custom Hooks
**Directory**: `src/hooks/`

```typescript
// usePlans.ts
export const usePlans = (userId: string) => {
  const [plans, setPlans] = useState<Plan[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchPlans = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await plansService.getPlans();
      setPlans(response.data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [userId]);

  const createPlan = async (planData: CreatePlanData) => {
    try {
      const response = await plansService.createPlan(planData);
      setPlans(prev => [...prev, response.data]);
      return response.data;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };

  const updatePlan = async (planId: string, updates: Partial<Plan>) => {
    try {
      const response = await plansService.updatePlan(planId, updates);
      setPlans(prev => prev.map(plan => 
        plan.id === planId ? response.data : plan
      ));
      return response.data;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };

  return {
    plans,
    loading,
    error,
    fetchPlans,
    createPlan,
    updatePlan
  };
};
```

#### API Client Components

##### 1. HTTP Client Configuration
**File**: `src/services/apiClient.ts`

```typescript
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';

class APIClient {
  private client: AxiosInstance;

  constructor(baseURL: string) {
    this.client = axios.create({
      baseURL,
      timeout: 10000,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    this.setupInterceptors();
  }

  private setupInterceptors() {
    // Request interceptor for auth token
    this.client.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    // Response interceptor for error handling
    this.client.interceptors.response.use(
      (response) => response,
      (error) => {
        if (error.response?.status === 401) {
          localStorage.removeItem('token');
          window.location.href = '/login';
        }
        return Promise.reject(error);
      }
    );
  }

  async get<T>(url: string, config?: AxiosRequestConfig): Promise<AxiosResponse<T>> {
    return this.client.get<T>(url, config);
  }

  async post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<AxiosResponse<T>> {
    return this.client.post<T>(url, data, config);
  }

  // Additional HTTP methods...
}

export const apiClient = new APIClient(process.env.REACT_APP_API_URL!);
```

## ⚙️ Backend Components

### Service Layer Architecture

#### 1. Authentication Service
**File**: `backend/apps/authentication/services.py`

```python
from typing import Optional, Dict, Any
from django.contrib.auth import authenticate
from django.core.exceptions import ValidationError
from rest_framework_simplejwt.tokens import RefreshToken
from .models import CustomUser
from .serializers import UserSerializer

class AuthenticationService:
    """Service for handling user authentication operations."""
    
    @staticmethod
    def register_user(user_data: Dict[str, Any]) -> Dict[str, Any]:
        """Register a new user and return user data with tokens."""
        serializer = UserRegistrationSerializer(data=user_data)
        serializer.is_valid(raise_exception=True)
        
        user = serializer.save()
        refresh = RefreshToken.for_user(user)
        
        return {
            'user': UserSerializer(user).data,
            'tokens': {
                'refresh': str(refresh),
                'access': str(refresh.access_token),
            }
        }
    
    @staticmethod
    def authenticate_user(email: str, password: str) -> Dict[str, Any]:
        """Authenticate user credentials and return tokens."""
        user = authenticate(username=email, password=password)
        
        if not user:
            raise ValidationError('Invalid credentials')
        
        if not user.is_active:
            raise ValidationError('User account is disabled')
        
        refresh = RefreshToken.for_user(user)
        
        return {
            'user': UserSerializer(user).data,
            'tokens': {
                'refresh': str(refresh),
                'access': str(refresh.access_token),
            }
        }
    
    @staticmethod
    def refresh_token(refresh_token: str) -> Dict[str, str]:
        """Refresh access token using refresh token."""
        try:
            refresh = RefreshToken(refresh_token)
            return {
                'access': str(refresh.access_token)
            }
        except Exception:
            raise ValidationError('Invalid refresh token')
```

#### 2. Plan Management Service
**File**: `backend/apps/plans/services.py`

```python
from typing import List, Dict, Any, Optional
from django.db.models import QuerySet
from django.core.exceptions import ValidationError, PermissionDenied
from .models import Plan, Activity
from .serializers import PlanSerializer, ActivitySerializer
from ..external_apis.services import ExternalAPIService

class PlanService:
    """Service for handling travel plan operations."""
    
    def __init__(self):
        self.external_api_service = ExternalAPIService()
    
    def get_user_plans(self, user_id: int) -> QuerySet[Plan]:
        """Get all plans for a specific user."""
        return Plan.objects.filter(user_id=user_id).order_by('-updated_at')
    
    def get_plan_by_id(self, plan_id: int, user_id: int) -> Plan:
        """Get a specific plan, ensuring user ownership."""
        try:
            plan = Plan.objects.get(id=plan_id, user_id=user_id)
            return plan
        except Plan.DoesNotExist:
            raise ValidationError('Plan not found')
    
    async def create_plan(self, user_id: int, plan_data: Dict[str, Any]) -> Plan:
        """Create a new travel plan with enhanced data."""
        # Validate and serialize plan data
        serializer = PlanSerializer(data=plan_data)
        serializer.is_valid(raise_exception=True)
        
        # Enhance plan with external API data
        if plan_data.get('destination'):
            enhanced_data = await self.external_api_service.enhance_destination_data(
                plan_data['destination']
            )
            serializer.validated_data['plan_data'].update(enhanced_data)
        
        # Save plan
        plan = serializer.save(user_id=user_id)
        return plan
    
    def update_plan(self, plan_id: int, user_id: int, update_data: Dict[str, Any]) -> Plan:
        """Update an existing plan."""
        plan = self.get_plan_by_id(plan_id, user_id)
        
        serializer = PlanSerializer(plan, data=update_data, partial=True)
        serializer.is_valid(raise_exception=True)
        
        updated_plan = serializer.save()
        return updated_plan
    
    def delete_plan(self, plan_id: int, user_id: int) -> None:
        """Delete a plan."""
        plan = self.get_plan_by_id(plan_id, user_id)
        plan.delete()
    
    def add_activity(self, plan_id: int, user_id: int, activity_data: Dict[str, Any]) -> Activity:
        """Add an activity to a plan."""
        plan = self.get_plan_by_id(plan_id, user_id)
        
        serializer = ActivitySerializer(data=activity_data)
        serializer.is_valid(raise_exception=True)
        
        activity = serializer.save(plan=plan)
        return activity
```

#### 3. External API Service
**File**: `backend/apps/external_apis/services.py`

```python
import asyncio
from typing import Dict, Any, Optional
from django.core.cache import cache
from .clients import WeatherAPIClient, GeocodingAPIClient, PlacesAPIClient

class ExternalAPIService:
    """Service for managing external API integrations."""
    
    def __init__(self):
        self.weather_client = WeatherAPIClient()
        self.geocoding_client = GeocodingAPIClient()
        self.places_client = PlacesAPIClient()
    
    async def enhance_destination_data(self, destination: str) -> Dict[str, Any]:
        """Enhance destination data with information from external APIs."""
        # Check cache first
        cache_key = f"destination_data:{destination}"
        cached_data = cache.get(cache_key)
        if cached_data:
            return cached_data
        
        # Geocode destination
        location_data = await self.geocoding_client.geocode(destination)
        if not location_data:
            return {}
        
        lat, lon = location_data['latitude'], location_data['longitude']
        
        # Gather data from multiple APIs concurrently
        tasks = [
            self.weather_client.get_current_weather(lat, lon),
            self.weather_client.get_forecast(lat, lon),
            self.places_client.search_attractions(lat, lon),
            self.places_client.search_restaurants(lat, lon),
        ]
        
        weather, forecast, attractions, restaurants = await asyncio.gather(
            *tasks, return_exceptions=True
        )
        
        enhanced_data = {
            'location': location_data,
            'weather': {
                'current': weather if not isinstance(weather, Exception) else None,
                'forecast': forecast if not isinstance(forecast, Exception) else None,
            },
            'places': {
                'attractions': attractions if not isinstance(attractions, Exception) else [],
                'restaurants': restaurants if not isinstance(restaurants, Exception) else [],
            }
        }
        
        # Cache for 1 hour
        cache.set(cache_key, enhanced_data, 3600)
        return enhanced_data
```

### API Endpoint Layer

#### 1. Authentication Endpoints
**File**: `backend/apps/authentication/views.py`

```python
from rest_framework import status, generics
from rest_framework.response import Response
from rest_framework.permissions import AllowAny, IsAuthenticated
from rest_framework.decorators import api_view, permission_classes
from .services import AuthenticationService
from .serializers import LoginSerializer, UserRegistrationSerializer

class RegisterView(generics.CreateAPIView):
    """User registration endpoint."""
    serializer_class = UserRegistrationSerializer
    permission_classes = [AllowAny]
    
    def create(self, request, *args, **kwargs):
        try:
            result = AuthenticationService.register_user(request.data)
            return Response(result, status=status.HTTP_201_CREATED)
        except ValidationError as e:
            return Response(
                {'error': str(e)}, 
                status=status.HTTP_400_BAD_REQUEST
            )

class LoginView(generics.GenericAPIView):
    """User login endpoint."""
    serializer_class = LoginSerializer
    permission_classes = [AllowAny]
    
    def post(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        
        try:
            result = AuthenticationService.authenticate_user(
                email=serializer.validated_data['email'],
                password=serializer.validated_data['password']
            )
            return Response(result, status=status.HTTP_200_OK)
        except ValidationError as e:
            return Response(
                {'error': str(e)}, 
                status=status.HTTP_401_UNAUTHORIZED
            )

@api_view(['POST'])
@permission_classes([IsAuthenticated])
def logout_view(request):
    """User logout endpoint."""
    # Since we're using JWT, logout is handled client-side
    # This endpoint can be used for logging purposes
    return Response(
        {'message': 'Successfully logged out'}, 
        status=status.HTTP_200_OK
    )
```

#### 2. Plan Management Endpoints
**File**: `backend/apps/plans/views.py`

```python
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from .models import Plan
from .serializers import PlanSerializer
from .services import PlanService
from .permissions import IsPlanOwner

class PlanViewSet(viewsets.ModelViewSet):
    """ViewSet for managing travel plans."""
    serializer_class = PlanSerializer
    permission_classes = [IsAuthenticated, IsPlanOwner]
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.plan_service = PlanService()
    
    def get_queryset(self):
        """Return plans for the current user."""
        return self.plan_service.get_user_plans(self.request.user.id)
    
    async def create(self, request, *args, **kwargs):
        """Create a new plan with enhanced data."""
        try:
            plan = await self.plan_service.create_plan(
                user_id=request.user.id,
                plan_data=request.data
            )
            serializer = self.get_serializer(plan)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        except ValidationError as e:
            return Response(
                {'error': str(e)}, 
                status=status.HTTP_400_BAD_REQUEST
            )
    
    @action(detail=True, methods=['post'])
    def add_activity(self, request, pk=None):
        """Add an activity to a plan."""
        try:
            activity = self.plan_service.add_activity(
                plan_id=pk,
                user_id=request.user.id,
                activity_data=request.data
            )
            from .serializers import ActivitySerializer
            serializer = ActivitySerializer(activity)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        except ValidationError as e:
            return Response(
                {'error': str(e)}, 
                status=status.HTTP_400_BAD_REQUEST
            )
```

### Middleware Components

#### 1. Rate Limiting Middleware
**File**: `backend/apps/core/middleware.py`

```python
import redis
from django.http import JsonResponse
from django.utils.deprecation import MiddlewareMixin
from django.conf import settings

class RateLimitMiddleware(MiddlewareMixin):
    """Middleware for rate limiting anonymous users."""
    
    def __init__(self, get_response):
        self.get_response = get_response
        self.redis_client = redis.Redis.from_url(settings.REDIS_URL)
    
    def process_request(self, request):
        """Process incoming request for rate limiting."""
        # Skip rate limiting for authenticated users
        if hasattr(request, 'user') and request.user.is_authenticated:
            return None
        
        # Skip rate limiting for certain paths
        excluded_paths = ['/api/auth/login/', '/api/auth/register/', '/api/health/']
        if any(request.path.startswith(path) for path in excluded_paths):
            return None
        
        # Check rate limit
        if not self._check_rate_limit(request):
            return JsonResponse({
                'error': 'Rate limit exceeded. Please sign up for unlimited access.',
                'limit': 10,
                'period': 'month'
            }, status=429)
        
        return None
    
    def _check_rate_limit(self, request) -> bool:
        """Check if request is within rate limit."""
        ip_address = self._get_client_ip(request)
        current_month = timezone.now().strftime("%Y-%m")
        key = f"rate_limit:{ip_address}:{current_month}"
        
        current_count = self.redis_client.get(key)
        if current_count is None:
            # First request this month
            self.redis_client.setex(key, 30 * 24 * 60 * 60, 1)  # 30 days
            return True
        
        current_count = int(current_count)
        if current_count >= 10:  # Rate limit exceeded
            return False
        
        # Increment counter
        self.redis_client.incr(key)
        return True
    
    def _get_client_ip(self, request) -> str:
        """Get client IP address from request."""
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip
```

## 🗄️ Data Layer Components

### Database Models

#### 1. User Model
**File**: `backend/apps/authentication/models.py`

```python
from django.contrib.auth.models import AbstractUser
from django.db import models
from django.core.validators import EmailValidator

class CustomUser(AbstractUser):
    """Extended user model with additional fields."""
    
    email = models.EmailField(
        unique=True,
        validators=[EmailValidator()],
        help_text='User email address'
    )
    first_name = models.CharField(max_length=30, blank=True)
    last_name = models.CharField(max_length=30, blank=True)
    
    # Account status
    is_verified = models.BooleanField(default=False)
    
    # OAuth integration
    google_id = models.CharField(max_length=100, blank=True, null=True, unique=True)
    facebook_id = models.CharField(max_length=100, blank=True, null=True, unique=True)
    
    # Preferences
    preferred_currency = models.CharField(max_length=3, default='USD')
    timezone = models.CharField(max_length=50, default='UTC')
    
    # Timestamps
    date_joined = models.DateTimeField(auto_now_add=True)
    last_login = models.DateTimeField(null=True, blank=True)
    
    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']
    
    class Meta:
        db_table = 'users'
        indexes = [
            models.Index(fields=['email']),
            models.Index(fields=['google_id']),
            models.Index(fields=['facebook_id']),
        ]
    
    def __str__(self):
        return self.email
    
    @property
    def full_name(self):
        """Return full name."""
        return f"{self.first_name} {self.last_name}".strip()
```

#### 2. Plan Model
**File**: `backend/apps/plans/models.py`

```python
from django.db import models
from django.contrib.auth import get_user_model
from django.core.validators import MinValueValidator

User = get_user_model()

class Plan(models.Model):
    """Model for travel plans."""
    
    STATUS_CHOICES = [
        ('draft', 'Draft'),
        ('published', 'Published'),
        ('archived', 'Archived'),
    ]
    
    # Relationships
    user = models.ForeignKey(
        User, 
        on_delete=models.CASCADE, 
        related_name='plans'
    )
    
    # Basic information
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    destination = models.CharField(max_length=100)
    
    # Trip details
    start_date = models.DateField(null=True, blank=True)
    end_date = models.DateField(null=True, blank=True)
    budget = models.DecimalField(
        max_digits=10, 
        decimal_places=2, 
        null=True, 
        blank=True,
        validators=[MinValueValidator(0)]
    )
    currency = models.CharField(max_length=3, default='USD')
    
    # Status and visibility
    status = models.CharField(
        max_length=20, 
        choices=STATUS_CHOICES, 
        default='draft'
    )
    is_public = models.BooleanField(default=False)
    share_token = models.CharField(max_length=100, unique=True, null=True, blank=True)
    
    # Flexible data storage
    plan_data = models.JSONField(default=dict, blank=True)
    
    # Timestamps
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        db_table = 'plans'
        ordering = ['-updated_at']
        indexes = [
            models.Index(fields=['user', '-updated_at']),
            models.Index(fields=['status']),
            models.Index(fields=['destination']),
            models.Index(fields=['share_token']),
        ]
    
    def __str__(self):
        return f"{self.title} - {self.user.email}"
    
    @property
    def duration_days(self):
        """Calculate trip duration in days."""
        if self.start_date and self.end_date:
            return (self.end_date - self.start_date).days + 1
        return None
```

### Repository Pattern Implementation

#### 1. Abstract Repository
**File**: `backend/apps/core/repositories.py`

```python
from abc import ABC, abstractmethod
from typing import List, Optional, Dict, Any
from django.db.models import Model, QuerySet

class BaseRepository(ABC):
    """Abstract base repository for data access operations."""
    
    @abstractmethod
    def get_by_id(self, id: int) -> Optional[Model]:
        """Get entity by ID."""
        pass
    
    @abstractmethod
    def get_all(self) -> QuerySet:
        """Get all entities."""
        pass
    
    @abstractmethod
    def create(self, data: Dict[str, Any]) -> Model:
        """Create new entity."""
        pass
    
    @abstractmethod
    def update(self, id: int, data: Dict[str, Any]) -> Model:
        """Update existing entity."""
        pass
    
    @abstractmethod
    def delete(self, id: int) -> bool:
        """Delete entity."""
        pass

class PlanRepository(BaseRepository):
    """Repository for Plan model operations."""
    
    def __init__(self):
        from apps.plans.models import Plan
        self.model = Plan
    
    def get_by_id(self, id: int) -> Optional['Plan']:
        """Get plan by ID."""
        try:
            return self.model.objects.get(id=id)
        except self.model.DoesNotExist:
            return None
    
    def get_by_user(self, user_id: int) -> QuerySet:
        """Get all plans for a user."""
        return self.model.objects.filter(user_id=user_id)
    
    def get_all(self) -> QuerySet:
        """Get all plans."""
        return self.model.objects.all()
    
    def create(self, data: Dict[str, Any]) -> 'Plan':
        """Create new plan."""
        return self.model.objects.create(**data)
    
    def update(self, id: int, data: Dict[str, Any]) -> 'Plan':
        """Update existing plan."""
        plan = self.get_by_id(id)
        if not plan:
            raise ValueError(f"Plan with id {id} not found")
        
        for key, value in data.items():
            setattr(plan, key, value)
        plan.save()
        return plan
    
    def delete(self, id: int) -> bool:
        """Delete plan."""
        plan = self.get_by_id(id)
        if plan:
            plan.delete()
            return True
        return False
```

## 🔄 Component Interactions

### Request Flow Patterns

#### 1. Authentication Flow
```
User Input → LoginForm → AuthContext → AuthService → API Client → Backend Auth Service → Database → JWT Response → Token Storage → State Update
```

#### 2. Plan Creation Flow
```
User Input → PlanForm → Validation → PlanService → External API Enhancement → Database Save → State Update → UI Refresh
```

#### 3. Data Fetching Flow
```
Component Mount → useEffect → Custom Hook → API Service → Cache Check → Backend Request → Database Query → Response → State Update → Re-render
```

### Inter-Component Communication

#### 1. Parent-Child Communication
- Props for data passing down
- Callback functions for events passing up
- Context for deeply nested components

#### 2. Sibling Component Communication
- Shared state through common parent
- Context providers for related components
- Event emitters for loosely coupled components

#### 3. Cross-Layer Communication
- API services for frontend-backend communication
- Event system for real-time updates
- WebSocket connections for live features

## 📊 Component Dependencies

### Frontend Dependencies
```typescript
// Core dependencies
"react": "^18.2.0",
"react-dom": "^18.2.0",
"react-router-dom": "^6.8.0",
"typescript": "^4.9.0",

// State management
"@reduxjs/toolkit": "^1.9.0",
"react-redux": "^8.0.0",

// HTTP client
"axios": "^1.3.0",

// UI components
"@mui/material": "^5.11.0",
"@emotion/react": "^11.10.0",
"@emotion/styled": "^11.10.0",

// Forms and validation
"react-hook-form": "^7.43.0",
"yup": "^1.0.0",

// Date handling
"date-fns": "^2.29.0",

// Maps and location
"leaflet": "^1.9.0",
"react-leaflet": "^4.2.0"
```

### Backend Dependencies
```python
# Core framework
Django==4.1.7
djangorestframework==3.14.0
django-cors-headers==3.14.0

# Database
psycopg2-binary==2.9.5
django-redis==5.2.0

# Authentication
djangorestframework-simplejwt==5.2.2
social-auth-app-django==5.0.0

# API integrations
requests==2.28.2
aiohttp==3.8.4

# Validation and serialization
marshmallow==3.19.0
django-filter==22.1

# Testing
pytest-django==4.5.2
factory-boy==3.2.1

# Production
gunicorn==20.1.0
whitenoise==6.4.0
```

## 🔍 Monitoring and Health Checks

### Component Health Monitoring

#### 1. Frontend Health Check
```typescript
// Health check service
export class HealthCheckService {
  async checkComponentHealth(): Promise<HealthStatus> {
    const checks = await Promise.allSettled([
      this.checkAPIConnection(),
      this.checkLocalStorage(),
      this.checkBrowserFeatures(),
    ]);

    return {
      api: checks[0].status === 'fulfilled' ? 'healthy' : 'unhealthy',
      storage: checks[1].status === 'fulfilled' ? 'healthy' : 'unhealthy',
      browser: checks[2].status === 'fulfilled' ? 'healthy' : 'unhealthy',
    };
  }

  private async checkAPIConnection(): Promise<void> {
    await apiClient.get('/health/');
  }

  private async checkLocalStorage(): Promise<void> {
    localStorage.setItem('health_check', 'test');
    const value = localStorage.getItem('health_check');
    if (value !== 'test') throw new Error('LocalStorage not working');
    localStorage.removeItem('health_check');
  }

  private async checkBrowserFeatures(): Promise<void> {
    if (!window.fetch) throw new Error('Fetch API not supported');
    if (!window.localStorage) throw new Error('LocalStorage not supported');
  }
}
```

#### 2. Backend Health Check
```python
# Health check endpoint
@api_view(['GET'])
@permission_classes([AllowAny])
def health_check(request):
    """Comprehensive health check endpoint."""
    status = {
        'status': 'healthy',
        'timestamp': timezone.now().isoformat(),
        'version': '1.0.0',
        'components': {}
    }
    
    # Database check
    try:
        from django.db import connection
        with connection.cursor() as cursor:
            cursor.execute("SELECT 1")
        status['components']['database'] = 'healthy'
    except Exception as e:
        status['components']['database'] = f'unhealthy: {str(e)}'
        status['status'] = 'unhealthy'
    
    # Redis check
    try:
        from django.core.cache import cache
        cache.set('health_check', 'test', 10)
        if cache.get('health_check') != 'test':
            raise Exception('Cache write/read failed')
        status['components']['redis'] = 'healthy'
    except Exception as e:
        status['components']['redis'] = f'unhealthy: {str(e)}'
        status['status'] = 'degraded'
    
    # External API check
    try:
        # Quick check of external APIs
        external_api_service = ExternalAPIService()
        # Implement a lightweight check
        status['components']['external_apis'] = 'healthy'
    except Exception as e:
        status['components']['external_apis'] = f'unhealthy: {str(e)}'
        status['status'] = 'degraded'
    
    return Response(status)
```

---

**Last Updated**: August 16, 2025  
**Version**: 1.0.0  
**Next Review**: September 1, 2025