# Backend Database Integration

## 🎯 Database Integration Overview

The backend handles database integration through a well-structured ORM layer that connects to our dual database architecture (PostgreSQL + Qdrant + Redis). This document covers backend-specific database integration patterns.

> **Note**: Complete database schemas and architecture details are now documented in the [`../database/`](../database/) folder. This document focuses on backend implementation patterns.

## 🔗 Database Connection Architecture

### SQLAlchemy Models Structure

```python
# app/models/base.py
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, DateTime, Boolean
from sqlalchemy.dialects.postgresql import UUID
import uuid
from datetime import datetime

Base = declarative_base()

class BaseModel(Base):
    """Base model with common fields"""
    __abstract__ = True
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    deleted_at = Column(DateTime, nullable=True)
    is_active = Column(Boolean, default=True)
```

### Database Service Layer

```python
# app/services/database.py
from sqlalchemy.orm import Session
from app.core.database import get_db
from app.core.vector_database import vector_db
from app.core.cache import cache
from typing import Optional, List, Dict, Any
import logging

logger = logging.getLogger(__name__)

class DatabaseService:
    """Unified database service for PostgreSQL, Qdrant, and Redis operations"""
    
    def __init__(self, db: Session):
        self.db = db
        self.vector_db = vector_db
        self.cache = cache
    
    async def create_travel_plan(self, user_id: str, plan_data: dict) -> dict:
        """Create travel plan with vector embedding sync"""
        try:
            # Create in PostgreSQL
            plan = TravelPlan(user_id=user_id, **plan_data)
            self.db.add(plan)
            self.db.commit()
            
            # Sync to vector database
            await self._sync_plan_to_vector_db(plan.id, plan_data)
            
            # Cache for quick access
            self.cache.set(f"plan:{plan.id}", plan_data, ttl=3600)
            
            return {"id": plan.id, "status": "created"}
            
        except Exception as e:
            self.db.rollback()
            logger.error(f"Failed to create travel plan: {e}")
            raise
    
    async def _sync_plan_to_vector_db(self, plan_id: str, plan_data: dict):
        """Sync travel plan to vector database for semantic search"""
        # This would generate embeddings and store in Qdrant
        # Implementation details in vector database documentation
        pass
```

## 🔄 Agent Integration Patterns

### Agent Execution Tracking

```python
# app/services/agent_service.py
from app.models.agent_execution import AgentExecution
from app.models.cost_tracking import CostTracking
from datetime import datetime
import uuid

class AgentService:
    """Service for managing agent executions and costs"""
    
    def __init__(self, db: Session):
        self.db = db
    
    async def track_agent_execution(
        self, 
        plan_id: str, 
        user_id: str, 
        agent_type: str,
        task_data: dict,
        model_config: dict
    ) -> str:
        """Track agent execution with cost monitoring"""
        
        execution_id = str(uuid.uuid4())
        
        # Create execution record
        execution = AgentExecution(
            id=execution_id,
            plan_id=plan_id,
            user_id=user_id,
            agent_type=agent_type,
            agent_version="1.0",
            model_provider=model_config["provider"],
            model_name=model_config["model"],
            task_data=task_data,
            status="pending",
            created_at=datetime.utcnow()
        )
        
        self.db.add(execution)
        self.db.commit()
        
        return execution_id
    
    async def complete_agent_execution(
        self,
        execution_id: str,
        result_data: dict,
        performance_metrics: dict,
        cost_data: dict
    ):
        """Complete agent execution and track costs"""
        
        # Update execution record
        execution = self.db.query(AgentExecution).filter(
            AgentExecution.id == execution_id
        ).first()
        
        if execution:
            execution.result_data = result_data
            execution.execution_time_ms = performance_metrics.get("execution_time_ms")
            execution.prompt_tokens = performance_metrics.get("prompt_tokens", 0)
            execution.completion_tokens = performance_metrics.get("completion_tokens", 0)
            execution.total_tokens = performance_metrics.get("total_tokens", 0)
            execution.status = "completed"
            execution.completed_at = datetime.utcnow()
            
            # Track costs
            cost_record = CostTracking(
                user_id=execution.user_id,
                plan_id=execution.plan_id,
                agent_execution_id=execution_id,
                model_provider=execution.model_provider,
                model_name=execution.model_name,
                prompt_tokens=execution.prompt_tokens,
                completion_tokens=execution.completion_tokens,
                total_cost_usd=cost_data.get("total_cost_usd", 0.0),
                usage_category=cost_data.get("category", "planning")
            )
            
            self.db.add(cost_record)
            self.db.commit()
```

## 📊 Performance Optimization Patterns

### Query Optimization

```python
# app/services/query_optimization.py
from sqlalchemy import and_, or_, func
from sqlalchemy.orm import joinedload, selectinload
from app.models import TravelPlan, AgentExecution, CostTracking

class QueryService:
    """Optimized database queries for performance"""
    
    def __init__(self, db: Session):
        self.db = db
    
    def get_user_plans_optimized(self, user_id: str, limit: int = 20):
        """Optimized query for user plans with related data"""
        return self.db.query(TravelPlan)\
            .options(
                selectinload(TravelPlan.agent_executions),
                selectinload(TravelPlan.cost_tracking)
            )\
            .filter(
                and_(
                    TravelPlan.user_id == user_id,
                    TravelPlan.deleted_at.is_(None),
                    TravelPlan.is_active == True
                )
            )\
            .order_by(TravelPlan.updated_at.desc())\
            .limit(limit)\
            .all()
    
    def get_cost_analytics(self, user_id: str, days: int = 30):
        """Get cost analytics with proper aggregation"""
        return self.db.query(
            func.date_trunc('day', CostTracking.created_at).label('date'),
            func.sum(CostTracking.total_cost_usd).label('daily_cost'),
            func.sum(CostTracking.total_tokens).label('daily_tokens'),
            func.count(CostTracking.id).label('request_count')
        )\
        .filter(
            and_(
                CostTracking.user_id == user_id,
                CostTracking.created_at >= func.now() - func.interval(f'{days} days')
            )
        )\
        .group_by(func.date_trunc('day', CostTracking.created_at))\
        .order_by('date')\
        .all()
```

### Caching Strategies

```python
# app/services/cache_service.py
from app.core.cache import cache
import json
from typing import Optional, Dict, Any

class CacheService:
    """Intelligent caching for frequently accessed data"""
    
    @staticmethod
    def get_cached_plan(plan_id: str) -> Optional[Dict[Any, Any]]:
        """Get cached travel plan"""
        return cache.get(f"plan:{plan_id}")
    
    @staticmethod
    def cache_plan(plan_id: str, plan_data: dict, ttl: int = 3600):
        """Cache travel plan data"""
        cache.set(f"plan:{plan_id}", plan_data, ttl)
    
    @staticmethod
    def get_cached_user_preferences(user_id: str) -> Optional[Dict[Any, Any]]:
        """Get cached user preferences"""
        return cache.get(f"user_prefs:{user_id}")
    
    @staticmethod
    def cache_search_results(query_hash: str, results: list, ttl: int = 600):
        """Cache search results from vector database"""
        cache.set(f"search:{query_hash}", results, ttl)
    
    @staticmethod
    def invalidate_user_cache(user_id: str):
        """Invalidate all user-related cache"""
        # Implementation would depend on Redis pattern matching
        patterns = [
            f"user_prefs:{user_id}",
            f"user_plans:{user_id}",
            f"user_costs:{user_id}"
        ]
        for pattern in patterns:
            cache.delete(pattern)
```

## 🔐 Security Integration

### Row Level Security Implementation

```python
# app/core/security.py
from sqlalchemy import event
from sqlalchemy.engine import Engine
from app.core.database import engine

@event.listens_for(Engine, "connect")
def set_user_context(dbapi_connection, connection_record):
    """Set user context for row-level security"""
    # This would be called with each request to set the current user
    # for PostgreSQL's row-level security policies
    pass

class DatabaseSecurity:
    """Database security utilities"""
    
    @staticmethod
    def set_current_user(db: Session, user_id: str):
        """Set current user for RLS policies"""
        db.execute(f"SET app.current_user_id = '{user_id}'")
    
    @staticmethod
    def ensure_user_access(db: Session, user_id: str, resource_id: str, resource_type: str) -> bool:
        """Verify user has access to resource"""
        if resource_type == "travel_plan":
            plan = db.query(TravelPlan).filter(
                and_(
                    TravelPlan.id == resource_id,
                    TravelPlan.user_id == user_id,
                    TravelPlan.deleted_at.is_(None)
                )
            ).first()
            return plan is not None
        
        return False
```

## 📈 Monitoring and Health Checks

### Database Health Monitoring

```python
# app/api/health.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.core.database import get_db
from app.core.vector_database import vector_db
from app.core.cache import cache
from app.core.health import get_system_health

router = APIRouter()

@router.get("/health/database")
async def database_health(db: Session = Depends(get_db)):
    """Comprehensive database health check"""
    try:
        health_status = await get_system_health()
        return health_status
    except Exception as e:
        raise HTTPException(status_code=503, detail=f"Database health check failed: {str(e)}")

@router.get("/health/detailed")
async def detailed_health(db: Session = Depends(get_db)):
    """Detailed health check with metrics"""
    try:
        # Check connection pool status
        pool_status = {
            "size": db.bind.pool.size(),
            "checked_in": db.bind.pool.checkedin(),
            "overflow": db.bind.pool.overflow(),
            "checked_out": db.bind.pool.checkedout()
        }
        
        # Check recent performance
        recent_executions = db.query(AgentExecution)\
            .filter(AgentExecution.created_at >= func.now() - func.interval('1 hour'))\
            .count()
        
        avg_execution_time = db.query(func.avg(AgentExecution.execution_time_ms))\
            .filter(AgentExecution.created_at >= func.now() - func.interval('1 hour'))\
            .scalar() or 0
        
        return {
            "status": "healthy",
            "pool_status": pool_status,
            "performance": {
                "recent_executions": recent_executions,
                "avg_execution_time_ms": float(avg_execution_time)
            },
            "timestamp": datetime.utcnow().isoformat()
        }
    except Exception as e:
        raise HTTPException(status_code=503, detail=str(e))
```

## 🔄 Migration Patterns

### Database Migration Management

```python
# app/core/migrations.py
from alembic import command
from alembic.config import Config
from app.core.database import engine
import logging

logger = logging.getLogger(__name__)

class MigrationManager:
    """Database migration management"""
    
    def __init__(self):
        self.alembic_cfg = Config("alembic.ini")
    
    def run_migrations(self):
        """Run pending migrations"""
        try:
            command.upgrade(self.alembic_cfg, "head")
            logger.info("Database migrations completed successfully")
        except Exception as e:
            logger.error(f"Migration failed: {e}")
            raise
    
    def create_migration(self, message: str):
        """Create new migration"""
        try:
            command.revision(self.alembic_cfg, message=message, autogenerate=True)
            logger.info(f"Created migration: {message}")
        except Exception as e:
            logger.error(f"Failed to create migration: {e}")
            raise

# Startup migration check
def check_and_run_migrations():
    """Check and run migrations on startup"""
    migration_manager = MigrationManager()
    migration_manager.run_migrations()
```

## 📋 Backend Database Integration Checklist

### Implementation Tasks
- [ ] Set up SQLAlchemy models based on PostgreSQL schema
- [ ] Implement database service layer
- [ ] Create agent execution tracking service
- [ ] Set up caching strategies with Redis
- [ ] Implement vector database integration
- [ ] Configure row-level security
- [ ] Set up health monitoring endpoints
- [ ] Create migration management system
- [ ] Implement query optimization patterns
- [ ] Set up performance monitoring

### Testing Tasks
- [ ] Unit tests for database services
- [ ] Integration tests for multi-database operations
- [ ] Performance tests for query optimization
- [ ] Load tests for connection pooling
- [ ] Security tests for access control

---

**Integration Points**: 
- **[Database Architecture](../database/README.md)**: Complete schema and configuration details
- **[Agent System](../agents/README.md)**: Agent coordination and memory patterns
- **[Security](../security/README.md)**: Authentication and authorization integration

**Last Updated**: September 4, 2025  
**Version**: 2.0.0  
**Focus**: Backend database integration patterns