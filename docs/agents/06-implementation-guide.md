# Implementation Guide & Development Roadmap

## 🎯 Updated Implementation Strategy

This guide provides a streamlined implementation roadmap for the multi-agent Travel Planner system based on our finalized architecture decisions.

> **📋 For complete technical details, see:**
> - **Backend Architecture**: [`/backend/01-architecture-decisions.md`](../backend/01-architecture-decisions.md)
> - **Database Schema**: [`/backend/02-database-schema.md`](../backend/02-database-schema.md)
> - **Monitoring System**: [`/backend/03-monitoring-system.md`](../backend/03-monitoring-system.md)

## 🏗️ Finalized Technology Stack

### **Core Architecture**
- **Backend Framework**: FastAPI 0.104+ (async-first, high performance)
- **Agent Framework**: LangGraph (state management, workflow orchestration)
- **Model Abstraction**: Universal provider layer (OpenAI, Anthropic, Azure, etc.)
- **Primary Database**: PostgreSQL 15+ (comprehensive tracking & versioning)
- **Vector Database**: Qdrant (RAG, similarity search, response caching)
- **Message Queue**: Redis 7+ + Celery (agent coordination)
- **Monitoring**: LangSmith + Prometheus + Grafana
- **Deployment**: Oracle Cloud ARM64 (Ampere A1 optimized)

### **Key Architectural Decisions**

📝 **Why FastAPI + LangGraph?**
- Native async/await for concurrent agent execution
- Automatic API documentation and type safety
- LangGraph provides purpose-built multi-agent state management
- Superior performance compared to traditional frameworks for our use case

📝 **Why PostgreSQL + Qdrant?**
- PostgreSQL: Complete audit trail, version control, cost tracking
- Qdrant: Vector search for plan recommendations and response caching
- Both optimized for Oracle Cloud ARM64 deployment

📝 **Why LangSmith + Prometheus?**
- LangSmith: AI-specific monitoring, token tracking, model comparisons
- Prometheus: Infrastructure metrics, custom business metrics
- Combined: Complete visibility into both technical and business performance

#### Model Configuration System
```python
# config/models.py
from enum import Enum
from pydantic import BaseModel
from typing import Dict, Optional

class ModelTier(str, Enum):
    TIER1 = "tier1"  # Premium models for complex reasoning
    TIER2 = "tier2"  # Standard models for moderate tasks  
    TIER3 = "tier3"  # Efficient models for simple tasks

class ModelConfig(BaseModel):
    model_name: str
    provider: str  # "openai", "anthropic", "azure"
    max_tokens: int
    temperature: float = 0.7
    top_p: float = 0.9
    frequency_penalty: float = 0.0
    presence_penalty: float = 0.0
    timeout_seconds: int = 30
    retry_attempts: int = 3
    cost_per_1k_tokens: float

# Default configurations
DEFAULT_MODEL_CONFIGS: Dict[ModelTier, ModelConfig] = {
    ModelTier.TIER1: ModelConfig(
        model_name="gpt-5",
        provider="openai",
        max_tokens=8192,
        cost_per_1k_tokens=1.25
    ),
    ModelTier.TIER2: ModelConfig(
        model_name="gpt-5-mini", 
        provider="openai",
        max_tokens=6144,
        cost_per_1k_tokens=0.25
    ),
    ModelTier.TIER3: ModelConfig(
        model_name="gpt-5-nano",
        provider="openai", 
        max_tokens=4096,
        cost_per_1k_tokens=0.05
    )
}

# Agent-specific model assignments
AGENT_MODEL_MAPPING: Dict[str, ModelTier] = {
    "planning": ModelTier.TIER1,
    "transport": ModelTier.TIER1,
    "location": ModelTier.TIER2,
    "accommodation": ModelTier.TIER2,
    "activity": ModelTier.TIER2,
    "budget": ModelTier.TIER3,
    "weather": ModelTier.TIER3
}
```

### 2. Environment Configuration System

#### .env Template with Model Configuration
```env
# ===========================================
# TRAVEL PLANNER AGENT SYSTEM CONFIGURATION
# ===========================================

# Database Configuration
DATABASE_URL=postgresql://user:password@localhost:5432/travel_planner
REDIS_URL=redis://localhost:6379/0

# Model Provider API Keys
OPENAI_API_KEY=your_openai_api_key_here
ANTHROPIC_API_KEY=your_anthropic_api_key_here
AZURE_OPENAI_API_KEY=your_azure_api_key_here
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/

# ===========================================
# AGENT MODEL CONFIGURATION
# ===========================================

# Default Model Settings by Tier
DEFAULT_MODEL_TIER1=gpt-5
DEFAULT_MODEL_TIER2=gpt-5-mini
DEFAULT_MODEL_TIER3=gpt-5-nano
DEFAULT_PROVIDER=openai

# Default Token Limits by Tier
DEFAULT_TOKENS_TIER1=8192
DEFAULT_TOKENS_TIER2=6144
DEFAULT_TOKENS_TIER3=4096

# Agent-Specific Model Overrides
PLANNING_AGENT_MODEL=gpt-5
PLANNING_AGENT_PROVIDER=openai
PLANNING_AGENT_MAX_TOKENS=8192
PLANNING_AGENT_TEMPERATURE=0.7

TRANSPORT_AGENT_MODEL=gpt-5
TRANSPORT_AGENT_PROVIDER=openai
TRANSPORT_AGENT_MAX_TOKENS=8192
TRANSPORT_AGENT_TEMPERATURE=0.6

LOCATION_AGENT_MODEL=gpt-5-mini
LOCATION_AGENT_PROVIDER=openai
LOCATION_AGENT_MAX_TOKENS=6144
LOCATION_AGENT_TEMPERATURE=0.7

ACCOMMODATION_AGENT_MODEL=gpt-5-mini
ACCOMMODATION_AGENT_PROVIDER=openai
ACCOMMODATION_AGENT_MAX_TOKENS=6144
ACCOMMODATION_AGENT_TEMPERATURE=0.7

ACTIVITY_AGENT_MODEL=gpt-5-mini
ACTIVITY_AGENT_PROVIDER=openai
ACTIVITY_AGENT_MAX_TOKENS=6144
ACTIVITY_AGENT_TEMPERATURE=0.8

BUDGET_AGENT_MODEL=gpt-5-nano
BUDGET_AGENT_PROVIDER=openai
BUDGET_AGENT_MAX_TOKENS=4096
BUDGET_AGENT_TEMPERATURE=0.3

WEATHER_AGENT_MODEL=gpt-5-nano
WEATHER_AGENT_PROVIDER=openai
WEATHER_AGENT_MAX_TOKENS=2048
WEATHER_AGENT_TEMPERATURE=0.1

# ===========================================
# AGENT SYSTEM CONFIGURATION
# ===========================================

# Orchestrator Settings
MAX_CONCURRENT_AGENTS=10
AGENT_TIMEOUT_SECONDS=30
ORCHESTRATOR_RETRY_ATTEMPTS=3
CIRCUIT_BREAKER_ENABLED=true
CIRCUIT_BREAKER_THRESHOLD=5

# Communication Settings
MESSAGE_QUEUE_TYPE=redis
MESSAGE_RETENTION_HOURS=24
MAX_MESSAGE_SIZE_KB=64
COMPRESSION_ENABLED=true
ENCRYPTION_ENABLED=true

# Context Management
CONTEXT_COMPRESSION_ENABLED=true
LAZY_LOADING_ENABLED=true
SHARED_CONTEXT_POOL_ENABLED=true
CONTEXT_TTL_SECONDS=3600

# Performance Monitoring
METRICS_ENABLED=true
DETAILED_LOGGING=false
COST_TRACKING_ENABLED=true
PERFORMANCE_ALERTS=true

# ===========================================
# FRONTEND INTEGRATION
# ===========================================

# WebSocket Configuration
WS_AGENT_COORDINATION_URL=ws://localhost:8080/agent-coordination
WS_RECONNECT_ATTEMPTS=5
WS_HEARTBEAT_INTERVAL=30000

# UI Settings
GRAPH_VISUALIZATION_ENABLED=true
REAL_TIME_UPDATES_ENABLED=true
VOICE_INTERFACE_ENABLED=false
MOBILE_OPTIMIZED=true

# ===========================================
# EXTERNAL API CONFIGURATION
# ===========================================

# Travel APIs
AMADEUS_API_KEY=your_amadeus_key
BOOKING_COM_API_KEY=your_booking_key
GOOGLE_PLACES_API_KEY=your_google_places_key
WEATHER_API_KEY=your_weather_key

# Development/Production Mode
ENVIRONMENT=development
DEBUG=true
LOG_LEVEL=INFO
```

### 3. Agent Base Classes & Model Integration

#### Base Agent Implementation
```python
# agents/base_agent.py
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional
from pydantic import BaseModel
import asyncio
import time
from openai import AsyncOpenAI
from anthropic import AsyncAnthropic

class AgentConfig(BaseModel):
    agent_type: str
    model_config: ModelConfig
    context_manager: str
    timeout_seconds: int = 30
    retry_attempts: int = 3

class BaseAgent(ABC):
    def __init__(self, config: AgentConfig):
        self.config = config
        self.model_client = self._initialize_model_client()
        self.context_manager = self._initialize_context_manager()
        self.metrics = AgentMetrics(self.config.agent_type)
        
    def _initialize_model_client(self):
        """Initialize the appropriate model client based on provider"""
        provider = self.config.model_config.provider
        
        if provider == "openai":
            return AsyncOpenAI(api_key=os.getenv("OPENAI_API_KEY"))
        elif provider == "anthropic":
            return AsyncAnthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))
        elif provider == "azure":
            return AsyncOpenAI(
                api_key=os.getenv("AZURE_OPENAI_API_KEY"),
                azure_endpoint=os.getenv("AZURE_OPENAI_ENDPOINT"),
                api_version="2024-02-15-preview"
            )
        else:
            raise ValueError(f"Unsupported provider: {provider}")
    
    @abstractmethod
    async def process_task(self, task_data: Dict[str, Any]) -> Dict[str, Any]:
        """Process a task and return results"""
        pass
    
    async def execute_with_retry(self, task_data: Dict[str, Any]) -> Dict[str, Any]:
        """Execute task with retry logic and metrics tracking"""
        start_time = time.time()
        last_exception = None
        
        for attempt in range(self.config.retry_attempts):
            try:
                # Prepare context for this attempt
                context = await self.context_manager.prepare_context(
                    self.config.agent_type, 
                    task_data
                )
                
                # Execute the task
                result = await asyncio.wait_for(
                    self._execute_with_model(context, task_data),
                    timeout=self.config.timeout_seconds
                )
                
                # Track successful execution
                execution_time = time.time() - start_time
                await self.metrics.record_success(execution_time, result)
                
                return result
                
            except asyncio.TimeoutError as e:
                last_exception = e
                await self.metrics.record_timeout(attempt + 1)
                if attempt < self.config.retry_attempts - 1:
                    await asyncio.sleep(2 ** attempt)  # Exponential backoff
                    
            except Exception as e:
                last_exception = e
                await self.metrics.record_error(e, attempt + 1)
                if attempt < self.config.retry_attempts - 1:
                    await asyncio.sleep(2 ** attempt)
        
        # All retries failed
        total_time = time.time() - start_time
        await self.metrics.record_failure(last_exception, total_time)
        raise AgentExecutionError(
            f"Agent {self.config.agent_type} failed after {self.config.retry_attempts} attempts",
            last_exception
        )
    
    async def _execute_with_model(self, context: Dict[str, Any], task_data: Dict[str, Any]) -> Dict[str, Any]:
        """Execute the actual model call"""
        prompt = await self._prepare_prompt(context, task_data)
        
        if self.config.model_config.provider in ["openai", "azure"]:
            response = await self.model_client.chat.completions.create(
                model=self.config.model_config.model_name,
                messages=prompt,
                max_tokens=self.config.model_config.max_tokens,
                temperature=self.config.model_config.temperature,
                top_p=self.config.model_config.top_p,
                frequency_penalty=self.config.model_config.frequency_penalty,
                presence_penalty=self.config.model_config.presence_penalty
            )
            
            result = self._parse_openai_response(response)
            
        elif self.config.model_config.provider == "anthropic":
            response = await self.model_client.messages.create(
                model=self.config.model_config.model_name,
                messages=prompt,
                max_tokens=self.config.model_config.max_tokens,
                temperature=self.config.model_config.temperature
            )
            
            result = self._parse_anthropic_response(response)
        
        # Track token usage and cost
        await self.metrics.record_token_usage(
            prompt_tokens=result.get("prompt_tokens", 0),
            completion_tokens=result.get("completion_tokens", 0),
            cost=self._calculate_cost(result)
        )
        
        return result
    
    @abstractmethod
    async def _prepare_prompt(self, context: Dict[str, Any], task_data: Dict[str, Any]) -> list:
        """Prepare the prompt for the model"""
        pass
    
    def _calculate_cost(self, result: Dict[str, Any]) -> float:
        """Calculate the cost of the model call"""
        total_tokens = result.get("prompt_tokens", 0) + result.get("completion_tokens", 0)
        cost_per_1k = self.config.model_config.cost_per_1k_tokens
        return (total_tokens / 1000) * cost_per_1k
```

#### Specific Agent Implementation Example
```python
# agents/planning_agent.py
from .base_agent import BaseAgent
from typing import Dict, Any

class PlanningAgent(BaseAgent):
    async def process_task(self, task_data: Dict[str, Any]) -> Dict[str, Any]:
        """Process planning task - create trip structure and itinerary framework"""
        return await self.execute_with_retry(task_data)
    
    async def _prepare_prompt(self, context: Dict[str, Any], task_data: Dict[str, Any]) -> list:
        """Prepare planning-specific prompt"""
        
        system_prompt = """You are an expert travel planning agent responsible for creating 
        comprehensive trip structures and itinerary frameworks. You specialize in:
        
        1. Analyzing complex travel requirements and constraints
        2. Creating logical trip structures with optimal sequencing
        3. Balancing competing priorities (time, budget, preferences)
        4. Generating flexible itinerary frameworks
        5. Handling multi-destination trip coordination
        
        Always respond with structured JSON output following the specified schema."""
        
        user_prompt = f"""
        Create a comprehensive trip plan based on the following requirements:
        
        Trip Context:
        {self._format_context(context)}
        
        Task Details:
        {self._format_task_data(task_data)}
        
        Please provide a detailed itinerary framework with:
        - Trip phases and sequences
        - Daily structure recommendations
        - Transition schedules
        - Optimization priorities
        - Flexibility points for user customization
        """
        
        return [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt}
        ]
    
    def _format_context(self, context: Dict[str, Any]) -> str:
        """Format context for planning agent"""
        # Format context specific to planning needs
        return f"""
        Destinations: {context.get('destinations', [])}
        Duration: {context.get('duration', 'Not specified')} days
        Travelers: {context.get('travelers', {})}
        Budget: ${context.get('budget', {}).get('total', 'Not specified')}
        Preferences: {context.get('preferences', [])}
        Constraints: {context.get('constraints', [])}
        """
    
    def _format_task_data(self, task_data: Dict[str, Any]) -> str:
        """Format task-specific data"""
        return f"""
        Request Type: {task_data.get('request_type', 'create_itinerary')}
        Special Requirements: {task_data.get('special_requirements', [])}
        Priority Level: {task_data.get('priority', 'standard')}
        """
```

## 🛠️ Development Environment Setup

### 1. Project Structure
```
travel-planner/
├── backend/
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── base_agent.py
│   │   ├── planning_agent.py
│   │   ├── location_agent.py
│   │   ├── transport_agent.py
│   │   ├── accommodation_agent.py
│   │   ├── activity_agent.py
│   │   ├── budget_agent.py
│   │   └── weather_agent.py
│   ├── orchestrator/
│   │   ├── __init__.py
│   │   ├── master_orchestrator.py
│   │   ├── task_delegator.py
│   │   ├── state_manager.py
│   │   └── communication_hub.py
│   ├── config/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── agents.py
│   │   └── settings.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── routes.py
│   │   ├── websocket.py
│   │   └── middleware.py
│   ├── database/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── migrations/
│   │   └── repositories.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── metrics.py
│   │   ├── context_manager.py
│   │   └── error_handlers.py
│   ├── tests/
│   │   ├── unit/
│   │   ├── integration/
│   │   └── load/
│   ├── requirements.txt
│   ├── Dockerfile
│   └── docker-compose.yml
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── TripGraph/
│   │   │   ├── AgentDashboard/
│   │   │   ├── ModificationPanel/
│   │   │   └── ConflictResolution/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── utils/
│   │   └── types/
│   ├── package.json
│   └── Dockerfile
├── docs/
│   └── agents/  # This documentation
├── scripts/
│   ├── setup-dev.sh
│   ├── run-tests.sh
│   └── deploy.sh
├── .env.example
├── .gitignore
├── docker-compose.yml
└── README.md
```

### 2. Quick Setup Script
```bash
#!/bin/bash
# scripts/setup-dev.sh

echo "Setting up Travel Planner Agent System Development Environment..."

# Check prerequisites
command -v python3 >/dev/null 2>&1 || { echo "Python 3.11+ required"; exit 1; }
command -v node >/dev/null 2>&1 || { echo "Node.js 18+ required"; exit 1; }
command -v docker >/dev/null 2>&1 || { echo "Docker required"; exit 1; }

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install backend dependencies
cd backend
pip install -r requirements.txt

# Setup database
docker-compose up -d postgres redis
sleep 10  # Wait for services to start

# Run database migrations
alembic upgrade head

# Install frontend dependencies
cd ../frontend
npm install

# Copy environment template
cp ../.env.example ../.env

echo "Development environment setup complete!"
echo "1. Update .env file with your API keys"
echo "2. Run 'docker-compose up' to start services"
echo "3. Run 'python backend/main.py' to start backend"
echo "4. Run 'npm start' in frontend/ to start frontend"
```

### 3. Testing Strategy

#### Unit Tests for Agents
```python
# tests/unit/test_planning_agent.py
import pytest
from unittest.mock import AsyncMock, patch
from agents.planning_agent import PlanningAgent
from config.models import ModelConfig, ModelTier

@pytest.fixture
def planning_agent():
    config = AgentConfig(
        agent_type="planning",
        model_config=ModelConfig(
            model_name="gpt-5",
            provider="openai",
            max_tokens=8192,
            cost_per_1k_tokens=1.25
        ),
        context_manager="default",
        timeout_seconds=30
    )
    return PlanningAgent(config)

@pytest.mark.asyncio
async def test_planning_agent_basic_trip(planning_agent):
    task_data = {
        "destinations": ["Tokyo", "Kyoto"],
        "duration": 7,
        "budget": 5000,
        "travelers": {"adults": 2}
    }
    
    with patch.object(planning_agent, '_execute_with_model') as mock_execute:
        mock_execute.return_value = {
            "itinerary_framework": {
                "total_duration": 7,
                "phases": [{"phase_id": "tokyo_exploration", "duration": 3}]
            },
            "prompt_tokens": 1500,
            "completion_tokens": 800
        }
        
        result = await planning_agent.process_task(task_data)
        
        assert "itinerary_framework" in result
        assert result["itinerary_framework"]["total_duration"] == 7
        mock_execute.assert_called_once()

@pytest.mark.asyncio
async def test_agent_retry_mechanism(planning_agent):
    task_data = {"destinations": ["Tokyo"]}
    
    with patch.object(planning_agent, '_execute_with_model') as mock_execute:
        # First two calls fail, third succeeds
        mock_execute.side_effect = [
            asyncio.TimeoutError(),
            Exception("API Error"),
            {"itinerary_framework": {"total_duration": 5}}
        ]
        
        result = await planning_agent.process_task(task_data)
        
        assert mock_execute.call_count == 3
        assert "itinerary_framework" in result
```

#### Integration Tests
```python
# tests/integration/test_orchestrator_flow.py
import pytest
from orchestrator.master_orchestrator import MasterOrchestrator

@pytest.mark.asyncio
async def test_full_planning_flow():
    orchestrator = MasterOrchestrator()
    
    request = {
        "destinations": ["Tokyo", "Kyoto"],
        "dates": {"start": "2024-03-15", "end": "2024-03-22"},
        "travelers": {"adults": 2},
        "budget": 5000,
        "preferences": ["cultural", "food"]
    }
    
    result = await orchestrator.process_trip_request(request)
    
    assert result["status"] == "success"
    assert "trip_plan" in result
    assert "agent_coordination" in result
    assert len(result["agent_coordination"]["agents_involved"]) >= 5

@pytest.mark.asyncio
async def test_model_configuration_override():
    # Test that environment variables override default model configurations
    with patch.dict(os.environ, {"PLANNING_AGENT_MODEL": "gpt-5-mini"}):
        orchestrator = MasterOrchestrator()
        planning_agent = orchestrator.get_agent("planning")
        
        assert planning_agent.config.model_config.model_name == "gpt-5-mini"
```

#### Load Testing
```python
# tests/load/test_agent_performance.py
import asyncio
import pytest
from concurrent.futures import ThreadPoolExecutor

@pytest.mark.asyncio
async def test_concurrent_agent_load():
    """Test system performance under concurrent agent requests"""
    orchestrator = MasterOrchestrator()
    
    # Create 50 concurrent trip requests
    requests = [
        {
            "destinations": [f"City{i}"],
            "duration": 5,
            "budget": 3000 + (i * 100)
        }
        for i in range(50)
    ]
    
    start_time = time.time()
    
    # Execute all requests concurrently
    results = await asyncio.gather(*[
        orchestrator.process_trip_request(req)
        for req in requests
    ])
    
    end_time = time.time()
    
    # Verify all requests succeeded
    success_count = sum(1 for r in results if r["status"] == "success")
    assert success_count >= 45  # 90% success rate minimum
    
    # Verify reasonable performance
    total_time = end_time - start_time
    assert total_time < 120  # Should complete within 2 minutes
    
    # Verify cost tracking
    total_cost = sum(r.get("cost", 0) for r in results)
    assert total_cost > 0  # Cost tracking working
```

## 🚀 Deployment & Production Setup

### 1. Docker Configuration

#### Backend Dockerfile
```dockerfile
# backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first for better caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd --create-home --shell /bin/bash agent_user
RUN chown -R agent_user:agent_user /app
USER agent_user

# Health check
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD python health_check.py

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### Production Docker Compose
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  # Agent Backend Services
  agent-orchestrator:
    build: ./backend
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/travel_planner
      - REDIS_URL=redis://redis:6379/0
      - ENVIRONMENT=production
    depends_on:
      - postgres
      - redis
    volumes:
      - ./backend/.env:/app/.env:ro
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 1G
          cpus: '0.5'
        reservations:
          memory: 512M
          cpus: '0.25'
  
  # Specialized Agent Workers
  planning-agent-worker:
    build: ./backend
    command: celery -A tasks worker --loglevel=info --queues=planning_tasks
    environment:
      - REDIS_URL=redis://redis:6379/0
      - AGENT_TYPE=planning
    depends_on:
      - redis
    deploy:
      replicas: 2
  
  transport-agent-worker:
    build: ./backend  
    command: celery -A tasks worker --loglevel=info --queues=transport_tasks
    environment:
      - REDIS_URL=redis://redis:6379/0
      - AGENT_TYPE=transport
    depends_on:
      - redis
    deploy:
      replicas: 2
  
  # Database Services
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: travel_planner
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: '1.0'
  
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.25'
  
  # Frontend
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - agent-orchestrator
    deploy:
      replicas: 2
  
  # Monitoring
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
  
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  postgres_data:
  redis_data:
  grafana_data:
```

### 2. Production Configuration Management

#### Environment-Specific Configurations
```python
# config/environments.py
import os
from typing import Dict, Any

class Environment:
    def __init__(self, name: str):
        self.name = name
        self._load_config()
    
    def _load_config(self):
        # Load base configuration
        self.config = self._get_base_config()
        
        # Override with environment-specific settings
        env_config = self._get_environment_config()
        self.config.update(env_config)
        
        # Override with environment variables
        self._apply_env_overrides()
    
    def _get_base_config(self) -> Dict[str, Any]:
        return {
            "agent_timeouts": {
                "planning": 30,
                "transport": 25,
                "location": 15,
                "accommodation": 20,
                "activity": 15,
                "budget": 8,
                "weather": 10
            },
            "model_configs": DEFAULT_MODEL_CONFIGS,
            "agent_model_mapping": AGENT_MODEL_MAPPING
        }
    
    def _get_environment_config(self) -> Dict[str, Any]:
        if self.name == "production":
            return {
                "debug": False,
                "log_level": "INFO",
                "agent_pool_sizes": {
                    "planning": 3,
                    "transport": 3,
                    "location": 5,
                    "accommodation": 4,
                    "activity": 5,
                    "budget": 2,
                    "weather": 2
                },
                "circuit_breaker_enabled": True,
                "metrics_enabled": True,
                "cost_alerts_enabled": True
            }
        elif self.name == "staging":
            return {
                "debug": False,
                "log_level": "DEBUG",
                "agent_pool_sizes": {
                    "planning": 1,
                    "transport": 1,
                    "location": 2,
                    "accommodation": 2,
                    "activity": 2,
                    "budget": 1,
                    "weather": 1
                },
                "circuit_breaker_enabled": True,
                "metrics_enabled": True
            }
        else:  # development
            return {
                "debug": True,
                "log_level": "DEBUG",
                "agent_pool_sizes": {agent: 1 for agent in AGENT_MODEL_MAPPING.keys()},
                "circuit_breaker_enabled": False,
                "metrics_enabled": False
            }
    
    def _apply_env_overrides(self):
        """Apply environment variable overrides"""
        for agent_type in AGENT_MODEL_MAPPING.keys():
            # Check for model overrides
            model_key = f"{agent_type.upper()}_AGENT_MODEL"
            if model_key in os.environ:
                self.config["agent_model_overrides"][agent_type] = {
                    "model_name": os.environ[model_key],
                    "provider": os.environ.get(f"{agent_type.upper()}_AGENT_PROVIDER", "openai"),
                    "max_tokens": int(os.environ.get(f"{agent_type.upper()}_AGENT_MAX_TOKENS", "4096"))
                }
```

### 3. Monitoring & Observability

#### Metrics Collection
```python
# utils/metrics.py
import time
import logging
from typing import Dict, Any
from prometheus_client import Counter, Histogram, Gauge

class AgentMetrics:
    def __init__(self, agent_type: str):
        self.agent_type = agent_type
        
        # Prometheus metrics
        self.request_count = Counter(
            'agent_requests_total',
            'Total agent requests',
            ['agent_type', 'status']
        )
        
        self.request_duration = Histogram(
            'agent_request_duration_seconds',
            'Agent request duration',
            ['agent_type']
        )
        
        self.token_usage = Counter(
            'agent_tokens_total',
            'Total tokens used',
            ['agent_type', 'token_type']
        )
        
        self.cost_tracking = Counter(
            'agent_cost_total',
            'Total cost incurred',
            ['agent_type']
        )
        
        self.active_requests = Gauge(
            'agent_active_requests',
            'Currently active requests',
            ['agent_type']
        )
    
    async def record_success(self, duration: float, result: Dict[str, Any]):
        self.request_count.labels(
            agent_type=self.agent_type,
            status='success'
        ).inc()
        
        self.request_duration.labels(
            agent_type=self.agent_type
        ).observe(duration)
        
        logging.info(f"Agent {self.agent_type} completed successfully in {duration:.2f}s")
    
    async def record_token_usage(self, prompt_tokens: int, completion_tokens: int, cost: float):
        self.token_usage.labels(
            agent_type=self.agent_type,
            token_type='prompt'
        ).inc(prompt_tokens)
        
        self.token_usage.labels(
            agent_type=self.agent_type,
            token_type='completion'  
        ).inc(completion_tokens)
        
        self.cost_tracking.labels(
            agent_type=self.agent_type
        ).inc(cost)
```

#### Health Checks
```python
# health_check.py
import asyncio
import sys
from agents.base_agent import AgentFactory
from orchestrator.master_orchestrator import MasterOrchestrator

async def health_check():
    """Comprehensive health check for agent system"""
    try:
        # Test orchestrator
        orchestrator = MasterOrchestrator()
        if not await orchestrator.health_check():
            return False
        
        # Test each agent type
        for agent_type in ['planning', 'transport', 'location', 'budget']:
            agent = AgentFactory.create_agent(agent_type)
            if not await agent.health_check():
                print(f"Health check failed for {agent_type} agent")
                return False
        
        # Test database connectivity
        if not await test_database_connection():
            return False
        
        # Test Redis connectivity
        if not await test_redis_connection():
            return False
        
        print("All health checks passed")
        return True
        
    except Exception as e:
        print(f"Health check failed: {e}")
        return False

if __name__ == "__main__":
    result = asyncio.run(health_check())
    sys.exit(0 if result else 1)
```

## 📊 Cost Optimization & Model Management

### 1. Dynamic Model Selection
```python
# utils/model_optimizer.py
class ModelOptimizer:
    def __init__(self):
        self.usage_patterns = {}
        self.cost_thresholds = {
            "daily_budget": 50.0,
            "hourly_budget": 5.0,
            "per_request_max": 1.0
        }
    
    async def select_optimal_model(self, agent_type: str, task_complexity: str, current_cost: float) -> ModelConfig:
        """Select the most cost-effective model for the task"""
        
        # Check if we're approaching cost limits
        if current_cost > self.cost_thresholds["hourly_budget"]:
            return self._get_cost_optimized_model(agent_type)
        
        # Check task complexity requirements
        if task_complexity == "high":
            return self._get_performance_optimized_model(agent_type)
        elif task_complexity == "low":
            return self._get_efficiency_optimized_model(agent_type)
        
        # Default to configured model
        return self._get_default_model(agent_type)
    
    def _get_cost_optimized_model(self, agent_type: str) -> ModelConfig:
        """Return the most cost-effective model that can handle the agent type"""
        cost_models = {
            "planning": ModelConfig(model_name="gpt-5-mini", provider="openai", max_tokens=6144, cost_per_1k_tokens=0.25),
            "transport": ModelConfig(model_name="gpt-5-mini", provider="openai", max_tokens=6144, cost_per_1k_tokens=0.25),
            "location": ModelConfig(model_name="gpt-5-nano", provider="openai", max_tokens=4096, cost_per_1k_tokens=0.05),
            "budget": ModelConfig(model_name="gpt-5-nano", provider="openai", max_tokens=2048, cost_per_1k_tokens=0.05),
            "weather": ModelConfig(model_name="gpt-5-nano", provider="openai", max_tokens=1024, cost_per_1k_tokens=0.05)
        }
        return cost_models.get(agent_type, cost_models["budget"])
```

## 🔧 Troubleshooting & Common Issues

### 1. Model Configuration Issues
```python
# utils/diagnostics.py
async def diagnose_model_config():
    """Diagnose common model configuration issues"""
    issues = []
    
    # Check API keys
    required_keys = ["OPENAI_API_KEY"]
    for key in required_keys:
        if not os.getenv(key):
            issues.append(f"Missing API key: {key}")
    
    # Check model availability
    for agent_type, model_tier in AGENT_MODEL_MAPPING.items():
        config = get_model_config(agent_type)
        if not await test_model_connectivity(config):
            issues.append(f"Cannot connect to model for {agent_type} agent")
    
    # Check token limits
    for agent_type in AGENT_MODEL_MAPPING.keys():
        config = get_model_config(agent_type)
        if config.max_tokens > 8192:
            issues.append(f"Token limit too high for {agent_type}: {config.max_tokens}")
    
    return issues
```

---

**Implementation Roadmap:**
1. **Week 1-2**: Core agent infrastructure and model configuration system
2. **Week 3-4**: Individual agent implementations and testing
3. **Week 5-6**: Orchestrator and communication layer
4. **Week 7-8**: Frontend integration and UI development  
5. **Week 9-10**: Testing, optimization, and deployment preparation
6. **Week 11-12**: Production deployment and monitoring setup

This implementation guide provides a complete foundation for building the configurable multi-agent Travel Planner system with flexible model selection and comprehensive monitoring capabilities.