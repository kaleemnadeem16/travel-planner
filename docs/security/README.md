# Authentication & Security

## Authentication Strategy

### Multi-Modal Authentication
- **Primary**: Email/password authentication
- **Secondary**: Social OAuth (Google, Facebook)
- **Token-based**: JWT (JSON Web Tokens) for stateless authentication
- **Session Management**: Optional server-side sessions for enhanced security

## Password Security

### Password Requirements
- Minimum 8 characters
- Must contain at least one uppercase letter
- Must contain at least one lowercase letter  
- Must contain at least one number
- Must contain at least one special character
- Cannot be a common password (use blacklist)
- Cannot be too similar to user information

### Password Storage
```python
# Django example using Argon2 (recommended)
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
    'django.contrib.auth.hashers.BCryptPasswordHasher',
]

# Manual password hashing
import argon2
ph = argon2.PasswordHasher()
hashed = ph.hash("user_password")
verified = ph.verify(hashed, "user_password")
```

## JWT Token Implementation

### Token Structure
```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "user_id": 123,
    "email": "user@example.com",
    "exp": 1635724800,
    "iat": 1635721200,
    "jti": "unique-token-id"
  },
  "signature": "encrypted-signature"
}
```

### Token Configuration
```python
# Django JWT Settings
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'UPDATE_LAST_LOGIN': True,
    
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': SECRET_KEY,
    'VERIFYING_KEY': None,
    
    'AUTH_HEADER_TYPES': ('Bearer',),
    'AUTH_HEADER_NAME': 'HTTP_AUTHORIZATION',
    'USER_ID_FIELD': 'id',
    'USER_ID_CLAIM': 'user_id',
}
```

### Token Management
```python
# Token generation
from rest_framework_simplejwt.tokens import RefreshToken

def get_tokens_for_user(user):
    refresh = RefreshToken.for_user(user)
    return {
        'refresh': str(refresh),
        'access': str(refresh.access_token),
    }

# Token verification middleware
def jwt_middleware(get_response):
    def middleware(request):
        token = request.META.get('HTTP_AUTHORIZATION')
        if token and token.startswith('Bearer '):
            try:
                # Verify and decode token
                payload = jwt.decode(token[7:], SECRET_KEY, algorithms=['HS256'])
                request.user_id = payload['user_id']
            except jwt.ExpiredSignatureError:
                return JsonResponse({'error': 'Token expired'}, status=401)
            except jwt.InvalidTokenError:
                return JsonResponse({'error': 'Invalid token'}, status=401)
        
        response = get_response(request)
        return response
    return middleware
```

## Social OAuth Integration

### Google OAuth 2.0

#### Setup Configuration
```python
# Environment variables
GOOGLE_OAUTH2_CLIENT_ID = "your-google-client-id"
GOOGLE_OAUTH2_CLIENT_SECRET = "your-google-client-secret"
GOOGLE_OAUTH2_REDIRECT_URI = "http://localhost:8000/auth/google/callback/"

# Django social auth settings
SOCIAL_AUTH_GOOGLE_OAUTH2_KEY = GOOGLE_OAUTH2_CLIENT_ID
SOCIAL_AUTH_GOOGLE_OAUTH2_SECRET = GOOGLE_OAUTH2_CLIENT_SECRET
SOCIAL_AUTH_GOOGLE_OAUTH2_SCOPE = [
    'https://www.googleapis.com/auth/userinfo.email',
    'https://www.googleapis.com/auth/userinfo.profile',
]
```

#### OAuth Flow Implementation
```python
# Backend OAuth view
class GoogleOAuth2LoginView(APIView):
    def post(self, request):
        code = request.data.get('code')
        if not code:
            return Response({'error': 'Authorization code required'}, status=400)
        
        # Exchange code for access token
        token_url = 'https://oauth2.googleapis.com/token'
        token_data = {
            'code': code,
            'client_id': GOOGLE_OAUTH2_CLIENT_ID,
            'client_secret': GOOGLE_OAUTH2_CLIENT_SECRET,
            'redirect_uri': GOOGLE_OAUTH2_REDIRECT_URI,
            'grant_type': 'authorization_code',
        }
        
        token_response = requests.post(token_url, data=token_data)
        token_json = token_response.json()
        
        if 'access_token' not in token_json:
            return Response({'error': 'Failed to obtain access token'}, status=400)
        
        # Get user info from Google
        user_info_url = f"https://www.googleapis.com/oauth2/v2/userinfo?access_token={token_json['access_token']}"
        user_response = requests.get(user_info_url)
        user_data = user_response.json()
        
        # Create or update user
        user, created = CustomUser.objects.get_or_create(
            email=user_data['email'],
            defaults={
                'username': user_data['email'],
                'first_name': user_data.get('given_name', ''),
                'last_name': user_data.get('family_name', ''),
                'google_id': user_data['id'],
                'is_verified': True,
            }
        )
        
        if not created and not user.google_id:
            user.google_id = user_data['id']
            user.save()
        
        # Generate JWT tokens
        tokens = get_tokens_for_user(user)
        
        return Response({
            'user': UserSerializer(user).data,
            'tokens': tokens,
        })
```

#### Frontend OAuth Implementation
```javascript
// React Google OAuth
import { GoogleOAuth2Strategy } from 'passport-google-oauth20';

const GoogleLoginButton = () => {
  const handleGoogleLogin = () => {
    const params = new URLSearchParams({
      client_id: process.env.REACT_APP_GOOGLE_CLIENT_ID,
      redirect_uri: `${window.location.origin}/auth/google/callback`,
      scope: 'openid email profile',
      response_type: 'code',
      state: generateRandomState(), // CSRF protection
    });
    
    window.location.href = `https://accounts.google.com/oauth/authorize?${params}`;
  };

  return (
    <button onClick={handleGoogleLogin} className="google-login-btn">
      Login with Google
    </button>
  );
};

// Handle OAuth callback
const GoogleCallback = () => {
  useEffect(() => {
    const urlParams = new URLSearchParams(window.location.search);
    const code = urlParams.get('code');
    const state = urlParams.get('state');
    
    if (code && validateState(state)) {
      // Send code to backend
      axios.post('/api/auth/google/', { code })
        .then(response => {
          // Store tokens and user data
          localStorage.setItem('access_token', response.data.tokens.access);
          localStorage.setItem('refresh_token', response.data.tokens.refresh);
          // Redirect to dashboard
          navigate('/dashboard');
        })
        .catch(error => {
          console.error('OAuth login failed:', error);
        });
    }
  }, []);

  return <div>Processing login...</div>;
};
```

### Facebook OAuth (Similar Implementation)
```python
SOCIAL_AUTH_FACEBOOK_KEY = "your-facebook-app-id"
SOCIAL_AUTH_FACEBOOK_SECRET = "your-facebook-app-secret"
SOCIAL_AUTH_FACEBOOK_SCOPE = ['email']
```

## Rate Limiting & Security

### Anonymous User Rate Limiting

#### Redis-based Implementation
```python
import redis
from django.core.cache import cache

class RateLimitMixin:
    def check_rate_limit(self, request):
        if request.user.is_authenticated:
            return True  # No limit for authenticated users
        
        ip_address = self.get_client_ip(request)
        cache_key = f"rate_limit:{ip_address}"
        
        current_month = datetime.now().strftime("%Y-%m")
        monthly_key = f"{cache_key}:{current_month}"
        
        current_count = cache.get(monthly_key, 0)
        
        if current_count >= 10:  # 10 requests per month limit
            return False
        
        # Increment counter
        cache.set(monthly_key, current_count + 1, 
                 timeout=60*60*24*31)  # 31 days
        return True
    
    def get_client_ip(self, request):
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip

# Decorator for rate limiting
def rate_limit_required(view_func):
    def wrapper(request, *args, **kwargs):
        rate_limiter = RateLimitMixin()
        if not rate_limiter.check_rate_limit(request):
            return JsonResponse({
                'error': 'Rate limit exceeded. Please sign up for unlimited access.',
                'limit': 10,
                'period': 'month'
            }, status=429)
        return view_func(request, *args, **kwargs)
    return wrapper
```

#### Database-based Alternative
```python
# If Redis is not available
class DatabaseRateLimiter:
    @staticmethod
    def check_rate_limit(ip_address):
        today = timezone.now().date()
        rate_limit, created = RateLimit.objects.get_or_create(
            identifier=ip_address,
            identifier_type='ip',
            reset_date=today,
            defaults={'request_count': 0}
        )
        
        if rate_limit.request_count >= 10:
            return False
        
        rate_limit.request_count += 1
        rate_limit.last_request = timezone.now()
        rate_limit.save()
        return True
```

## Authorization & Permissions

### Permission Classes
```python
from rest_framework.permissions import BasePermission

class IsOwnerOrReadOnly(BasePermission):
    """
    Custom permission to only allow owners of an object to edit it.
    """
    def has_object_permission(self, request, view, obj):
        # Read permissions for any request
        if request.method in ['GET', 'HEAD', 'OPTIONS']:
            return True
        
        # Write permissions only to the owner
        return obj.user == request.user

class IsPlanOwner(BasePermission):
    """
    Permission class for plan-specific access
    """
    def has_permission(self, request, view):
        return request.user.is_authenticated
    
    def has_object_permission(self, request, view, obj):
        return obj.user == request.user

# Usage in views
class PlanViewSet(viewsets.ModelViewSet):
    serializer_class = PlanSerializer
    permission_classes = [IsAuthenticated, IsPlanOwner]
    
    def get_queryset(self):
        return Plan.objects.filter(user=self.request.user)
```

## Security Headers & HTTPS

### Django Security Settings
```python
# Security settings for production
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

X_FRAME_OPTIONS = 'DENY'
SECURE_REFERRER_POLICY = 'strict-origin-when-cross-origin'

# HTTPS settings
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# CORS settings
CORS_ALLOW_CREDENTIALS = True
CORS_ALLOWED_ORIGINS = [
    "https://yourdomain.com",
    "https://www.yourdomain.com",
]

# Content Security Policy
CSP_DEFAULT_SRC = ["'self'"]
CSP_SCRIPT_SRC = ["'self'", "'unsafe-inline'", "https://apis.google.com"]
CSP_STYLE_SRC = ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"]
```

### Nginx Security Configuration
```nginx
# SSL configuration
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
limit_req zone=api burst=20 nodelay;
```

## Security Best Practices

### Input Validation & Sanitization
```python
from django.core.validators import EmailValidator, RegexValidator
from bleach import clean

class SecureUserSerializer(serializers.ModelSerializer):
    email = serializers.EmailField(validators=[EmailValidator()])
    username = serializers.CharField(
        validators=[RegexValidator(r'^[\w.@+-]+$', 'Invalid username format')]
    )
    
    def validate_description(self, value):
        # Sanitize HTML input
        allowed_tags = ['p', 'br', 'strong', 'em']
        return clean(value, tags=allowed_tags, strip=True)
```

### SQL Injection Prevention
```python
# Always use parameterized queries
# Django ORM automatically handles this

# Good - parameterized
User.objects.filter(email=user_email)

# Bad - string formatting (vulnerable)
# cursor.execute(f"SELECT * FROM users WHERE email = '{user_email}'")

# Good - raw SQL with parameters
cursor.execute("SELECT * FROM users WHERE email = %s", [user_email])
```

### CSRF Protection
```javascript
// Frontend CSRF handling
const getCSRFToken = () => {
  return document.querySelector('[name=csrfmiddlewaretoken]').value;
};

axios.defaults.headers.common['X-CSRFToken'] = getCSRFToken();
```

## Monitoring & Logging

### Security Event Logging
```python
import logging

security_logger = logging.getLogger('security')

class SecurityMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Log authentication attempts
        if request.path.startswith('/api/auth/'):
            self.log_auth_attempt(request)
        
        response = self.get_response(request)
        
        # Log suspicious activity
        if response.status_code in [401, 403, 429]:
            self.log_security_event(request, response)
        
        return response
    
    def log_auth_attempt(self, request):
        security_logger.info(f"Auth attempt from {request.META.get('REMOTE_ADDR')} - {request.path}")
    
    def log_security_event(self, request, response):
        security_logger.warning(
            f"Security event: {response.status_code} from {request.META.get('REMOTE_ADDR')} "
            f"accessing {request.path}"
        )
```

## TODO: Implementation Tasks

- [ ] Set up JWT authentication system
- [ ] Implement password hashing and validation
- [ ] Configure Google OAuth integration
- [ ] Set up Facebook OAuth (optional)
- [ ] Implement rate limiting for anonymous users
- [ ] Add permission classes for authorization
- [ ] Configure security headers and HTTPS
- [ ] Set up input validation and sanitization
- [ ] Implement security event logging
- [ ] Add CSRF protection
- [ ] Create user registration flow
- [ ] Implement password reset functionality
- [ ] Add two-factor authentication (future enhancement)