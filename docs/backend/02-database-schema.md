# Database Architecture & Schema Design

## 🎯 Database Strategy Overview

Our database architecture supports comprehensive tracking, versioning, and monitoring for a multi-agent travel planning system with the following requirements:

✅ **Version Control** - Complete history of every plan iteration  
✅ **Agent Tracking** - Individual agent execution records  
✅ **Cost Monitoring** - Detailed token usage and cost breakdown  
✅ **User Sessions** - Context continuity across interactions  
✅ **Vector Search** - RAG and similarity matching capabilities  
✅ **Performance Optimization** - Indexes and query optimization for Oracle Cloud ARM

## 🗄️ Primary Database: PostgreSQL 15+

### Core Design Principles

1. **Immutable History** - Never delete historical data, only mark as inactive
2. **Granular Tracking** - Track every agent execution, token usage, and cost
3. **Flexible Schema** - JSONB for dynamic plan data and agent results
4. **User-Centric** - All data organized around users and their plans
5. **Performance First** - Optimized indexes for common query patterns

### Complete Database Schema

```sql
-- =====================================================
-- USERS & AUTHENTICATION
-- =====================================================

-- Enhanced user table with subscription and preference tracking
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    
    -- Personal information
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone_number VARCHAR(20),
    date_of_birth DATE,
    
    -- User preferences and profile data
    profile_data JSONB DEFAULT '{}',
    travel_preferences JSONB DEFAULT '{}',
    communication_preferences JSONB DEFAULT '{}',
    
    -- Subscription and billing
    subscription_tier VARCHAR(50) DEFAULT 'free',
    billing_data JSONB DEFAULT '{}',
    
    -- Account status
    is_active BOOLEAN DEFAULT true,
    is_verified BOOLEAN DEFAULT false,
    verification_token VARCHAR(255),
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login TIMESTAMP WITH TIME ZONE,
    email_verified_at TIMESTAMP WITH TIME ZONE,
    
    -- Soft deletion
    deleted_at TIMESTAMP WITH TIME ZONE
);

-- =====================================================
-- TRAVEL PLANS & VERSIONING
-- =====================================================

-- Comprehensive travel plans with full version control
CREATE TABLE travel_plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    -- Plan identification
    title VARCHAR(255) NOT NULL,
    description TEXT,
    
    -- Trip details
    destinations JSONB NOT NULL DEFAULT '[]',
    start_date DATE,
    end_date DATE,
    duration_days INTEGER GENERATED ALWAYS AS (
        CASE WHEN start_date IS NOT NULL AND end_date IS NOT NULL 
        THEN (end_date - start_date) 
        ELSE NULL END
    ) STORED,
    
    -- Travelers information
    traveler_count INTEGER DEFAULT 1,
    traveler_details JSONB DEFAULT '{}', -- ages, special needs, etc.
    
    -- Budget information
    budget_total DECIMAL(12, 2),
    budget_currency VARCHAR(3) DEFAULT 'USD',
    budget_breakdown JSONB DEFAULT '{}',
    
    -- Preferences and constraints
    preferences JSONB DEFAULT '{}',
    constraints JSONB DEFAULT '{}',
    special_requirements JSONB DEFAULT '{}',
    
    -- Generated plan data
    plan_data JSONB DEFAULT '{}',
    itinerary JSONB DEFAULT '{}',
    
    -- Agent coordination results
    agent_results JSONB DEFAULT '{}',
    agent_execution_summary JSONB DEFAULT '{}',
    
    -- Version control
    version INTEGER DEFAULT 1,
    parent_plan_id UUID REFERENCES travel_plans(id),
    is_active BOOLEAN DEFAULT true,
    is_template BOOLEAN DEFAULT false,
    is_public BOOLEAN DEFAULT false,
    
    -- Status tracking
    status VARCHAR(50) DEFAULT 'draft', 
    -- draft, processing, agent_processing, completed, error, cancelled
    completion_percentage INTEGER DEFAULT 0,
    
    -- Cost tracking
    total_cost_usd DECIMAL(10, 6) DEFAULT 0.00,
    token_usage_total INTEGER DEFAULT 0,
    
    -- Quality metrics
    user_rating INTEGER CHECK (user_rating >= 1 AND user_rating <= 5),
    user_feedback TEXT,
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE,
    archived_at TIMESTAMP WITH TIME ZONE,
    
    -- Soft deletion
    deleted_at TIMESTAMP WITH TIME ZONE
);

-- =====================================================
-- AGENT EXECUTION TRACKING
-- =====================================================

-- Detailed tracking of every agent execution
CREATE TABLE agent_executions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    plan_id UUID NOT NULL REFERENCES travel_plans(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    -- Agent identification
    agent_type VARCHAR(50) NOT NULL,
    agent_version VARCHAR(20) DEFAULT '1.0',
    agent_config JSONB DEFAULT '{}',
    
    -- Model information
    model_provider VARCHAR(50) NOT NULL,
    model_name VARCHAR(100) NOT NULL,
    model_config JSONB DEFAULT '{}',
    
    -- Execution data
    task_data JSONB NOT NULL,
    context_data JSONB DEFAULT '{}',
    prompt_data JSONB DEFAULT '{}',
    result_data JSONB DEFAULT '{}',
    error_data JSONB DEFAULT '{}',
    
    -- Performance metrics
    execution_time_ms INTEGER,
    queue_time_ms INTEGER,
    prompt_tokens INTEGER DEFAULT 0,
    completion_tokens INTEGER DEFAULT 0,
    total_tokens INTEGER DEFAULT 0,
    cost_usd DECIMAL(10, 6) DEFAULT 0.00,
    
    -- Quality metrics
    result_quality_score DECIMAL(3, 2),
    user_satisfaction_score INTEGER CHECK (user_satisfaction_score >= 1 AND user_satisfaction_score <= 5),
    
    -- Status and retries
    status VARCHAR(50) NOT NULL, 
    -- pending, queued, running, completed, failed, timeout, cancelled
    retry_count INTEGER DEFAULT 0,
    max_retries INTEGER DEFAULT 3,
    
    -- Timing
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    
    -- Distributed tracing
    trace_id VARCHAR(100),
    span_id VARCHAR(100),
    parent_span_id VARCHAR(100),
    correlation_id VARCHAR(100),
    
    -- Session context
    session_id VARCHAR(100),
    request_id VARCHAR(100)
);

-- =====================================================
-- COMPREHENSIVE COST TRACKING
-- =====================================================

-- Granular cost tracking for billing and optimization
CREATE TABLE cost_tracking (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    plan_id UUID REFERENCES travel_plans(id) ON DELETE CASCADE,
    agent_execution_id UUID REFERENCES agent_executions(id) ON DELETE CASCADE,
    
    -- Model and provider details
    model_provider VARCHAR(50) NOT NULL,
    model_name VARCHAR(100) NOT NULL,
    model_tier VARCHAR(20), -- tier1, tier2, tier3
    
    -- Token breakdown
    prompt_tokens INTEGER DEFAULT 0,
    completion_tokens INTEGER DEFAULT 0,
    total_tokens INTEGER DEFAULT 0,
    
    -- Pricing details
    prompt_cost_per_token DECIMAL(12, 10),
    completion_cost_per_token DECIMAL(12, 10),
    total_cost_usd DECIMAL(10, 6) NOT NULL,
    
    -- Billing information
    billing_date DATE DEFAULT CURRENT_DATE,
    billing_period VARCHAR(20) DEFAULT 'monthly',
    invoice_id VARCHAR(100),
    
    -- Usage categorization
    usage_category VARCHAR(50), -- planning, research, optimization, etc.
    usage_type VARCHAR(50), -- regular, retry, optimization
    
    -- Optimization metrics
    cost_efficiency_score DECIMAL(4, 2),
    alternative_cost_usd DECIMAL(10, 6), -- what it would cost with different model
    savings_usd DECIMAL(10, 6),
    
    -- Metadata
    tags JSONB DEFAULT '{}',
    metadata JSONB DEFAULT '{}',
    
    -- Timestamps
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Session context
    session_id VARCHAR(100),
    request_id VARCHAR(100)
);

-- =====================================================
-- USER SESSIONS & CONTEXT
-- =====================================================

-- User sessions for conversation continuity
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    session_token VARCHAR(255) NOT NULL UNIQUE,
    plan_id UUID REFERENCES travel_plans(id) ON DELETE SET NULL,
    
    -- Session data
    context_data JSONB DEFAULT '{}',
    conversation_history JSONB DEFAULT '[]',
    user_preferences JSONB DEFAULT '{}',
    agent_states JSONB DEFAULT '{}',
    
    -- Session metadata
    device_info JSONB DEFAULT '{}',
    ip_address INET,
    user_agent TEXT,
    location_data JSONB DEFAULT '{}',
    
    -- Activity tracking
    page_views INTEGER DEFAULT 0,
    actions_count INTEGER DEFAULT 0,
    last_activity_type VARCHAR(50),
    
    -- Timing
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_activity TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE,
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    ended_at TIMESTAMP WITH TIME ZONE
);

-- =====================================================
-- AGENT COMMUNICATION & COORDINATION
-- =====================================================

-- Track inter-agent communications for debugging and optimization
CREATE TABLE agent_communications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    plan_id UUID NOT NULL REFERENCES travel_plans(id) ON DELETE CASCADE,
    session_id VARCHAR(100),
    
    -- Communication details
    from_agent VARCHAR(50),
    to_agent VARCHAR(50),
    message_type VARCHAR(50) NOT NULL,
    message_data JSONB NOT NULL,
    priority INTEGER DEFAULT 5 CHECK (priority >= 1 AND priority <= 10),
    
    -- Response tracking
    response_data JSONB DEFAULT '{}',
    response_time_ms INTEGER,
    
    -- Status
    status VARCHAR(50) DEFAULT 'sent', 
    -- sent, delivered, received, processed, failed, timeout
    error_message TEXT,
    retry_count INTEGER DEFAULT 0,
    
    -- Timing
    sent_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    delivered_at TIMESTAMP WITH TIME ZONE,
    received_at TIMESTAMP WITH TIME ZONE,
    processed_at TIMESTAMP WITH TIME ZONE,
    
    -- Tracing
    trace_id VARCHAR(100),
    correlation_id VARCHAR(100)
);

-- =====================================================
-- SYSTEM MONITORING & HEALTH
-- =====================================================

-- System health and performance metrics
CREATE TABLE system_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Metric details
    metric_name VARCHAR(100) NOT NULL,
    metric_type VARCHAR(50) NOT NULL, -- counter, gauge, histogram, timer
    value DECIMAL(15, 6) NOT NULL,
    unit VARCHAR(20),
    
    -- Dimensions/tags
    tags JSONB DEFAULT '{}',
    
    -- Context
    service_name VARCHAR(50),
    environment VARCHAR(20),
    version VARCHAR(20),
    
    -- Timestamp
    recorded_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Retention hint
    retention_days INTEGER DEFAULT 30
);

-- =====================================================
-- PLAN SHARING & COLLABORATION
-- =====================================================

-- Plan sharing and collaboration features
CREATE TABLE plan_shares (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    plan_id UUID NOT NULL REFERENCES travel_plans(id) ON DELETE CASCADE,
    shared_by_user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    shared_with_user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    
    -- Sharing details
    share_token VARCHAR(255) UNIQUE NOT NULL,
    share_type VARCHAR(20) NOT NULL, -- public, private, link
    permissions JSONB DEFAULT '{}', -- read, comment, edit
    
    -- Public sharing
    is_public BOOLEAN DEFAULT false,
    public_url_slug VARCHAR(100) UNIQUE,
    
    -- Access control
    password_hash VARCHAR(255),
    access_count INTEGER DEFAULT 0,
    max_access_count INTEGER,
    
    -- Timing
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE,
    last_accessed_at TIMESTAMP WITH TIME ZONE,
    
    -- Status
    is_active BOOLEAN DEFAULT true,
    revoked_at TIMESTAMP WITH TIME ZONE
);

-- =====================================================
-- COMPREHENSIVE INDEXING STRATEGY
-- =====================================================

-- Users table indexes
CREATE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_username ON users(username) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_active ON users(is_active) WHERE is_active = true AND deleted_at IS NULL;
CREATE INDEX idx_users_subscription ON users(subscription_tier);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Travel plans indexes
CREATE INDEX idx_travel_plans_user_id ON travel_plans(user_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_travel_plans_status ON travel_plans(status);
CREATE INDEX idx_travel_plans_active ON travel_plans(is_active) WHERE is_active = true;
CREATE INDEX idx_travel_plans_dates ON travel_plans(start_date, end_date);
CREATE INDEX idx_travel_plans_created_at ON travel_plans(created_at);
CREATE INDEX idx_travel_plans_version ON travel_plans(parent_plan_id, version);
CREATE INDEX idx_travel_plans_destinations ON travel_plans USING GIN(destinations);
CREATE INDEX idx_travel_plans_preferences ON travel_plans USING GIN(preferences);

-- Agent executions indexes  
CREATE INDEX idx_agent_executions_plan_id ON agent_executions(plan_id);
CREATE INDEX idx_agent_executions_user_id ON agent_executions(user_id);
CREATE INDEX idx_agent_executions_agent_type ON agent_executions(agent_type);
CREATE INDEX idx_agent_executions_status ON agent_executions(status);
CREATE INDEX idx_agent_executions_created_at ON agent_executions(created_at);
CREATE INDEX idx_agent_executions_model ON agent_executions(model_provider, model_name);
CREATE INDEX idx_agent_executions_trace ON agent_executions(trace_id);
CREATE INDEX idx_agent_executions_session ON agent_executions(session_id);

-- Cost tracking indexes
CREATE INDEX idx_cost_tracking_user_id ON cost_tracking(user_id);
CREATE INDEX idx_cost_tracking_plan_id ON cost_tracking(plan_id);
CREATE INDEX idx_cost_tracking_billing_date ON cost_tracking(billing_date);
CREATE INDEX idx_cost_tracking_model ON cost_tracking(model_provider, model_name);
CREATE INDEX idx_cost_tracking_created_at ON cost_tracking(created_at);
CREATE INDEX idx_cost_tracking_agent_execution ON cost_tracking(agent_execution_id);

-- User sessions indexes
CREATE INDEX idx_user_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_user_sessions_token ON user_sessions(session_token);
CREATE INDEX idx_user_sessions_active ON user_sessions(is_active) WHERE is_active = true;
CREATE INDEX idx_user_sessions_last_activity ON user_sessions(last_activity);
CREATE INDEX idx_user_sessions_plan_id ON user_sessions(plan_id);

-- Agent communications indexes
CREATE INDEX idx_agent_communications_plan_id ON agent_communications(plan_id);
CREATE INDEX idx_agent_communications_session ON agent_communications(session_id);
CREATE INDEX idx_agent_communications_agents ON agent_communications(from_agent, to_agent);
CREATE INDEX idx_agent_communications_created_at ON agent_communications(sent_at);
CREATE INDEX idx_agent_communications_status ON agent_communications(status);

-- System metrics indexes
CREATE INDEX idx_system_metrics_name ON system_metrics(metric_name);
CREATE INDEX idx_system_metrics_recorded_at ON system_metrics(recorded_at);
CREATE INDEX idx_system_metrics_tags ON system_metrics USING GIN(tags);
CREATE INDEX idx_system_metrics_service ON system_metrics(service_name);

-- Plan shares indexes
CREATE INDEX idx_plan_shares_plan_id ON plan_shares(plan_id);
CREATE INDEX idx_plan_shares_token ON plan_shares(share_token);
CREATE INDEX idx_plan_shares_public ON plan_shares(is_public) WHERE is_public = true;
CREATE INDEX idx_plan_shares_active ON plan_shares(is_active) WHERE is_active = true;

-- =====================================================
-- TRIGGERS & FUNCTIONS
-- =====================================================

-- Automatic timestamp updates
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Apply timestamp triggers
CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_travel_plans_updated_at BEFORE UPDATE ON travel_plans
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- Auto-increment plan version
CREATE OR REPLACE FUNCTION auto_increment_plan_version()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.parent_plan_id IS NOT NULL THEN
        NEW.version = (
            SELECT COALESCE(MAX(version), 0) + 1 
            FROM travel_plans 
            WHERE parent_plan_id = NEW.parent_plan_id OR id = NEW.parent_plan_id
        );
    END IF;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER auto_increment_plan_version_trigger BEFORE INSERT ON travel_plans
    FOR EACH ROW EXECUTE FUNCTION auto_increment_plan_version();

-- Update plan cost summary
CREATE OR REPLACE FUNCTION update_plan_cost_summary()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE travel_plans 
    SET 
        total_cost_usd = (
            SELECT COALESCE(SUM(total_cost_usd), 0) 
            FROM cost_tracking 
            WHERE plan_id = NEW.plan_id
        ),
        token_usage_total = (
            SELECT COALESCE(SUM(total_tokens), 0) 
            FROM agent_executions 
            WHERE plan_id = NEW.plan_id
        )
    WHERE id = NEW.plan_id;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_plan_cost_trigger AFTER INSERT ON cost_tracking
    FOR EACH ROW EXECUTE FUNCTION update_plan_cost_summary();

-- =====================================================
-- VIEWS FOR COMMON QUERIES
-- =====================================================

-- Active plans with cost summary
CREATE VIEW v_active_plans_with_costs AS
SELECT 
    p.*,
    COALESCE(cost_summary.total_cost, 0) as total_cost_calculated,
    COALESCE(cost_summary.total_tokens, 0) as total_tokens_calculated,
    COALESCE(cost_summary.execution_count, 0) as agent_execution_count
FROM travel_plans p
LEFT JOIN (
    SELECT 
        plan_id,
        SUM(total_cost_usd) as total_cost,
        SUM(total_tokens) as total_tokens,
        COUNT(*) as execution_count
    FROM cost_tracking
    GROUP BY plan_id
) cost_summary ON p.id = cost_summary.plan_id
WHERE p.is_active = true AND p.deleted_at IS NULL;

-- User cost summary by period
CREATE VIEW v_user_cost_summary AS
SELECT 
    user_id,
    DATE_TRUNC('month', created_at) as month,
    SUM(total_cost_usd) as monthly_cost,
    SUM(total_tokens) as monthly_tokens,
    COUNT(*) as monthly_requests,
    COUNT(DISTINCT plan_id) as plans_worked_on
FROM cost_tracking
GROUP BY user_id, DATE_TRUNC('month', created_at);

-- Agent performance metrics
CREATE VIEW v_agent_performance AS
SELECT 
    agent_type,
    model_provider,
    model_name,
    COUNT(*) as execution_count,
    AVG(execution_time_ms) as avg_execution_time,
    AVG(total_tokens) as avg_tokens,
    AVG(cost_usd) as avg_cost,
    SUM(CASE WHEN status = 'completed' THEN 1 ELSE 0 END)::FLOAT / COUNT(*) as success_rate
FROM agent_executions
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY agent_type, model_provider, model_name;

-- =====================================================
-- DATA RETENTION POLICIES
-- =====================================================

-- Function to clean old system metrics
CREATE OR REPLACE FUNCTION cleanup_old_system_metrics()
RETURNS INTEGER AS $$
DECLARE
    deleted_count INTEGER;
BEGIN
    DELETE FROM system_metrics 
    WHERE recorded_at < NOW() - INTERVAL '1 day' * 
          COALESCE((tags->>'retention_days')::INTEGER, retention_days, 30);
    
    GET DIAGNOSTICS deleted_count = ROW_COUNT;
    RETURN deleted_count;
END;
$$ language 'plpgsql';

-- =====================================================
-- PARTITIONING STRATEGY (for high-volume tables)
-- =====================================================

-- Partition agent_executions by date for better performance
-- CREATE TABLE agent_executions_y2024m01 PARTITION OF agent_executions
-- FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- =====================================================
-- SECURITY & ROW LEVEL SECURITY
-- =====================================================

-- Enable RLS on sensitive tables
ALTER TABLE travel_plans ENABLE ROW LEVEL SECURITY;
ALTER TABLE agent_executions ENABLE ROW LEVEL SECURITY;
ALTER TABLE cost_tracking ENABLE ROW LEVEL SECURITY;

-- Policies to ensure users can only see their own data
CREATE POLICY user_travel_plans_policy ON travel_plans
    FOR ALL TO authenticated_users
    USING (user_id = current_setting('app.current_user_id')::UUID);

CREATE POLICY user_agent_executions_policy ON agent_executions
    FOR ALL TO authenticated_users
    USING (user_id = current_setting('app.current_user_id')::UUID);

CREATE POLICY user_cost_tracking_policy ON cost_tracking
    FOR ALL TO authenticated_users
    USING (user_id = current_setting('app.current_user_id')::UUID);
```

## 📊 Database Performance Optimization

### Query Optimization Strategies

```sql
-- 1. Efficient pagination for large datasets
SELECT * FROM travel_plans 
WHERE user_id = $1 
  AND created_at < $2 
ORDER BY created_at DESC 
LIMIT $3;

-- 2. Cost analysis with proper indexing
SELECT 
    DATE_TRUNC('day', created_at) as date,
    SUM(total_cost_usd) as daily_cost,
    COUNT(*) as request_count
FROM cost_tracking 
WHERE user_id = $1 
  AND created_at >= $2 
GROUP BY DATE_TRUNC('day', created_at)
ORDER BY date DESC;

-- 3. Agent performance analytics
SELECT 
    agent_type,
    AVG(execution_time_ms) as avg_time,
    COUNT(*) as executions,
    SUM(total_cost_usd) as total_cost
FROM agent_executions ae
JOIN cost_tracking ct ON ae.id = ct.agent_execution_id
WHERE ae.created_at >= NOW() - INTERVAL '7 days'
  AND ae.status = 'completed'
GROUP BY agent_type
ORDER BY total_cost DESC;
```

### Connection Pool Configuration for ARM64

```python
# app/core/database.py
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool
import os

# Optimized for Oracle Cloud ARM64 (24GB RAM)
DATABASE_CONFIG = {
    "pool_size": 20,          # Base connections
    "max_overflow": 30,       # Additional connections under load
    "pool_pre_ping": True,    # Validate connections
    "pool_recycle": 3600,     # Recycle connections every hour
    "echo": False,            # Disable SQL logging in production
}

# ARM64 optimized engine
engine = create_engine(
    os.getenv("DATABASE_URL"),
    poolclass=QueuePool,
    **DATABASE_CONFIG
)
```

This database architecture provides:

✅ **Complete Audit Trail** - Every action tracked with full context  
✅ **Cost Optimization** - Granular cost tracking per user/plan/agent  
✅ **Performance Monitoring** - Detailed metrics for optimization  
✅ **Version Control** - Full history of plan iterations  
✅ **Scalability** - Optimized for Oracle Cloud ARM architecture  
✅ **Security** - Row-level security and proper access control  
✅ **Flexibility** - JSONB fields for dynamic data structures  
✅ **Monitoring Ready** - Built-in metrics and health tracking

The schema is designed to handle high-volume operations while maintaining data integrity and providing comprehensive insights for cost optimization and performance tuning.