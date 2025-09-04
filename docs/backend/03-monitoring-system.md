# Monitoring & Cost Tracking Architecture

## 🎯 Comprehensive Monitoring Strategy

Our monitoring system provides end-to-end visibility into agent performance, cost optimization, and system health, specifically designed for production deployment on Oracle Cloud ARM infrastructure.

## 📊 Multi-Layer Monitoring Stack

### 1. LangSmith Integration (AI-Specific Monitoring)

```python
# app/core/langsmith_monitor.py
from langsmith import Client, RunTree
from langsmith.run_helpers import trace
from typing import Dict, Any, Optional
import os
import asyncio
import time

class LangSmithMonitor:
    """LangSmith integration for AI agent monitoring"""
    
    def __init__(self):
        self.client = Client() if os.getenv("LANGSMITH_API_KEY") else None
        self.project_name = os.getenv("LANGSMITH_PROJECT", "travel-planner-agents")
        self.enabled = bool(self.client)
    
    @trace(name="travel_planning_session")
    async def monitor_planning_session(
        self,
        user_id: str,
        plan_id: str,
        session_data: Dict[str, Any]
    ) -> RunTree:
        """Monitor complete planning session with LangSmith"""
        
        if not self.enabled:
            return None
            
        session_run = RunTree(
            name="travel_planning_session",
            inputs={
                "user_id": user_id,
                "plan_id": plan_id,
                "destinations": session_data.get("destinations"),
                "duration": session_data.get("duration"),
                "budget": session_data.get("budget"),
                "travelers": session_data.get("travelers")
            },
            project_name=self.project_name,
            tags=["travel-planning", "session"]
        )
        
        return session_run
    
    @trace(name="agent_execution")
    async def monitor_agent_execution(
        self,
        agent_type: str,
        task_data: Dict[str, Any],
        model_config: Dict[str, Any],
        parent_run: Optional[RunTree] = None
    ) -> RunTree:
        """Monitor individual agent execution"""
        
        if not self.enabled:
            return None
            
        agent_run = RunTree(
            name=f"{agent_type}_agent_execution",
            inputs={
                "agent_type": agent_type,
                "task_data": task_data,
                "model_config": model_config
            },
            parent_run=parent_run,
            tags=[agent_type, "agent-execution", model_config.get("model_name")]
        )
        
        return agent_run
    
    async def record_agent_success(
        self,
        run: RunTree,
        result: Dict[str, Any],
        metrics: Dict[str, Any]
    ):
        """Record successful agent execution"""
        
        if not self.enabled or not run:
            return
            
        run.end(
            outputs=result,
            extra={
                "execution_time_ms": metrics.get("execution_time_ms"),
                "prompt_tokens": metrics.get("prompt_tokens"),
                "completion_tokens": metrics.get("completion_tokens"),
                "total_cost": metrics.get("cost_usd"),
                "model_provider": metrics.get("model_provider"),
                "success": True
            }
        )
        
        run.post()
    
    async def record_agent_error(
        self,
        run: RunTree,
        error: Exception,
        metrics: Dict[str, Any]
    ):
        """Record agent execution error"""
        
        if not self.enabled or not run:
            return
            
        run.end(
            error=str(error),
            extra={
                "execution_time_ms": metrics.get("execution_time_ms"),
                "retry_count": metrics.get("retry_count"),
                "error_type": type(error).__name__,
                "success": False
            }
        )
        
        run.post()
    
    async def get_session_analytics(
        self,
        session_id: str,
        days: int = 7
    ) -> Dict[str, Any]:
        """Get analytics for a specific session"""
        
        if not self.enabled:
            return {}
            
        # Query LangSmith for session analytics
        runs = self.client.list_runs(
            project_name=self.project_name,
            filter=f"has_tag('session') and extra.session_id = '{session_id}'",
            limit=1000
        )
        
        analytics = {
            "total_runs": len(runs),
            "successful_runs": len([r for r in runs if not r.error]),
            "failed_runs": len([r for r in runs if r.error]),
            "total_cost": sum(r.extra.get("total_cost", 0) for r in runs),
            "total_tokens": sum(r.extra.get("prompt_tokens", 0) + r.extra.get("completion_tokens", 0) for r in runs),
            "avg_execution_time": sum(r.extra.get("execution_time_ms", 0) for r in runs) / len(runs) if runs else 0
        }
        
        return analytics
```

### 2. Prometheus Metrics Collection

```python
# app/core/prometheus_metrics.py
from prometheus_client import Counter, Histogram, Gauge, Summary, Info
from typing import Dict, Any
import time

class PrometheusMetrics:
    """Prometheus metrics for travel planner monitoring"""
    
    def __init__(self):
        # Agent execution metrics
        self.agent_requests_total = Counter(
            'travel_planner_agent_requests_total',
            'Total agent requests',
            ['agent_type', 'status', 'model_provider', 'model_name']
        )
        
        self.agent_execution_duration = Histogram(
            'travel_planner_agent_execution_duration_seconds',
            'Agent execution duration in seconds',
            ['agent_type', 'model_provider'],
            buckets=(0.1, 0.5, 1.0, 2.5, 5.0, 10.0, 30.0, 60.0, float('inf'))
        )
        
        self.agent_token_usage = Counter(
            'travel_planner_agent_tokens_total',
            'Total tokens used by agents',
            ['agent_type', 'token_type', 'model_provider', 'model_name']
        )
        
        self.agent_cost_total = Counter(
            'travel_planner_agent_cost_total',
            'Total cost incurred by agents',
            ['agent_type', 'model_provider', 'model_name']
        )
        
        # System health metrics
        self.active_sessions = Gauge(
            'travel_planner_active_sessions',
            'Number of active user sessions'
        )
        
        self.database_connections = Gauge(
            'travel_planner_database_connections',
            'Number of active database connections'
        )
        
        self.cache_hit_rate = Gauge(
            'travel_planner_cache_hit_rate',
            'Cache hit rate percentage'
        )
        
        # Plan creation metrics
        self.plans_created_total = Counter(
            'travel_planner_plans_created_total',
            'Total travel plans created',
            ['user_tier', 'plan_complexity']
        )
        
        self.plan_completion_time = Histogram(
            'travel_planner_plan_completion_duration_seconds',
            'Time to complete a travel plan',
            ['plan_complexity', 'agent_count'],
            buckets=(30, 60, 120, 300, 600, 1200, 3600, float('inf'))
        )
        
        # Cost optimization metrics
        self.cost_savings_total = Counter(
            'travel_planner_cost_savings_total',
            'Total cost savings from optimization',
            ['optimization_type']
        )
        
        self.model_switches_total = Counter(
            'travel_planner_model_switches_total',
            'Total model switches for cost optimization',
            ['from_model', 'to_model', 'reason']
        )
        
        # Error metrics
        self.errors_total = Counter(
            'travel_planner_errors_total',
            'Total errors by type',
            ['error_type', 'component', 'severity']
        )
        
        self.retry_attempts_total = Counter(
            'travel_planner_retry_attempts_total',
            'Total retry attempts',
            ['component', 'reason']
        )
        
        # Business metrics
        self.user_satisfaction = Histogram(
            'travel_planner_user_satisfaction',
            'User satisfaction scores',
            ['plan_type'],
            buckets=(1, 2, 3, 4, 5)
        )
        
        self.revenue_total = Counter(
            'travel_planner_revenue_total',
            'Total revenue by subscription tier',
            ['subscription_tier']
        )
    
    def record_agent_execution(
        self,
        agent_type: str,
        duration: float,
        status: str,
        model_provider: str,
        model_name: str,
        prompt_tokens: int,
        completion_tokens: int,
        cost: float
    ):
        """Record agent execution metrics"""
        
        # Record request
        self.agent_requests_total.labels(
            agent_type=agent_type,
            status=status,
            model_provider=model_provider,
            model_name=model_name
        ).inc()
        
        # Record duration
        self.agent_execution_duration.labels(
            agent_type=agent_type,
            model_provider=model_provider
        ).observe(duration)
        
        # Record token usage
        self.agent_token_usage.labels(
            agent_type=agent_type,
            token_type='prompt',
            model_provider=model_provider,
            model_name=model_name
        ).inc(prompt_tokens)
        
        self.agent_token_usage.labels(
            agent_type=agent_type,
            token_type='completion',
            model_provider=model_provider,
            model_name=model_name
        ).inc(completion_tokens)
        
        # Record cost
        self.agent_cost_total.labels(
            agent_type=agent_type,
            model_provider=model_provider,
            model_name=model_name
        ).inc(cost)
    
    def record_plan_creation(
        self,
        user_tier: str,
        plan_complexity: str,
        completion_time: float,
        agent_count: int
    ):
        """Record plan creation metrics"""
        
        self.plans_created_total.labels(
            user_tier=user_tier,
            plan_complexity=plan_complexity
        ).inc()
        
        self.plan_completion_time.labels(
            plan_complexity=plan_complexity,
            agent_count=str(agent_count)
        ).observe(completion_time)
    
    def record_cost_optimization(
        self,
        optimization_type: str,
        savings: float,
        from_model: str = None,
        to_model: str = None
    ):
        """Record cost optimization metrics"""
        
        self.cost_savings_total.labels(
            optimization_type=optimization_type
        ).inc(savings)
        
        if from_model and to_model:
            self.model_switches_total.labels(
                from_model=from_model,
                to_model=to_model,
                reason=optimization_type
            ).inc()
```

### 3. Custom Cost Tracking Service

```python
# app/services/cost_service.py
from typing import Dict, Any, List, Optional
from datetime import datetime, timedelta
from sqlalchemy.orm import Session
from app.models.database import CostTracking, AgentExecution, User
from app.core.database import get_db
import logging

class CostTrackingService:
    """Comprehensive cost tracking and optimization service"""
    
    def __init__(self, db: Session):
        self.db = db
        self.logger = logging.getLogger(__name__)
    
    async def record_cost(
        self,
        user_id: str,
        plan_id: str,
        agent_execution_id: str,
        model_provider: str,
        model_name: str,
        prompt_tokens: int,
        completion_tokens: int,
        cost_usd: float,
        session_id: str = None,
        metadata: Dict[str, Any] = None
    ) -> str:
        """Record cost data in database"""
        
        cost_record = CostTracking(
            user_id=user_id,
            plan_id=plan_id,
            agent_execution_id=agent_execution_id,
            model_provider=model_provider,
            model_name=model_name,
            prompt_tokens=prompt_tokens,
            completion_tokens=completion_tokens,
            total_tokens=prompt_tokens + completion_tokens,
            total_cost_usd=cost_usd,
            session_id=session_id,
            metadata=metadata or {}
        )
        
        self.db.add(cost_record)
        self.db.commit()
        
        # Check for cost alerts
        await self._check_cost_alerts(user_id, cost_usd)
        
        return cost_record.id
    
    async def get_user_cost_summary(
        self,
        user_id: str,
        period: str = "month",
        start_date: datetime = None,
        end_date: datetime = None
    ) -> Dict[str, Any]:
        """Get comprehensive cost summary for user"""
        
        # Determine date range
        if not start_date:
            if period == "day":
                start_date = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)
            elif period == "week":
                start_date = datetime.now() - timedelta(days=7)
            elif period == "month":
                start_date = datetime.now().replace(day=1, hour=0, minute=0, second=0, microsecond=0)
            else:
                start_date = datetime.now() - timedelta(days=30)
        
        if not end_date:
            end_date = datetime.now()
        
        # Query cost data
        costs = self.db.query(CostTracking).filter(
            CostTracking.user_id == user_id,
            CostTracking.created_at >= start_date,
            CostTracking.created_at <= end_date
        ).all()
        
        if not costs:
            return {
                "total_cost": 0.0,
                "total_tokens": 0,
                "total_requests": 0,
                "avg_cost_per_request": 0.0,
                "cost_by_agent": {},
                "cost_by_model": {},
                "cost_trend": []
            }
        
        # Calculate summary statistics
        total_cost = sum(c.total_cost_usd for c in costs)
        total_tokens = sum(c.total_tokens for c in costs)
        total_requests = len(costs)
        
        # Cost breakdown by agent
        cost_by_agent = {}
        for cost in costs:
            agent_execution = self.db.query(AgentExecution).filter(
                AgentExecution.id == cost.agent_execution_id
            ).first()
            
            if agent_execution:
                agent_type = agent_execution.agent_type
                if agent_type not in cost_by_agent:
                    cost_by_agent[agent_type] = {
                        "cost": 0.0,
                        "tokens": 0,
                        "requests": 0
                    }
                
                cost_by_agent[agent_type]["cost"] += cost.total_cost_usd
                cost_by_agent[agent_type]["tokens"] += cost.total_tokens
                cost_by_agent[agent_type]["requests"] += 1
        
        # Cost breakdown by model
        cost_by_model = {}
        for cost in costs:
            model_key = f"{cost.model_provider}:{cost.model_name}"
            if model_key not in cost_by_model:
                cost_by_model[model_key] = {
                    "cost": 0.0,
                    "tokens": 0,
                    "requests": 0
                }
            
            cost_by_model[model_key]["cost"] += cost.total_cost_usd
            cost_by_model[model_key]["tokens"] += cost.total_tokens
            cost_by_model[model_key]["requests"] += 1
        
        # Daily cost trend
        cost_trend = self._calculate_cost_trend(costs, start_date, end_date)
        
        return {
            "total_cost": round(total_cost, 6),
            "total_tokens": total_tokens,
            "total_requests": total_requests,
            "avg_cost_per_request": round(total_cost / total_requests, 6) if total_requests > 0 else 0.0,
            "cost_by_agent": cost_by_agent,
            "cost_by_model": cost_by_model,
            "cost_trend": cost_trend,
            "period": period,
            "start_date": start_date.isoformat(),
            "end_date": end_date.isoformat()
        }
    
    async def get_cost_optimization_suggestions(
        self,
        user_id: str
    ) -> List[Dict[str, Any]]:
        """Generate cost optimization suggestions"""
        
        suggestions = []
        
        # Get recent usage patterns
        recent_costs = self.db.query(CostTracking).filter(
            CostTracking.user_id == user_id,
            CostTracking.created_at >= datetime.now() - timedelta(days=7)
        ).all()
        
        if not recent_costs:
            return suggestions
        
        # Analyze model usage efficiency
        model_usage = {}
        for cost in recent_costs:
            agent_execution = self.db.query(AgentExecution).filter(
                AgentExecution.id == cost.agent_execution_id
            ).first()
            
            if agent_execution:
                key = (agent_execution.agent_type, cost.model_name)
                if key not in model_usage:
                    model_usage[key] = {
                        "cost": 0.0,
                        "requests": 0,
                        "avg_execution_time": 0,
                        "success_rate": 0
                    }
                
                model_usage[key]["cost"] += cost.total_cost_usd
                model_usage[key]["requests"] += 1
                
                if agent_execution.execution_time_ms:
                    model_usage[key]["avg_execution_time"] += agent_execution.execution_time_ms
                
                if agent_execution.status == "completed":
                    model_usage[key]["success_rate"] += 1
        
        # Calculate averages and identify optimization opportunities
        for (agent_type, model_name), usage in model_usage.items():
            avg_cost = usage["cost"] / usage["requests"]
            avg_time = usage["avg_execution_time"] / usage["requests"] if usage["requests"] > 0 else 0
            success_rate = usage["success_rate"] / usage["requests"] if usage["requests"] > 0 else 0
            
            # Suggest model downgrades for simple tasks
            if agent_type in ["budget", "weather"] and "gpt-5" in model_name:
                potential_savings = avg_cost * 0.8  # Estimate 80% savings
                suggestions.append({
                    "type": "model_downgrade",
                    "agent_type": agent_type,
                    "current_model": model_name,
                    "suggested_model": "gpt-5-nano",
                    "potential_savings_per_request": potential_savings,
                    "reason": f"Simple {agent_type} tasks can use more efficient models",
                    "confidence": 0.9
                })
            
            # Suggest optimization for slow, expensive operations
            if avg_cost > 0.5 and avg_time > 30000:  # >$0.50 and >30s
                suggestions.append({
                    "type": "performance_optimization",
                    "agent_type": agent_type,
                    "current_model": model_name,
                    "avg_cost": avg_cost,
                    "avg_time_ms": avg_time,
                    "reason": "High cost and slow execution detected",
                    "suggestions": [
                        "Consider reducing max_tokens",
                        "Optimize prompt structure",
                        "Use model with better cost/performance ratio"
                    ],
                    "confidence": 0.7
                })
        
        return suggestions
    
    async def _check_cost_alerts(self, user_id: str, current_cost: float):
        """Check if cost alerts should be triggered"""
        
        # Get user info for alert thresholds
        user = self.db.query(User).filter(User.id == user_id).first()
        if not user:
            return
        
        # Daily cost limit check
        today_start = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)
        daily_total = self.db.query(CostTracking).filter(
            CostTracking.user_id == user_id,
            CostTracking.created_at >= today_start
        ).with_entities(func.sum(CostTracking.total_cost_usd)).scalar() or 0.0
        
        # Alert thresholds based on subscription tier
        alert_thresholds = {
            "free": {"daily": 1.0, "request": 0.10},
            "basic": {"daily": 10.0, "request": 0.50},
            "premium": {"daily": 50.0, "request": 2.00}
        }
        
        thresholds = alert_thresholds.get(user.subscription_tier, alert_thresholds["free"])
        
        # Check daily limit
        if daily_total > thresholds["daily"]:
            await self._send_cost_alert(
                user_id, 
                "daily_limit_exceeded", 
                {
                    "daily_total": daily_total,
                    "limit": thresholds["daily"],
                    "tier": user.subscription_tier
                }
            )
        
        # Check per-request limit
        if current_cost > thresholds["request"]:
            await self._send_cost_alert(
                user_id,
                "request_limit_exceeded",
                {
                    "request_cost": current_cost,
                    "limit": thresholds["request"],
                    "tier": user.subscription_tier
                }
            )
    
    async def _send_cost_alert(
        self,
        user_id: str,
        alert_type: str,
        alert_data: Dict[str, Any]
    ):
        """Send cost alert to user"""
        
        self.logger.warning(f"Cost alert for user {user_id}: {alert_type} - {alert_data}")
        
        # TODO: Implement actual alert mechanism (email, push notification, etc.)
        # For now, just log the alert
        
    def _calculate_cost_trend(
        self,
        costs: List[CostTracking],
        start_date: datetime,
        end_date: datetime
    ) -> List[Dict[str, Any]]:
        """Calculate daily cost trend"""
        
        # Group costs by date
        daily_costs = {}
        current_date = start_date.date()
        end_date_only = end_date.date()
        
        # Initialize all dates with 0
        while current_date <= end_date_only:
            daily_costs[current_date] = 0.0
            current_date += timedelta(days=1)
        
        # Sum costs by date
        for cost in costs:
            cost_date = cost.created_at.date()
            if cost_date in daily_costs:
                daily_costs[cost_date] += cost.total_cost_usd
        
        # Convert to list format
        trend = []
        for date, cost in sorted(daily_costs.items()):
            trend.append({
                "date": date.isoformat(),
                "cost": round(cost, 6)
            })
        
        return trend
```

### 4. Health Check System

```python
# app/api/routes/health.py
from fastapi import APIRouter, Depends, HTTPException
from typing import Dict, Any
import asyncio
import time
import psutil
from app.core.database import get_db
from app.core.vector_store import TravelPlannerVectorStore
from app.core.monitoring import PrometheusMetrics

router = APIRouter(prefix="/health", tags=["health"])

class HealthChecker:
    """Comprehensive health checking for all system components"""
    
    def __init__(self):
        self.metrics = PrometheusMetrics()
    
    async def check_all_systems(self) -> Dict[str, Any]:
        """Perform comprehensive health check"""
        
        start_time = time.time()
        
        health_checks = {
            "database": self._check_database(),
            "redis": self._check_redis(),
            "vector_db": self._check_vector_db(),
            "external_apis": self._check_external_apis(),
            "system_resources": self._check_system_resources(),
            "agent_workers": self._check_agent_workers()
        }
        
        # Run all checks concurrently
        results = {}
        for name, check_coroutine in health_checks.items():
            try:
                results[name] = await asyncio.wait_for(check_coroutine, timeout=10.0)
            except asyncio.TimeoutError:
                results[name] = {
                    "status": "unhealthy",
                    "error": "Health check timeout",
                    "response_time_ms": 10000
                }
            except Exception as e:
                results[name] = {
                    "status": "unhealthy", 
                    "error": str(e),
                    "response_time_ms": None
                }
        
        # Calculate overall health
        total_time = time.time() - start_time
        all_healthy = all(r.get("status") == "healthy" for r in results.values())
        
        return {
            "status": "healthy" if all_healthy else "unhealthy",
            "timestamp": time.time(),
            "total_check_time_ms": int(total_time * 1000),
            "components": results
        }
    
    async def _check_database(self) -> Dict[str, Any]:
        """Check PostgreSQL database connectivity"""
        start_time = time.time()
        
        try:
            db = next(get_db())
            
            # Simple connectivity test
            result = db.execute("SELECT 1").scalar()
            
            # Check connection pool status
            pool_info = db.get_bind().pool.status() if hasattr(db.get_bind(), 'pool') else "unknown"
            
            response_time = int((time.time() - start_time) * 1000)
            
            return {
                "status": "healthy" if result == 1 else "unhealthy",
                "response_time_ms": response_time,
                "pool_info": pool_info,
                "details": "Database connectivity verified"
            }
            
        except Exception as e:
            return {
                "status": "unhealthy",
                "error": str(e),
                "response_time_ms": int((time.time() - start_time) * 1000)
            }
    
    async def _check_redis(self) -> Dict[str, Any]:
        """Check Redis connectivity"""
        start_time = time.time()
        
        try:
            import redis
            r = redis.Redis.from_url(os.getenv("REDIS_URL"))
            
            # Test connectivity
            r.ping()
            
            # Get Redis info
            info = r.info()
            
            response_time = int((time.time() - start_time) * 1000)
            
            return {
                "status": "healthy",
                "response_time_ms": response_time,
                "memory_usage": info.get("used_memory_human"),
                "connected_clients": info.get("connected_clients"),
                "details": "Redis connectivity verified"
            }
            
        except Exception as e:
            return {
                "status": "unhealthy",
                "error": str(e),
                "response_time_ms": int((time.time() - start_time) * 1000)
            }
    
    async def _check_vector_db(self) -> Dict[str, Any]:
        """Check Qdrant vector database"""
        start_time = time.time()
        
        try:
            vector_store = TravelPlannerVectorStore()
            
            # Test connectivity by getting collection info
            collections = vector_store.client.get_collections()
            
            response_time = int((time.time() - start_time) * 1000)
            
            return {
                "status": "healthy",
                "response_time_ms": response_time,
                "collections_count": len(collections.collections),
                "details": "Vector database connectivity verified"
            }
            
        except Exception as e:
            return {
                "status": "unhealthy",
                "error": str(e),
                "response_time_ms": int((time.time() - start_time) * 1000)
            }
    
    async def _check_external_apis(self) -> Dict[str, Any]:
        """Check external API connectivity"""
        start_time = time.time()
        
        try:
            # Test OpenAI API
            from openai import AsyncOpenAI
            client = AsyncOpenAI(api_key=os.getenv("OPENAI_API_KEY"))
            
            # Simple test - list models
            models = await client.models.list()
            
            response_time = int((time.time() - start_time) * 1000)
            
            return {
                "status": "healthy",
                "response_time_ms": response_time,
                "available_models": len(models.data),
                "details": "External APIs accessible"
            }
            
        except Exception as e:
            return {
                "status": "degraded",  # External APIs not critical for basic operation
                "error": str(e),
                "response_time_ms": int((time.time() - start_time) * 1000)
            }
    
    async def _check_system_resources(self) -> Dict[str, Any]:
        """Check system resource utilization (ARM64 optimized)"""
        
        try:
            # Get CPU usage
            cpu_percent = psutil.cpu_percent(interval=1)
            
            # Get memory usage
            memory = psutil.virtual_memory()
            
            # Get disk usage
            disk = psutil.disk_usage('/')
            
            # ARM64 specific optimizations
            cpu_count = psutil.cpu_count()
            load_avg = psutil.getloadavg() if hasattr(psutil, 'getloadavg') else [0, 0, 0]
            
            # Determine health status
            status = "healthy"
            warnings = []
            
            if cpu_percent > 80:
                status = "degraded"
                warnings.append("High CPU usage")
            
            if memory.percent > 85:
                status = "degraded" 
                warnings.append("High memory usage")
            
            if disk.percent > 90:
                status = "degraded"
                warnings.append("High disk usage")
            
            if load_avg[0] > cpu_count * 2:
                status = "degraded"
                warnings.append("High system load")
            
            return {
                "status": status,
                "cpu_percent": cpu_percent,
                "memory_percent": memory.percent,
                "memory_available_gb": round(memory.available / (1024**3), 2),
                "disk_percent": disk.percent,
                "disk_free_gb": round(disk.free / (1024**3), 2),
                "cpu_count": cpu_count,
                "load_average": load_avg,
                "warnings": warnings
            }
            
        except Exception as e:
            return {
                "status": "unknown",
                "error": str(e)
            }
    
    async def _check_agent_workers(self) -> Dict[str, Any]:
        """Check Celery agent worker status"""
        
        try:
            from celery import Celery
            
            app = Celery('travel_planner_agents')
            app.config_from_object('app.core.celery_config')
            
            # Get worker statistics
            inspect = app.control.inspect()
            
            active_workers = inspect.active()
            worker_stats = inspect.stats()
            
            if not active_workers:
                return {
                    "status": "unhealthy",
                    "error": "No active workers found",
                    "worker_count": 0
                }
            
            total_workers = len(active_workers)
            total_active_tasks = sum(len(tasks) for tasks in active_workers.values())
            
            return {
                "status": "healthy",
                "worker_count": total_workers,
                "active_tasks": total_active_tasks,
                "worker_details": worker_stats
            }
            
        except Exception as e:
            return {
                "status": "degraded",
                "error": str(e),
                "worker_count": 0
            }

@router.get("/")
async def health_check():
    """Basic health check endpoint"""
    checker = HealthChecker()
    return await checker.check_all_systems()

@router.get("/quick")
async def quick_health_check():
    """Quick health check for load balancers"""
    return {"status": "healthy", "timestamp": time.time()}

@router.get("/detailed")
async def detailed_health_check():
    """Detailed health check with metrics"""
    checker = HealthChecker()
    health_data = await checker.check_all_systems()
    
    # Add additional metrics
    health_data["version"] = os.getenv("APP_VERSION", "unknown")
    health_data["environment"] = os.getenv("ENVIRONMENT", "unknown")
    health_data["deployment_id"] = os.getenv("DEPLOYMENT_ID", "unknown")
    
    return health_data
```

### 5. Grafana Dashboard Configuration

```yaml
# monitoring/grafana/dashboards/travel-planner-dashboard.json
{
  "dashboard": {
    "title": "Travel Planner Agent Monitoring",
    "panels": [
      {
        "title": "Agent Execution Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(travel_planner_agent_requests_total[5m])",
            "legendFormat": "{{agent_type}} - {{status}}"
          }
        ]
      },
      {
        "title": "Cost per Hour",
        "type": "graph", 
        "targets": [
          {
            "expr": "rate(travel_planner_agent_cost_total[1h])",
            "legendFormat": "{{agent_type}} - {{model_provider}}"
          }
        ]
      },
      {
        "title": "Token Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(travel_planner_agent_tokens_total[5m])",
            "legendFormat": "{{agent_type}} - {{token_type}}"
          }
        ]
      },
      {
        "title": "System Resources (ARM64)",
        "type": "graph",
        "targets": [
          {
            "expr": "100 - (avg(rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "legendFormat": "CPU Usage %"
          },
          {
            "expr": "100 * (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)",
            "legendFormat": "Memory Usage %"
          }
        ]
      }
    ]
  }
}
```

This comprehensive monitoring system provides:

✅ **AI-Specific Monitoring** - LangSmith integration for agent tracing  
✅ **Infrastructure Monitoring** - Prometheus + Grafana for system metrics  
✅ **Cost Optimization** - Real-time cost tracking and optimization suggestions  
✅ **Health Monitoring** - Comprehensive health checks for all components  
✅ **ARM64 Optimization** - Tailored for Oracle Cloud Ampere A1 infrastructure  
✅ **Business Intelligence** - User satisfaction and revenue tracking  
✅ **Alerting** - Proactive alerts for cost limits and system issues

The system is designed to provide complete visibility into both technical performance and business metrics, enabling data-driven optimization of the multi-agent travel planning system.