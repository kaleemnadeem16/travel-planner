# Backend Architecture Decisions & Technical Stack

## 🎯 Architecture Overview

This document outlines the technical decisions for building a production-ready, multi-agent travel planning system with comprehensive monitoring, database architecture, and cloud deployment on Oracle Cloud ARM.

## 🏗️ Core Technology Stack

### 1. Backend Framework: **FastAPI + LangGraph**

#### Why FastAPI?
```python
# Performance & Features
- Native async/await support for concurrent agent execution
- Automatic API documentation (OpenAPI/Swagger)
- Built-in request validation with Pydantic
- WebSocket support for real-time agent communication
- High performance (comparable to Node.js/Go)
- Type hints throughout for better maintainability
```

#### Why LangGraph over LangChain?
```yaml
LangGraph Advantages:
  - Better state management for multi-agent workflows
  - Built-in conversation memory and persistence  
  - Visual workflow representation and debugging
  - Advanced routing and conditional execution
  - Better error handling and recovery
  - Native support for human-in-the-loop
  - Optimized for production deployment
```

### 2. Agent Monitoring Framework: **LangSmith + Custom Metrics**

#### LangSmith Integration
```python
# Comprehensive agent monitoring
Features:
  - Automatic request/response logging
  - Token usage and cost tracking per agent
  - Performance metrics and latency monitoring
  - Error rate tracking and alerting
  - Model comparison and A/B testing
  - Real-time debugging and tracing
  - Custom metrics and dashboards
```

#### Custom Monitoring Layer
```python
# Additional monitoring capabilities
class AgentMonitoringSystem:
    - Cost tracking per user/session/plan
    - Agent performance benchmarking
    - Resource utilization monitoring
    - API rate limit management
    - Custom business metrics
    - Integration with Prometheus/Grafana
```

### 3. Database Architecture: **PostgreSQL + Qdrant Vector DB**

#### Primary Database: PostgreSQL 15+
```sql
-- Core data storage
- User management and authentication
- Plan storage and versioning
- Agent execution history and logs
- Cost tracking and billing data
- System configuration and settings
```

#### Vector Database: Qdrant
```python
# Vector storage for RAG and search
Use Cases:
  - Previous trip plans for recommendations
  - Location descriptions and preferences
  - User interaction patterns
  - Agent response caching
  - Similarity search for plan suggestions
```

## 🔧 Framework Flexibility Architecture

### 1. Model Provider Abstraction Layer

```python
# backend/core/model_providers.py
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional
from enum import Enum

class ModelProvider(str, Enum):
    OPENAI = "openai"
    ANTHROPIC = "anthropic"
    AZURE_OPENAI = "azure_openai"
    GOOGLE = "google"
    COHERE = "cohere"
    OLLAMA = "ollama"  # For local deployment

class BaseModelProvider(ABC):
    """Abstract base class for model providers"""
    
    def __init__(self, api_key: str, base_url: Optional[str] = None):
        self.api_key = api_key
        self.base_url = base_url
        self.client = self._initialize_client()
    
    @abstractmethod
    async def chat_completion(
        self, 
        messages: list,
        model: str,
        **kwargs
    ) -> Dict[str, Any]:
        """Unified chat completion interface"""
        pass
    
    @abstractmethod
    async def get_embedding(
        self, 
        text: str, 
        model: str
    ) -> list[float]:
        """Unified embedding interface"""
        pass
    
    @abstractmethod
    def calculate_cost(
        self, 
        prompt_tokens: int, 
        completion_tokens: int, 
        model: str
    ) -> float:
        """Calculate cost for token usage"""
        pass

class OpenAIProvider(BaseModelProvider):
    """OpenAI provider implementation"""
    
    def _initialize_client(self):
        from openai import AsyncOpenAI
        return AsyncOpenAI(api_key=self.api_key)
    
    async def chat_completion(self, messages: list, model: str, **kwargs) -> Dict[str, Any]:
        response = await self.client.chat.completions.create(
            model=model,
            messages=messages,
            **kwargs
        )
        
        return {
            "content": response.choices[0].message.content,
            "prompt_tokens": response.usage.prompt_tokens,
            "completion_tokens": response.usage.completion_tokens,
            "total_tokens": response.usage.total_tokens,
            "model": model,
            "provider": "openai"
        }
    
    def calculate_cost(self, prompt_tokens: int, completion_tokens: int, model: str) -> float:
        # GPT-5 series pricing
        pricing = {
            "gpt-5": {"input": 1.25, "output": 10.0},
            "gpt-5-mini": {"input": 0.25, "output": 1.0},
            "gpt-5-nano": {"input": 0.05, "output": 0.40},
        }
        
        model_pricing = pricing.get(model, pricing["gpt-5-mini"])
        input_cost = (prompt_tokens / 1000) * model_pricing["input"]
        output_cost = (completion_tokens / 1000) * model_pricing["output"]
        
        return input_cost + output_cost

class AnthropicProvider(BaseModelProvider):
    """Anthropic provider implementation"""
    
    def _initialize_client(self):
        from anthropic import AsyncAnthropic
        return AsyncAnthropic(api_key=self.api_key)
    
    async def chat_completion(self, messages: list, model: str, **kwargs) -> Dict[str, Any]:
        # Convert OpenAI format to Anthropic format
        system_message = None
        user_messages = []
        
        for msg in messages:
            if msg["role"] == "system":
                system_message = msg["content"]
            else:
                user_messages.append(msg)
        
        response = await self.client.messages.create(
            model=model,
            system=system_message,
            messages=user_messages,
            **kwargs
        )
        
        return {
            "content": response.content[0].text,
            "prompt_tokens": response.usage.input_tokens,
            "completion_tokens": response.usage.output_tokens,
            "total_tokens": response.usage.input_tokens + response.usage.output_tokens,
            "model": model,
            "provider": "anthropic"
        }

# Provider Factory
class ModelProviderFactory:
    """Factory for creating model providers"""
    
    @staticmethod
    def create_provider(provider: ModelProvider, api_key: str, **kwargs) -> BaseModelProvider:
        if provider == ModelProvider.OPENAI:
            return OpenAIProvider(api_key, **kwargs)
        elif provider == ModelProvider.ANTHROPIC:
            return AnthropicProvider(api_key, **kwargs)
        elif provider == ModelProvider.AZURE_OPENAI:
            return AzureOpenAIProvider(api_key, **kwargs)
        else:
            raise ValueError(f"Unsupported provider: {provider}")
```

### 2. Universal Agent Configuration

```python
# backend/core/agent_config.py
from pydantic import BaseModel
from typing import Dict, Any, Optional

class UniversalAgentConfig(BaseModel):
    """Universal configuration that works with any model provider"""
    
    agent_type: str
    provider: ModelProvider
    model_name: str
    max_tokens: int = 4096
    temperature: float = 0.7
    timeout_seconds: int = 30
    retry_attempts: int = 3
    
    # Provider-specific overrides
    provider_config: Dict[str, Any] = {}
    
    # Cost management
    max_cost_per_request: float = 1.0
    cost_optimization_enabled: bool = True
    
    # Monitoring configuration
    logging_enabled: bool = True
    metrics_enabled: bool = True
    langsmith_enabled: bool = True

# Environment-based configuration
class AgentConfigManager:
    """Manages agent configurations from environment variables"""
    
    def __init__(self):
        self.configs = self._load_configurations()
    
    def _load_configurations(self) -> Dict[str, UniversalAgentConfig]:
        configs = {}
        
        for agent_type in ["planning", "transport", "location", "accommodation", "activity", "budget", "weather"]:
            configs[agent_type] = self._load_agent_config(agent_type)
        
        return configs
    
    def _load_agent_config(self, agent_type: str) -> UniversalAgentConfig:
        # Load from environment with fallback to defaults
        env_prefix = f"{agent_type.upper()}_AGENT"
        
        return UniversalAgentConfig(
            agent_type=agent_type,
            provider=ModelProvider(os.getenv(f"{env_prefix}_PROVIDER", "openai")),
            model_name=os.getenv(f"{env_prefix}_MODEL", self._get_default_model(agent_type)),
            max_tokens=int(os.getenv(f"{env_prefix}_MAX_TOKENS", "4096")),
            temperature=float(os.getenv(f"{env_prefix}_TEMPERATURE", "0.7")),
            timeout_seconds=int(os.getenv(f"{env_prefix}_TIMEOUT", "30")),
            retry_attempts=int(os.getenv(f"{env_prefix}_RETRIES", "3")),
            max_cost_per_request=float(os.getenv(f"{env_prefix}_MAX_COST", "1.0")),
            cost_optimization_enabled=os.getenv(f"{env_prefix}_COST_OPT", "true").lower() == "true",
            logging_enabled=os.getenv(f"{env_prefix}_LOGGING", "true").lower() == "true",
            metrics_enabled=os.getenv(f"{env_prefix}_METRICS", "true").lower() == "true",
            langsmith_enabled=os.getenv("LANGSMITH_ENABLED", "true").lower() == "true"
        )
    
    def _get_default_model(self, agent_type: str) -> str:
        """Get default model based on agent complexity"""
        tier1_agents = ["planning", "transport"]
        tier2_agents = ["location", "accommodation", "activity"]
        tier3_agents = ["budget", "weather"]
        
        if agent_type in tier1_agents:
            return "gpt-5"
        elif agent_type in tier2_agents:
            return "gpt-5-mini"
        else:
            return "gpt-5-nano"
```

## 🗄️ Database Schema & Architecture

### 1. PostgreSQL Schema Design

```sql
-- backend/database/migrations/001_initial_schema.sql

-- Users table with comprehensive tracking
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    profile_data JSONB DEFAULT '{}',
    preferences JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_login TIMESTAMP WITH TIME ZONE,
    is_active BOOLEAN DEFAULT true,
    subscription_tier VARCHAR(50) DEFAULT 'free'
);

-- Travel plans with comprehensive versioning
CREATE TABLE travel_plans (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    destinations JSONB NOT NULL DEFAULT '[]',
    start_date DATE,
    end_date DATE,
    budget_total DECIMAL(10, 2),
    traveler_count INTEGER DEFAULT 1,
    traveler_details JSONB DEFAULT '{}',
    preferences JSONB DEFAULT '{}',
    constraints JSONB DEFAULT '{}',
    
    -- Plan data and agent results
    plan_data JSONB DEFAULT '{}',
    agent_results JSONB DEFAULT '{}',
    
    -- Version control
    version INTEGER DEFAULT 1,
    parent_plan_id UUID REFERENCES travel_plans(id),
    is_active BOOLEAN DEFAULT true,
    is_template BOOLEAN DEFAULT false,
    
    -- Status tracking
    status VARCHAR(50) DEFAULT 'draft', -- draft, processing, completed, error
    completion_percentage INTEGER DEFAULT 0,
    
    -- Metadata
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE,
    archived_at TIMESTAMP WITH TIME ZONE
);

-- Agent execution history for detailed tracking
CREATE TABLE agent_executions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    plan_id UUID NOT NULL REFERENCES travel_plans(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    -- Agent details
    agent_type VARCHAR(50) NOT NULL,
    agent_version VARCHAR(20) DEFAULT '1.0',
    
    -- Model information
    model_provider VARCHAR(50) NOT NULL,
    model_name VARCHAR(100) NOT NULL,
    model_config JSONB DEFAULT '{}',
    
    -- Execution details
    task_data JSONB NOT NULL,
    context_data JSONB DEFAULT '{}',
    result_data JSONB DEFAULT '{}',
    error_data JSONB DEFAULT '{}',
    
    -- Performance metrics
    execution_time_ms INTEGER,
    prompt_tokens INTEGER DEFAULT 0,
    completion_tokens INTEGER DEFAULT 0,
    total_tokens INTEGER DEFAULT 0,
    cost_usd DECIMAL(10, 6) DEFAULT 0.00,
    
    -- Status and timing
    status VARCHAR(50) NOT NULL, -- pending, running, completed, failed, timeout
    retry_count INTEGER DEFAULT 0,
    started_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE,
    
    -- Tracing
    trace_id VARCHAR(100),
    span_id VARCHAR(100),
    parent_span_id VARCHAR(100)
);

-- Detailed cost tracking for billing and optimization
CREATE TABLE cost_tracking (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    plan_id UUID REFERENCES travel_plans(id) ON DELETE CASCADE,
    agent_execution_id UUID REFERENCES agent_executions(id) ON DELETE CASCADE,
    
    -- Cost breakdown
    model_provider VARCHAR(50) NOT NULL,
    model_name VARCHAR(100) NOT NULL,
    prompt_tokens INTEGER DEFAULT 0,
    completion_tokens INTEGER DEFAULT 0,
    total_tokens INTEGER DEFAULT 0,
    cost_per_token DECIMAL(10, 8),
    total_cost_usd DECIMAL(10, 6) NOT NULL,
    
    -- Billing period
    billing_date DATE DEFAULT CURRENT_DATE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Additional metadata
    session_id VARCHAR(100),
    request_id VARCHAR(100),
    tags JSONB DEFAULT '{}'
);

-- User sessions for context continuity
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    session_token VARCHAR(255) NOT NULL UNIQUE,
    plan_id UUID REFERENCES travel_plans(id) ON DELETE SET NULL,
    
    -- Session data
    context_data JSONB DEFAULT '{}',
    conversation_history JSONB DEFAULT '[]',
    preferences JSONB DEFAULT '{}',
    
    -- Timing
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_activity TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expires_at TIMESTAMP WITH TIME ZONE,
    
    -- Status
    is_active BOOLEAN DEFAULT true
);

-- Agent communication logs for debugging
CREATE TABLE agent_communications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    plan_id UUID NOT NULL REFERENCES travel_plans(id) ON DELETE CASCADE,
    session_id VARCHAR(100),
    
    -- Communication details
    from_agent VARCHAR(50),
    to_agent VARCHAR(50),
    message_type VARCHAR(50) NOT NULL,
    message_data JSONB NOT NULL,
    
    -- Timing
    sent_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    received_at TIMESTAMP WITH TIME ZONE,
    processed_at TIMESTAMP WITH TIME ZONE,
    
    -- Status
    status VARCHAR(50) DEFAULT 'sent', -- sent, received, processed, failed
    error_message TEXT
);

-- Indexes for performance
CREATE INDEX idx_travel_plans_user_id ON travel_plans(user_id);
CREATE INDEX idx_travel_plans_status ON travel_plans(status);
CREATE INDEX idx_travel_plans_created_at ON travel_plans(created_at);
CREATE INDEX idx_travel_plans_active ON travel_plans(is_active) WHERE is_active = true;

CREATE INDEX idx_agent_executions_plan_id ON agent_executions(plan_id);
CREATE INDEX idx_agent_executions_agent_type ON agent_executions(agent_type);
CREATE INDEX idx_agent_executions_status ON agent_executions(status);
CREATE INDEX idx_agent_executions_created_at ON agent_executions(started_at);

CREATE INDEX idx_cost_tracking_user_id ON cost_tracking(user_id);
CREATE INDEX idx_cost_tracking_billing_date ON cost_tracking(billing_date);
CREATE INDEX idx_cost_tracking_plan_id ON cost_tracking(plan_id);

CREATE INDEX idx_user_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_user_sessions_token ON user_sessions(session_token);
CREATE INDEX idx_user_sessions_active ON user_sessions(is_active) WHERE is_active = true;

-- Functions for automatic updates
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

-- Triggers
CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_travel_plans_updated_at BEFORE UPDATE ON travel_plans
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 2. Vector Database Integration (Qdrant)

```python
# backend/core/vector_store.py
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct
from typing import List, Dict, Any, Optional
import uuid

class TravelPlannerVectorStore:
    """Vector store for RAG and similarity search"""
    
    def __init__(self, url: str = "http://localhost:6333"):
        self.client = QdrantClient(url=url)
        self.collections = {
            "plans": "travel_plans_collection",
            "locations": "locations_collection", 
            "preferences": "user_preferences_collection",
            "agents": "agent_responses_collection"
        }
        self._initialize_collections()
    
    def _initialize_collections(self):
        """Initialize vector collections if they don't exist"""
        for collection_name in self.collections.values():
            try:
                self.client.create_collection(
                    collection_name=collection_name,
                    vectors_config=VectorParams(
                        size=1536,  # OpenAI embedding size
                        distance=Distance.COSINE
                    )
                )
            except Exception:
                pass  # Collection already exists
    
    async def store_plan_embedding(
        self, 
        plan_id: str, 
        plan_data: Dict[str, Any], 
        embedding: List[float]
    ):
        """Store travel plan embedding for similarity search"""
        point = PointStruct(
            id=str(uuid.uuid4()),
            vector=embedding,
            payload={
                "plan_id": plan_id,
                "destinations": plan_data.get("destinations", []),
                "duration": plan_data.get("duration"),
                "budget": plan_data.get("budget"),
                "preferences": plan_data.get("preferences", []),
                "created_at": plan_data.get("created_at"),
                "user_id": plan_data.get("user_id")
            }
        )
        
        self.client.upsert(
            collection_name=self.collections["plans"],
            points=[point]
        )
    
    async def search_similar_plans(
        self, 
        query_embedding: List[float], 
        user_id: Optional[str] = None,
        limit: int = 5
    ) -> List[Dict[str, Any]]:
        """Search for similar travel plans"""
        search_filter = None
        if user_id:
            search_filter = {"must": [{"key": "user_id", "match": {"value": user_id}}]}
        
        results = self.client.search(
            collection_name=self.collections["plans"],
            query_vector=query_embedding,
            query_filter=search_filter,
            limit=limit
        )
        
        return [
            {
                "plan_id": result.payload["plan_id"],
                "score": result.score,
                "destinations": result.payload["destinations"],
                "preferences": result.payload["preferences"]
            }
            for result in results
        ]
    
    async def store_agent_response(
        self,
        agent_type: str,
        query: str,
        response: str,
        embedding: List[float],
        metadata: Dict[str, Any]
    ):
        """Cache agent responses for faster similar queries"""
        point = PointStruct(
            id=str(uuid.uuid4()),
            vector=embedding,
            payload={
                "agent_type": agent_type,
                "query": query,
                "response": response,
                "created_at": metadata.get("created_at"),
                "cost": metadata.get("cost", 0),
                "tokens": metadata.get("tokens", 0)
            }
        )
        
        self.client.upsert(
            collection_name=self.collections["agents"],
            points=[point]
        )
```

## 📊 Comprehensive Monitoring System

### 1. LangSmith Integration

```python
# backend/core/monitoring.py
from langsmith import Client
from langsmith.run_helpers import trace
from typing import Dict, Any, Optional
import os
import time

class AgentMonitoringSystem:
    """Comprehensive monitoring for multi-agent system"""
    
    def __init__(self):
        self.langsmith_client = Client() if os.getenv("LANGSMITH_API_KEY") else None
        self.metrics_collector = MetricsCollector()
        self.cost_tracker = CostTracker()
    
    @trace(name="agent_execution")
    async def monitor_agent_execution(
        self,
        agent_type: str,
        task_data: Dict[str, Any],
        user_id: str,
        plan_id: str,
        session_id: str
    ):
        """Monitor complete agent execution with LangSmith tracing"""
        
        execution_start = time.time()
        execution_id = str(uuid.uuid4())
        
        try:
            # Record execution start
            await self._record_execution_start(
                execution_id, agent_type, task_data, user_id, plan_id
            )
            
            # Execute agent with monitoring
            result = await self._execute_agent_with_monitoring(
                agent_type, task_data, execution_id
            )
            
            # Record successful completion
            execution_time = time.time() - execution_start
            await self._record_execution_success(
                execution_id, result, execution_time
            )
            
            return result
            
        except Exception as e:
            # Record failure
            execution_time = time.time() - execution_start
            await self._record_execution_failure(
                execution_id, e, execution_time
            )
            raise
    
    async def _execute_agent_with_monitoring(
        self,
        agent_type: str,
        task_data: Dict[str, Any],
        execution_id: str
    ) -> Dict[str, Any]:
        """Execute agent with detailed monitoring"""
        
        # Get agent configuration
        agent_config = AgentConfigManager().configs[agent_type]
        
        # Initialize model provider
        provider = ModelProviderFactory.create_provider(
            agent_config.provider,
            self._get_api_key(agent_config.provider)
        )
        
        # Prepare prompt
        messages = await self._prepare_agent_prompt(agent_type, task_data)
        
        # Execute with LangSmith tracing
        with trace(name=f"{agent_type}_model_call") as run:
            response = await provider.chat_completion(
                messages=messages,
                model=agent_config.model_name,
                max_tokens=agent_config.max_tokens,
                temperature=agent_config.temperature
            )
            
            # Record metrics in LangSmith
            run.update(
                inputs={"messages": messages, "config": agent_config.dict()},
                outputs={"response": response},
                tags=[agent_type, agent_config.provider, agent_config.model_name]
            )
        
        # Calculate and record cost
        cost = provider.calculate_cost(
            response["prompt_tokens"],
            response["completion_tokens"], 
            agent_config.model_name
        )
        
        await self.cost_tracker.record_cost(
            execution_id=execution_id,
            provider=agent_config.provider,
            model=agent_config.model_name,
            prompt_tokens=response["prompt_tokens"],
            completion_tokens=response["completion_tokens"],
            cost=cost
        )
        
        return response

class CostTracker:
    """Detailed cost tracking and optimization"""
    
    def __init__(self):
        self.db = get_database()
    
    async def record_cost(
        self,
        execution_id: str,
        provider: str,
        model: str,
        prompt_tokens: int,
        completion_tokens: int,
        cost: float,
        user_id: Optional[str] = None,
        plan_id: Optional[str] = None
    ):
        """Record cost data in database"""
        
        await self.db.execute("""
            INSERT INTO cost_tracking (
                agent_execution_id, user_id, plan_id, model_provider,
                model_name, prompt_tokens, completion_tokens, total_tokens,
                cost_per_token, total_cost_usd, created_at
            ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, NOW())
        """, 
        execution_id, user_id, plan_id, provider, model,
        prompt_tokens, completion_tokens, prompt_tokens + completion_tokens,
        cost / (prompt_tokens + completion_tokens) if (prompt_tokens + completion_tokens) > 0 else 0,
        cost
        )
        
        # Check cost alerts
        await self._check_cost_alerts(user_id, cost)
    
    async def get_cost_summary(
        self, 
        user_id: str, 
        period: str = "month"
    ) -> Dict[str, Any]:
        """Get cost summary for user"""
        
        if period == "month":
            date_filter = "created_at >= date_trunc('month', CURRENT_DATE)"
        elif period == "day":
            date_filter = "created_at >= CURRENT_DATE"
        else:
            date_filter = "created_at >= date_trunc('week', CURRENT_DATE)"
        
        result = await self.db.fetchrow(f"""
            SELECT 
                SUM(total_cost_usd) as total_cost,
                SUM(total_tokens) as total_tokens,
                COUNT(*) as total_requests,
                AVG(total_cost_usd) as avg_cost_per_request,
                COUNT(DISTINCT plan_id) as unique_plans
            FROM cost_tracking 
            WHERE user_id = $1 AND {date_filter}
        """, user_id)
        
        return dict(result) if result else {}
    
    async def _check_cost_alerts(self, user_id: str, current_cost: float):
        """Check if cost alerts should be triggered"""
        
        # Daily cost check
        daily_total = await self.db.fetchval("""
            SELECT COALESCE(SUM(total_cost_usd), 0)
            FROM cost_tracking 
            WHERE user_id = $1 AND created_at >= CURRENT_DATE
        """, user_id)
        
        if daily_total > 10.0:  # $10 daily limit
            await self._send_cost_alert(user_id, "daily", daily_total)
        
        # Single request cost check
        if current_cost > 1.0:  # $1 per request limit
            await self._send_cost_alert(user_id, "request", current_cost)
```

## ☁️ Oracle Cloud ARM Deployment Architecture

### 1. Ampere A1 Optimized Configuration

```yaml
# docker-compose.arm64.yml - Optimized for Oracle Cloud ARM
version: '3.8'

services:
  # FastAPI Backend - ARM64 optimized
  travel-planner-api:
    build:
      context: ./backend
      dockerfile: Dockerfile.arm64
    image: travel-planner-api:arm64
    platform: linux/arm64
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/travel_planner
      - REDIS_URL=redis://redis:6379/0
      - QDRANT_URL=http://qdrant:6333
    volumes:
      - ./backend/.env:/app/.env:ro
    deploy:
      resources:
        limits:
          memory: 4G      # 4GB out of 24GB for main API
          cpus: '2.0'     # 2 cores out of 4 for main service
        reservations:
          memory: 2G
          cpus: '1.0'
    depends_on:
      - postgres
      - redis
      - qdrant
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Agent Workers - Multiple ARM64 instances
  agent-worker-planning:
    build:
      context: ./backend
      dockerfile: Dockerfile.arm64
    platform: linux/arm64
    command: celery -A tasks worker --loglevel=info --queues=planning_tasks --concurrency=2
    environment:
      - REDIS_URL=redis://redis:6379/0
      - AGENT_TYPE=planning
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 1G
          cpus: '0.5'
        reservations:
          memory: 512M
          cpus: '0.25'
    depends_on:
      - redis
    restart: unless-stopped

  agent-worker-general:
    build:
      context: ./backend
      dockerfile: Dockerfile.arm64
    platform: linux/arm64
    command: celery -A tasks worker --loglevel=info --queues=location_tasks,transport_tasks,accommodation_tasks,activity_tasks --concurrency=4
    environment:
      - REDIS_URL=redis://redis:6379/0
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 1.5G
          cpus: '0.5'
        reservations:
          memory: 1G
          cpus: '0.25'
    depends_on:
      - redis
    restart: unless-stopped

  # PostgreSQL - ARM64 optimized
  postgres:
    image: postgres:15-alpine  # ARM64 compatible
    platform: linux/arm64
    environment:
      POSTGRES_DB: travel_planner
      POSTGRES_USER: user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_INITDB_ARGS: "--auth-host=scram-sha-256"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    deploy:
      resources:
        limits:
          memory: 4G      # 4GB for database
          cpus: '1.0'
        reservations:
          memory: 2G
          cpus: '0.5'
    restart: unless-stopped
    command: |
      postgres 
      -c shared_preload_libraries=pg_stat_statements
      -c max_connections=200
      -c shared_buffers=1GB
      -c effective_cache_size=3GB
      -c work_mem=64MB
      -c maintenance_work_mem=512MB

  # Redis - ARM64 optimized  
  redis:
    image: redis:7-alpine  # ARM64 compatible
    platform: linux/arm64
    command: redis-server --appendonly yes --maxmemory 2gb --maxmemory-policy allkeys-lru
    volumes:
      - redis_data:/data
    deploy:
      resources:
        limits:
          memory: 2G      # 2GB for Redis
          cpus: '0.5'
        reservations:
          memory: 1G
          cpus: '0.25'
    restart: unless-stopped

  # Qdrant Vector Database - ARM64 optimized
  qdrant:
    image: qdrant/qdrant:latest  # ARM64 compatible
    platform: linux/arm64
    ports:
      - "6333:6333"
    volumes:
      - qdrant_data:/qdrant/storage
    environment:
      - QDRANT__SERVICE__HTTP_PORT=6333
      - QDRANT__SERVICE__GRPC_PORT=6334
    deploy:
      resources:
        limits:
          memory: 4G      # 4GB for vector operations
          cpus: '1.0'
        reservations:
          memory: 2G
          cpus: '0.5'
    restart: unless-stopped

  # Monitoring Stack - Lightweight for ARM64
  prometheus:
    image: prom/prometheus:latest  # ARM64 compatible
    platform: linux/arm64
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '0.25'
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest  # ARM64 compatible
    platform: linux/arm64
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '0.25'
    restart: unless-stopped

  # Frontend - React optimized for ARM64
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.arm64
    platform: linux/arm64
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/ssl/certs:ro
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '0.5'
        reservations:
          memory: 512M
          cpus: '0.25'
    depends_on:
      - travel-planner-api
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
  qdrant_data:
  prometheus_data:
  grafana_data:

# Networks for proper isolation
networks:
  default:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### 2. ARM64 Optimized Dockerfile

```dockerfile
# backend/Dockerfile.arm64
FROM python:3.11-slim-bullseye

# Set platform explicitly
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PLATFORM=linux/arm64

WORKDIR /app

# Install ARM64 optimized system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    libffi-dev \
    libssl-dev \
    libpq-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies with ARM64 optimizations
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user optimized for Oracle Cloud
RUN useradd --create-home --shell /bin/bash --uid 1001 appuser && \
    chown -R appuser:appuser /app
USER appuser

# Health check endpoint
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Expose port
EXPOSE 8000

# Optimized startup for ARM64
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4", "--worker-class", "uvicorn.workers.UvicornWorker"]
```

### 3. Oracle Cloud ARM64 Setup Script

```bash
#!/bin/bash
# scripts/setup-oracle-cloud-arm64.sh

echo "Setting up Travel Planner on Oracle Cloud ARM64..."

# System optimization for Ampere A1
echo "Optimizing system for ARM64..."
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-plugin htop nginx certbot

# Docker optimization for ARM64
echo "Configuring Docker for ARM64..."
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

# Create optimized Docker daemon configuration
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  }
}
EOF

sudo systemctl restart docker

# System resource optimization for 24GB RAM
echo "Optimizing system resources..."
sudo sysctl -w vm.swappiness=10
sudo sysctl -w vm.max_map_count=262144
sudo sysctl -w net.core.somaxconn=65535

# Make optimizations permanent
sudo tee -a /etc/sysctl.conf > /dev/null <<EOF
vm.swappiness=10
vm.max_map_count=262144
net.core.somaxconn=65535
net.core.netdev_max_backlog=5000
net.ipv4.tcp_max_syn_backlog=8192
EOF

# Setup environment
echo "Setting up application environment..."
cp .env.example .env

# Build and start services
echo "Building ARM64 optimized containers..."
docker compose -f docker-compose.arm64.yml build --parallel

echo "Starting services..."
docker compose -f docker-compose.arm64.yml up -d

# Setup SSL with Let's Encrypt
echo "Setting up SSL certificates..."
sudo certbot --nginx -d yourdomain.com

# Setup monitoring
echo "Configuring monitoring..."
docker compose -f docker-compose.arm64.yml logs -f &

echo "Setup complete! Access your application at https://yourdomain.com"
echo "Monitoring: http://yourdomain.com:3000 (Grafana)"
echo "Metrics: http://yourdomain.com:9090 (Prometheus)"
```

This architecture provides:

✅ **Framework Flexibility** - Works with OpenAI, Anthropic, Azure, Google, etc.  
✅ **Comprehensive Monitoring** - LangSmith + custom metrics + cost tracking  
✅ **Database Versioning** - Full history and version control  
✅ **Vector Search** - Qdrant for RAG and similarity matching  
✅ **Oracle Cloud ARM Optimized** - Efficient resource usage on Ampere A1  
✅ **Production Ready** - Health checks, auto-scaling, monitoring

Would you like me to continue with the next phase of documentation updates?
