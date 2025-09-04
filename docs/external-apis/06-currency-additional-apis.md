# Currency & Additional APIs

## 💱 Overview

This document covers currency exchange, country information, and additional utility APIs needed for the travel planner.

## 1. **ExchangeRate-API** (Primary - FREE)

**Free Tier**: 1,500 requests/month (no signup required)  
**Pro Tier**: 100,000 requests/month for $10

### No Registration Required (Free Tier)
For basic usage, no account setup is needed.

### Enhanced Setup (Optional)
1. Sign up at [exchangerate-api.com](https://app.exchangerate-api.com/sign-up)
2. Verify email
3. Get API key for higher limits and more features

### Implementation

```python
import os
import requests
from typing import Dict, Any

class CurrencyService:
    def __init__(self):
        self.api_key = os.getenv('EXCHANGERATE_API_KEY')  # Optional for free tier
        if self.api_key:
            self.base_url = f"https://v6.exchangerate-api.com/v6/{self.api_key}"
        else:
            self.base_url = "https://api.exchangerate-api.com/v4"
    
    async def get_exchange_rates(self, base: str = 'USD') -> Dict[str, Any]:
        """Get current exchange rates for a base currency"""
        if self.api_key:
            # Pro endpoint with API key
            response = requests.get(f"{self.base_url}/latest/{base}")
        else:
            # Free endpoint (no API key required)
            response = requests.get(f"{self.base_url}/latest/{base}")
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Exchange rate API failed: {response.status_code} - {response.text}")
    
    async def convert_currency(self, amount: float, from_currency: str, 
                             to_currency: str) -> Dict[str, Any]:
        """Convert amount from one currency to another"""
        if self.api_key:
            # Pro endpoint with direct conversion
            url = f"{self.base_url}/pair/{from_currency}/{to_currency}/{amount}"
            response = requests.get(url)
            
            if response.status_code == 200:
                return response.json()
            else:
                raise Exception(f"Currency conversion failed: {response.status_code} - {response.text}")
        else:
            # Free endpoint calculation
            rates = await self.get_exchange_rates(from_currency)
            if 'rates' in rates and to_currency in rates['rates']:
                converted_amount = amount * rates['rates'][to_currency]
                return {
                    'base_code': from_currency,
                    'target_code': to_currency,
                    'conversion_rate': rates['rates'][to_currency],
                    'conversion_result': converted_amount
                }
            else:
                raise Exception("Currency conversion failed - currency not found")
    
    async def get_supported_currencies(self) -> Dict[str, Any]:
        """Get list of supported currencies"""
        if self.api_key:
            response = requests.get(f"{self.base_url}/codes")
        else:
            # For free tier, get from a sample request
            response = requests.get(f"{self.base_url}/latest/USD")
            if response.status_code == 200:
                data = response.json()
                return {'supported_codes': list(data.get('rates', {}).keys())}
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Supported currencies failed: {response.status_code} - {response.text}")
```

### Environment Setup
```bash
# Add to .env file (optional for free tier)
EXCHANGERATE_API_KEY=your_api_key_here
```

### Frontend Currency Converter

```javascript
const CurrencyConverter = ({ amount, fromCurrency, toCurrency }) => {
  const [convertedAmount, setConvertedAmount] = useState(null);
  const [loading, setLoading] = useState(false);
  const [exchangeRate, setExchangeRate] = useState(null);

  useEffect(() => {
    const convertCurrency = async () => {
      if (!amount || !fromCurrency || !toCurrency) return;
      
      setLoading(true);
      try {
        const response = await api.get('/currency/convert', {
          params: {
            amount,
            from: fromCurrency,
            to: toCurrency
          }
        });
        
        setConvertedAmount(response.data.conversion_result);
        setExchangeRate(response.data.conversion_rate);
      } catch (error) {
        console.error('Currency conversion failed:', error);
      } finally {
        setLoading(false);
      }
    };

    convertCurrency();
  }, [amount, fromCurrency, toCurrency]);

  if (loading) return <div>Converting...</div>;

  return (
    <div className="currency-converter">
      {convertedAmount && (
        <div className="conversion-result">
          <p>
            {amount} {fromCurrency} = {convertedAmount.toFixed(2)} {toCurrency}
          </p>
          {exchangeRate && (
            <p className="exchange-rate">
              Rate: 1 {fromCurrency} = {exchangeRate.toFixed(4)} {toCurrency}
            </p>
          )}
        </div>
      )}
    </div>
  );
};
```

## 2. **REST Countries API** (FREE)

**No registration required**  
**Features**: Country information, currencies, languages, flags

### Implementation

```python
class CountryInfoService:
    def __init__(self):
        self.base_url = "https://restcountries.com/v3.1"
    
    async def get_country_info(self, country_code: str) -> Dict[str, Any]:
        """Get country information by country code"""
        response = requests.get(f"{self.base_url}/alpha/{country_code}")
        
        if response.status_code == 200:
            data = response.json()
            return data[0] if data else None
        else:
            raise Exception(f"Country info failed: {response.status_code} - {response.text}")
    
    async def get_all_countries(self) -> List[Dict[str, Any]]:
        """Get information for all countries"""
        response = requests.get(f"{self.base_url}/all")
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"All countries failed: {response.status_code} - {response.text}")
    
    async def search_countries(self, name: str) -> List[Dict[str, Any]]:
        """Search countries by name"""
        response = requests.get(f"{self.base_url}/name/{name}")
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Country search failed: {response.status_code} - {response.text}")
    
    async def get_countries_by_currency(self, currency_code: str) -> List[Dict[str, Any]]:
        """Get countries that use a specific currency"""
        response = requests.get(f"{self.base_url}/currency/{currency_code}")
        
        if response.status_code == 200:
            return response.json()
        else:
            raise Exception(f"Countries by currency failed: {response.status_code} - {response.text}")
    
    async def get_country_summary(self, country_code: str) -> Dict[str, Any]:
        """Get formatted country summary for travel planning"""
        country_data = await self.get_country_info(country_code)
        if not country_data:
            return None
        
        return {
            'name': country_data.get('name', {}).get('common'),
            'official_name': country_data.get('name', {}).get('official'),
            'capital': country_data.get('capital', [None])[0],
            'region': country_data.get('region'),
            'currencies': [
                {
                    'code': code,
                    'name': info.get('name'),
                    'symbol': info.get('symbol')
                }
                for code, info in country_data.get('currencies', {}).items()
            ],
            'languages': list(country_data.get('languages', {}).values()),
            'flag': country_data.get('flags', {}).get('png'),
            'timezone': country_data.get('timezones', [])[0] if country_data.get('timezones') else None,
            'calling_code': country_data.get('idd', {}).get('root', '') + 
                           (country_data.get('idd', {}).get('suffixes', [''])[0])
        }
```

### Frontend Country Info Component

```javascript
const CountryInfo = ({ countryCode }) => {
  const [countryData, setCountryData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchCountryInfo = async () => {
      if (!countryCode) return;
      
      setLoading(true);
      try {
        const response = await api.get(`/countries/${countryCode}`);
        setCountryData(response.data);
      } catch (error) {
        console.error('Failed to fetch country info:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchCountryInfo();
  }, [countryCode]);

  if (loading) return <div>Loading country information...</div>;
  if (!countryData) return <div>Country information unavailable</div>;

  return (
    <div className="country-info">
      <div className="country-header">
        <img src={countryData.flag} alt={`${countryData.name} flag`} className="country-flag" />
        <div>
          <h2>{countryData.name}</h2>
          <p className="official-name">{countryData.official_name}</p>
        </div>
      </div>
      
      <div className="country-details">
        <div className="detail-item">
          <strong>Capital:</strong> {countryData.capital}
        </div>
        <div className="detail-item">
          <strong>Region:</strong> {countryData.region}
        </div>
        <div className="detail-item">
          <strong>Currencies:</strong>
          {countryData.currencies?.map((currency, index) => (
            <span key={currency.code}>
              {currency.name} ({currency.symbol})
              {index < countryData.currencies.length - 1 ? ', ' : ''}
            </span>
          ))}
        </div>
        <div className="detail-item">
          <strong>Languages:</strong> {countryData.languages?.join(', ')}
        </div>
        <div className="detail-item">
          <strong>Calling Code:</strong> {countryData.calling_code}
        </div>
        <div className="detail-item">
          <strong>Timezone:</strong> {countryData.timezone}
        </div>
      </div>
    </div>
  );
};
```

## 3. **Fixer.io** (Alternative Currency API)

**Free Tier**: 100 requests/month  
**Paid**: $10/month for 1K requests

### Setup Steps
1. Sign up at [fixer.io](https://fixer.io/signup/free)
2. Verify email
3. Get API key from dashboard

### Implementation

```python
class FixerCurrencyService:
    def __init__(self):
        self.api_key = os.getenv('FIXER_API_KEY')
        self.base_url = "http://data.fixer.io/api"
        
        if not self.api_key:
            raise ValueError("FIXER_API_KEY environment variable is required")
    
    async def get_latest_rates(self, base: str = 'EUR') -> Dict[str, Any]:
        """Get latest exchange rates (EUR base for free tier)"""
        params = {'access_key': self.api_key}
        if base != 'EUR':  # Free tier only supports EUR base
            params['base'] = base  # This will require paid plan
        
        response = requests.get(f"{self.base_url}/latest", params=params)
        return response.json() if response.status_code == 200 else None
```

## 4. **Abstract API** (Holiday & Travel Info)

**Free Tier**: 1,000 requests/month per service  
**Features**: Holidays, time zones, phone validation

### Setup Steps
1. Sign up at [abstractapi.com](https://app.abstractapi.com/users/signup)
2. Navigate to specific API (Holidays, Timezone, etc.)
3. Get API key for free tier

### Implementation

```python
class HolidayService:
    def __init__(self):
        self.api_key = os.getenv('ABSTRACT_API_KEY')
        self.base_url = "https://holidays.abstractapi.com/v1"
        
        if not self.api_key:
            raise ValueError("ABSTRACT_API_KEY environment variable is required")
    
    async def get_holidays(self, country: str, year: int) -> Dict[str, Any]:
        """Get holidays for a country and year"""
        params = {
            'api_key': self.api_key,
            'country': country,
            'year': year
        }
        response = requests.get(self.base_url, params=params)
        return response.json() if response.status_code == 200 else None

class TimezoneService:
    def __init__(self):
        self.api_key = os.getenv('ABSTRACT_API_KEY')
        self.base_url = "https://timezone.abstractapi.com/v1"
    
    async def get_timezone(self, lat: float, lon: float) -> Dict[str, Any]:
        """Get timezone information for coordinates"""
        params = {
            'api_key': self.api_key,
            'location': f"{lat},{lon}"
        }
        response = requests.get(f"{self.base_url}/current_time", params=params)
        return response.json() if response.status_code == 200 else None
```

## Unified API Service Manager

```python
class UtilityAPIManager:
    def __init__(self):
        self.currency = CurrencyService()
        self.countries = CountryInfoService()
        self.holidays = HolidayService()
        self.timezone = TimezoneService()
    
    async def get_destination_essentials(self, country_code: str, 
                                       lat: float, lon: float) -> Dict[str, Any]:
        """Get essential travel information for a destination"""
        tasks = [
            self.countries.get_country_summary(country_code),
            self.timezone.get_timezone(lat, lon),
            self.holidays.get_holidays(country_code, datetime.now().year)
        ]
        
        country_info, timezone_info, holidays_info = await asyncio.gather(*tasks, return_exceptions=True)
        
        essentials = {}
        
        if not isinstance(country_info, Exception):
            essentials['country'] = country_info
        
        if not isinstance(timezone_info, Exception):
            essentials['timezone'] = timezone_info
        
        if not isinstance(holidays_info, Exception):
            essentials['holidays'] = holidays_info
        
        return essentials
    
    async def convert_travel_budget(self, amount: float, from_currency: str, 
                                  destination_country: str) -> Dict[str, Any]:
        """Convert travel budget to destination currency"""
        # Get destination country currency
        country_info = await self.countries.get_country_info(destination_country)
        if not country_info or not country_info.get('currencies'):
            return None
        
        destination_currency = list(country_info['currencies'].keys())[0]
        
        # Convert currency
        conversion = await self.currency.convert_currency(
            amount, from_currency, destination_currency
        )
        
        return {
            'original_amount': amount,
            'original_currency': from_currency,
            'converted_amount': conversion['conversion_result'],
            'destination_currency': destination_currency,
            'exchange_rate': conversion['conversion_rate'],
            'currency_info': country_info['currencies'][destination_currency]
        }
```

## Implementation Status

| Service | Status | Free Tier | Setup Required | Features |
|---------|--------|-----------|----------------|----------|
| ExchangeRate-API | ✅ Ready | 1.5K calls/month | No signup | Currency conversion |
| REST Countries | ✅ Ready | Unlimited | No signup | Country information |
| Fixer.io | ✅ Ready | 100 calls/month | Account required | Currency rates |
| Abstract API | ✅ Ready | 1K calls/month | Account required | Holidays, timezones |

## Environment Setup

```bash
# Add to .env file
EXCHANGERATE_API_KEY=your_exchangerate_key_here  # Optional
FIXER_API_KEY=your_fixer_key_here                # Optional
ABSTRACT_API_KEY=your_abstract_key_here          # Optional
```

## Best Practices

### Caching Strategy
- Cache exchange rates for 6 hours
- Cache country information for 24 hours
- Cache holiday information for 7 days

### Error Handling
```python
async def get_exchange_rate_with_fallback(from_currency: str, to_currency: str) -> float:
    """Get exchange rate with fallback between services"""
    try:
        result = await exchangerate_service.convert_currency(1, from_currency, to_currency)
        return result['conversion_rate']
    except Exception:
        try:
            result = await fixer_service.get_latest_rates()
            return result['rates'].get(to_currency, 1.0)
        except Exception:
            logger.error("All currency services failed")
            return 1.0  # Fallback to 1:1 ratio
```