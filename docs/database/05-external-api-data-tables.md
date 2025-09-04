# External APIs Comprehensive Guide

## 🎯 Overview

This document lists all external APIs needed for the travel planner application, organized by category. Each API includes free tier options, setup instructions, and implementation examples.

## 🗺️ Maps & Geolocation APIs

### 1. **OpenStreetMap + Nominatim** (Primary - FREE)
**Cost**: Completely free  
**Limits**: 1 request/second for geocoding  
**Features**: Maps, geocoding, reverse geocoding  
**Setup Steps**:
1. No registration required
2. Add user-agent header to requests
3. Respect rate limits (1 req/sec)

```python
# Implementation
class OpenStreetMapService:
    def __init__(self):
        self.base_url = "https://nominatim.openstreetmap.org"
        self.headers = {'User-Agent': 'TravelPlanner/1.0 (your-email@domain.com)'}
    
    def geocode(self, address):
        params = {'q': address, 'format': 'json', 'limit': 1}
        response = requests.get(f"{self.base_url}/search", params=params, headers=self.headers)
        return response.json()[0] if response.json() else None
```

### 2. **Google Maps Platform** (Premium Option)
**Free Tier**: $200 credit monthly (~28K map loads)  
**Setup Steps**:
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create new project
3. Enable Maps JavaScript API, Geocoding API, Places API
4. Create API key with restrictions
5. Add billing account (required even for free tier)

**Pricing**:
- Maps: $7/1K loads
- Geocoding: $5/1K requests
- Places: $32/1K requests

### 3. **Mapbox** (Alternative)
**Free Tier**: 50K map loads/month  
**Setup Steps**:
1. Sign up at [mapbox.com](https://account.mapbox.com/auth/signup/)
2. Get access token from account dashboard
3. No credit card required for free tier

## 🌤️ Weather APIs

### 1. **OpenWeatherMap** (Primary - FREE)
**Free Tier**: 1K calls/day, 60 calls/minute  
**Setup Steps**:
1. Sign up at [openweathermap.org](https://home.openweathermap.org/users/sign_up)
2. Verify email
3. Get API key from dashboard
4. Wait 10 minutes for activation

**Features**: Current weather, 5-day forecast, historical data

```python
class WeatherService:
    def __init__(self):
        self.api_key = os.getenv('OPENWEATHER_API_KEY')
        self.base_url = "https://api.openweathermap.org/data/2.5"
```

### 2. **WeatherAPI** (Alternative)
**Free Tier**: 1M calls/month  
**Setup Steps**:
1. Sign up at [weatherapi.com](https://www.weatherapi.com/signup.aspx)
2. Get API key instantly
3. No credit card required

### 3. **AccuWeather** (Premium Option)
**Free Tier**: 50 calls/day  
**Paid**: $25/month for 1K calls/day  

## ✈️ Flight APIs

### 1. **Amadeus Travel API** (Primary - FREE)
**Free Tier**: 2K API calls/month  
**Setup Steps**:
1. Sign up at [developers.amadeus.com](https://developers.amadeus.com/register)
2. Create application
3. Get API key and secret
4. No credit card required

**Features**: Flight search, booking, airport info, airline codes

```python
class AmadeusFlightService:
    def __init__(self):
        self.api_key = os.getenv('AMADEUS_API_KEY')
        self.api_secret = os.getenv('AMADEUS_API_SECRET')
        self.base_url = "https://test.api.amadeus.com"
        self.access_token = None
    
    def authenticate(self):
        auth_url = f"{self.base_url}/v1/security/oauth2/token"
        data = {
            'grant_type': 'client_credentials',
            'client_id': self.api_key,
            'client_secret': self.api_secret
        }
        response = requests.post(auth_url, data=data)
        self.access_token = response.json()['access_token']
    
    def search_flights(self, origin, destination, departure_date, return_date=None):
        headers = {'Authorization': f'Bearer {self.access_token}'}
        params = {
            'originLocationCode': origin,
            'destinationLocationCode': destination,
            'departureDate': departure_date,
            'adults': 1
        }
        if return_date:
            params['returnDate'] = return_date
        
        response = requests.get(
            f"{self.base_url}/v2/shopping/flight-offers",
            headers=headers,
            params=params
        )
        return response.json()
```

### 2. **Skyscanner Rapid API** (Alternative)
**Free Tier**: 500 requests/month  
**Setup Steps**:
1. Sign up at [rapidapi.com](https://rapidapi.com/auth/sign-up)
2. Subscribe to Skyscanner API
3. Get RapidAPI key

### 3. **Kiwi.com Tequila API** (Alternative)
**Free Tier**: Limited trial  
**Features**: Flight search, booking

## 🏨 Hotel APIs

### 1. **Booking.com Affiliate Partner API** (FREE)
**Setup Steps**:
1. Apply at [partners.booking.com](https://partners.booking.com/)
2. Get affiliate partner status
3. Access to API after approval

### 2. **Hotels.com API** (Alternative)
**Free Tier**: Available through RapidAPI  
**Setup Steps**:
1. Sign up at RapidAPI
2. Subscribe to Hotels.com API

### 3. **Amadeus Hotel API** (Primary)
**Included in Amadeus free tier**  
**Features**: Hotel search, availability, booking

```python
class AmadeusHotelService:
    def search_hotels(self, city_code, check_in, check_out, adults=1):
        headers = {'Authorization': f'Bearer {self.access_token}'}
        params = {
            'cityCode': city_code,
            'checkInDate': check_in,
            'checkOutDate': check_out,
            'adults': adults,
            'radius': 5,
            'radiusUnit': 'KM'
        }
        
        response = requests.get(
            f"{self.base_url}/v1/reference-data/locations/hotels/by-city",
            headers=headers,
            params=params
        )
        return response.json()
```

## 🍽️ Places & Restaurants APIs

### 1. **Foursquare Places API** (Primary - FREE)
**Free Tier**: 100K regular calls/month  
**Setup Steps**:
1. Sign up at [foursquare.com/developers](https://foursquare.com/developers/)
2. Create app
3. Get API key
4. No credit card required

```python
class FoursquareService:
    def __init__(self):
        self.api_key = os.getenv('FOURSQUARE_API_KEY')
        self.base_url = "https://api.foursquare.com/v3/places"
        self.headers = {'Authorization': self.api_key}
    
    def search_places(self, query, lat, lon, categories=None):
        params = {
            'query': query,
            'll': f"{lat},{lon}",
            'radius': 1000,
            'limit': 20
        }
        if categories:
            params['categories'] = ','.join(categories)
        
        response = requests.get(f"{self.base_url}/search", headers=self.headers, params=params)
        return response.json()
```

### 2. **Yelp Fusion API** (Alternative - FREE)
**Free Tier**: 5K calls/day  
**Setup Steps**:
1. Sign up at [yelp.com/developers](https://www.yelp.com/developers)
2. Create app
3. Get API key

### 3. **Google Places API** (Premium)
**Free Tier**: $200 credit/month  
**Pricing**: $32/1K requests  

## 💱 Currency Exchange APIs

### 1. **ExchangeRate-API** (Primary - FREE)
**Free Tier**: 1,500 requests/month  
**Setup Steps**:
1. No registration required for basic tier
2. For higher limits: [exchangerate-api.com](https://app.exchangerate-api.com/sign-up)

```python
class CurrencyService:
    def get_exchange_rates(self, base='USD'):
        response = requests.get(f"https://api.exchangerate-api.com/v4/latest/{base}")
        return response.json()
```

### 2. **Fixer.io** (Alternative)
**Free Tier**: 100 requests/month  
**Paid**: $10/month for 1K requests  

### 3. **Open Exchange Rates** (Alternative)
**Free Tier**: 1K requests/month  

## 🚗 Transportation APIs

### 1. **OpenRouteService** (Primary - FREE)
**Free Tier**: 2K requests/day  
**Setup Steps**:
1. Sign up at [openrouteservice.org](https://openrouteservice.org/dev/#/signup)
2. Get API key
3. No credit card required

```python
class RoutingService:
    def __init__(self):
        self.api_key = os.getenv('OPENROUTESERVICE_API_KEY')
        self.base_url = "https://api.openrouteservice.org/v2"
    
    def get_directions(self, start_coords, end_coords, profile='driving-car'):
        headers = {'Authorization': self.api_key}
        data = {
            'coordinates': [start_coords, end_coords],
            'format': 'geojson'
        }
        response = requests.post(f"{self.base_url}/directions/{profile}/geojson", headers=headers, json=data)
        return response.json()
```

### 2. **GraphHopper** (Alternative)
**Free Tier**: 500 requests/day  

### 3. **HERE Routing API** (Alternative)
**Free Tier**: 250K transactions/month  

## 🎯 Attractions & Activities APIs

### 1. **GetYourGuide API** (Partner Program)
**Setup Steps**:
1. Apply for partner program
2. Commission-based model

### 2. **Viator API** (TripAdvisor)
**Setup Steps**:
1. Apply for API access
2. Partner agreement required

### 3. **Musement API** (Alternative)
**Setup Steps**:
1. Contact for API access
2. Partner program available

## 🏛️ Government & Travel Info APIs

### 1. **REST Countries API** (FREE)
**No registration required**  
**Features**: Country information, currencies, languages

```python
class CountryInfoService:
    def get_country_info(self, country_code):
        response = requests.get(f"https://restcountries.com/v3.1/alpha/{country_code}")
        return response.json()
```

### 2. **Abstract API** (FREE)
**Free Tier**: Various limits per service  
**Features**: Holidays, time zones, phone validation

### 3. **World Bank API** (FREE)
**Features**: Economic indicators, country data

## 🚨 Travel Alerts & Safety APIs

### 1. **US State Department Travel Advisories** (FREE)
**No registration required**  
**Features**: Travel warnings, country safety info

### 2. **UK Foreign Office Travel Advice** (FREE)
**Features**: Travel advisories, safety information

### 3. **Global Disaster Alert API** (FREE)
**Features**: Natural disaster alerts

## 📊 Database Schema Updates

Now let me update the database schema to store all this external API data:

<function_calls>
<invoke name="create_file">
<parameter name="content"># External API Data Tables

## Additional PostgreSQL Tables for External API Data

```sql
-- =====================================================
-- EXTERNAL API DATA STORAGE
-- =====================================================

-- Countries and destinations master data
CREATE TABLE countries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Basic country information
    country_code VARCHAR(3) UNIQUE NOT NULL, -- ISO 3166-1 alpha-3
    country_code_2 VARCHAR(2) UNIQUE NOT NULL, -- ISO 3166-1 alpha-2
    name VARCHAR(255) NOT NULL,
    official_name VARCHAR(255),
    
    -- Geographic data
    capital VARCHAR(255),
    region VARCHAR(100),
    subregion VARCHAR(100),
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    area_km2 DECIMAL(15, 2),
    
    -- Currency and language
    currencies JSONB DEFAULT '[]', -- [{"code": "USD", "name": "US Dollar", "symbol": "$"}]
    languages JSONB DEFAULT '[]', -- [{"code": "en", "name": "English"}]
    
    -- Travel information
    timezone VARCHAR(50),
    calling_code VARCHAR(10),
    drive_side VARCHAR(10), -- left, right
    
    -- Visa and entry requirements
    visa_requirements JSONB DEFAULT '{}', -- By nationality
    entry_requirements JSONB DEFAULT '{}',
    
    -- Travel safety
    safety_level INTEGER DEFAULT 3, -- 1-5 scale
    travel_advisory JSONB DEFAULT '{}',
    
    -- Additional data from REST Countries API
    api_data JSONB DEFAULT '{}',
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_synced_at TIMESTAMP WITH TIME ZONE
);

-- Cities and major destinations
CREATE TABLE cities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Basic city information
    name VARCHAR(255) NOT NULL,
    country_id UUID NOT NULL REFERENCES countries(id),
    admin_name VARCHAR(255), -- State/Province name
    
    -- Geographic data
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    elevation_m INTEGER,
    
    -- City details
    population INTEGER,
    timezone VARCHAR(50),
    
    -- Travel information
    airport_codes JSONB DEFAULT '[]', -- ["NYC", "JFK", "LGA", "EWR"]
    public_transport JSONB DEFAULT '{}',
    
    -- From geocoding APIs
    formatted_address TEXT,
    place_id VARCHAR(255), -- External API place ID
    
    -- Additional data
    api_data JSONB DEFAULT '{}',
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_synced_at TIMESTAMP WITH TIME ZONE
);

-- =====================================================
-- FLIGHT DATA
-- =====================================================

-- Airlines data
CREATE TABLE airlines (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Airline identification
    iata_code VARCHAR(2) UNIQUE, -- e.g., "AA"
    icao_code VARCHAR(3) UNIQUE, -- e.g., "AAL"
    name VARCHAR(255) NOT NULL,
    
    -- Airline details
    country_id UUID REFERENCES countries(id),
    fleet_size INTEGER,
    destinations_count INTEGER,
    
    -- API data
    api_data JSONB DEFAULT '{}',
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Airports data
CREATE TABLE airports (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Airport identification
    iata_code VARCHAR(3) UNIQUE, -- e.g., "JFK"
    icao_code VARCHAR(4) UNIQUE, -- e.g., "KJFK"
    name VARCHAR(255) NOT NULL,
    
    -- Location
    city_id UUID REFERENCES cities(id),
    country_id UUID NOT NULL REFERENCES countries(id),
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    elevation_m INTEGER,
    
    -- Airport details
    airport_type VARCHAR(50), -- international, domestic, regional
    timezone VARCHAR(50),
    
    -- API data
    api_data JSONB DEFAULT '{}',
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Flight search results cache
CREATE TABLE flight_searches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Search parameters
    origin_airport_id UUID REFERENCES airports(id),
    destination_airport_id UUID REFERENCES airports(id),
    departure_date DATE NOT NULL,
    return_date DATE,
    adults INTEGER DEFAULT 1,
    children INTEGER DEFAULT 0,
    infants INTEGER DEFAULT 0,
    cabin_class VARCHAR(20) DEFAULT 'economy', -- economy, premium, business, first
    
    -- Search metadata
    search_hash VARCHAR(64) UNIQUE, -- Hash of search parameters
    user_id UUID REFERENCES users(id),
    search_timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Results from API
    flight_results JSONB NOT NULL DEFAULT '[]',
    search_duration_ms INTEGER,
    api_provider VARCHAR(50), -- amadeus, skyscanner, etc.
    
    -- Cache management
    expires_at TIMESTAMP WITH TIME ZONE,
    result_count INTEGER DEFAULT 0,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Individual flight offers/results
CREATE TABLE flight_offers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Link to search
    search_id UUID NOT NULL REFERENCES flight_searches(id),
    
    -- Flight details
    airline_id UUID REFERENCES airlines(id),
    flight_number VARCHAR(10),
    
    -- Route
    origin_airport_id UUID NOT NULL REFERENCES airports(id),
    destination_airport_id UUID NOT NULL REFERENCES airports(id),
    
    -- Timing
    departure_datetime TIMESTAMP WITH TIME ZONE,
    arrival_datetime TIMESTAMP WITH TIME ZONE,
    duration_minutes INTEGER,
    
    -- Pricing
    price_amount DECIMAL(10, 2),
    price_currency VARCHAR(3),
    
    -- Flight details
    aircraft_type VARCHAR(50),
    cabin_class VARCHAR(20),
    stops INTEGER DEFAULT 0,
    layover_airports JSONB DEFAULT '[]',
    
    -- Booking information
    booking_url TEXT,
    deep_link TEXT,
    
    -- API data
    external_id VARCHAR(255), -- ID from external API
    api_provider VARCHAR(50),
    raw_data JSONB DEFAULT '{}',
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- =====================================================
-- ACCOMMODATION DATA
-- =====================================================

-- Hotel chains and brands
CREATE TABLE hotel_chains (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    name VARCHAR(255) NOT NULL,
    brand_names JSONB DEFAULT '[]', -- ["Marriott", "Courtyard", "Residence Inn"]
    country_id UUID REFERENCES countries(id),
    
    -- API data
    api_data JSONB DEFAULT '{}',
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Hotels and accommodations
CREATE TABLE accommodations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Basic information
    name VARCHAR(255) NOT NULL,
    type VARCHAR(50), -- hotel, apartment, hostel, villa, etc.
    
    -- Location
    city_id UUID REFERENCES cities(id),
    country_id UUID NOT NULL REFERENCES countries(id),
    address TEXT,
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    
    -- Hotel details
    hotel_chain_id UUID REFERENCES hotel_chains(id),
    star_rating DECIMAL(2, 1), -- 1.0 to 5.0
    
    -- Amenities and features
    amenities JSONB DEFAULT '[]', -- ["wifi", "pool", "gym", "spa"]
    room_types JSONB DEFAULT '[]',
    
    -- Contact information
    phone VARCHAR(50),
    email VARCHAR(255),
    website TEXT,
    
    -- Ratings and reviews
    rating_average DECIMAL(3, 2),
    review_count INTEGER DEFAULT 0,
    
    -- API identifiers
    booking_com_id VARCHAR(50),
    expedia_id VARCHAR(50),
    amadeus_id VARCHAR(50),
    
    -- API data
    api_data JSONB DEFAULT '{}',
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_synced_at TIMESTAMP WITH TIME ZONE
);

-- Hotel search results cache
CREATE TABLE accommodation_searches (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Search parameters
    city_id UUID REFERENCES cities(id),
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    radius_km INTEGER DEFAULT 5,
    
    check_in_date DATE NOT NULL,
    check_out_date DATE NOT NULL,
    adults INTEGER DEFAULT 1,
    children INTEGER DEFAULT 0,
    rooms INTEGER DEFAULT 1,
    
    -- Filters
    min_price DECIMAL(10, 2),
    max_price DECIMAL(10, 2),
    min_rating DECIMAL(2, 1),
    amenities_required JSONB DEFAULT '[]',
    
    -- Search metadata
    search_hash VARCHAR(64) UNIQUE,
    user_id UUID REFERENCES users(id),
    search_timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Results
    accommodation_results JSONB NOT NULL DEFAULT '[]',
    api_provider VARCHAR(50),
    result_count INTEGER DEFAULT 0,
    
    -- Cache management
    expires_at TIMESTAMP WITH TIME ZONE,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- =====================================================
-- PLACES & ATTRACTIONS DATA
-- =====================================================

-- Place categories (attractions, restaurants, etc.)
CREATE TABLE place_categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    name VARCHAR(255) NOT NULL,
    category_type VARCHAR(50), -- attraction, restaurant, shopping, etc.
    parent_category_id UUID REFERENCES place_categories(id),
    
    -- External API mappings
    foursquare_category_id VARCHAR(50),
    google_place_type VARCHAR(50),
    yelp_category VARCHAR(50),
    
    icon_name VARCHAR(100),
    color_hex VARCHAR(7),
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Places of interest (restaurants, attractions, etc.)
CREATE TABLE places (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Basic information
    name VARCHAR(255) NOT NULL,
    description TEXT,
    
    -- Location
    city_id UUID REFERENCES cities(id),
    country_id UUID NOT NULL REFERENCES countries(id),
    address TEXT,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    
    -- Categories
    primary_category_id UUID REFERENCES place_categories(id),
    categories JSONB DEFAULT '[]', -- Additional category IDs
    
    -- Contact information
    phone VARCHAR(50),
    email VARCHAR(255),
    website TEXT,
    
    -- Operating hours
    opening_hours JSONB DEFAULT '{}', -- {"monday": {"open": "09:00", "close": "17:00"}}
    
    -- Pricing
    price_level INTEGER, -- 1-4 scale ($ to $$$$)
    average_cost_per_person DECIMAL(10, 2),
    currency VARCHAR(3),
    
    -- Ratings and reviews
    rating_average DECIMAL(3, 2),
    review_count INTEGER DEFAULT 0,
    
    -- Visit information
    recommended_duration_minutes INTEGER,
    best_time_to_visit JSONB DEFAULT '{}',
    
    -- External API IDs
    foursquare_id VARCHAR(50),
    google_place_id VARCHAR(255),
    yelp_id VARCHAR(50),
    tripadvisor_id VARCHAR(50),
    
    -- API data
    api_data JSONB DEFAULT '{}',
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_synced_at TIMESTAMP WITH TIME ZONE
);

-- Place photos
CREATE TABLE place_photos (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    place_id UUID NOT NULL REFERENCES places(id) ON DELETE CASCADE,
    
    -- Photo information
    url TEXT NOT NULL,
    thumbnail_url TEXT,
    caption TEXT,
    
    -- Photo metadata
    width INTEGER,
    height INTEGER,
    source VARCHAR(50), -- foursquare, google, user_upload
    
    -- Attribution
    photographer_name VARCHAR(255),
    license_type VARCHAR(50),
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- =====================================================
-- WEATHER DATA
-- =====================================================

-- Weather data cache
CREATE TABLE weather_data (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Location
    city_id UUID REFERENCES cities(id),
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    
    -- Weather information
    weather_date DATE NOT NULL,
    temperature_celsius DECIMAL(5, 2),
    temperature_fahrenheit DECIMAL(5, 2),
    humidity INTEGER, -- percentage
    pressure_mb DECIMAL(7, 2),
    
    -- Weather conditions
    condition_main VARCHAR(50), -- Clear, Clouds, Rain, etc.
    condition_description VARCHAR(255),
    weather_icon VARCHAR(20),
    
    -- Wind
    wind_speed_mps DECIMAL(5, 2), -- meters per second
    wind_direction_degrees INTEGER,
    
    -- Precipitation
    precipitation_mm DECIMAL(6, 2),
    
    -- Additional data
    uv_index DECIMAL(3, 1),
    visibility_km DECIMAL(5, 2),
    
    -- Forecast vs current
    is_forecast BOOLEAN DEFAULT false,
    forecast_datetime TIMESTAMP WITH TIME ZONE,
    
    -- API information
    api_provider VARCHAR(50), -- openweathermap, weatherapi, etc.
    api_data JSONB DEFAULT '{}',
    
    -- Cache management
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE
);

-- =====================================================
-- CURRENCY & EXCHANGE RATES
-- =====================================================

-- Exchange rates cache
CREATE TABLE exchange_rates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Currency pair
    base_currency VARCHAR(3) NOT NULL,
    target_currency VARCHAR(3) NOT NULL,
    
    -- Rate information
    exchange_rate DECIMAL(15, 8) NOT NULL,
    rate_date DATE NOT NULL,
    
    -- API information
    api_provider VARCHAR(50),
    api_data JSONB DEFAULT '{}',
    
    -- Cache management
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE,
    
    -- Ensure unique combination per date
    UNIQUE(base_currency, target_currency, rate_date)
);

-- =====================================================
-- API USAGE TRACKING
-- =====================================================

-- Track external API usage for monitoring and limits
CREATE TABLE api_usage_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- API information
    api_provider VARCHAR(50) NOT NULL,
    endpoint VARCHAR(255) NOT NULL,
    method VARCHAR(10) DEFAULT 'GET',
    
    -- Request details
    user_id UUID REFERENCES users(id),
    request_params JSONB DEFAULT '{}',
    response_status INTEGER,
    response_time_ms INTEGER,
    
    -- Usage tracking
    request_timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    date_only DATE DEFAULT CURRENT_DATE,
    
    -- Cost tracking (if applicable)
    cost_usd DECIMAL(10, 6) DEFAULT 0.0,
    
    -- Error tracking
    error_message TEXT,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Daily API usage summary
CREATE TABLE api_usage_summary (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    api_provider VARCHAR(50) NOT NULL,
    endpoint VARCHAR(255) NOT NULL,
    usage_date DATE NOT NULL,
    
    total_requests INTEGER DEFAULT 0,
    successful_requests INTEGER DEFAULT 0,
    failed_requests INTEGER DEFAULT 0,
    
    avg_response_time_ms DECIMAL(10, 2),
    total_cost_usd DECIMAL(10, 4) DEFAULT 0.0,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(api_provider, endpoint, usage_date)
);

-- =====================================================
-- INDEXES FOR EXTERNAL API TABLES
-- =====================================================

-- Countries indexes
CREATE INDEX idx_countries_country_code ON countries(country_code);
CREATE INDEX idx_countries_country_code_2 ON countries(country_code_2);
CREATE INDEX idx_countries_name ON countries(name);

-- Cities indexes
CREATE INDEX idx_cities_country_id ON cities(country_id);
CREATE INDEX idx_cities_name ON cities(name);
CREATE INDEX idx_cities_location ON cities(latitude, longitude);

-- Airlines and airports indexes
CREATE INDEX idx_airlines_iata_code ON airlines(iata_code);
CREATE INDEX idx_airlines_icao_code ON airlines(icao_code);
CREATE INDEX idx_airports_iata_code ON airports(iata_code);
CREATE INDEX idx_airports_icao_code ON airports(icao_code);
CREATE INDEX idx_airports_city_id ON airports(city_id);
CREATE INDEX idx_airports_country_id ON airports(country_id);

-- Flight searches indexes
CREATE INDEX idx_flight_searches_hash ON flight_searches(search_hash);
CREATE INDEX idx_flight_searches_user_id ON flight_searches(user_id);
CREATE INDEX idx_flight_searches_route ON flight_searches(origin_airport_id, destination_airport_id, departure_date);
CREATE INDEX idx_flight_searches_expires ON flight_searches(expires_at);

-- Flight offers indexes
CREATE INDEX idx_flight_offers_search_id ON flight_offers(search_id);
CREATE INDEX idx_flight_offers_airline ON flight_offers(airline_id);
CREATE INDEX idx_flight_offers_route ON flight_offers(origin_airport_id, destination_airport_id);
CREATE INDEX idx_flight_offers_price ON flight_offers(price_amount);

-- Accommodations indexes
CREATE INDEX idx_accommodations_city_id ON accommodations(city_id);
CREATE INDEX idx_accommodations_country_id ON accommodations(country_id);
CREATE INDEX idx_accommodations_location ON accommodations(latitude, longitude);
CREATE INDEX idx_accommodations_rating ON accommodations(rating_average);
CREATE INDEX idx_accommodations_active ON accommodations(is_active) WHERE is_active = true;

-- Accommodation searches indexes
CREATE INDEX idx_accommodation_searches_hash ON accommodation_searches(search_hash);
CREATE INDEX idx_accommodation_searches_city ON accommodation_searches(city_id);
CREATE INDEX idx_accommodation_searches_dates ON accommodation_searches(check_in_date, check_out_date);
CREATE INDEX idx_accommodation_searches_expires ON accommodation_searches(expires_at);

-- Places indexes
CREATE INDEX idx_places_city_id ON places(city_id);
CREATE INDEX idx_places_country_id ON places(country_id);
CREATE INDEX idx_places_location ON places(latitude, longitude);
CREATE INDEX idx_places_category ON places(primary_category_id);
CREATE INDEX idx_places_rating ON places(rating_average);
CREATE INDEX idx_places_name ON places(name);
CREATE INDEX idx_places_active ON places(is_active) WHERE is_active = true;
CREATE INDEX idx_places_foursquare_id ON places(foursquare_id) WHERE foursquare_id IS NOT NULL;
CREATE INDEX idx_places_google_id ON places(google_place_id) WHERE google_place_id IS NOT NULL;

-- Weather data indexes
CREATE INDEX idx_weather_data_city_id ON weather_data(city_id);
CREATE INDEX idx_weather_data_location ON weather_data(latitude, longitude);
CREATE INDEX idx_weather_data_date ON weather_data(weather_date);
CREATE INDEX idx_weather_data_expires ON weather_data(expires_at);

-- Exchange rates indexes
CREATE INDEX idx_exchange_rates_pair ON exchange_rates(base_currency, target_currency);
CREATE INDEX idx_exchange_rates_date ON exchange_rates(rate_date);
CREATE INDEX idx_exchange_rates_expires ON exchange_rates(expires_at);

-- API usage indexes
CREATE INDEX idx_api_usage_logs_provider ON api_usage_logs(api_provider);
CREATE INDEX idx_api_usage_logs_date ON api_usage_logs(date_only);
CREATE INDEX idx_api_usage_logs_user ON api_usage_logs(user_id);
CREATE INDEX idx_api_usage_summary_provider_date ON api_usage_summary(api_provider, usage_date);

-- =====================================================
-- FUNCTIONS FOR API DATA MANAGEMENT
-- =====================================================

-- Function to clean expired cache data
CREATE OR REPLACE FUNCTION cleanup_expired_api_cache()
RETURNS INTEGER AS $$
DECLARE
    deleted_count INTEGER := 0;
    temp_count INTEGER;
BEGIN
    -- Clean expired flight searches
    DELETE FROM flight_searches WHERE expires_at < NOW();
    GET DIAGNOSTICS temp_count = ROW_COUNT;
    deleted_count := deleted_count + temp_count;
    
    -- Clean expired accommodation searches
    DELETE FROM accommodation_searches WHERE expires_at < NOW();
    GET DIAGNOSTICS temp_count = ROW_COUNT;
    deleted_count := deleted_count + temp_count;
    
    -- Clean expired weather data
    DELETE FROM weather_data WHERE expires_at < NOW();
    GET DIAGNOSTICS temp_count = ROW_COUNT;
    deleted_count := deleted_count + temp_count;
    
    -- Clean expired exchange rates
    DELETE FROM exchange_rates WHERE expires_at < NOW();
    GET DIAGNOSTICS temp_count = ROW_COUNT;
    deleted_count := deleted_count + temp_count;
    
    RETURN deleted_count;
END;
$$ language 'plpgsql';

-- Function to update API usage summary
CREATE OR REPLACE FUNCTION update_api_usage_summary()
RETURNS VOID AS $$
BEGIN
    INSERT INTO api_usage_summary (
        api_provider,
        endpoint,
        usage_date,
        total_requests,
        successful_requests,
        failed_requests,
        avg_response_time_ms,
        total_cost_usd
    )
    SELECT 
        api_provider,
        endpoint,
        date_only,
        COUNT(*) as total_requests,
        COUNT(*) FILTER (WHERE response_status BETWEEN 200 AND 299) as successful_requests,
        COUNT(*) FILTER (WHERE response_status NOT BETWEEN 200 AND 299) as failed_requests,
        AVG(response_time_ms) as avg_response_time_ms,
        SUM(cost_usd) as total_cost_usd
    FROM api_usage_logs
    WHERE date_only = CURRENT_DATE - INTERVAL '1 day'
    GROUP BY api_provider, endpoint, date_only
    ON CONFLICT (api_provider, endpoint, usage_date)
    DO UPDATE SET
        total_requests = EXCLUDED.total_requests,
        successful_requests = EXCLUDED.successful_requests,
        failed_requests = EXCLUDED.failed_requests,
        avg_response_time_ms = EXCLUDED.avg_response_time_ms,
        total_cost_usd = EXCLUDED.total_cost_usd;
END;
$$ language 'plpgsql';

-- =====================================================
-- SCHEDULED JOBS (to be set up with pg_cron or external scheduler)
-- =====================================================

-- Schedule cleanup of expired cache data (run daily)
-- SELECT cron.schedule('cleanup-api-cache', '0 2 * * *', 'SELECT cleanup_expired_api_cache();');

-- Schedule API usage summary update (run daily)
-- SELECT cron.schedule('update-api-usage', '0 1 * * *', 'SELECT update_api_usage_summary();');
```

## 🔄 Vector Database Collections for External API Data

```python
# Qdrant collections for external API data
EXTERNAL_API_COLLECTIONS = {
    "destinations_content": {
        "vector_size": 1536,
        "distance": "Cosine",
        "description": "Destination descriptions and travel content"
    },
    "places_content": {
        "vector_size": 1536,
        "distance": "Cosine", 
        "description": "Places, attractions, and restaurant descriptions"
    },
    "travel_guides": {
        "vector_size": 1536,
        "distance": "Cosine",
        "description": "Travel guides and destination information"
    },
    "weather_insights": {
        "vector_size": 1536,
        "distance": "Cosine",
        "description": "Weather-based travel recommendations"
    }
}

# Example vector data structures
destinations_vector_example = {
    "id": "dest_content_tokyo_12345",
    "vector": [0.1, 0.2, ...],  # 1536-dimensional embedding
    "payload": {
        "city_id": "tokyo_uuid",
        "country_id": "japan_uuid",
        "content_type": "destination_overview",
        "title": "Tokyo: Modern Metropolis with Traditional Heart",
        "description": "Tokyo seamlessly blends cutting-edge technology with ancient traditions...",
        "highlights": ["technology", "temples", "food", "shopping", "nightlife"],
        "best_time_to_visit": "spring_autumn",
        "travel_style_fit": ["urban", "cultural", "foodie", "technology"],
        "content_source": "aggregated_travel_guides",
        "last_updated": "2025-09-04T10:00:00Z"
    }
}

places_vector_example = {
    "id": "place_content_senso_ji_67890",
    "vector": [0.1, 0.2, ...],
    "payload": {
        "place_id": "senso_ji_uuid",
        "city_id": "tokyo_uuid",
        "place_type": "attraction",
        "category": "religious_site",
        "name": "Senso-ji Temple",
        "description": "Tokyo's oldest temple, founded in 628 AD in Asakusa district...",
        "experience_tags": ["spiritual", "historic", "architecture", "cultural"],
        "visit_duration": "1-2 hours",
        "best_time": "early morning",
        "nearby_attractions": ["Asakusa Shrine", "Nakamise Shopping Street"],
        "visitor_tips": ["free admission", "photography allowed", "dress modestly"],
        "content_source": "foursquare_yelp_aggregated"
    }
}
```