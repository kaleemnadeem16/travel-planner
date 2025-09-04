# Weather APIs

## 🌤️ Overview

This document covers weather APIs needed for the travel planner application to provide current weather and forecasts for destinations.

## 1. **OpenWeatherMap** (Primary - FREE)

**Free Tier**: 1,000 calls/day, 60 calls/minute  
**Features**: Current weather, 5-day forecast, historical data

### Setup Steps
1. Go to [openweathermap.org](https://home.openweathermap.org/users/sign_up)
2. Click "Sign Up" in top right corner
3. Fill registration form:
   - Username: your_username
   - Email: your_email@domain.com
   - Password: strong_password
   - Purpose: Personal/Educational
4. Click "Create Account"
5. **Verify email address** (check inbox/spam)
6. Login to your account
7. Navigate to "API keys" tab
8. Copy your default API key
9. **Wait 10-15 minutes** for API key activation
10. Test with: `https://api.openweathermap.org/data/2.5/weather?q=London&appid=YOUR_API_KEY`

### Implementation

```python
import os
import requests
from typing import Dict, Any

class WeatherService:
    def __init__(self):
        self.api_key = os.getenv('OPENWEATHER_API_KEY')
        self.base_url = "https://api.openweathermap.org/data/2.5"
        if not self.api_key:
            raise ValueError("OPENWEATHER_API_KEY environment variable is required")
    
    async def get_current_weather(self, lat: float, lon: float) -> Dict[str, Any]:
        params = {
            'lat': lat,
            'lon': lon,
            'appid': self.api_key,
            'units': 'metric'
        }
        response = requests.get(f"{self.base_url}/weather", params=params)
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Weather API error: {response.status_code} - {response.text}")
    
    async def get_forecast(self, lat: float, lon: float, days: int = 5) -> Dict[str, Any]:
        params = {
            'lat': lat,
            'lon': lon,
            'appid': self.api_key,
            'units': 'metric',
            'cnt': days * 8  # 8 forecasts per day (every 3 hours)
        }
        response = requests.get(f"{self.base_url}/forecast", params=params)
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Forecast API error: {response.status_code} - {response.text}")
    
    async def get_weather_by_city(self, city_name: str) -> Dict[str, Any]:
        params = {
            'q': city_name,
            'appid': self.api_key,
            'units': 'metric'
        }
        response = requests.get(f"{self.base_url}/weather", params=params)
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Weather API error: {response.status_code} - {response.text}")
```

### Frontend Component

```javascript
const WeatherWidget = ({ latitude, longitude }) => {
  const [weather, setWeather] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const fetchWeather = async () => {
      try {
        setLoading(true);
        const response = await api.get(`/weather/?lat=${latitude}&lon=${longitude}`);
        setWeather(response.data);
      } catch (error) {
        console.error('Failed to fetch weather:', error);
      } finally {
        setLoading(false);
      }
    };
    
    if (latitude && longitude) {
      fetchWeather();
    }
  }, [latitude, longitude]);
  
  if (loading) return <div>Loading weather...</div>;
  if (!weather) return <div>Weather unavailable</div>;
  
  return (
    <div className="weather-widget">
      <h3>Current Weather</h3>
      <div className="weather-info">
        <img 
          src={`https://openweathermap.org/img/w/${weather.weather[0].icon}.png`}
          alt={weather.weather[0].description}
        />
        <div>
          <p className="temperature">{Math.round(weather.main.temp)}°C</p>
          <p className="description">{weather.weather[0].description}</p>
          <p className="details">
            Humidity: {weather.main.humidity}% | 
            Wind: {weather.wind.speed} m/s
          </p>
        </div>
      </div>
    </div>
  );
};
```

### Environment Setup
```bash
# Add to .env file
OPENWEATHER_API_KEY=your_api_key_here
```

## 2. **WeatherAPI** (Alternative - FREE)

**Free Tier**: 1 million calls/month  
**Setup Steps**:
1. Go to [weatherapi.com](https://www.weatherapi.com/signup.aspx)
2. Fill registration form (no credit card required)
3. Verify email
4. Get API key from dashboard immediately

### Implementation

```python
class WeatherAPIService:
    def __init__(self):
        self.api_key = os.getenv('WEATHERAPI_KEY')
        self.base_url = "http://api.weatherapi.com/v1"
    
    async def get_current_weather(self, lat: float, lon: float) -> Dict[str, Any]:
        params = {
            'key': self.api_key,
            'q': f"{lat},{lon}",
            'aqi': 'no'
        }
        response = requests.get(f"{self.base_url}/current.json", params=params)
        return response.json() if response.status_code == 200 else None
    
    async def get_forecast(self, lat: float, lon: float, days: int = 3) -> Dict[str, Any]:
        params = {
            'key': self.api_key,
            'q': f"{lat},{lon}",
            'days': days,
            'aqi': 'no',
            'alerts': 'no'
        }
        response = requests.get(f"{self.base_url}/forecast.json", params=params)
        return response.json() if response.status_code == 200 else None
```

## Weather Data Caching Strategy

```python
class WeatherCacheManager:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.current_weather_ttl = 600  # 10 minutes
        self.forecast_ttl = 3600       # 1 hour
    
    async def get_cached_weather(self, lat: float, lon: float) -> Optional[Dict]:
        cache_key = f"weather:current:{lat:.3f}:{lon:.3f}"
        cached_data = await self.redis.get(cache_key)
        if cached_data:
            return json.loads(cached_data)
        return None
    
    async def cache_weather(self, lat: float, lon: float, weather_data: Dict):
        cache_key = f"weather:current:{lat:.3f}:{lon:.3f}"
        await self.redis.setex(
            cache_key, 
            self.current_weather_ttl, 
            json.dumps(weather_data, default=str)
        )
```

## Implementation Status

| Service | Status | Free Tier | Monthly Limit | Setup Required |
|---------|--------|-----------|---------------|----------------|
| OpenWeatherMap | ✅ Ready | 1K calls/day | 30K calls | Account required |
| WeatherAPI | ✅ Ready | 1M calls/month | 1M calls | Account required |

## Best Practices

### Rate Limiting
- Cache weather data for 10-15 minutes for current weather
- Cache forecasts for 1 hour
- Implement exponential backoff for failed requests
- Monitor API usage to stay within limits

### Error Handling
```python
async def get_weather_with_fallback(lat: float, lon: float) -> Dict[str, Any]:
    """Try primary weather service, fallback to secondary if needed"""
    try:
        return await openweather_service.get_current_weather(lat, lon)
    except Exception as primary_error:
        logger.warning(f"Primary weather service failed: {primary_error}")
        try:
            return await weatherapi_service.get_current_weather(lat, lon)
        except Exception as fallback_error:
            logger.error(f"All weather services failed: {fallback_error}")
            return None
```