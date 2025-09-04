# API Integration Architecture

## 🔧 Overview

This document covers the complete integration architecture for managing all external APIs in the travel planner application, including service management, caching, rate limiting, and error handling.

## Unified API Service Manager

```python
import asyncio
import aioredis
import json
import logging
from datetime import datetime, timedelta
from typing import Dict, Any, Optional

logger = logging.getLogger(__name__)

class APIServiceManager:
    """Unified manager for all external API services"""
    
    def __init__(self):
        # Initialize all service instances
        self.services = {
            'weather': WeatherService(),
            'flights': AmadeusFlightService(),
            'hotels': AmadeusHotelService(),
            'places': FoursquareService(),
            'currency': CurrencyService(),
            'maps': OpenStreetMapService(),
            'routing': RoutingService(),
            'countries': CountryInfoService(),
            'yelp': YelpService(),
            'holidays': HolidayService()
        }
        
        # Initialize supporting services
        self.redis = None
        self.usage_tracker = APIUsageTracker()
        self.rate_limiter = RateLimiter()
        self.circuit_breaker = CircuitBreaker()
    
    async def initialize(self):
        """Initialize Redis connection and other async services"""
        try:
            redis_url = os.getenv('REDIS_URL', 'redis://localhost:6379')
            self.redis = await aioredis.from_url(redis_url)
            await self.redis.ping()
            logger.info("✅ Redis connected successfully")
        except Exception as e:
            logger.warning(f"⚠️ Redis connection failed: {e}")
            self.redis = None
    
    async def get_cached_or_fetch(self, cache_key: str, fetch_func, 
                                ttl_seconds: int = 3600) -> Optional[Dict[str, Any]]:
        """Get data from cache or fetch from API with comprehensive error handling"""
        
        service_name = fetch_func.__self__.__class__.__name__
        endpoint_name = fetch_func.__name__
        
        # Check circuit breaker
        if not self.circuit_breaker.can_execute(service_name):
            logger.warning(f"🚫 Circuit breaker open for {service_name}")
            return None
        
        # Check rate limits
        if not await self.rate_limiter.can_proceed(service_name):
            logger.warning(f"⏳ Rate limit exceeded for {service_name}")
            return None
        
        # Try cache first
        if self.redis:
            try:
                cached_data = await self.redis.get(cache_key)
                if cached_data:
                    logger.info(f"🔄 Cache hit for {cache_key}")
                    return json.loads(cached_data)
            except Exception as e:
                logger.warning(f"⚠️ Cache read error: {e}")
        
        # Fetch from API
        start_time = datetime.now()
        try:
            logger.info(f"🌐 Fetching from API: {service_name}.{endpoint_name}")
            
            # Execute the API call
            data = await fetch_func()
            
            # Calculate response time
            response_time = (datetime.now() - start_time).total_seconds() * 1000
            
            # Cache the result
            if self.redis and data:
                try:
                    await self.redis.setex(
                        cache_key, 
                        ttl_seconds, 
                        json.dumps(data, default=str)
                    )
                    logger.info(f"💾 Cached result for {cache_key}")
                except Exception as e:
                    logger.warning(f"⚠️ Cache write error: {e}")
            
            # Track successful usage
            await self.usage_tracker.log_request(
                api_provider=service_name,
                endpoint=endpoint_name,
                success=True,
                response_time_ms=response_time
            )
            
            # Record success in circuit breaker
            self.circuit_breaker.record_success(service_name)
            
            return data
            
        except Exception as e:
            response_time = (datetime.now() - start_time).total_seconds() * 1000
            
            # Track failed usage
            await self.usage_tracker.log_request(
                api_provider=service_name,
                endpoint=endpoint_name,
                success=False,
                response_time_ms=response_time,
                error=str(e)
            )
            
            # Record failure in circuit breaker
            self.circuit_breaker.record_failure(service_name)
            
            logger.error(f"❌ API call failed: {service_name}.{endpoint_name} - {e}")
            raise
    
    # Convenience methods for each service with optimized caching
    
    async def search_flights(self, origin: str, destination: str, 
                           departure_date: str, **kwargs) -> Dict[str, Any]:
        """Search flights with 30-minute caching"""
        cache_key = f"flights:{origin}:{destination}:{departure_date}:{hash(str(kwargs))}"
        
        async def fetch():
            return await self.services['flights'].search_flights(
                origin, destination, departure_date, **kwargs
            )
        
        return await self.get_cached_or_fetch(cache_key, fetch, ttl_seconds=1800)
    
    async def get_weather(self, lat: float, lon: float) -> Dict[str, Any]:
        """Get current weather with 10-minute caching"""
        cache_key = f"weather:current:{lat:.3f}:{lon:.3f}"
        
        async def fetch():
            return await self.services['weather'].get_current_weather(lat, lon)
        
        return await self.get_cached_or_fetch(cache_key, fetch, ttl_seconds=600)
    
    async def get_weather_forecast(self, lat: float, lon: float, days: int = 5) -> Dict[str, Any]:
        """Get weather forecast with 1-hour caching"""
        cache_key = f"weather:forecast:{lat:.3f}:{lon:.3f}:{days}"
        
        async def fetch():
            return await self.services['weather'].get_forecast(lat, lon, days)
        
        return await self.get_cached_or_fetch(cache_key, fetch, ttl_seconds=3600)
    
    async def search_places(self, query: str, lat: float, lon: float, 
                          **kwargs) -> Dict[str, Any]:
        """Search places with 1-hour caching"""
        cache_key = f"places:{query}:{lat:.3f}:{lon:.3f}:{hash(str(kwargs))}"
        
        async def fetch():
            return await self.services['places'].search_places(query, lat, lon, **kwargs)
        
        return await self.get_cached_or_fetch(cache_key, fetch, ttl_seconds=3600)
    
    async def search_hotels(self, city_code: str, check_in: str, check_out: str, 
                          **kwargs) -> Dict[str, Any]:
        """Search hotels with 1-hour caching"""
        cache_key = f"hotels:{city_code}:{check_in}:{check_out}:{hash(str(kwargs))}"
        
        async def fetch():
            return await self.services['hotels'].search_hotels_by_city(
                city_code, check_in, check_out, **kwargs
            )
        
        return await self.get_cached_or_fetch(cache_key, fetch, ttl_seconds=3600)
    
    async def get_exchange_rates(self, base_currency: str = 'USD') -> Dict[str, Any]:
        """Get exchange rates with 6-hour caching"""
        cache_key = f"currency:rates:{base_currency}"
        
        async def fetch():
            return await self.services['currency'].get_exchange_rates(base_currency)
        
        return await self.get_cached_or_fetch(cache_key, fetch, ttl_seconds=21600)
    
    async def convert_currency(self, amount: float, from_currency: str, 
                             to_currency: str) -> Dict[str, Any]:
        """Convert currency with 6-hour caching"""
        cache_key = f"currency:convert:{from_currency}:{to_currency}"
        
        async def fetch():
            return await self.services['currency'].convert_currency(
                amount, from_currency, to_currency
            )
        
        return await self.get_cached_or_fetch(cache_key, fetch, ttl_seconds=21600)
    
    async def get_directions(self, start_coords: list, end_coords: list, 
                           profile: str = 'driving-car') -> Dict[str, Any]:
        """Get directions with 2-hour caching"""
        cache_key = f"routing:{start_coords[0]:.3f}:{start_coords[1]:.3f}:{end_coords[0]:.3f}:{end_coords[1]:.3f}:{profile}"
        
        async def fetch():
            return await self.services['routing'].get_directions(start_coords, end_coords, profile)
        
        return await self.get_cached_or_fetch(cache_key, fetch, ttl_seconds=7200)
    
    async def get_country_info(self, country_code: str) -> Dict[str, Any]:
        """Get country information with 24-hour caching"""
        cache_key = f"country:info:{country_code}"
        
        async def fetch():
            return await self.services['countries'].get_country_summary(country_code)
        
        return await self.get_cached_or_fetch(cache_key, fetch, ttl_seconds=86400)
    
    async def geocode(self, address: str) -> Dict[str, Any]:
        """Geocode address with 24-hour caching"""
        cache_key = f"geocode:{hash(address)}"
        
        async def fetch():
            return await self.services['maps'].geocode(address)
        
        return await self.get_cached_or_fetch(cache_key, fetch, ttl_seconds=86400)
```

## Rate Limiting System

```python
class RateLimiter:
    """Comprehensive rate limiting for different API providers"""
    
    def __init__(self):
        self.limits = {
            'OpenStreetMapService': {
                'requests_per_second': 1
            },
            'WeatherService': {
                'requests_per_minute': 60,
                'requests_per_day': 1000
            },
            'AmadeusFlightService': {
                'requests_per_day': 67,  # 2000/month ≈ 67/day
                'requests_per_month': 2000
            },
            'AmadeusHotelService': {
                'requests_per_day': 67,  # Shared with flights
                'requests_per_month': 2000
            },
            'FoursquareService': {
                'requests_per_day': 3333,  # 100K/month ≈ 3333/day
                'requests_per_month': 100000
            },
            'YelpService': {
                'requests_per_day': 5000
            },
            'RoutingService': {
                'requests_per_minute': 40,
                'requests_per_day': 2000
            },
            'CurrencyService': {
                'requests_per_day': 50,  # 1500/month ≈ 50/day
                'requests_per_month': 1500
            }
        }
        self.request_counts = {}
        self.last_reset = {}
    
    async def can_proceed(self, service_name: str) -> bool:
        """Check if request can proceed within rate limits"""
        if service_name not in self.limits:
            return True
        
        now = datetime.now()
        limits = self.limits[service_name]
        
        # Initialize counters if needed
        if service_name not in self.request_counts:
            self.request_counts[service_name] = {}
            self.last_reset[service_name] = {}
        
        # Check each limit type
        for limit_type, limit_value in limits.items():
            if not await self._check_limit(service_name, limit_type, limit_value, now):
                return False
        
        # Increment counters
        await self._increment_counters(service_name, now)
        return True
    
    async def _check_limit(self, service_name: str, limit_type: str, 
                         limit_value: int, now: datetime) -> bool:
        """Check individual limit"""
        if limit_type not in self.request_counts[service_name]:
            self.request_counts[service_name][limit_type] = 0
            self.last_reset[service_name][limit_type] = now
        
        # Determine reset interval
        if 'second' in limit_type:
            reset_interval = timedelta(seconds=1)
        elif 'minute' in limit_type:
            reset_interval = timedelta(minutes=1)
        elif 'day' in limit_type:
            reset_interval = timedelta(days=1)
        elif 'month' in limit_type:
            reset_interval = timedelta(days=30)
        else:
            return True
        
        # Reset counter if needed
        if now - self.last_reset[service_name][limit_type] >= reset_interval:
            self.request_counts[service_name][limit_type] = 0
            self.last_reset[service_name][limit_type] = now
        
        # Check limit
        return self.request_counts[service_name][limit_type] < limit_value
    
    async def _increment_counters(self, service_name: str, now: datetime):
        """Increment all counters for the service"""
        for limit_type in self.limits[service_name]:
            if limit_type in self.request_counts[service_name]:
                self.request_counts[service_name][limit_type] += 1
    
    def get_usage_stats(self, service_name: str) -> Dict[str, Any]:
        """Get current usage statistics for a service"""
        if service_name not in self.request_counts:
            return {}
        
        stats = {}
        for limit_type, count in self.request_counts[service_name].items():
            limit_value = self.limits[service_name][limit_type]
            stats[limit_type] = {
                'current': count,
                'limit': limit_value,
                'percentage': (count / limit_value) * 100
            }
        
        return stats
```

## Circuit Breaker Pattern

```python
class CircuitBreaker:
    """Circuit breaker pattern for API failure protection"""
    
    def __init__(self):
        self.failure_threshold = int(os.getenv('CIRCUIT_BREAKER_FAILURE_THRESHOLD', 5))
        self.timeout_seconds = int(os.getenv('CIRCUIT_BREAKER_TIMEOUT_SECONDS', 60))
        
        self.failure_counts = {}
        self.last_failure_time = {}
        self.states = {}  # 'closed', 'open', 'half-open'
    
    def can_execute(self, service_name: str) -> bool:
        """Check if service can be called"""
        state = self._get_state(service_name)
        
        if state == 'closed':
            return True
        elif state == 'open':
            return False
        else:  # half-open
            return True
    
    def record_success(self, service_name: str):
        """Record successful API call"""
        self.failure_counts[service_name] = 0
        self.states[service_name] = 'closed'
    
    def record_failure(self, service_name: str):
        """Record failed API call"""
        now = datetime.now()
        self.failure_counts[service_name] = self.failure_counts.get(service_name, 0) + 1
        self.last_failure_time[service_name] = now
        
        if self.failure_counts[service_name] >= self.failure_threshold:
            self.states[service_name] = 'open'
            logger.warning(f"🔴 Circuit breaker opened for {service_name}")
    
    def _get_state(self, service_name: str) -> str:
        """Get current circuit breaker state"""
        if service_name not in self.states:
            self.states[service_name] = 'closed'
            return 'closed'
        
        current_state = self.states[service_name]
        
        if current_state == 'open':
            # Check if timeout has passed
            last_failure = self.last_failure_time.get(service_name)
            if last_failure and datetime.now() - last_failure >= timedelta(seconds=self.timeout_seconds):
                self.states[service_name] = 'half-open'
                logger.info(f"🟡 Circuit breaker half-open for {service_name}")
                return 'half-open'
        
        return current_state
```

## Usage Tracking System

```python
class APIUsageTracker:
    """Track API usage for monitoring and cost management"""
    
    def __init__(self):
        self.daily_usage = {}
        self.monthly_usage = {}
    
    async def log_request(self, api_provider: str, endpoint: str, 
                        success: bool, response_time_ms: float = 0, 
                        error: str = None, user_id: str = None):
        """Log API usage with comprehensive details"""
        
        timestamp = datetime.now()
        date_key = timestamp.strftime("%Y-%m-%d")
        month_key = timestamp.strftime("%Y-%m")
        
        # Update in-memory counters
        key = f"{api_provider}:{endpoint}"
        
        # Daily tracking
        if date_key not in self.daily_usage:
            self.daily_usage[date_key] = {}
        if key not in self.daily_usage[date_key]:
            self.daily_usage[date_key][key] = {
                'total': 0, 'success': 0, 'error': 0, 
                'total_response_time': 0, 'avg_response_time': 0
            }
        
        stats = self.daily_usage[date_key][key]
        stats['total'] += 1
        stats['total_response_time'] += response_time_ms
        stats['avg_response_time'] = stats['total_response_time'] / stats['total']
        
        if success:
            stats['success'] += 1
        else:
            stats['error'] += 1
        
        # Monthly tracking
        if month_key not in self.monthly_usage:
            self.monthly_usage[month_key] = {}
        if key not in self.monthly_usage[month_key]:
            self.monthly_usage[month_key][key] = {'total': 0, 'success': 0, 'error': 0}
        
        monthly_stats = self.monthly_usage[month_key][key]
        monthly_stats['total'] += 1
        if success:
            monthly_stats['success'] += 1
        else:
            monthly_stats['error'] += 1
        
        # Log to console
        status_emoji = "✅" if success else "❌"
        logger.info(f"📊 API Usage: {api_provider}.{endpoint} - {status_emoji} ({response_time_ms:.1f}ms)")
        
        # TODO: Log to database (api_usage_logs table)
        # await self._log_to_database(api_provider, endpoint, success, response_time_ms, error, user_id)
    
    def get_usage_summary(self, days: int = 7) -> Dict[str, Any]:
        """Get usage statistics for the last N days"""
        summary = {}
        
        for i in range(days):
            date = (datetime.now() - timedelta(days=i)).strftime("%Y-%m-%d")
            if date in self.daily_usage:
                summary[date] = self.daily_usage[date]
        
        return summary
    
    def get_monthly_summary(self, months: int = 3) -> Dict[str, Any]:
        """Get monthly usage statistics"""
        summary = {}
        
        for i in range(months):
            date = datetime.now() - timedelta(days=i*30)
            month_key = date.strftime("%Y-%m")
            if month_key in self.monthly_usage:
                summary[month_key] = self.monthly_usage[month_key]
        
        return summary
    
    def get_service_health(self) -> Dict[str, Any]:
        """Get overall service health metrics"""
        today = datetime.now().strftime("%Y-%m-%d")
        
        if today not in self.daily_usage:
            return {"status": "no_data", "services": {}}
        
        health_report = {"status": "healthy", "services": {}}
        
        for service_endpoint, stats in self.daily_usage[today].items():
            service_name = service_endpoint.split(':')[0]
            
            if service_name not in health_report["services"]:
                health_report["services"][service_name] = {
                    "total_requests": 0,
                    "success_rate": 0,
                    "avg_response_time": 0,
                    "status": "healthy"
                }
            
            service_stats = health_report["services"][service_name]
            service_stats["total_requests"] += stats["total"]
            
            if stats["total"] > 0:
                success_rate = (stats["success"] / stats["total"]) * 100
                service_stats["success_rate"] = success_rate
                service_stats["avg_response_time"] = stats["avg_response_time"]
                
                # Determine service health
                if success_rate < 80:
                    service_stats["status"] = "unhealthy"
                    health_report["status"] = "degraded"
                elif success_rate < 95:
                    service_stats["status"] = "degraded"
                    if health_report["status"] == "healthy":
                        health_report["status"] = "degraded"
        
        return health_report
```

## Environment Configuration

```bash
# API Integration Settings
REDIS_URL=redis://localhost:6379
ENABLE_API_CACHING=true
ENABLE_RATE_LIMITING=true

# Cache TTL Settings (seconds)
WEATHER_CACHE_TTL=600
FLIGHT_CACHE_TTL=1800
HOTEL_CACHE_TTL=3600
PLACES_CACHE_TTL=3600
CURRENCY_CACHE_TTL=21600
ROUTING_CACHE_TTL=7200

# Circuit Breaker Settings
ENABLE_CIRCUIT_BREAKER=true
CIRCUIT_BREAKER_FAILURE_THRESHOLD=5
CIRCUIT_BREAKER_TIMEOUT_SECONDS=60

# Rate Limiting Settings
MAX_RETRIES=3
RETRY_DELAY_SECONDS=2

# Monitoring Settings
LOG_API_REQUESTS=true
LOG_API_RESPONSES=false
API_DEBUG_MODE=false
```

## Usage Examples

### Basic Usage

```python
# Initialize API manager
api_manager = APIServiceManager()
await api_manager.initialize()

# Search for flights
flights = await api_manager.search_flights(
    origin='NYC',
    destination='LAX',
    departure_date='2025-12-01',
    adults=2
)

# Get weather for destination
weather = await api_manager.get_weather(lat=34.0522, lon=-118.2437)

# Find restaurants
restaurants = await api_manager.search_places(
    query='restaurants',
    lat=34.0522,
    lon=-118.2437,
    categories=['13065']
)
```

### Health Monitoring

```python
# Get service health report
health_report = api_manager.usage_tracker.get_service_health()
print(f"Overall Status: {health_report['status']}")

for service, stats in health_report['services'].items():
    print(f"{service}: {stats['success_rate']:.1f}% success rate")

# Get rate limit status
for service_name in api_manager.services.keys():
    usage_stats = api_manager.rate_limiter.get_usage_stats(service_name)
    for limit_type, stats in usage_stats.items():
        print(f"{service_name} {limit_type}: {stats['percentage']:.1f}% used")
```

This comprehensive integration architecture provides robust, scalable, and monitored access to all external APIs while respecting rate limits and providing graceful fallbacks.