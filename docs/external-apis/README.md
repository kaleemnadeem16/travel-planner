# External APIs & Integrations

## API Selection Criteria

All external APIs must meet the following requirements:
- **Free Tier Available**: Generous free usage limits
- **Reliable Service**: Good uptime and performance
- **Easy Integration**: Well-documented REST APIs
- **No Credit Card Required**: For initial development and testing

## Maps & Geolocation Services

### OpenStreetMap + Leaflet (Recommended)
**Service**: OpenStreetMap with Leaflet.js
**Cost**: Completely free, no limits
**Features**: Maps, geocoding, routing

```javascript
// Frontend implementation
import L from 'leaflet';

const MapComponent = ({ center, zoom = 13 }) => {
  useEffect(() => {
    const map = L.map('map').setView(center, zoom);
    
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap contributors'
    }).addTo(map);
    
    // Add markers
    L.marker(center).addTo(map)
      .bindPopup('Destination')
      .openPopup();
  }, [center, zoom]);

  return <div id="map" style={{ height: '400px' }}></div>;
};
```

### Mapbox (Alternative)
**Service**: Mapbox GL JS
**Free Tier**: 50,000 map loads per month
**Features**: Interactive maps, geocoding, directions

```javascript
// Mapbox implementation
import mapboxgl from 'mapbox-gl';

mapboxgl.accessToken = process.env.REACT_APP_MAPBOX_TOKEN;

const MapboxComponent = ({ center, zoom = 13 }) => {
  useEffect(() => {
    const map = new mapboxgl.Map({
      container: 'mapbox-container',
      style: 'mapbox://styles/mapbox/streets-v11',
      center: center,
      zoom: zoom
    });

    new mapboxgl.Marker()
      .setLngLat(center)
      .addTo(map);

    return () => map.remove();
  }, [center, zoom]);

  return <div id="mapbox-container" style={{ height: '400px' }}></div>;
};
```

### Geocoding Services

#### Nominatim (OpenStreetMap)
**Service**: Nominatim geocoding
**Cost**: Free, rate limited to 1 request per second
**API**: `https://nominatim.openstreetmap.org/`

```python
# Backend geocoding service
import requests
import time

class GeocodingService:
    def __init__(self):
        self.base_url = "https://nominatim.openstreetmap.org"
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'TravelPlanner/1.0 (contact@example.com)'
        })
    
    def geocode(self, address):
        """Convert address to coordinates"""
        params = {
            'q': address,
            'format': 'json',
            'limit': 1,
            'addressdetails': 1
        }
        
        response = self.session.get(
            f"{self.base_url}/search",
            params=params
        )
        
        if response.status_code == 200:
            data = response.json()
            if data:
                result = data[0]
                return {
                    'latitude': float(result['lat']),
                    'longitude': float(result['lon']),
                    'display_name': result['display_name'],
                    'address': result.get('address', {})
                }
        
        return None
    
    def reverse_geocode(self, lat, lon):
        """Convert coordinates to address"""
        params = {
            'lat': lat,
            'lon': lon,
            'format': 'json',
            'addressdetails': 1
        }
        
        time.sleep(1)  # Rate limiting
        response = self.session.get(
            f"{self.base_url}/reverse",
            params=params
        )
        
        if response.status_code == 200:
            data = response.json()
            return {
                'display_name': data.get('display_name'),
                'address': data.get('address', {})
            }
        
        return None

# Usage example
geocoder = GeocodingService()
location = geocoder.geocode("Paris, France")
```

## Routing & Directions

### OpenRouteService
**Service**: OpenRouteService
**Free Tier**: 2,000 requests per day
**Features**: Routing, isochrones, matrix calculations

```python
# Backend routing service
class RoutingService:
    def __init__(self):
        self.api_key = os.getenv('OPENROUTESERVICE_API_KEY')
        self.base_url = "https://api.openrouteservice.org/v2"
    
    def get_directions(self, start_coords, end_coords, profile='driving-car'):
        """Get directions between two points"""
        headers = {
            'Authorization': self.api_key,
            'Content-Type': 'application/json'
        }
        
        data = {
            'coordinates': [start_coords, end_coords],
            'format': 'geojson',
            'instructions': True,
            'language': 'en'
        }
        
        response = requests.post(
            f"{self.base_url}/directions/{profile}/geojson",
            headers=headers,
            json=data
        )
        
        if response.status_code == 200:
            return response.json()
        return None
    
    def calculate_distance_matrix(self, coordinates):
        """Calculate distances between multiple points"""
        headers = {
            'Authorization': self.api_key,
            'Content-Type': 'application/json'
        }
        
        data = {
            'locations': coordinates,
            'metrics': ['distance', 'duration'],
            'units': 'km'
        }
        
        response = requests.post(
            f"{self.base_url}/matrix/driving-car",
            headers=headers,
            json=data
        )
        
        if response.status_code == 200:
            return response.json()
        return None
```

### GraphHopper (Alternative)
**Service**: GraphHopper Directions API
**Free Tier**: 500 requests per day
**Features**: Routing, route optimization

```python
class GraphHopperService:
    def __init__(self):
        self.api_key = os.getenv('GRAPHHOPPER_API_KEY')
        self.base_url = "https://graphhopper.com/api/1"
    
    def get_route(self, points, vehicle='car'):
        params = {
            'key': self.api_key,
            'point': points,  # List of [lat, lon] pairs
            'vehicle': vehicle,
            'instructions': True,
            'calc_points': True
        }
        
        response = requests.get(f"{self.base_url}/route", params=params)
        return response.json() if response.status_code == 200 else None
```

## Weather Services

### OpenWeatherMap
**Service**: OpenWeatherMap API
**Free Tier**: 1,000 calls per day
**Features**: Current weather, 5-day forecast

```python
class WeatherService:
    def __init__(self):
        self.api_key = os.getenv('OPENWEATHER_API_KEY')
        self.base_url = "https://api.openweathermap.org/data/2.5"
    
    def get_current_weather(self, lat, lon):
        """Get current weather for coordinates"""
        params = {
            'lat': lat,
            'lon': lon,
            'appid': self.api_key,
            'units': 'metric'
        }
        
        response = requests.get(f"{self.base_url}/weather", params=params)
        
        if response.status_code == 200:
            data = response.json()
            return {
                'temperature': data['main']['temp'],
                'description': data['weather'][0]['description'],
                'humidity': data['main']['humidity'],
                'wind_speed': data['wind']['speed'],
                'icon': data['weather'][0]['icon']
            }
        return None
    
    def get_forecast(self, lat, lon, days=5):
        """Get weather forecast"""
        params = {
            'lat': lat,
            'lon': lon,
            'appid': self.api_key,
            'units': 'metric',
            'cnt': days * 8  # 8 forecasts per day (every 3 hours)
        }
        
        response = requests.get(f"{self.base_url}/forecast", params=params)
        
        if response.status_code == 200:
            data = response.json()
            forecasts = []
            
            for item in data['list']:
                forecasts.append({
                    'datetime': item['dt_txt'],
                    'temperature': item['main']['temp'],
                    'description': item['weather'][0]['description'],
                    'icon': item['weather'][0]['icon']
                })
            
            return forecasts
        return None

# Frontend weather component
const WeatherWidget = ({ latitude, longitude }) => {
  const [weather, setWeather] = useState(null);
  
  useEffect(() => {
    const fetchWeather = async () => {
      try {
        const response = await api.get(`/weather/?lat=${latitude}&lon=${longitude}`);
        setWeather(response.data);
      } catch (error) {
        console.error('Failed to fetch weather:', error);
      }
    };
    
    if (latitude && longitude) {
      fetchWeather();
    }
  }, [latitude, longitude]);
  
  if (!weather) return <div>Loading weather...</div>;
  
  return (
    <div className="weather-widget">
      <h3>Current Weather</h3>
      <p>{weather.temperature}°C</p>
      <p>{weather.description}</p>
      <img 
        src={`https://openweathermap.org/img/w/${weather.icon}.png`}
        alt={weather.description}
      />
    </div>
  );
};
```

### Weatherbit.io (Alternative)
**Service**: Weatherbit API
**Free Tier**: 500 calls per day
**Features**: Current weather, forecasts, historical data

## Currency & Exchange Rates

### ExchangeRate-API
**Service**: ExchangeRate-API
**Free Tier**: 1,500 requests per month
**Features**: Real-time exchange rates

```python
class CurrencyService:
    def __init__(self):
        self.base_url = "https://api.exchangerate-api.com/v4/latest"
    
    def get_exchange_rates(self, base_currency='USD'):
        """Get current exchange rates"""
        response = requests.get(f"{self.base_url}/{base_currency}")
        
        if response.status_code == 200:
            data = response.json()
            return {
                'base': data['base'],
                'date': data['date'],
                'rates': data['rates']
            }
        return None
    
    def convert_currency(self, amount, from_currency, to_currency):
        """Convert between currencies"""
        rates = self.get_exchange_rates(from_currency)
        if rates and to_currency in rates['rates']:
            return amount * rates['rates'][to_currency]
        return None
```

## Places & Points of Interest

### Foursquare Places API
**Service**: Foursquare Places API
**Free Tier**: 100,000 regular calls per month
**Features**: Place search, details, photos

```python
class PlacesService:
    def __init__(self):
        self.api_key = os.getenv('FOURSQUARE_API_KEY')
        self.base_url = "https://api.foursquare.com/v3/places"
        self.headers = {
            'Authorization': self.api_key,
            'Accept': 'application/json'
        }
    
    def search_places(self, query, lat, lon, radius=1000, categories=None):
        """Search for places near coordinates"""
        params = {
            'query': query,
            'll': f"{lat},{lon}",
            'radius': radius,
            'limit': 20
        }
        
        if categories:
            params['categories'] = ','.join(categories)
        
        response = requests.get(
            f"{self.base_url}/search",
            headers=self.headers,
            params=params
        )
        
        if response.status_code == 200:
            data = response.json()
            places = []
            
            for place in data.get('results', []):
                places.append({
                    'id': place['fsq_id'],
                    'name': place['name'],
                    'address': place.get('location', {}).get('formatted_address'),
                    'categories': [cat['name'] for cat in place.get('categories', [])],
                    'distance': place.get('distance'),
                    'coordinates': place.get('geocodes', {}).get('main', {})
                })
            
            return places
        return []
    
    def get_place_details(self, place_id):
        """Get detailed information about a place"""
        response = requests.get(
            f"{self.base_url}/{place_id}",
            headers=self.headers,
            params={'fields': 'name,location,categories,website,tel,rating,photos'}
        )
        
        if response.status_code == 200:
            return response.json()
        return None
```

## API Integration Architecture

### API Service Manager
```python
# services/api_manager.py
class APIManager:
    def __init__(self):
        self.geocoding = GeocodingService()
        self.routing = RoutingService()
        self.weather = WeatherService()
        self.currency = CurrencyService()
        self.places = PlacesService()
    
    async def get_destination_info(self, destination_name):
        """Get comprehensive information about a destination"""
        # Geocode the destination
        location = self.geocoding.geocode(destination_name)
        if not location:
            return None
        
        lat, lon = location['latitude'], location['longitude']
        
        # Get weather and places in parallel
        tasks = [
            self.weather.get_current_weather(lat, lon),
            self.weather.get_forecast(lat, lon),
            self.places.search_places('tourist attraction', lat, lon),
            self.places.search_places('restaurant', lat, lon),
        ]
        
        weather, forecast, attractions, restaurants = await asyncio.gather(*tasks)
        
        return {
            'location': location,
            'weather': {
                'current': weather,
                'forecast': forecast
            },
            'places': {
                'attractions': attractions,
                'restaurants': restaurants
            }
        }
```

### Rate Limiting & Caching
```python
from django.core.cache import cache
from functools import wraps

def api_cache(timeout=3600):
    """Cache API responses to reduce external calls"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Create cache key from function name and arguments
            cache_key = f"api_cache:{func.__name__}:{hash(str(args) + str(kwargs))}"
            
            # Try to get from cache first
            result = cache.get(cache_key)
            if result is not None:
                return result
            
            # Make API call and cache result
            result = func(*args, **kwargs)
            if result is not None:
                cache.set(cache_key, result, timeout)
            
            return result
        return wrapper
    return decorator

# Usage
@api_cache(timeout=1800)  # Cache for 30 minutes
def get_weather_cached(lat, lon):
    return weather_service.get_current_weather(lat, lon)
```

### Error Handling & Fallbacks
```python
class APIFallbackService:
    def __init__(self):
        self.primary_geocoder = GeocodingService()
        self.fallback_geocoder = MapboxGeocodingService()
    
    def geocode_with_fallback(self, address):
        """Try primary service, fallback to secondary if needed"""
        try:
            result = self.primary_geocoder.geocode(address)
            if result:
                return result
        except Exception as e:
            logger.warning(f"Primary geocoding failed: {e}")
        
        try:
            return self.fallback_geocoder.geocode(address)
        except Exception as e:
            logger.error(f"All geocoding services failed: {e}")
            return None
```

## Frontend API Integration

### API Service Layer
```javascript
// services/externalApis.js
class ExternalAPIService {
  constructor() {
    this.baseURL = process.env.REACT_APP_API_URL;
  }
  
  async getDestinationInfo(destination) {
    try {
      const response = await axios.get(`${this.baseURL}/external/destination-info/`, {
        params: { destination }
      });
      return response.data;
    } catch (error) {
      console.error('Failed to fetch destination info:', error);
      throw error;
    }
  }
  
  async searchPlaces(query, coordinates) {
    try {
      const response = await axios.get(`${this.baseURL}/external/places/search/`, {
        params: {
          query,
          lat: coordinates.lat,
          lon: coordinates.lon
        }
      });
      return response.data;
    } catch (error) {
      console.error('Failed to search places:', error);
      return [];
    }
  }
}

export default new ExternalAPIService();
```

## API Usage Monitoring

### Usage Tracking
```python
class APIUsageTracker:
    def __init__(self):
        self.redis_client = redis.Redis()
    
    def track_usage(self, service_name, endpoint, user_id=None):
        """Track API usage for monitoring limits"""
        today = datetime.now().strftime("%Y-%m-%d")
        
        # Track overall usage
        key = f"api_usage:{service_name}:{endpoint}:{today}"
        self.redis_client.incr(key)
        self.redis_client.expire(key, 86400)  # 24 hours
        
        # Track per-user usage if authenticated
        if user_id:
            user_key = f"user_api_usage:{user_id}:{service_name}:{today}"
            self.redis_client.incr(user_key)
            self.redis_client.expire(user_key, 86400)
    
    def get_usage_stats(self, service_name, days=7):
        """Get usage statistics"""
        stats = {}
        for i in range(days):
            date = (datetime.now() - timedelta(days=i)).strftime("%Y-%m-%d")
            pattern = f"api_usage:{service_name}:*:{date}"
            keys = self.redis_client.keys(pattern)
            
            daily_total = 0
            for key in keys:
                daily_total += int(self.redis_client.get(key) or 0)
            
            stats[date] = daily_total
        
        return stats
```

## TODO: Implementation Tasks

- [ ] Set up OpenStreetMap integration with Leaflet
- [ ] Implement Nominatim geocoding service
- [ ] Configure OpenRouteService for directions
- [ ] Set up OpenWeatherMap integration
- [ ] Implement Foursquare Places API
- [ ] Add currency conversion service
- [ ] Create API caching layer
- [ ] Implement error handling and fallbacks
- [ ] Set up usage tracking and monitoring
- [ ] Add rate limiting for external API calls
- [ ] Create comprehensive API documentation
- [ ] Test all API integrations