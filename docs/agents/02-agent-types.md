# Agent Types & Specifications

## 🎯 Agent Taxonomy Overview

The Travel Planner system employs seven specialized agent types, each designed for specific domain expertise and optimized for their particular tasks. This modular approach ensures high accuracy, maintainable code, and cost-effective operations.

## 🏗️ Complete Agent Specifications

### 1. Planning Agent
**Purpose**: High-level trip structure and itinerary framework creation  
**Complexity**: High (orchestrates overall trip logic)

#### Core Responsibilities
- Parse complex trip requirements and constraints
- Create logical trip structure with optimal sequencing
- Balance competing priorities (time, budget, preferences)
- Generate flexible itinerary frameworks
- Handle multi-destination trip coordination

#### Input Interface
```json
{
  "request_type": "create_itinerary|modify_itinerary|optimize_itinerary",
  "trip_context": {
    "destinations": ["Tokyo", "Kyoto", "Osaka"],
    "duration": 7,
    "dates": {"start": "2024-03-15", "end": "2024-03-22"},
    "travelers": {"adults": 2, "children": 1, "seniors": 0},
    "budget_range": {"min": 3000, "max": 5000, "currency": "USD"}
  },
  "preferences": {
    "pace": "relaxed|moderate|fast",
    "interests": ["culture", "food", "nature"],
    "accommodation_style": "budget|mid-range|luxury",
    "transport_preferences": ["train", "flight", "car"]
  },
  "constraints": {
    "must_visit": ["Fushimi Inari Shrine"],
    "must_avoid": ["crowded_areas"],
    "accessibility_needs": ["wheelchair_access"]
  }
}
```

#### Output Interface
```json
{
  "itinerary_framework": {
    "total_duration": 7,
    "phases": [
      {
        "phase_id": "tokyo_exploration",
        "location": "Tokyo",
        "duration": 3,
        "sequence": 1,
        "activities_slots": 6,
        "accommodation_nights": 2,
        "key_objectives": ["cultural_immersion", "food_experiences"]
      }
    ],
    "daily_structure": {
      "morning_slot": "09:00-12:00",
      "afternoon_slot": "13:00-17:00", 
      "evening_slot": "18:00-21:00"
    },
    "transition_schedule": [
      {
        "from": "Tokyo",
        "to": "Kyoto", 
        "date": "2024-03-18",
        "transport_type": "train",
        "duration": "3h"
      }
    ]
  },
  "optimization_priorities": ["budget_efficiency", "travel_time", "user_preferences"],
  "flexibility_points": ["extra_day_tokyo", "optional_osaka_extension"]
}
```

#### Context Requirements
- **Maximum Token Limit**: 8,192 tokens
- **Essential Context**: Trip requirements, user preferences, constraints
- **Optional Context**: Weather data, local events, price trends

---

### 2. Location Agent
**Purpose**: Geographic analysis and destination intelligence  
**Complexity**: Medium (focused geographic expertise)

#### Core Responsibilities
- Analyze destination characteristics and appeal
- Discover points of interest and hidden gems
- Optimize geographic routing and logistics
- Provide cultural and practical destination insights
- Handle multi-city coordination and connections

#### Input Interface
```json
{
  "analysis_type": "destination_overview|poi_discovery|route_optimization",
  "locations": [
    {
      "name": "Tokyo",
      "type": "city|region|landmark",
      "coordinates": {"lat": 35.6762, "lng": 139.6503},
      "visit_duration": 3
    }
  ],
  "user_interests": ["temples", "modern_architecture", "food_markets"],
  "mobility_constraints": ["public_transport_only", "walking_distance_max_2km"],
  "seasonal_context": {"month": 3, "weather": "spring"}
}
```

#### Output Interface
```json
{
  "destination_analysis": [
    {
      "location": "Tokyo",
      "appeal_score": 9.2,
      "recommended_duration": 4,
      "best_neighborhoods": [
        {
          "name": "Shibuya",
          "characteristics": ["modern", "shopping", "youth_culture"],
          "visit_priority": "high",
          "time_needed": "half_day"
        }
      ],
      "points_of_interest": [
        {
          "name": "Senso-ji Temple",
          "category": "cultural",
          "rating": 4.7,
          "visit_duration": "2h",
          "best_time": "early_morning",
          "coordinates": {"lat": 35.7148, "lng": 139.7967}
        }
      ],
      "practical_info": {
        "transportation_hubs": ["Tokyo Station", "Shinjuku Station"],
        "local_customs": ["bow_greeting", "remove_shoes_temples"],
        "language_notes": "English signage common in tourist areas"
      }
    }
  ],
  "route_optimization": {
    "suggested_flow": ["Shibuya", "Harajuku", "Omotesando"],
    "travel_times": [{"from": "Shibuya", "to": "Harajuku", "duration": "15min"}],
    "efficiency_score": 8.5
  }
}
```

#### Context Requirements
- **Maximum Token Limit**: 6,144 tokens
- **Essential Context**: Destination data, user interests, geographic constraints
- **Optional Context**: Real-time crowd data, local events, seasonal considerations

---

### 3. Transport Agent
**Purpose**: Transportation booking and coordination across all modes  
**Complexity**: High (complex booking logic and real-time data)

#### Core Responsibilities
- Find and compare flight, train, and ground transport options
- Handle complex multi-leg journeys and connections
- Optimize for cost, time, and user preferences
- Manage booking coordination and change management
- Provide real-time updates and alternatives

#### Input Interface
```json
{
  "transport_request": {
    "type": "flight|train|car|local_transport",
    "routes": [
      {
        "from": {"city": "New York", "airport": "JFK"},
        "to": {"city": "Tokyo", "airport": "NRT"},
        "date": "2024-03-15",
        "flexibility": "+/- 2 days"
      }
    ],
    "passengers": [
      {
        "type": "adult",
        "preferences": {
          "seat": "aisle",
          "meal": "vegetarian",
          "class": "economy"
        }
      }
    ],
    "priorities": ["price", "duration", "convenience"],
    "constraints": {
      "max_connections": 1,
      "preferred_airlines": ["ANA", "JAL"],
      "budget_max": 1200
    }
  }
}
```

#### Output Interface
```json
{
  "transport_options": [
    {
      "option_id": "flight_001",
      "type": "flight",
      "route": {
        "segments": [
          {
            "from": "JFK",
            "to": "NRT", 
            "departure": "2024-03-15T18:30:00",
            "arrival": "2024-03-16T22:15:00",
            "airline": "ANA",
            "flight_number": "NH110",
            "aircraft": "Boeing 777",
            "duration": "14h 45m"
          }
        ]
      },
      "pricing": {
        "total": 1089,
        "breakdown": {"base": 950, "taxes": 139},
        "currency": "USD"
      },
      "features": ["wifi", "entertainment", "meal_included"],
      "booking_deadline": "2024-03-01T23:59:00",
      "change_policy": "free_change_24h",
      "carbon_footprint": "2.3 tons CO2"
    }
  ],
  "recommendations": {
    "best_value": "flight_001",
    "fastest": "flight_003",
    "most_convenient": "flight_001"
  },
  "booking_coordination": {
    "suggested_booking_order": ["flights", "trains", "local_transport"],
    "timing_dependencies": ["flight_confirmation_required_for_hotel"]
  }
}
```

#### Context Requirements
- **Maximum Token Limit**: 8,192 tokens
- **Essential Context**: Route requirements, passenger details, preferences
- **Optional Context**: Price history, weather impacts, alternative routes

---

### 4. Accommodation Agent
**Purpose**: Hotel and lodging selection and optimization  
**Complexity**: Medium (booking logic with location coordination)

#### Core Responsibilities
- Find accommodations matching budget and preferences
- Optimize location based on planned activities
- Handle special requirements and group bookings
- Coordinate with transport schedules and local plans
- Manage booking modifications and alternatives

#### Input Interface
```json
{
  "accommodation_request": {
    "locations": [
      {
        "city": "Tokyo",
        "neighborhood_preferences": ["Shibuya", "Shinjuku"],
        "check_in": "2024-03-15",
        "check_out": "2024-03-18",
        "nights": 3
      }
    ],
    "room_requirements": {
      "guests": 2,
      "room_type": "double|twin|suite",
      "accessibility": "wheelchair_accessible"
    },
    "preferences": {
      "style": "modern|traditional|boutique",
      "amenities": ["wifi", "breakfast", "gym", "spa"],
      "budget_per_night": {"min": 100, "max": 300}
    },
    "proximity_requirements": [
      {
        "poi": "Tokyo Station",
        "max_distance": "1km",
        "transport": "walking"
      }
    ]
  }
}
```

#### Output Interface
```json
{
  "accommodation_options": [
    {
      "hotel_id": "hotel_001",
      "name": "Grand Tokyo Hotel",
      "category": "luxury",
      "location": {
        "address": "1-1-1 Marunouchi, Tokyo",
        "neighborhood": "Marunouchi",
        "coordinates": {"lat": 35.6812, "lng": 139.7671}
      },
      "room_details": {
        "type": "Superior Double",
        "size": "30 sqm",
        "view": "city",
        "amenities": ["king_bed", "city_view", "work_desk"]
      },
      "pricing": {
        "per_night": 250,
        "total": 750,
        "includes": ["breakfast", "wifi", "taxes"],
        "cancellation": "free_until_24h"
      },
      "proximity_scores": {
        "tokyo_station": {"distance": "500m", "walk_time": "6min"},
        "shibuya": {"distance": "8km", "train_time": "15min"}
      },
      "ratings": {
        "overall": 4.6,
        "location": 4.8,
        "service": 4.5,
        "cleanliness": 4.7
      }
    }
  ],
  "optimization_analysis": {
    "best_value": "hotel_003",
    "best_location": "hotel_001", 
    "best_amenities": "hotel_002"
  },
  "booking_strategy": {
    "recommended_timing": "book_immediately",
    "price_trend": "increasing",
    "availability_risk": "medium"
  }
}
```

#### Context Requirements
- **Maximum Token Limit**: 6,144 tokens
- **Essential Context**: Location requirements, dates, guest details
- **Optional Context**: Local events affecting availability, seasonal pricing

---

### 5. Activity Agent
**Purpose**: Experience curation and activity recommendation  
**Complexity**: Medium (diverse activity types and personalization)

#### Core Responsibilities
- Curate experiences matching user interests and trip style
- Optimize activity timing and logistics
- Handle reservations and special requirements
- Balance popular attractions with unique experiences
- Coordinate with weather and local conditions

#### Input Interface
```json
{
  "activity_request": {
    "location": "Kyoto",
    "available_time_slots": [
      {
        "date": "2024-03-18",
        "slot": "morning|afternoon|evening",
        "duration": "3h"
      }
    ],
    "interests": ["temples", "traditional_crafts", "gardens"],
    "experience_level": "beginner|intermediate|expert",
    "group_composition": {
      "adults": 2,
      "children": 1,
      "mobility_constraints": []
    },
    "preferences": {
      "crowd_tolerance": "low",
      "physical_activity": "light",
      "cultural_immersion": "high",
      "photo_opportunities": "important"
    },
    "budget_per_activity": {"min": 20, "max": 100}
  }
}
```

#### Output Interface
```json
{
  "activity_recommendations": [
    {
      "activity_id": "kyoto_001",
      "name": "Fushimi Inari Shrine Visit",
      "category": "cultural|adventure|culinary|nature",
      "description": "Explore thousands of vermillion torii gates",
      "location": {
        "name": "Fushimi Inari Shrine",
        "coordinates": {"lat": 34.9671, "lng": 135.7727},
        "access": "JR Inari Station (2min walk)"
      },
      "scheduling": {
        "recommended_slot": "early_morning",
        "duration": "2-3h",
        "best_time": "07:00-10:00",
        "busy_periods": ["11:00-15:00"]
      },
      "experience_details": {
        "difficulty": "easy",
        "walking_distance": "2-5km",
        "highlights": ["torii_tunnel", "mountain_views", "fox_statues"],
        "cultural_significance": "Inari deity, rice harvest god"
      },
      "practical_info": {
        "cost": 0,
        "reservation_required": false,
        "duration_flexibility": "high",
        "weather_dependency": "low"
      },
      "enhancement_options": [
        {
          "name": "Guided Cultural Tour",
          "cost": 45,
          "duration": "+1h",
          "value": "deep_cultural_context"
        }
      ]
    }
  ],
  "itinerary_integration": {
    "suggested_combinations": [
      {
        "morning": "Fushimi Inari Shrine",
        "afternoon": "Bamboo Grove", 
        "synergy": "nature_cultural_theme"
      }
    ],
    "logistics_optimization": {
      "transport_efficiency": 8.5,
      "time_optimization": "minimal_travel_between_activities"
    }
  }
}
```

#### Context Requirements
- **Maximum Token Limit**: 6,144 tokens
- **Essential Context**: User interests, location, time constraints
- **Optional Context**: Weather conditions, local events, seasonal considerations

---

### 6. Budget Agent
**Purpose**: Cost optimization and financial planning  
**Complexity**: Medium (complex financial calculations and optimization)

#### Core Responsibilities
- Track and optimize trip costs across all categories
- Provide budget alternatives and trade-off analysis
- Handle currency conversions and local pricing
- Identify cost-saving opportunities without compromising experience
- Generate detailed financial breakdowns and reports

#### Input Interface
```json
{
  "budget_analysis": {
    "total_budget": 5000,
    "currency": "USD",
    "budget_breakdown": {
      "transport": {"max": 2000, "priority": "high"},
      "accommodation": {"max": 1500, "priority": "medium"},
      "activities": {"max": 800, "priority": "high"},
      "food": {"max": 700, "priority": "medium"}
    },
    "cost_components": [
      {
        "category": "transport",
        "item": "Flight NYC-Tokyo",
        "cost": 1089,
        "currency": "USD",
        "flexibility": "low"
      }
    ],
    "optimization_preferences": {
      "priority": "value|experience|savings",
      "trade_off_tolerance": "low|medium|high"
    }
  }
}
```

#### Output Interface
```json
{
  "budget_analysis": {
    "total_estimated": 4650,
    "budget_utilization": 93,
    "category_breakdown": {
      "transport": {"allocated": 2000, "estimated": 1800, "variance": -200},
      "accommodation": {"allocated": 1500, "estimated": 1350, "variance": -150},
      "activities": {"allocated": 800, "estimated": 750, "variance": -50},
      "food": {"allocated": 700, "estimated": 750, "variance": 50}
    },
    "optimization_opportunities": [
      {
        "category": "accommodation",
        "suggestion": "Move from Shibuya to Shinjuku hotel",
        "savings": 150,
        "impact": "5min longer commute to main activities",
        "recommendation": "high_value"
      }
    ],
    "cost_alerts": [
      {
        "type": "budget_exceeded",
        "category": "food",
        "amount": 50,
        "suggestions": ["cook_some_meals", "lunch_at_convenience_stores"]
      }
    ],
    "financial_summary": {
      "daily_average": 664,
      "cost_per_person": 2325,
      "value_score": 8.7,
      "cost_confidence": "high"
    }
  }
}
```

#### Context Requirements
- **Maximum Token Limit**: 4,096 tokens
- **Essential Context**: Budget constraints, cost components, preferences
- **Optional Context**: Exchange rates, local pricing trends, seasonal variations

---

### 7. Weather Agent
**Purpose**: Climate analysis and weather-optimized recommendations  
**Complexity**: Low (focused weather data analysis)

#### Core Responsibilities
- Provide accurate weather forecasts for trip dates and locations
- Recommend weather-appropriate activities and clothing
- Suggest itinerary modifications based on weather conditions
- Handle seasonal considerations and climate impacts
- Provide real-time weather updates and alerts

#### Input Interface
```json
{
  "weather_request": {
    "locations": [
      {
        "city": "Tokyo",
        "dates": {"start": "2024-03-15", "end": "2024-03-18"}
      }
    ],
    "activity_types": ["outdoor", "indoor", "mixed"],
    "weather_sensitivity": {
      "rain_tolerance": "low",
      "temperature_range": {"min": 10, "max": 25},
      "wind_sensitivity": "medium"
    }
  }
}
```

#### Output Interface
```json
{
  "weather_forecast": [
    {
      "location": "Tokyo",
      "date": "2024-03-15",
      "conditions": {
        "temperature": {"high": 18, "low": 12, "unit": "celsius"},
        "precipitation": {"probability": 20, "type": "light_rain"},
        "wind": {"speed": 15, "direction": "NE", "unit": "kmh"},
        "humidity": 65,
        "visibility": "good"
      },
      "activity_recommendations": {
        "ideal_for": ["indoor_activities", "covered_attractions"],
        "avoid": ["outdoor_hiking", "beach_activities"],
        "optimal_times": "afternoon (warmest period)"
      },
      "clothing_suggestions": {
        "base": ["light_jacket", "long_pants"],
        "accessories": ["umbrella", "light_scarf"],
        "footwear": "comfortable_walking_shoes"
      }
    }
  ],
  "itinerary_adjustments": [
    {
      "original_plan": "Ueno Park morning walk",
      "weather_impact": "high_rain_probability",
      "suggested_change": "Visit Ueno museums instead",
      "impact_level": "low"
    }
  ],
  "seasonal_insights": {
    "cherry_blossom_status": "peak_bloom_expected",
    "tourist_season": "high",
    "local_weather_patterns": "spring_rain_common"
  }
}
```

#### Context Requirements
- **Maximum Token Limit**: 2,048 tokens
- **Essential Context**: Locations, dates, activity types
- **Optional Context**: User weather preferences, clothing constraints

## 🔄 Agent Interaction Patterns

### 1. Sequential Processing
```
Planning Agent → Location Agent → Transport Agent → Accommodation Agent
```
**Use Case**: Simple trip with clear dependencies

### 2. Parallel Processing
```
Location Agent + Weather Agent + Activity Agent (parallel)
↓
Accommodation Agent + Transport Agent (parallel)
```
**Use Case**: Complex trips with independent analysis phases

### 3. Iterative Refinement
```
Initial Pass: All agents provide basic recommendations
↓
Optimization Pass: Budget Agent + Planning Agent refine
↓
Final Pass: All agents adjust based on optimization
```
**Use Case**: Budget-constrained trips requiring multiple optimization rounds

## 📊 Agent Performance Optimization

### Response Time Targets
| Agent Type | Target Response | Max Acceptable | Complexity Factor |
|------------|----------------|----------------|-------------------|
| Weather | 1-2s | 5s | Low |
| Budget | 2-3s | 8s | Medium |
| Location | 3-5s | 10s | Medium |
| Activity | 3-5s | 10s | Medium |
| Accommodation | 4-6s | 12s | Medium |
| Transport | 5-8s | 15s | High |
| Planning | 6-10s | 20s | High |

### Context Window Optimization
- **Minimal Context**: Only essential data for task completion
- **Shared Context Pool**: Common data accessible to all agents
- **Dynamic Context Loading**: Load additional context on-demand
- **Context Compression**: Summarize large datasets for efficiency

### Cost Management
- **Agent Pooling**: Share instances across multiple requests
- **Batch Processing**: Group similar requests for efficiency
- **Smart Caching**: Cache frequent queries and responses
- **Model Selection**: Use appropriate model size for each agent type

---

**Key Agent Design Principles:**
1. **Single Responsibility**: Each agent has a clearly defined, focused purpose
2. **Loose Coupling**: Agents operate independently with minimal dependencies
3. **High Cohesion**: Related functionality grouped within each agent
4. **Scalable Architecture**: Individual agents can be scaled based on demand
5. **Cost Efficiency**: Optimized context usage and appropriate model selection