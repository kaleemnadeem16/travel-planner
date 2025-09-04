# Places & Restaurants APIs

## 🍽️ Overview

This document covers place discovery and restaurant APIs needed for finding points of interest, restaurants, and attractions.

## 1. **Foursquare Places API** (Primary - FREE)

**Free Tier**: 100,000 regular calls/month  
**Features**: Place search, details, photos, reviews

### Setup Steps
1. Go to [foursquare.com/developers](https://foursquare.com/developers/)
2. Click "Get Started" button
3. Click "Create Account" 
4. Fill registration form:
   - First name, Last name
   - Email address
   - Password
   - Company: "Personal Project"
5. Verify email address
6. Login to developer portal
7. Click "Create new app" 
8. Fill app details:
   - App name: "Travel Planner"
   - App description: "Personal travel planning application"
   - App URL: Your domain or http://localhost:3000
   - Category: "Travel"
9. Get API key from app dashboard (it's your Authorization header value)

### Implementation

```python
import os
import requests
from typing import Dict, Any, List, Optional

class FoursquareService:
    def __init__(self):
        self.api_key = os.getenv('FOURSQUARE_API_KEY')
        self.base_url = "https://api.foursquare.com/v3/places"
        self.headers = {
            'Authorization': self.api_key,
            'Accept': 'application/json'
        }
        
        if not self.api_key:
            raise ValueError("FOURSQUARE_API_KEY environment variable is required")
    
    async def search_places(self, query: str, lat: float, lon: float, 
                          categories: List[str] = None, radius: int = 1000) -> Dict[str, Any]:
        """Search for places near coordinates"""
        params = {
            'query': query,
            'll': f"{lat},{lon}",
            'radius': radius,
            'limit': 20
        }
        if categories:
            params['categories'] = ','.join(categories)
        
        response = requests.get(f"{self.base_url}/search", headers=self.headers, params=params)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Foursquare search failed: {response.status_code} - {response.text}")
    
    async def get_place_details(self, place_id: str) -> Dict[str, Any]:
        """Get detailed information about a place"""
        response = requests.get(f"{self.base_url}/{place_id}", headers=self.headers)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Foursquare place details failed: {response.status_code} - {response.text}")
    
    async def get_place_photos(self, place_id: str) -> Dict[str, Any]:
        """Get photos for a place"""
        response = requests.get(f"{self.base_url}/{place_id}/photos", headers=self.headers)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Foursquare photos failed: {response.status_code} - {response.text}")
    
    async def search_restaurants(self, lat: float, lon: float, 
                               cuisine: str = None, radius: int = 1000) -> Dict[str, Any]:
        """Search for restaurants specifically"""
        params = {
            'll': f"{lat},{lon}",
            'radius': radius,
            'categories': '13065',  # Food and Beverage category
            'limit': 20
        }
        if cuisine:
            params['query'] = cuisine
        
        response = requests.get(f"{self.base_url}/search", headers=self.headers, params=params)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Restaurant search failed: {response.status_code} - {response.text}")
    
    async def search_attractions(self, lat: float, lon: float, 
                               radius: int = 1000) -> Dict[str, Any]:
        """Search for tourist attractions"""
        params = {
            'll': f"{lat},{lon}",
            'radius': radius,
            'categories': '16000',  # Arts and Entertainment
            'limit': 20
        }
        
        response = requests.get(f"{self.base_url}/search", headers=self.headers, params=params)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Attractions search failed: {response.status_code} - {response.text}")
```

### Foursquare Categories
**Common Categories**:
- Food & Beverage: `13065`
- Hotel: `19014`
- Arts & Entertainment: `16000`
- Retail: `17000`
- Travel & Transportation: `19000`
- Recreation: `18000`
- Health & Medicine: `15000`

### Environment Setup
```bash
# Add to .env file
FOURSQUARE_API_KEY=your_api_key_here
```

## 2. **Yelp Fusion API** (Alternative - FREE)

**Free Tier**: 5,000 calls/day  
**Features**: Business search, reviews, ratings

### Setup Steps
1. Go to [yelp.com/developers](https://www.yelp.com/developers)
2. Click "Create App"
3. Fill app information form
4. Get API key immediately

### Implementation

```python
class YelpService:
    def __init__(self):
        self.api_key = os.getenv('YELP_API_KEY')
        self.base_url = "https://api.yelp.com/v3"
        self.headers = {'Authorization': f'Bearer {self.api_key}'}
        
        if not self.api_key:
            raise ValueError("YELP_API_KEY environment variable is required")
    
    async def search_businesses(self, lat: float, lon: float, 
                              term: str = None, categories: str = None,
                              radius: int = 1000) -> Dict[str, Any]:
        """Search for businesses near coordinates"""
        params = {
            'latitude': lat,
            'longitude': lon,
            'radius': radius,
            'limit': 20
        }
        if term:
            params['term'] = term
        if categories:
            params['categories'] = categories
        
        response = requests.get(f"{self.base_url}/businesses/search", 
                              headers=self.headers, params=params)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Yelp search failed: {response.status_code} - {response.text}")
    
    async def get_business_details(self, business_id: str) -> Dict[str, Any]:
        """Get detailed business information"""
        response = requests.get(f"{self.base_url}/businesses/{business_id}", 
                              headers=self.headers)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Yelp business details failed: {response.status_code} - {response.text}")
    
    async def get_business_reviews(self, business_id: str) -> Dict[str, Any]:
        """Get business reviews"""
        response = requests.get(f"{self.base_url}/businesses/{business_id}/reviews", 
                              headers=self.headers)
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Yelp reviews failed: {response.status_code} - {response.text}")
```

### Environment Setup
```bash
# Add to .env file  
YELP_API_KEY=your_yelp_key_here
```

## Frontend Integration Examples

### Places Search Component

```javascript
const PlacesSearch = ({ location }) => {
  const [places, setPlaces] = useState([]);
  const [loading, setLoading] = useState(false);
  const [searchTerm, setSearchTerm] = useState('');
  const [category, setCategory] = useState('all');

  const searchPlaces = async () => {
    if (!location?.lat || !location?.lon) return;
    
    setLoading(true);
    try {
      const response = await api.get('/places/search', {
        params: {
          query: searchTerm,
          lat: location.lat,
          lon: location.lon,
          categories: category !== 'all' ? category : undefined,
          radius: 2000
        }
      });
      setPlaces(response.data.results || []);
    } catch (error) {
      console.error('Places search failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="places-search">
      <div className="search-controls">
        <input
          type="text"
          placeholder="Search for places..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
        />
        <select value={category} onChange={(e) => setCategory(e.target.value)}>
          <option value="all">All Categories</option>
          <option value="13065">Restaurants</option>
          <option value="16000">Attractions</option>
          <option value="17000">Shopping</option>
          <option value="19014">Hotels</option>
        </select>
        <button onClick={searchPlaces} disabled={loading}>
          {loading ? 'Searching...' : 'Search'}
        </button>
      </div>
      
      <div className="places-results">
        {places.map((place) => (
          <div key={place.fsq_id} className="place-card">
            <h3>{place.name}</h3>
            <p>{place.location?.formatted_address}</p>
            <div className="place-categories">
              {place.categories?.map((cat) => (
                <span key={cat.id} className="category-tag">
                  {cat.name}
                </span>
              ))}
            </div>
            {place.distance && (
              <p className="distance">{place.distance}m away</p>
            )}
          </div>
        ))}
      </div>
    </div>
  );
};
```

### Restaurant Finder Component

```javascript
const RestaurantFinder = ({ location }) => {
  const [restaurants, setRestaurants] = useState([]);
  const [loading, setLoading] = useState(false);
  const [cuisineType, setCuisineType] = useState('');

  const findRestaurants = async () => {
    if (!location?.lat || !location?.lon) return;
    
    setLoading(true);
    try {
      const response = await api.get('/places/restaurants', {
        params: {
          lat: location.lat,
          lon: location.lon,
          cuisine: cuisineType || undefined,
          radius: 1500
        }
      });
      setRestaurants(response.data.results || []);
    } catch (error) {
      console.error('Restaurant search failed:', error);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    findRestaurants();
  }, [location, cuisineType]);

  return (
    <div className="restaurant-finder">
      <div className="filters">
        <select 
          value={cuisineType} 
          onChange={(e) => setCuisineType(e.target.value)}
        >
          <option value="">All Cuisines</option>
          <option value="italian">Italian</option>
          <option value="chinese">Chinese</option>
          <option value="mexican">Mexican</option>
          <option value="japanese">Japanese</option>
          <option value="indian">Indian</option>
        </select>
      </div>
      
      {loading ? (
        <div>Finding restaurants...</div>
      ) : (
        <div className="restaurant-grid">
          {restaurants.map((restaurant) => (
            <div key={restaurant.fsq_id} className="restaurant-card">
              <h3>{restaurant.name}</h3>
              <p>{restaurant.location?.formatted_address}</p>
              <div className="restaurant-info">
                {restaurant.rating && (
                  <span className="rating">★ {restaurant.rating}</span>
                )}
                {restaurant.price && (
                  <span className="price">{restaurant.price}</span>
                )}
              </div>
              <div className="cuisine-tags">
                {restaurant.categories?.map((cat) => (
                  <span key={cat.id} className="cuisine-tag">
                    {cat.name}
                  </span>
                ))}
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  );
};
```

## Unified Places Service

```python
class UnifiedPlacesService:
    def __init__(self):
        self.foursquare = FoursquareService()
        self.yelp = YelpService()
    
    async def search_with_fallback(self, query: str, lat: float, lon: float, 
                                 service_preference: str = 'foursquare') -> Dict[str, Any]:
        """Search places with fallback between services"""
        try:
            if service_preference == 'foursquare':
                return await self.foursquare.search_places(query, lat, lon)
            else:
                return await self.yelp.search_businesses(lat, lon, term=query)
        except Exception as primary_error:
            logger.warning(f"Primary places service failed: {primary_error}")
            try:
                # Fallback to other service
                if service_preference == 'foursquare':
                    return await self.yelp.search_businesses(lat, lon, term=query)
                else:
                    return await self.foursquare.search_places(query, lat, lon)
            except Exception as fallback_error:
                logger.error(f"All places services failed: {fallback_error}")
                return {"results": []}
    
    async def get_comprehensive_place_data(self, place_id: str, 
                                         service: str = 'foursquare') -> Dict[str, Any]:
        """Get detailed place information including photos"""
        if service == 'foursquare':
            details = await self.foursquare.get_place_details(place_id)
            photos = await self.foursquare.get_place_photos(place_id)
            return {**details, 'photos': photos}
        else:
            details = await self.yelp.get_business_details(place_id)
            reviews = await self.yelp.get_business_reviews(place_id)
            return {**details, 'reviews': reviews}
```

## Implementation Status

| Service | Status | Free Tier | Monthly Limit | Features |
|---------|--------|-----------|---------------|----------|
| Foursquare | ✅ Ready | 100K calls/month | 100K calls | Places, photos, categories |
| Yelp | ✅ Ready | 5K calls/day | 150K calls | Reviews, ratings, business details |

## Best Practices

### Caching Strategy
- Cache place search results for 1 hour
- Cache place details for 24 hours
- Cache photos/reviews for 7 days

### Rate Limiting
- Foursquare: 100K calls/month = ~3,333 calls/day
- Yelp: 5K calls/day
- Implement intelligent caching to maximize efficiency

### Search Optimization
```python
async def optimized_place_search(query: str, lat: float, lon: float) -> List[Dict]:
    """Optimized place search with deduplication"""
    # Search both services in parallel
    foursquare_task = asyncio.create_task(
        foursquare_service.search_places(query, lat, lon)
    )
    yelp_task = asyncio.create_task(
        yelp_service.search_businesses(lat, lon, term=query)
    )
    
    foursquare_results, yelp_results = await asyncio.gather(
        foursquare_task, yelp_task, return_exceptions=True
    )
    
    # Combine and deduplicate results
    combined_results = []
    if not isinstance(foursquare_results, Exception):
        combined_results.extend(foursquare_results.get('results', []))
    if not isinstance(yelp_results, Exception):
        combined_results.extend(yelp_results.get('businesses', []))
    
    # Remove duplicates based on name and location
    return deduplicate_places(combined_results)
```