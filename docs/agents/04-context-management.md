# Context Management & Memory Optimization

## 🎯 Context Management Overview

The Travel Planner's multi-agent system employs **intelligent context management** to optimize token usage, reduce costs, and maintain high accuracy across different model tiers. Each agent receives precisely the context needed for optimal performance.

## 🧠 Context Architecture

### 1. Context Hierarchy

#### Global Context (Shared Across All Agents)
```json
{
  "trip_metadata": {
    "trip_id": "trip_12345",
    "user_id": "user_789",
    "creation_timestamp": "2024-03-15T10:00:00Z",
    "status": "planning|confirmed|active|completed"
  },
  "core_requirements": {
    "destinations": ["Tokyo", "Kyoto"],
    "dates": {"start": "2024-03-15", "end": "2024-03-22"},
    "travelers": {"adults": 2, "children": 1},
    "budget": {"total": 5000, "currency": "USD"}
  },
  "user_profile": {
    "preferences": ["cultural", "food", "nature"],
    "accessibility_needs": ["wheelchair_access"],
    "past_trips": ["europe_2023", "asia_2022"],
    "communication_style": "detailed|brief"
  }
}
```

#### Domain Context (Agent-Specific)
```json
{
  "planning_context": {
    "itinerary_structure": {...},
    "timing_constraints": {...},
    "priority_matrix": {...}
  },
  "location_context": {
    "destination_analysis": {...},
    "poi_database": {...},
    "routing_data": {...}
  },
  "transport_context": {
    "booking_history": {...},
    "route_options": {...},
    "real_time_data": {...}
  }
}
```

#### Task Context (Request-Specific)
```json
{
  "current_task": {
    "task_id": "task_456",
    "task_type": "hotel_search",
    "parameters": {
      "location": "Tokyo",
      "check_in": "2024-03-15",
      "nights": 3,
      "room_type": "double"
    },
    "dependencies": ["transport_confirmed", "budget_validated"],
    "urgency": "high|medium|low"
  }
}
```

### 2. Model-Tier Context Strategies

#### Tier 1 Agents (Premium Models - GPT-5)
**Token Allocation**: 6,000-8,192 tokens  
**Context Strategy**: Rich, comprehensive context

```python
class Tier1ContextManager:
    def prepare_context(self, agent_type, task_data):
        context = {
            "global_context": self.get_full_global_context(),
            "domain_context": self.get_comprehensive_domain_context(agent_type),
            "task_context": task_data,
            "historical_context": self.get_relevant_history(agent_type, limit=10),
            "external_data": self.get_enriched_external_data(),
            "reasoning_framework": self.get_reasoning_templates(agent_type)
        }
        return self.optimize_for_reasoning(context)
    
    def optimize_for_reasoning(self, context):
        # Add reasoning chains, examples, and detailed instructions
        context["reasoning_examples"] = self.get_reasoning_examples()
        context["decision_framework"] = self.get_decision_framework()
        context["edge_case_handling"] = self.get_edge_cases()
        return context
```

#### Tier 2 Agents (Standard Models - GPT-5-mini)
**Token Allocation**: 3,000-6,144 tokens  
**Context Strategy**: Balanced, focused context

```python
class Tier2ContextManager:
    def prepare_context(self, agent_type, task_data):
        context = {
            "global_context": self.get_essential_global_context(),
            "domain_context": self.get_focused_domain_context(agent_type),
            "task_context": task_data,
            "historical_context": self.get_relevant_history(agent_type, limit=5),
            "templates": self.get_response_templates(agent_type)
        }
        return self.optimize_for_efficiency(context)
    
    def optimize_for_efficiency(self, context):
        # Balance detail with token efficiency
        context = self.compress_historical_data(context)
        context = self.focus_on_task_relevance(context)
        return context
```

#### Tier 3 Agents (Efficient Models - GPT-5-nano)
**Token Allocation**: 1,000-4,096 tokens  
**Context Strategy**: Minimal, task-focused context

```python
class Tier3ContextManager:
    def prepare_context(self, agent_type, task_data):
        context = {
            "essential_data": self.get_minimal_global_context(),
            "task_parameters": self.extract_task_essentials(task_data),
            "output_format": self.get_structured_format(agent_type)
        }
        return self.optimize_for_speed(context)
    
    def optimize_for_speed(self, context):
        # Minimize tokens while maintaining accuracy
        context = self.use_structured_templates(context)
        context = self.compress_to_essentials(context)
        return context
```

## 🔄 Dynamic Context Loading

### 1. Context Inheritance Patterns

#### Parent-Child Context Flow
```
Master Orchestrator Context
├── Global Trip Context (inherited by all)
├── Planning Phase Context (inherited by planning-related agents)
│   ├── Location Agent Context
│   ├── Transport Agent Context
│   └── Activity Agent Context
└── Optimization Phase Context (inherited by optimization agents)
    ├── Budget Agent Context
    └── Accommodation Agent Context
```

#### Context Propagation Rules
```python
class ContextInheritance:
    def propagate_context(self, parent_context, child_agent_type):
        # 1. Start with global context
        child_context = self.get_global_context()
        
        # 2. Add relevant parent context
        relevant_parent = self.filter_relevant_context(
            parent_context, 
            child_agent_type
        )
        child_context.update(relevant_parent)
        
        # 3. Add agent-specific context
        agent_specific = self.get_agent_context(child_agent_type)
        child_context.update(agent_specific)
        
        # 4. Optimize for agent's model tier
        return self.optimize_for_model_tier(child_context, child_agent_type)
```

### 2. Lazy Context Loading

#### On-Demand Context Retrieval
```python
class LazyContextLoader:
    def __init__(self):
        self.context_cache = {}
        self.loading_strategies = {
            "immediate": self.load_immediate,
            "lazy": self.load_on_demand,
            "predictive": self.preload_predicted
        }
    
    def get_context(self, agent_type, task_id, context_level="standard"):
        cache_key = f"{agent_type}_{task_id}_{context_level}"
        
        if cache_key in self.context_cache:
            return self.context_cache[cache_key]
        
        # Determine loading strategy based on agent tier
        agent_config = self.get_agent_config(agent_type)
        loading_strategy = self.get_loading_strategy(agent_config.model_tier)
        
        context = loading_strategy(agent_type, task_id, context_level)
        self.context_cache[cache_key] = context
        
        return context
    
    def load_on_demand(self, agent_type, task_id, context_level):
        # Load only what's immediately needed
        base_context = self.get_minimal_context(agent_type)
        
        # Add task-specific context as needed
        if self.requires_domain_context(task_id):
            base_context.update(self.load_domain_context(agent_type))
        
        if self.requires_historical_context(task_id):
            base_context.update(self.load_historical_context(agent_type, limit=3))
        
        return base_context
```

## 📊 Context Window Optimization

### 1. Token Budget Management

#### Per-Agent Token Allocation
```yaml
Agent Token Budgets:
  planning_agent:
    model: gpt-5
    max_tokens: 8192
    allocation:
      global_context: 1500      # 18%
      domain_context: 3000      # 37%
      task_context: 2000        # 24%
      reasoning_space: 1692     # 21%
  
  location_agent:
    model: gpt-5-mini
    max_tokens: 6144
    allocation:
      global_context: 1000      # 16%
      domain_context: 2500      # 41%
      task_context: 1644        # 27%
      response_space: 1000      # 16%
  
  weather_agent:
    model: gpt-5-nano
    max_tokens: 2048
    allocation:
      essential_context: 800    # 39%
      task_parameters: 400      # 20%
      response_format: 848      # 41%
```

#### Dynamic Token Reallocation
```python
class TokenBudgetManager:
    def allocate_tokens(self, agent_type, task_complexity, context_requirements):
        base_budget = self.get_base_token_budget(agent_type)
        
        # Adjust based on task complexity
        if task_complexity == "high":
            context_boost = int(base_budget * 0.2)
            reasoning_boost = int(base_budget * 0.1)
        elif task_complexity == "low":
            context_reduction = int(base_budget * 0.15)
            efficiency_focus = True
        
        # Reallocate based on context requirements
        allocation = self.calculate_optimal_allocation(
            base_budget,
            context_requirements,
            task_complexity
        )
        
        return allocation
    
    def monitor_token_usage(self, agent_type, actual_usage):
        # Track actual vs planned usage
        self.usage_metrics[agent_type].append(actual_usage)
        
        # Adjust future allocations based on patterns
        if self.detect_consistent_overusage(agent_type):
            self.increase_token_budget(agent_type, percentage=10)
        elif self.detect_consistent_underusage(agent_type):
            self.optimize_context_loading(agent_type)
```

### 2. Context Compression Techniques

#### Hierarchical Summarization
```python
class ContextCompressor:
    def compress_historical_context(self, historical_data, target_tokens):
        """Compress historical context using progressive summarization"""
        
        if len(historical_data) <= target_tokens:
            return historical_data
        
        # Group related items
        grouped_data = self.group_by_relevance(historical_data)
        
        # Summarize each group
        compressed_groups = []
        for group in grouped_data:
            if len(group) > 1:
                summary = self.create_group_summary(group)
                compressed_groups.append(summary)
            else:
                compressed_groups.extend(group)
        
        # If still too large, apply second-level compression
        if self.estimate_tokens(compressed_groups) > target_tokens:
            return self.apply_secondary_compression(compressed_groups, target_tokens)
        
        return compressed_groups
    
    def create_smart_abstracts(self, detailed_context, compression_ratio=0.3):
        """Create intelligent abstracts preserving key information"""
        
        # Identify key entities and relationships
        key_entities = self.extract_key_entities(detailed_context)
        relationships = self.extract_relationships(detailed_context)
        
        # Preserve critical decision points
        decision_points = self.identify_decision_points(detailed_context)
        
        # Create compressed representation
        abstract = self.build_abstract(
            key_entities, 
            relationships, 
            decision_points,
            target_ratio=compression_ratio
        )
        
        return abstract
```

#### Template-Based Compression
```python
class TemplateCompressor:
    def __init__(self):
        self.templates = {
            "location_summary": "Location: {name} | Type: {type} | Rating: {rating} | Key features: {features}",
            "transport_option": "Route: {from}->{to} | Time: {duration} | Cost: ${price} | Type: {transport_type}",
            "activity_brief": "Activity: {name} | Duration: {time} | Cost: ${price} | Best for: {interests}"
        }
    
    def compress_using_templates(self, data, data_type):
        template = self.templates.get(data_type)
        if not template:
            return self.fallback_compression(data)
        
        compressed_items = []
        for item in data:
            compressed = template.format(**item)
            compressed_items.append(compressed)
        
        return " | ".join(compressed_items)
```

## 🔄 Memory Patterns & State Management

### 1. Working Memory for Agents

#### Short-Term Context Buffer
```python
class AgentWorkingMemory:
    def __init__(self, agent_id, max_size_mb=50):
        self.agent_id = agent_id
        self.short_term_buffer = {}
        self.max_size = max_size_mb * 1024 * 1024  # Convert to bytes
        self.access_frequency = {}
        self.last_access = {}
    
    def store_context(self, context_key, context_data, ttl_seconds=3600):
        """Store context with automatic cleanup"""
        
        # Check size limits
        if self.get_buffer_size() + self.estimate_size(context_data) > self.max_size:
            self.cleanup_old_context()
        
        self.short_term_buffer[context_key] = {
            "data": context_data,
            "timestamp": time.time(),
            "ttl": ttl_seconds,
            "access_count": 0
        }
        
        self.access_frequency[context_key] = 1
        self.last_access[context_key] = time.time()
    
    def retrieve_context(self, context_key):
        """Retrieve context with access tracking"""
        
        if context_key not in self.short_term_buffer:
            return None
        
        context_item = self.short_term_buffer[context_key]
        
        # Check TTL
        if time.time() - context_item["timestamp"] > context_item["ttl"]:
            del self.short_term_buffer[context_key]
            return None
        
        # Update access metrics
        context_item["access_count"] += 1
        self.access_frequency[context_key] += 1
        self.last_access[context_key] = time.time()
        
        return context_item["data"]
```

#### Long-Term Memory Integration
```python
class LongTermMemory:
    def __init__(self, agent_type):
        self.agent_type = agent_type
        self.memory_store = self.initialize_memory_store()
        self.indexing_strategy = self.get_indexing_strategy(agent_type)
    
    def store_successful_patterns(self, context, result, success_metrics):
        """Store successful context-result patterns for future use"""
        
        pattern = {
            "context_signature": self.create_context_signature(context),
            "result_pattern": self.extract_result_pattern(result),
            "success_score": success_metrics["user_satisfaction"],
            "performance_metrics": success_metrics,
            "timestamp": time.time()
        }
        
        self.memory_store.store_pattern(pattern)
        self.update_pattern_index(pattern)
    
    def retrieve_similar_patterns(self, current_context, similarity_threshold=0.8):
        """Retrieve similar successful patterns"""
        
        context_signature = self.create_context_signature(current_context)
        similar_patterns = self.memory_store.find_similar(
            context_signature, 
            threshold=similarity_threshold
        )
        
        return self.rank_by_relevance(similar_patterns, current_context)
```

### 2. Context Sharing Between Agents

#### Shared Context Pool
```python
class SharedContextPool:
    def __init__(self):
        self.global_pool = {}
        self.agent_subscriptions = {}
        self.context_versions = {}
        self.access_permissions = {}
    
    def publish_context(self, context_key, context_data, publisher_agent, permissions="read_only"):
        """Publish context for other agents to use"""
        
        self.global_pool[context_key] = {
            "data": context_data,
            "publisher": publisher_agent,
            "timestamp": time.time(),
            "version": self.increment_version(context_key),
            "permissions": permissions
        }
        
        # Notify subscribed agents
        self.notify_subscribers(context_key, context_data)
    
    def subscribe_to_context(self, agent_id, context_patterns):
        """Subscribe agent to specific context patterns"""
        
        if agent_id not in self.agent_subscriptions:
            self.agent_subscriptions[agent_id] = []
        
        self.agent_subscriptions[agent_id].extend(context_patterns)
    
    def get_shared_context(self, context_key, requesting_agent):
        """Retrieve shared context with permission checking"""
        
        if context_key not in self.global_pool:
            return None
        
        context_item = self.global_pool[context_key]
        
        # Check permissions
        if not self.check_permissions(requesting_agent, context_item):
            return None
        
        return context_item["data"]
```

## ⚙️ Environment Configuration for Context Management

### .env Configuration for Context Settings
```env
# Context Management Settings
CONTEXT_MANAGEMENT_ENABLED=true
CONTEXT_COMPRESSION_ENABLED=true
LAZY_LOADING_ENABLED=true

# Model-Specific Token Limits
PLANNING_AGENT_MAX_TOKENS=8192
TRANSPORT_AGENT_MAX_TOKENS=8192
LOCATION_AGENT_MAX_TOKENS=6144
ACCOMMODATION_AGENT_MAX_TOKENS=6144
ACTIVITY_AGENT_MAX_TOKENS=6144
BUDGET_AGENT_MAX_TOKENS=4096
WEATHER_AGENT_MAX_TOKENS=2048

# Context Allocation Ratios (as percentages)
TIER1_GLOBAL_CONTEXT_RATIO=18
TIER1_DOMAIN_CONTEXT_RATIO=37
TIER1_TASK_CONTEXT_RATIO=24
TIER1_REASONING_SPACE_RATIO=21

TIER2_GLOBAL_CONTEXT_RATIO=16
TIER2_DOMAIN_CONTEXT_RATIO=41
TIER2_TASK_CONTEXT_RATIO=27
TIER2_RESPONSE_SPACE_RATIO=16

TIER3_ESSENTIAL_CONTEXT_RATIO=39
TIER3_TASK_PARAMETERS_RATIO=20
TIER3_RESPONSE_FORMAT_RATIO=41

# Memory Management
WORKING_MEMORY_MAX_SIZE_MB=50
CONTEXT_TTL_SECONDS=3600
LONG_TERM_MEMORY_ENABLED=true
PATTERN_STORAGE_ENABLED=true

# Context Compression
COMPRESSION_RATIO=0.3
HIERARCHICAL_SUMMARIZATION=true
TEMPLATE_COMPRESSION=true
SMART_ABSTRACTS_ENABLED=true

# Shared Context
SHARED_CONTEXT_POOL_ENABLED=true
CONTEXT_VERSIONING_ENABLED=true
CROSS_AGENT_SUBSCRIPTIONS=true

# Performance Monitoring
CONTEXT_USAGE_MONITORING=true
TOKEN_BUDGET_TRACKING=true
MEMORY_USAGE_ALERTS=true
OPTIMIZATION_SUGGESTIONS=true
```

### Context Configuration by Agent Type
```yaml
context_configurations:
  planning_agent:
    model_tier: "tier1"
    context_strategy: "comprehensive"
    token_budget: 8192
    compression_ratio: 0.2
    memory_retention: "high"
    
  transport_agent:
    model_tier: "tier1" 
    context_strategy: "detailed"
    token_budget: 8192
    compression_ratio: 0.2
    memory_retention: "high"
    
  location_agent:
    model_tier: "tier2"
    context_strategy: "focused"
    token_budget: 6144
    compression_ratio: 0.3
    memory_retention: "medium"
    
  weather_agent:
    model_tier: "tier3"
    context_strategy: "minimal"
    token_budget: 2048
    compression_ratio: 0.5
    memory_retention: "low"
```

## 📈 Performance Optimization & Monitoring

### Context Efficiency Metrics
```python
class ContextMetrics:
    def track_context_efficiency(self, agent_type, context_size, response_quality):
        metrics = {
            "context_token_count": context_size,
            "context_compression_ratio": self.calculate_compression_ratio(context_size),
            "response_quality_score": response_quality,
            "context_utilization": self.calculate_utilization(agent_type),
            "cost_per_token": self.get_cost_per_token(agent_type),
            "total_cost": context_size * self.get_cost_per_token(agent_type)
        }
        
        self.store_metrics(agent_type, metrics)
        return metrics
    
    def generate_optimization_recommendations(self, agent_type):
        historical_metrics = self.get_historical_metrics(agent_type)
        
        recommendations = []
        
        # Analyze token usage patterns
        if self.detect_token_waste(historical_metrics):
            recommendations.append({
                "type": "token_optimization",
                "suggestion": "Reduce context window by 15%",
                "expected_savings": self.calculate_savings(agent_type, 0.15)
            })
        
        # Analyze compression effectiveness
        if self.detect_compression_opportunities(historical_metrics):
            recommendations.append({
                "type": "compression_improvement",
                "suggestion": "Increase compression ratio to 0.4",
                "expected_savings": self.calculate_compression_savings(agent_type)
            })
        
        return recommendations
```

---

**Key Context Management Principles:**
1. **Efficiency**: Optimal token usage for each model tier and agent capability
2. **Relevance**: Context precisely matched to agent needs and task requirements
3. **Scalability**: Context strategies that scale with system load and complexity
4. **Adaptability**: Dynamic adjustment based on performance metrics and usage patterns
5. **Cost Optimization**: Intelligent context allocation to minimize operational costs