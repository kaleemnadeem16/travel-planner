# Vector Database (Qdrant) Schema & Configuration

## 🎯 Vector Database Strategy

Our **Qdrant vector database** handles all embedding-based operations for intelligent travel planning, including semantic search, recommendation engines, and AI agent contextual memory.

### 🔍 **Core Use Cases**

1. **Semantic Travel Search** - Find similar destinations, activities, and experiences
2. **Personalized Recommendations** - User preference-based suggestions
3. **Agent Memory** - Contextual memory for multi-agent conversations
4. **Content Discovery** - Similar place/activity discovery
5. **User Behavior Analysis** - Pattern recognition in travel preferences

## 🗄️ Qdrant Collection Schema

### Collection Architecture

```python
# Qdrant Collections Configuration
COLLECTIONS = {
    "destinations": {
        "vector_size": 1536,  # OpenAI text-embedding-3-small
        "distance": "Cosine",
        "description": "Global destinations and places"
    },
    "activities": {
        "vector_size": 1536,
        "distance": "Cosine", 
        "description": "Activities, attractions, and experiences"
    },
    "user_preferences": {
        "vector_size": 1536,
        "distance": "Cosine",
        "description": "User travel preferences and behavior"
    },
    "agent_memory": {
        "vector_size": 1536,
        "distance": "Cosine",
        "description": "Conversational context and agent memory"
    },
    "travel_plans": {
        "vector_size": 1536,
        "distance": "Cosine",
        "description": "Complete travel plan embeddings"
    },
    "content_embeddings": {
        "vector_size": 1536,
        "distance": "Cosine",
        "description": "General content for RAG operations"
    }
}
```

## 📊 Collection Schemas

### 1. Destinations Collection

```python
# Collection: destinations
{
    "id": "dest_12345",
    "vector": [0.1, 0.2, ..., 0.1536],  # 1536-dimensional embedding
    "payload": {
        "destination_id": "12345",
        "name": "Tokyo, Japan",
        "country": "Japan",
        "continent": "Asia",
        "city": "Tokyo",
        "type": "city",  # city, region, country, landmark
        "coordinates": {
            "lat": 35.6762,
            "lon": 139.6503
        },
        "description": "Bustling metropolis blending traditional and modern culture...",
        "tags": ["urban", "culture", "technology", "food", "nightlife"],
        "climate": {
            "type": "humid_subtropical",
            "seasons": ["spring", "summer", "autumn", "winter"]
        },
        "travel_info": {
            "visa_required": True,
            "language": "Japanese",
            "currency": "JPY",
            "safety_rating": 9.2
        },
        "popularity_score": 9.5,
        "budget_level": "medium_high",
        "last_updated": "2025-09-04T10:00:00Z",
        "source": "manual_curation"
    }
}
```

### 2. Activities Collection

```python
# Collection: activities
{
    "id": "activity_67890",
    "vector": [0.1, 0.2, ..., 0.1536],
    "payload": {
        "activity_id": "67890",
        "name": "Senso-ji Temple Visit",
        "destination_id": "12345",  # Links to destinations
        "type": "cultural_site",  # restaurant, attraction, activity, experience
        "category": "religious_site",
        "subcategory": "buddhist_temple",
        "coordinates": {
            "lat": 35.7148,
            "lon": 139.7967
        },
        "description": "Ancient Buddhist temple in Asakusa district...",
        "tags": ["temple", "historic", "spiritual", "architecture", "free"],
        "duration": {
            "typical_minutes": 90,
            "min_minutes": 45,
            "max_minutes": 180
        },
        "cost": {
            "currency": "JPY",
            "range": "free",
            "typical_cost": 0,
            "additional_costs": ["donation_optional"]
        },
        "accessibility": {
            "wheelchair_accessible": True,
            "mobility_difficulty": "easy"
        },
        "best_times": {
            "seasons": ["spring", "autumn"],
            "times_of_day": ["morning", "afternoon"],
            "avoid_times": ["evening"]
        },
        "ratings": {
            "overall": 4.7,
            "cultural_value": 4.9,
            "accessibility": 4.2,
            "value_for_money": 4.8
        },
        "last_updated": "2025-09-04T10:00:00Z",
        "source": "tripadvisor_api"
    }
}
```

### 3. User Preferences Collection

```python
# Collection: user_preferences
{
    "id": "user_pref_user123_session456",
    "vector": [0.1, 0.2, ..., 0.1536],  # Derived from preference text
    "payload": {
        "user_id": "user123",
        "session_id": "session456",
        "preference_type": "travel_style",  # travel_style, budget, activities, destinations
        "preferences": {
            "travel_style": ["cultural", "foodie", "adventure"],
            "budget_preference": "medium",
            "group_type": "couple",
            "pace": "relaxed",
            "accommodation": ["hotel", "ryokan"],
            "transport": ["train", "walking"],
            "interests": ["history", "temples", "food", "gardens"],
            "avoid": ["crowds", "nightlife", "extreme_sports"]
        },
        "derived_from": {
            "travel_plans": ["plan123", "plan124"],
            "explicit_input": True,
            "behavioral_analysis": True
        },
        "confidence_score": 0.85,
        "created_at": "2025-09-04T10:00:00Z",
        "updated_at": "2025-09-04T10:00:00Z",
        "expires_at": "2025-12-04T10:00:00Z"
    }
}
```

### 4. Agent Memory Collection

```python
# Collection: agent_memory
{
    "id": "memory_session456_msg789",
    "vector": [0.1, 0.2, ..., 0.1536],  # Context embedding
    "payload": {
        "session_id": "session456",
        "user_id": "user123",
        "plan_id": "plan123",
        "agent_type": "planning_agent",
        "memory_type": "conversation_context",  # conversation_context, user_preference, decision_rationale
        "content": {
            "conversation_turn": 5,
            "user_input": "I prefer temples over museums",
            "agent_response": "Noted your preference for temples. I'll prioritize traditional temples...",
            "context_summary": "User prefers cultural/spiritual experiences over modern attractions",
            "extracted_preferences": ["temples", "spiritual", "traditional"],
            "decision_factors": ["user_stated_preference", "previous_choices"]
        },
        "relationships": {
            "related_memories": ["memory_session456_msg123", "memory_session456_msg234"],
            "influences_decisions": ["activity_selection", "destination_ranking"]
        },
        "confidence": 0.92,
        "importance": 0.8,  # How important this memory is for future decisions
        "created_at": "2025-09-04T10:00:00Z",
        "expires_at": "2025-10-04T10:00:00Z"  # Memory retention policy
    }
}
```

### 5. Travel Plans Collection

```python
# Collection: travel_plans
{
    "id": "plan_embedding_123",
    "vector": [0.1, 0.2, ..., 0.1536],  # Full plan embedding
    "payload": {
        "plan_id": "123",
        "user_id": "user123",
        "plan_summary": "5-day cultural tour of Tokyo focusing on temples and traditional experiences",
        "destinations": ["Tokyo"],
        "key_activities": ["Senso-ji Temple", "Meiji Shrine", "Traditional Tea Ceremony"],
        "travel_style": ["cultural", "spiritual", "traditional"],
        "budget_level": "medium",
        "duration_days": 5,
        "traveler_count": 2,
        "season": "spring",
        "plan_characteristics": {
            "cultural_intensity": 0.9,
            "adventure_level": 0.2,
            "relaxation_factor": 0.7,
            "urban_vs_nature": 0.8,  # 1.0 = fully urban, 0.0 = fully nature
            "budget_intensity": 0.6
        },
        "success_metrics": {
            "user_rating": 4.8,
            "completion_rate": 0.95,
            "cost_accuracy": 0.88
        },
        "created_at": "2025-09-04T10:00:00Z",
        "version": 1
    }
}
```

### 6. Content Embeddings Collection

```python
# Collection: content_embeddings
{
    "id": "content_12345",
    "vector": [0.1, 0.2, ..., 0.1536],
    "payload": {
        "content_id": "12345",
        "content_type": "travel_guide",  # travel_guide, blog_post, review, faq
        "title": "Ultimate Guide to Tokyo Temples",
        "source": "travel_blog_xyz",
        "url": "https://example.com/tokyo-temples-guide",
        "author": "Travel Expert",
        "content_summary": "Comprehensive guide covering 15 must-visit temples in Tokyo...",
        "topics": ["temples", "tokyo", "spiritual", "buddhist", "shinto"],
        "target_audience": ["cultural_travelers", "spiritual_seekers"],
        "quality_score": 0.92,
        "freshness_score": 0.85,  # How up-to-date the content is
        "created_at": "2025-09-04T10:00:00Z",
        "indexed_at": "2025-09-04T10:00:00Z"
    }
}
```

## 🔍 Vector Search Operations

### Semantic Search Examples

```python
# 1. Find similar destinations
from qdrant_client import QdrantClient
from qdrant_client.models import Filter, FieldCondition, MatchValue

client = QdrantClient("localhost", port=6333)

# Search for destinations similar to "cultural exploration in Asia"
search_results = client.search(
    collection_name="destinations",
    query_vector=embed_text("cultural exploration in Asia"),
    query_filter=Filter(
        must=[
            FieldCondition(
                key="continent",
                match=MatchValue(value="Asia")
            )
        ]
    ),
    limit=10,
    score_threshold=0.7
)

# 2. Find activities matching user preferences
activity_results = client.search(
    collection_name="activities",
    query_vector=user_preference_vector,
    query_filter=Filter(
        must=[
            FieldCondition(
                key="destination_id",
                match=MatchValue(value="12345")
            ),
            FieldCondition(
                key="type",
                match=MatchValue(value="cultural_site")
            )
        ]
    ),
    limit=15
)

# 3. Retrieve relevant agent memory
memory_results = client.search(
    collection_name="agent_memory",
    query_vector=current_context_vector,
    query_filter=Filter(
        must=[
            FieldCondition(
                key="session_id",
                match=MatchValue(value="session456")
            ),
            FieldCondition(
                key="importance",
                range={"gte": 0.5}  # Only important memories
            )
        ]
    ),
    limit=5
)
```

### Recommendation Engine Queries

```python
# Find similar travel plans for recommendations
def find_similar_plans(user_plan_vector, exclude_user_id):
    return client.search(
        collection_name="travel_plans",
        query_vector=user_plan_vector,
        query_filter=Filter(
            must_not=[
                FieldCondition(
                    key="user_id",
                    match=MatchValue(value=exclude_user_id)
                )
            ],
            must=[
                FieldCondition(
                    key="success_metrics.user_rating",
                    range={"gte": 4.0}
                )
            ]
        ),
        limit=10,
        score_threshold=0.6
    )

# Hybrid search combining vector similarity and filters
def hybrid_activity_search(preference_vector, destination_id, budget_level):
    return client.search(
        collection_name="activities",
        query_vector=preference_vector,
        query_filter=Filter(
            must=[
                FieldCondition(key="destination_id", match=MatchValue(value=destination_id)),
                FieldCondition(key="cost.range", match=MatchValue(value=budget_level)),
                FieldCondition(key="ratings.overall", range={"gte": 4.0})
            ]
        ),
        limit=20,
        score_threshold=0.5
    )
```

## ⚙️ Qdrant Configuration

### Docker Compose Setup

```yaml
# docker-compose.yml
version: '3.8'
services:
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant_storage:/qdrant/storage
      - ./qdrant_config:/qdrant/config
    environment:
      - QDRANT__SERVICE__HTTP_PORT=6333
      - QDRANT__SERVICE__GRPC_PORT=6334
      - QDRANT__LOG_LEVEL=INFO
    restart: unless-stopped

volumes:
  qdrant_storage:
```

### Production Configuration

```yaml
# qdrant_config/production.yaml
service:
  host: 0.0.0.0
  http_port: 6333
  grpc_port: 6334
  max_request_size_mb: 32
  max_workers: 0  # Auto-detect based on CPU cores

storage:
  # Optimized for Oracle Cloud ARM64
  performance:
    max_search_threads: 0  # Auto-detect
    max_optimization_threads: 0  # Auto-detect
  
  # Optimistic concurrency for high-throughput
  optimizers:
    deleted_threshold: 0.2
    vacuum_min_vector_number: 1000
    default_segment_number: 0  # Auto-detect
    max_segment_size_kb: 5000000  # 5GB for ARM64
    memmap_threshold_kb: 200000   # 200MB
    indexing_threshold_kb: 20000  # 20MB

telemetry:
  disabled: true  # Disable in production

cluster:
  enabled: false  # Single node for initial deployment
```

### Python Client Configuration

```python
# app/core/vector_db.py
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, CreateCollection
import os

class VectorDatabase:
    def __init__(self):
        self.client = QdrantClient(
            host=os.getenv("QDRANT_HOST", "localhost"),
            port=int(os.getenv("QDRANT_PORT", 6333)),
            timeout=30.0,
            # Connection pooling for high performance
            pool_size=20,
            retries=3
        )
        
    def initialize_collections(self):
        """Create all required collections"""
        collections_config = {
            "destinations": {
                "vector_size": 1536,
                "distance": Distance.COSINE
            },
            "activities": {
                "vector_size": 1536, 
                "distance": Distance.COSINE
            },
            "user_preferences": {
                "vector_size": 1536,
                "distance": Distance.COSINE
            },
            "agent_memory": {
                "vector_size": 1536,
                "distance": Distance.COSINE
            },
            "travel_plans": {
                "vector_size": 1536,
                "distance": Distance.COSINE
            },
            "content_embeddings": {
                "vector_size": 1536,
                "distance": Distance.COSINE
            }
        }
        
        for collection_name, config in collections_config.items():
            try:
                self.client.create_collection(
                    collection_name=collection_name,
                    vectors_config=VectorParams(
                        size=config["vector_size"],
                        distance=config["distance"]
                    )
                )
                print(f"Created collection: {collection_name}")
            except Exception as e:
                print(f"Collection {collection_name} already exists or error: {e}")

# Initialize vector database
vector_db = VectorDatabase()
```

## 📈 Performance Optimization

### Indexing Strategy

```python
# Optimize collections for specific query patterns
def optimize_collections():
    # Create payload indexes for common filters
    payload_indexes = {
        "destinations": ["country", "continent", "type", "budget_level"],
        "activities": ["destination_id", "type", "cost.range", "ratings.overall"],
        "user_preferences": ["user_id", "preference_type"],
        "agent_memory": ["session_id", "user_id", "agent_type", "importance"],
        "travel_plans": ["user_id", "duration_days", "budget_level"],
        "content_embeddings": ["content_type", "topics", "quality_score"]
    }
    
    for collection, fields in payload_indexes.items():
        for field in fields:
            try:
                client.create_payload_index(
                    collection_name=collection,
                    field_name=field
                )
            except Exception as e:
                print(f"Index for {collection}.{field} exists or error: {e}")
```

### Memory Management

```python
# Automatic cleanup of expired memories and old data
def cleanup_expired_data():
    from datetime import datetime, timedelta
    
    current_time = datetime.utcnow().isoformat() + "Z"
    
    # Clean expired agent memories
    client.delete(
        collection_name="agent_memory",
        points_selector=Filter(
            must=[
                FieldCondition(
                    key="expires_at",
                    range={"lt": current_time}
                )
            ]
        )
    )
    
    # Clean old low-importance memories older than 30 days
    thirty_days_ago = (datetime.utcnow() - timedelta(days=30)).isoformat() + "Z"
    client.delete(
        collection_name="agent_memory",
        points_selector=Filter(
            must=[
                FieldCondition(key="created_at", range={"lt": thirty_days_ago}),
                FieldCondition(key="importance", range={"lt": 0.5})
            ]
        )
    )
```

## 🔄 Data Synchronization

### PostgreSQL ↔ Qdrant Sync

```python
# Sync travel plan embeddings when plans are updated
async def sync_plan_to_vector_db(plan_id: str, plan_data: dict):
    """Sync travel plan to vector database"""
    
    # Generate embedding from plan summary
    plan_text = f"""
    {plan_data['title']} - {plan_data['description']}
    Destinations: {', '.join(plan_data.get('destinations', []))}
    Duration: {plan_data.get('duration_days', 0)} days
    Budget: {plan_data.get('budget_level', 'unknown')}
    Activities: {', '.join([a.get('name', '') for a in plan_data.get('activities', [])])}
    """
    
    embedding = await generate_embedding(plan_text)
    
    # Upsert to Qdrant
    client.upsert(
        collection_name="travel_plans",
        points=[{
            "id": f"plan_embedding_{plan_id}",
            "vector": embedding,
            "payload": {
                "plan_id": plan_id,
                "user_id": plan_data["user_id"],
                "plan_summary": plan_text,
                **extract_plan_characteristics(plan_data)
            }
        }]
    )

# Batch sync for initial data population
async def batch_sync_destinations():
    """Sync all destinations from PostgreSQL to Qdrant"""
    # This would typically be run as a migration or setup script
    destinations = await get_all_destinations_from_postgres()
    
    points = []
    for dest in destinations:
        embedding = await generate_embedding(dest["description"])
        points.append({
            "id": f"dest_{dest['id']}",
            "vector": embedding,
            "payload": dest
        })
    
    # Batch upsert for efficiency
    client.upsert(
        collection_name="destinations", 
        points=points
    )
```

## 🛡️ Security Considerations

### Access Control

```python
# Implement user-specific data isolation
def get_user_specific_filter(user_id: str):
    """Ensure users only access their own data"""
    return Filter(
        must=[
            FieldCondition(
                key="user_id",
                match=MatchValue(value=user_id)
            )
        ]
    )

# Example usage in search
def search_user_memories(user_id: str, query_vector: list):
    return client.search(
        collection_name="agent_memory",
        query_vector=query_vector,
        query_filter=get_user_specific_filter(user_id),
        limit=10
    )
```

This vector database architecture provides:

✅ **Semantic Search** - Intelligent content discovery and matching  
✅ **Personalization** - User preference-based recommendations  
✅ **Agent Memory** - Contextual conversation continuity  
✅ **Performance** - Optimized for ARM64 deployment  
✅ **Scalability** - Efficient batch operations and indexing  
✅ **Data Privacy** - User-specific data isolation  
✅ **Synchronization** - Real-time sync with PostgreSQL  
✅ **Memory Management** - Automatic cleanup and retention policies