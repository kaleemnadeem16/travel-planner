# Database Design & Schema

## Database Technology

**Primary Database**: PostgreSQL
- **Rationale**: ACID compliance, JSON support, excellent performance, free and open-source
- **Alternative**: SQLite for development, MySQL as fallback

**Caching Layer**: Redis (optional)
- **Purpose**: Session storage, rate limiting counters, frequently accessed data
- **Alternative**: In-memory caching for simple setups

## Core Database Schema

### Users Table

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(150) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    
    -- Account status
    is_active BOOLEAN DEFAULT TRUE,
    is_verified BOOLEAN DEFAULT FALSE,
    is_staff BOOLEAN DEFAULT FALSE,
    
    -- OAuth integration
    google_id VARCHAR(100) UNIQUE,
    facebook_id VARCHAR(100) UNIQUE,
    
    -- Timestamps
    date_joined TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    
    -- Indexes
    CONSTRAINT users_email_check CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- Indexes for performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_google_id ON users(google_id) WHERE google_id IS NOT NULL;
CREATE INDEX idx_users_facebook_id ON users(facebook_id) WHERE facebook_id IS NOT NULL;
```

### Plans Table

```sql
CREATE TABLE plans (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    -- Basic plan information
    title VARCHAR(200) NOT NULL,
    description TEXT,
    destination VARCHAR(100),
    
    -- Trip dates
    start_date DATE,
    end_date DATE,
    
    -- Budget information
    budget DECIMAL(10, 2),
    currency VARCHAR(3) DEFAULT 'USD',
    
    -- Plan status
    status VARCHAR(20) DEFAULT 'draft' CHECK (status IN ('draft', 'published', 'archived')),
    
    -- Flexible plan data (JSON)
    plan_data JSONB DEFAULT '{}',
    
    -- Sharing and privacy
    is_public BOOLEAN DEFAULT FALSE,
    share_token VARCHAR(100) UNIQUE,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT plans_date_check CHECK (start_date IS NULL OR end_date IS NULL OR start_date <= end_date),
    CONSTRAINT plans_budget_check CHECK (budget IS NULL OR budget >= 0)
);

-- Indexes for performance
CREATE INDEX idx_plans_user_id ON plans(user_id);
CREATE INDEX idx_plans_user_updated ON plans(user_id, updated_at DESC);
CREATE INDEX idx_plans_status ON plans(status);
CREATE INDEX idx_plans_public ON plans(is_public) WHERE is_public = TRUE;
CREATE INDEX idx_plans_share_token ON plans(share_token) WHERE share_token IS NOT NULL;

-- GIN index for JSON queries
CREATE INDEX idx_plans_data ON plans USING GIN (plan_data);

-- Update timestamp trigger
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_plans_updated_at BEFORE UPDATE ON plans
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### Plan Activities Table (Optional - for structured data)

```sql
CREATE TABLE plan_activities (
    id SERIAL PRIMARY KEY,
    plan_id INTEGER NOT NULL REFERENCES plans(id) ON DELETE CASCADE,
    
    -- Activity details
    title VARCHAR(200) NOT NULL,
    description TEXT,
    activity_type VARCHAR(50), -- 'accommodation', 'transport', 'attraction', 'restaurant', etc.
    
    -- Location information
    location_name VARCHAR(200),
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    address TEXT,
    
    -- Timing
    scheduled_date DATE,
    start_time TIME,
    end_time TIME,
    duration_minutes INTEGER,
    
    -- Cost
    estimated_cost DECIMAL(10, 2),
    actual_cost DECIMAL(10, 2),
    
    -- Order and status
    order_index INTEGER DEFAULT 0,
    is_completed BOOLEAN DEFAULT FALSE,
    
    -- Additional data
    metadata JSONB DEFAULT '{}',
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes
CREATE INDEX idx_activities_plan_id ON plan_activities(plan_id);
CREATE INDEX idx_activities_plan_order ON plan_activities(plan_id, order_index);
CREATE INDEX idx_activities_type ON plan_activities(activity_type);
CREATE INDEX idx_activities_location ON plan_activities(latitude, longitude) WHERE latitude IS NOT NULL AND longitude IS NOT NULL;
```

### Rate Limiting Table (Alternative to Redis)

```sql
CREATE TABLE rate_limits (
    id SERIAL PRIMARY KEY,
    identifier VARCHAR(255) NOT NULL, -- IP address or user ID
    identifier_type VARCHAR(20) NOT NULL, -- 'ip' or 'user'
    
    request_count INTEGER DEFAULT 0,
    last_request TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    reset_date DATE DEFAULT CURRENT_DATE,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(identifier, identifier_type, reset_date)
);

CREATE INDEX idx_rate_limits_identifier ON rate_limits(identifier, identifier_type);
CREATE INDEX idx_rate_limits_reset_date ON rate_limits(reset_date);
```

### User Sessions Table (if not using JWT exclusively)

```sql
CREATE TABLE user_sessions (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    session_key VARCHAR(255) UNIQUE NOT NULL,
    session_data TEXT,
    
    ip_address INET,
    user_agent TEXT,
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    
    is_active BOOLEAN DEFAULT TRUE
);

CREATE INDEX idx_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_sessions_key ON user_sessions(session_key);
CREATE INDEX idx_sessions_expires ON user_sessions(expires_at);
```

## Database Relationships

### Entity Relationship Diagram

```
┌─────────────┐         ┌─────────────┐         ┌─────────────────┐
│    users    │         │    plans    │         │ plan_activities │
├─────────────┤         ├─────────────┤         ├─────────────────┤
│ id (PK)     │────────▶│ user_id (FK)│────────▶│ plan_id (FK)    │
│ email       │ 1     * │ id (PK)     │ 1     * │ id (PK)         │
│ username    │         │ title       │         │ title           │
│ password_hash│         │ description │         │ activity_type   │
│ first_name  │         │ destination │         │ location_name   │
│ last_name   │         │ start_date  │         │ scheduled_date  │
│ is_active   │         │ end_date    │         │ estimated_cost  │
│ google_id   │         │ budget      │         │ order_index     │
│ facebook_id │         │ status      │         │ metadata        │
│ date_joined │         │ plan_data   │         │ created_at      │
│ last_login  │         │ is_public   │         │ updated_at      │
└─────────────┘         │ share_token │         └─────────────────┘
                        │ created_at  │
                        │ updated_at  │
                        └─────────────┘
```

### Relationship Details

1. **Users → Plans**: One-to-Many
   - One user can have multiple travel plans
   - Foreign key: `plans.user_id` → `users.id`
   - Cascade delete: When user is deleted, all their plans are deleted

2. **Plans → Activities**: One-to-Many (Optional)
   - One plan can have multiple activities
   - Foreign key: `plan_activities.plan_id` → `plans.id`
   - Cascade delete: When plan is deleted, all activities are deleted

## JSON Data Structure Examples

### plan_data Field Examples

```json
{
  "preferences": {
    "budget_type": "budget",
    "travel_style": "cultural",
    "group_size": 2,
    "accessibility_needs": []
  },
  "accommodation": {
    "type": "hotel",
    "booking_details": {},
    "preferences": ["wifi", "breakfast"]
  },
  "transportation": {
    "arrival": {
      "method": "flight",
      "details": {}
    },
    "local": {
      "methods": ["walking", "public_transport"]
    }
  },
  "itinerary": [
    {
      "day": 1,
      "date": "2025-09-01",
      "activities": [
        {
          "time": "09:00",
          "activity": "Museum visit",
          "location": "City Museum",
          "duration": 120,
          "cost": 15.00
        }
      ]
    }
  ],
  "notes": "Remember to book tickets in advance",
  "external_bookings": {
    "flights": [],
    "hotels": [],
    "activities": []
  }
}
```

### activity metadata Examples

```json
{
  "booking_info": {
    "confirmation_number": "ABC123",
    "provider": "BookingProvider",
    "cancellation_policy": "free_cancellation_24h"
  },
  "contact_info": {
    "phone": "+1-555-0123",
    "website": "https://example.com",
    "email": "info@example.com"
  },
  "reviews": {
    "rating": 4.5,
    "source": "TripAdvisor",
    "review_count": 245
  },
  "tags": ["family_friendly", "indoor", "accessible"]
}
```

## Performance Considerations

### Database Optimization

1. **Indexing Strategy**
   - Primary keys on all tables
   - Foreign key indexes for joins
   - Composite indexes for common query patterns
   - GIN indexes for JSON queries

2. **Query Optimization**
   - Use EXPLAIN ANALYZE for slow queries
   - Implement pagination for large result sets
   - Consider read replicas for scaling

3. **Connection Pooling**
   - Use connection pooling (pgBouncer)
   - Configure appropriate pool sizes

### Data Archival

```sql
-- Archive old plans
CREATE TABLE archived_plans (LIKE plans INCLUDING ALL);

-- Move old inactive plans to archive
INSERT INTO archived_plans 
SELECT * FROM plans 
WHERE updated_at < CURRENT_DATE - INTERVAL '2 years' 
AND status = 'archived';
```

## Backup and Recovery

### Backup Strategy

```bash
# Daily backup script
pg_dump travel_planner > backups/travel_planner_$(date +%Y%m%d).sql

# Automated backup with compression
pg_dump travel_planner | gzip > backups/travel_planner_$(date +%Y%m%d).sql.gz
```

### Migration Management

```python
# Alembic migrations example
alembic revision --autogenerate -m "Create initial tables"
alembic upgrade head

# Data migration for schema changes
python manage.py datamigration app_name migration_name
```

## Security Considerations

1. **Data Encryption**
   - Encrypt sensitive data at rest
   - Use SSL/TLS for data in transit
   - Hash passwords with strong algorithms

2. **Access Control**
   - Row-level security for multi-tenant data
   - Database user permissions
   - Application-level authorization

3. **Data Privacy**
   - GDPR compliance for EU users
   - Data retention policies
   - User data deletion procedures

## TODO: Implementation Tasks

- [ ] Set up PostgreSQL database
- [ ] Create initial migration files
- [ ] Implement database models in chosen framework
- [ ] Set up connection pooling
- [ ] Configure backup procedures
- [ ] Implement data validation constraints
- [ ] Add database monitoring
- [ ] Create data seeding scripts
- [ ] Set up read replicas (if needed)
- [ ] Implement data archival strategy