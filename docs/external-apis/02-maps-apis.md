# Maps & Geolocation APIs

## 🗺️ Overview

This document covers map services, geocoding, and geolocation APIs needed for the travel planner application.

## 1. **OpenStreetMap + Nominatim** (Primary - FREE)

**Cost**: Completely free  
**Limits**: 1 request/second for geocoding  
**Features**: Maps, geocoding, reverse geocoding

### Setup Steps
1. No registration required
2. Add user-agent header: `TravelPlanner/1.0 (your-email@domain.com)`
3. Respect rate limits (1 req/sec)
4. Use Leaflet.js for map display

### Implementation

```python
import requests
import time
import asyncio
from typing import Optional, Dict, Any

class OpenStreetMapService:
    def __init__(self):
        self.base_url = "https://nominatim.openstreetmap.org"
        self.headers = {'User-Agent': 'TravelPlanner/1.0 (your-email@domain.com)'}
        self.last_request_time = 0
    
    async def _rate_limit(self):
        """Ensure 1 second between requests"""
        current_time = time.time()
        time_since_last = current_time - self.last_request_time
        if time_since_last < 1.0:
            await asyncio.sleep(1.0 - time_since_last)
        self.last_request_time = time.time()
    
    async def geocode(self, address: str) -> Optional[Dict[str, Any]]:
        await self._rate_limit()
        params = {
            'q': address,
            'format': 'json',
            'limit': 1,
            'addressdetails': 1,
            'extratags': 1
        }
        response = requests.get(f"{self.base_url}/search", params=params, headers=self.headers)
        results = response.json()
        return results[0] if results else None
    
    async def reverse_geocode(self, lat: float, lon: float) -> Optional[Dict[str, Any]]:
        await self._rate_limit()
        params = {
            'lat': lat,
            'lon': lon,
            'format': 'json',
            'addressdetails': 1
        }
        response = requests.get(f"{self.base_url}/reverse", params=params, headers=self.headers)
        return response.json()
```

### Frontend Integration

```javascript
// Leaflet.js integration
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

## 2. **Google Maps Platform** (Premium Option)

**Free Tier**: $200 credit monthly (~28K map loads)  
**Setup Steps**:
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create new project or select existing
3. Enable these APIs:
   - Maps JavaScript API
   - Geocoding API  
   - Places API
   - Directions API
4. Create API key:
   - Navigate to "Credentials"
   - Click "Create Credentials" → "API Key"
   - Restrict key to specific APIs and domains
5. Add billing account (required even for free tier)

### Environment Setup
```bash
# Add to .env file
GOOGLE_MAPS_API_KEY=your_api_key_here
GOOGLE_MAPS_RESTRICTED_KEY=your_restricted_frontend_key
```

## 3. **Routing & Directions - OpenRouteService** (FREE)

**Free Tier**: 2,000 requests/day  
**Setup Steps**:
1. Go to [openrouteservice.org](https://openrouteservice.org/dev/#/signup)
2. Click "Sign Up" button
3. Fill registration form
4. Confirm account via email
5. Generate API key in dashboard

### Implementation

```python
import os
import requests
from typing import Dict, Any, List

class RoutingService:
    def __init__(self):
        self.api_key = os.getenv('OPENROUTESERVICE_API_KEY')
        self.base_url = "https://api.openrouteservice.org/v2"
        
        if not self.api_key:
            raise ValueError("OPENROUTESERVICE_API_KEY environment variable is required")
    
    async def get_directions(self, start_coords: List[float], end_coords: List[float], 
                           profile: str = 'driving-car') -> Dict[str, Any]:
        """Get directions between two points"""
        headers = {'Authorization': self.api_key}
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
        else:
            raise Exception(f"Routing failed: {response.status_code} - {response.text}")
    
    async def get_distance_matrix(self, locations: List[List[float]], 
                                profile: str = 'driving-car') -> Dict[str, Any]:
        """Calculate distances between multiple points"""
        headers = {'Authorization': self.api_key}
        data = {
            'locations': locations,
            'metrics': ['distance', 'duration'],
            'units': 'km'
        }
        
        response = requests.post(
            f"{self.base_url}/matrix/{profile}",
            headers=headers,
            json=data
        )
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Distance matrix failed: {response.status_code} - {response.text}")
```

### Available Profiles
- `driving-car`: Car routing
- `driving-hgv`: Heavy goods vehicle
- `cycling-regular`: Regular cycling
- `cycling-road`: Road cycling
- `foot-walking`: Pedestrian
- `foot-hiking`: Hiking
- `wheelchair`: Wheelchair accessible

### Environment Setup
```bash
# Add to .env file
OPENROUTESERVICE_API_KEY=your_api_key_here
```

## Implementation Status

| Service | Status | Free Tier | Setup Required |
|---------|--------|-----------|----------------|
| OpenStreetMap | ✅ Ready | Unlimited | No registration |
| Google Maps | ✅ Ready | $200 credit | Account + billing |
| OpenRouteService | ✅ Ready | 2K requests/day | Account required |