# Travel APIs (Flights & Hotels)

## ✈️ Overview

This document covers flight and hotel booking APIs needed for the travel planner application.

## 1. **Amadeus Travel API** (Primary - FREE)

**Free Tier**: 2,000 API calls/month  
**Features**: Flight search, hotel search, airport/airline information

### Setup Steps
1. Go to [developers.amadeus.com](https://developers.amadeus.com/register)
2. Click "Sign up" or "Get Started"
3. Fill registration form:
   - First name, Last name
   - Email address
   - Password
   - Company: "Personal Project" or your company
   - Country: Your country
4. Accept terms and conditions
5. Click "Create account"
6. **Verify email address**
7. Login to developer portal
8. Click "Create new app" or "My First App"
9. Fill app details:
   - Application name: "Travel Planner"
   - Description: "Personal travel planning application"
   - Choose "Self-Service" tier (free)
10. Get API Key and API Secret from app dashboard
11. **Important**: Start with Test environment, switch to Production later

### Implementation

```python
import os
import requests
from datetime import datetime, timedelta
from typing import Dict, Any, Optional

class AmadeusFlightService:
    def __init__(self):
        self.api_key = os.getenv('AMADEUS_API_KEY')
        self.api_secret = os.getenv('AMADEUS_API_SECRET')
        self.environment = os.getenv('AMADEUS_ENVIRONMENT', 'test')
        
        if self.environment == 'production':
            self.base_url = "https://api.amadeus.com"
        else:
            self.base_url = "https://test.api.amadeus.com"
            
        self.access_token = None
        self.token_expires_at = None
        
        if not self.api_key or not self.api_secret:
            raise ValueError("AMADEUS_API_KEY and AMADEUS_API_SECRET environment variables are required")
    
    async def authenticate(self):
        """Get OAuth2 access token"""
        if self.access_token and self.token_expires_at > datetime.now():
            return self.access_token
            
        auth_url = f"{self.base_url}/v1/security/oauth2/token"
        data = {
            'grant_type': 'client_credentials',
            'client_id': self.api_key,
            'client_secret': self.api_secret
        }
        response = requests.post(auth_url, data=data)
        
        if response.status_code == 200:
            token_data = response.json()
            self.access_token = token_data['access_token']
            expires_in = token_data.get('expires_in', 1799)  # Default 30 minutes
            self.token_expires_at = datetime.now() + timedelta(seconds=expires_in - 60)
            return self.access_token
        else:
            raise Exception(f"Amadeus authentication failed: {response.status_code} - {response.text}")
    
    async def search_flights(self, origin: str, destination: str, departure_date: str, 
                           return_date: str = None, adults: int = 1) -> Dict[str, Any]:
        """Search for flights"""
        await self.authenticate()
        headers = {'Authorization': f'Bearer {self.access_token}'}
        
        params = {
            'originLocationCode': origin,
            'destinationLocationCode': destination,
            'departureDate': departure_date,
            'adults': adults,
            'max': 10  # Limit results
        }
        if return_date:
            params['returnDate'] = return_date
        
        response = requests.get(
            f"{self.base_url}/v2/shopping/flight-offers",
            headers=headers,
            params=params
        )
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Flight search failed: {response.status_code} - {response.text}")
    
    async def get_airport_info(self, iata_code: str) -> Dict[str, Any]:
        """Get airport information"""
        await self.authenticate()
        headers = {'Authorization': f'Bearer {self.access_token}'}
        
        response = requests.get(
            f"{self.base_url}/v1/reference-data/locations",
            headers=headers,
            params={'keyword': iata_code, 'subType': 'AIRPORT'}
        )
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Airport info failed: {response.status_code} - {response.text}")
    
    async def get_airline_info(self, iata_code: str) -> Dict[str, Any]:
        """Get airline information"""
        await self.authenticate()
        headers = {'Authorization': f'Bearer {self.access_token}'}
        
        response = requests.get(
            f"{self.base_url}/v1/reference-data/airlines",
            headers=headers,
            params={'airlineCodes': iata_code}
        )
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Airline info failed: {response.status_code} - {response.text}")
```

### Hotel Service

```python
class AmadeusHotelService:
    def __init__(self):
        # Reuse AmadeusFlightService authentication
        self.flight_service = AmadeusFlightService()
    
    async def search_hotels_by_city(self, city_code: str, check_in: str, 
                                  check_out: str, adults: int = 1) -> Dict[str, Any]:
        """Search hotels in a city"""
        await self.flight_service.authenticate()
        headers = {'Authorization': f'Bearer {self.flight_service.access_token}'}
        
        params = {
            'cityCode': city_code,
            'checkInDate': check_in,
            'checkOutDate': check_out,
            'adults': adults,
            'radius': 5,
            'radiusUnit': 'KM'
        }
        
        response = requests.get(
            f"{self.flight_service.base_url}/v1/reference-data/locations/hotels/by-city",
            headers=headers,
            params=params
        )
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Hotel search failed: {response.status_code} - {response.text}")
    
    async def get_hotel_offers(self, hotel_ids: list, check_in: str, 
                             check_out: str, adults: int = 1) -> Dict[str, Any]:
        """Get hotel offers for specific hotels"""
        await self.flight_service.authenticate()
        headers = {'Authorization': f'Bearer {self.flight_service.access_token}'}
        
        params = {
            'hotelIds': ','.join(hotel_ids),
            'checkInDate': check_in,
            'checkOutDate': check_out,
            'adults': adults
        }
        
        response = requests.get(
            f"{self.flight_service.base_url}/v3/shopping/hotel-offers",
            headers=headers,
            params=params
        )
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Hotel offers failed: {response.status_code} - {response.text}")
    
    async def search_hotels_by_coordinates(self, lat: float, lon: float, 
                                         check_in: str, check_out: str, 
                                         adults: int = 1, radius: int = 5) -> Dict[str, Any]:
        """Search hotels by coordinates"""
        await self.flight_service.authenticate()
        headers = {'Authorization': f'Bearer {self.flight_service.access_token}'}
        
        params = {
            'latitude': lat,
            'longitude': lon,
            'checkInDate': check_in,
            'checkOutDate': check_out,
            'adults': adults,
            'radius': radius,
            'radiusUnit': 'KM'
        }
        
        response = requests.get(
            f"{self.flight_service.base_url}/v1/reference-data/locations/hotels/by-geocode",
            headers=headers,
            params=params
        )
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Hotel search by coordinates failed: {response.status_code} - {response.text}")
```

### Environment Setup
```bash
# Add to .env file
AMADEUS_API_KEY=your_api_key_here
AMADEUS_API_SECRET=your_api_secret_here
AMADEUS_ENVIRONMENT=test  # Change to 'production' when ready
```

## 2. **Alternative APIs**

### Skyscanner RapidAPI (Alternative)
**Free Tier**: 500 requests/month  
**Setup Steps**:
1. Sign up at [rapidapi.com](https://rapidapi.com/auth/sign-up)
2. Search for "Skyscanner API"
3. Subscribe to free tier
4. Get RapidAPI key from dashboard

### Booking.com Partner API (Alternative)
**Setup Steps**:
1. Apply at [partners.booking.com](https://partners.booking.com/)
2. Fill affiliate application form
3. Wait for approval (can take days/weeks)
4. Get API access after partner status approval

## Frontend Integration Examples

### Flight Search Component

```javascript
const FlightSearch = () => {
  const [searchParams, setSearchParams] = useState({
    origin: '',
    destination: '',
    departureDate: '',
    returnDate: '',
    adults: 1
  });
  const [flights, setFlights] = useState([]);
  const [loading, setLoading] = useState(false);

  const handleSearch = async () => {
    setLoading(true);
    try {
      const response = await api.post('/flights/search', searchParams);
      setFlights(response.data.data || []);
    } catch (error) {
      console.error('Flight search failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="flight-search">
      <div className="search-form">
        <input
          type="text"
          placeholder="From (Airport Code)"
          value={searchParams.origin}
          onChange={(e) => setSearchParams({...searchParams, origin: e.target.value})}
        />
        <input
          type="text"
          placeholder="To (Airport Code)"
          value={searchParams.destination}
          onChange={(e) => setSearchParams({...searchParams, destination: e.target.value})}
        />
        <input
          type="date"
          value={searchParams.departureDate}
          onChange={(e) => setSearchParams({...searchParams, departureDate: e.target.value})}
        />
        <input
          type="date"
          value={searchParams.returnDate}
          onChange={(e) => setSearchParams({...searchParams, returnDate: e.target.value})}
        />
        <button onClick={handleSearch} disabled={loading}>
          {loading ? 'Searching...' : 'Search Flights'}
        </button>
      </div>
      
      <div className="flight-results">
        {flights.map((flight, index) => (
          <div key={index} className="flight-card">
            <div className="flight-route">
              {flight.itineraries[0].segments[0].departure.iataCode} → 
              {flight.itineraries[0].segments.slice(-1)[0].arrival.iataCode}
            </div>
            <div className="flight-price">
              {flight.price.currency} {flight.price.total}
            </div>
            <div className="flight-duration">
              Duration: {flight.itineraries[0].duration}
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};
```

### Hotel Search Component

```javascript
const HotelSearch = ({ destination }) => {
  const [hotels, setHotels] = useState([]);
  const [loading, setLoading] = useState(false);
  const [checkIn, setCheckIn] = useState('');
  const [checkOut, setCheckOut] = useState('');

  const searchHotels = async () => {
    if (!destination || !checkIn || !checkOut) return;
    
    setLoading(true);
    try {
      const response = await api.get('/hotels/search', {
        params: {
          cityCode: destination,
          checkInDate: checkIn,
          checkOutDate: checkOut,
          adults: 1
        }
      });
      setHotels(response.data.data || []);
    } catch (error) {
      console.error('Hotel search failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="hotel-search">
      <div className="search-controls">
        <input
          type="date"
          value={checkIn}
          onChange={(e) => setCheckIn(e.target.value)}
          placeholder="Check-in"
        />
        <input
          type="date"
          value={checkOut}
          onChange={(e) => setCheckOut(e.target.value)}
          placeholder="Check-out"
        />
        <button onClick={searchHotels} disabled={loading}>
          {loading ? 'Searching...' : 'Search Hotels'}
        </button>
      </div>
      
      <div className="hotel-results">
        {hotels.map((hotel, index) => (
          <div key={index} className="hotel-card">
            <h3>{hotel.name}</h3>
            <p>{hotel.address?.lines?.join(', ')}</p>
            <p>Rating: {hotel.rating || 'N/A'}</p>
          </div>
        ))}
      </div>
    </div>
  );
};
```

## Implementation Status

| Service | Status | Free Tier | Monthly Limit | Setup Required |
|---------|--------|-----------|---------------|----------------|
| Amadeus Flights | ✅ Ready | 2K calls/month | 2K calls | Account + verification |
| Amadeus Hotels | ✅ Ready | Included above | 2K calls | Same as flights |
| Skyscanner | ✅ Ready | 500 calls/month | 500 calls | RapidAPI account |

## Best Practices

### Caching Strategy
- Cache flight search results for 30 minutes
- Cache hotel search results for 1 hour
- Cache airport/airline reference data for 24 hours

### Error Handling
```python
async def search_flights_with_retry(search_params: dict, max_retries: int = 3) -> dict:
    """Search flights with retry logic"""
    for attempt in range(max_retries):
        try:
            return await amadeus_service.search_flights(**search_params)
        except Exception as e:
            if attempt == max_retries - 1:
                raise e
            await asyncio.sleep(2 ** attempt)  # Exponential backoff
```

### Rate Limiting
- Monitor API usage daily to stay within 2K monthly limit
- Implement request queuing for high-traffic periods
- Cache frequently requested routes and destinations