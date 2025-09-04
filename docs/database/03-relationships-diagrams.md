# Database Relationships & Architecture Diagrams

## 🎯 Database Architecture Overview

This document provides visual representations of our dual database architecture, showing relationships between PostgreSQL tables and Qdrant collections, as well as data flow patterns.

## 📊 PostgreSQL Entity Relationship Diagram

### Core Tables Relationship

```mermaid
erDiagram
    users ||--o{ travel_plans : "creates"
    users ||--o{ agent_executions : "initiates"
    users ||--o{ cost_tracking : "incurs"
    users ||--o{ user_sessions : "has"
    users ||--o{ plan_shares : "creates/receives"
    
    travel_plans ||--o{ agent_executions : "processes"
    travel_plans ||--o{ cost_tracking : "tracks"
    travel_plans ||--o{ agent_communications : "coordinates"
    travel_plans ||--o{ plan_shares : "shared"
    travel_plans ||--o{ travel_plans : "versions"
    
    agent_executions ||--|| cost_tracking : "costs"
    agent_executions ||--o{ agent_communications : "communicates"
    
    users {
        uuid id PK
        string email UK
        string username UK
        string password_hash
        string first_name
        string last_name
        string phone_number
        date date_of_birth
        jsonb profile_data
        jsonb travel_preferences
        string subscription_tier
        boolean is_active
        boolean is_verified
        timestamp created_at
        timestamp updated_at
    }
    
    travel_plans {
        uuid id PK
        uuid user_id FK
        string title
        text description
        jsonb destinations
        date start_date
        date end_date
        integer duration_days
        integer traveler_count
        decimal budget_total
        string budget_currency
        jsonb preferences
        jsonb plan_data
        jsonb itinerary
        jsonb agent_results
        integer version
        uuid parent_plan_id FK
        string status
        decimal total_cost_usd
        integer token_usage_total
        timestamp created_at
        timestamp updated_at
    }
    
    agent_executions {
        uuid id PK
        uuid plan_id FK
        uuid user_id FK
        string agent_type
        string agent_version
        jsonb agent_config
        string model_provider
        string model_name
        jsonb task_data
        jsonb result_data
        integer execution_time_ms
        integer prompt_tokens
        integer completion_tokens
        integer total_tokens
        decimal cost_usd
        string status
        timestamp created_at
        timestamp completed_at
    }
    
    cost_tracking {
        uuid id PK
        uuid user_id FK
        uuid plan_id FK
        uuid agent_execution_id FK
        string model_provider
        string model_name
        integer prompt_tokens
        integer completion_tokens
        decimal total_cost_usd
        string usage_category
        timestamp created_at
    }
```

### Agent Communication Flow

```mermaid
erDiagram
    agent_communications {
        uuid id PK
        uuid plan_id FK
        string session_id
        string from_agent
        string to_agent
        string message_type
        jsonb message_data
        jsonb response_data
        integer response_time_ms
        string status
        timestamp sent_at
        timestamp processed_at
    }
    
    system_metrics {
        uuid id PK
        string metric_name
        string metric_type
        decimal value
        string unit
        jsonb tags
        string service_name
        timestamp recorded_at
    }
    
    user_sessions {
        uuid id PK
        uuid user_id FK
        string session_token UK
        uuid plan_id FK
        jsonb context_data
        jsonb conversation_history
        jsonb agent_states
        boolean is_active
        timestamp created_at
        timestamp last_activity
    }
```

## 🔍 Vector Database Collection Relationships

### Qdrant Collections Architecture

```mermaid
graph TB
    subgraph "User Data"
        UP[User Preferences<br/>Collection]
        AM[Agent Memory<br/>Collection]
        TP[Travel Plans<br/>Collection]
    end
    
    subgraph "Content Data"
        DEST[Destinations<br/>Collection]
        ACT[Activities<br/>Collection]
        CONT[Content Embeddings<br/>Collection]
    end
    
    subgraph "Search Operations"
        SEM[Semantic Search]
        REC[Recommendations]
        CTX[Context Retrieval]
    end
    
    UP -->|User Profile| SEM
    AM -->|Conversation Context| CTX
    TP -->|Similar Plans| REC
    DEST -->|Location Matching| SEM
    ACT -->|Activity Suggestions| REC
    CONT -->|RAG Operations| CTX
    
    SEM -->|Results| APP[Application Layer]
    REC -->|Results| APP
    CTX -->|Results| APP
```

### Vector Search Flow

```mermaid
sequenceDiagram
    participant User
    participant App
    participant PG as PostgreSQL
    participant QD as Qdrant
    participant AI as AI Agents
    
    User->>App: "Find temples in Tokyo"
    App->>AI: Generate query embedding
    AI-->>App: Vector embedding
    
    App->>QD: Search destinations collection
    QD-->>App: Matching destinations
    
    App->>QD: Search activities collection
    QD-->>App: Relevant activities
    
    App->>PG: Get detailed data
    PG-->>App: Complete records
    
    App->>QD: Store search context in agent_memory
    App-->>User: Personalized results
```

## 🔄 Data Flow Architecture

### PostgreSQL ↔ Qdrant Synchronization

```mermaid
flowchart LR
    subgraph "PostgreSQL Database"
        USR[Users Table]
        PLN[Travel Plans Table]
        AGT[Agent Executions Table]
        SES[User Sessions Table]
    end
    
    subgraph "Synchronization Layer"
        SYNC[Data Sync Service]
        EMB[Embedding Generator]
        BATCH[Batch Processor]
    end
    
    subgraph "Qdrant Database"
        QUPLANS[Travel Plans Collection]
        QUPREF[User Preferences Collection]
        QUMEM[Agent Memory Collection]
        QUDEST[Destinations Collection]
    end
    
    PLN -->|Plan Updates| SYNC
    USR -->|Preference Changes| SYNC
    AGT -->|Context Data| SYNC
    SES -->|Conversation Data| SYNC
    
    SYNC --> EMB
    EMB --> BATCH
    
    BATCH -->|Upsert Vectors| QUPLANS
    BATCH -->|Upsert Vectors| QUPREF
    BATCH -->|Upsert Vectors| QUMEM
    BATCH -->|Initial Load| QUDEST
```

### Multi-Agent Data Access Pattern

```mermaid
graph TD
    subgraph "Agent Types"
        ORCH[Orchestrator Agent]
        PLAN[Planning Agent]
        LOC[Location Agent]
        ACT[Activity Agent]
        TRANS[Transport Agent]
    end
    
    subgraph "Data Access Layer"
        PG[(PostgreSQL)]
        QD[(Qdrant)]
        CACHE[(Redis Cache)]
    end
    
    subgraph "Data Types"
        STRUCT[Structured Data<br/>Users, Plans, Costs]
        VECTOR[Vector Data<br/>Embeddings, Similarities]
        TEMP[Temporary Data<br/>Sessions, State]
    end
    
    ORCH -->|Read/Write| PG
    ORCH -->|Search| QD
    ORCH -->|Cache| CACHE
    
    PLAN -->|Plan CRUD| PG
    PLAN -->|Similar Plans| QD
    
    LOC -->|Destination Search| QD
    LOC -->|Location Data| PG
    
    ACT -->|Activity Search| QD
    ACT -->|Activity Details| PG
    
    TRANS -->|Booking Data| PG
    TRANS -->|Route Planning| QD
    
    PG -.->|Contains| STRUCT
    QD -.->|Contains| VECTOR
    CACHE -.->|Contains| TEMP
```

## 📈 Performance Architecture

### Database Scaling Strategy

```mermaid
graph TB
    subgraph "Application Tier"
        APP1[App Instance 1]
        APP2[App Instance 2]
        APP3[App Instance 3]
    end
    
    subgraph "Connection Pool"
        POOL[pgBouncer<br/>Connection Pool]
    end
    
    subgraph "PostgreSQL Cluster"
        PG_MASTER[(PostgreSQL Primary)]
        PG_READ1[(Read Replica 1)]
        PG_READ2[(Read Replica 2)]
    end
    
    subgraph "Vector Database"
        QD_MAIN[(Qdrant Primary)]
        QD_SHARD1[(Qdrant Shard 1)]
        QD_SHARD2[(Qdrant Shard 2)]
    end
    
    subgraph "Caching Layer"
        REDIS_MAIN[(Redis Primary)]
        REDIS_REP[(Redis Replica)]
    end
    
    APP1 --> POOL
    APP2 --> POOL
    APP3 --> POOL
    
    POOL -->|Writes| PG_MASTER
    POOL -->|Reads| PG_READ1
    POOL -->|Reads| PG_READ2
    
    APP1 -->|Vector Search| QD_MAIN
    APP2 -->|Vector Search| QD_SHARD1
    APP3 -->|Vector Search| QD_SHARD2
    
    APP1 -->|Cache| REDIS_MAIN
    APP2 -->|Cache| REDIS_REP
    APP3 -->|Cache| REDIS_MAIN
    
    PG_MASTER -.->|Replication| PG_READ1
    PG_MASTER -.->|Replication| PG_READ2
    REDIS_MAIN -.->|Replication| REDIS_REP
```

### Query Performance Optimization

```mermaid
flowchart TD
    subgraph "Query Types"
        Q1[User Travel Plans]
        Q2[Agent Executions by Plan]
        Q3[Cost Analytics]
        Q4[Semantic Search]
        Q5[Recommendations]
    end
    
    subgraph "Optimization Strategies"
        IDX[Database Indexes]
        PART[Table Partitioning]
        MAT[Materialized Views]
        CACHE[Query Caching]
        VEC[Vector Indexing]
    end
    
    subgraph "Performance Targets"
        T1[< 100ms Simple Queries]
        T2[< 500ms Complex Analytics]
        T3[< 200ms Vector Search]
        T4[< 50ms Cached Queries]
    end
    
    Q1 -->|Uses| IDX
    Q2 -->|Uses| PART
    Q3 -->|Uses| MAT
    Q4 -->|Uses| VEC
    Q5 -->|Uses| CACHE
    
    IDX --> T1
    PART --> T2
    MAT --> T2
    VEC --> T3
    CACHE --> T4
```

## 🔐 Security Architecture

### Data Access Control

```mermaid
graph TB
    subgraph "User Authentication"
        AUTH[Authentication Service]
        JWT[JWT Tokens]
        OAUTH[OAuth Providers]
    end
    
    subgraph "Authorization Layer"
        RLS[Row Level Security]
        RBAC[Role-Based Access]
        FILTER[Query Filters]
    end
    
    subgraph "Database Security"
        PG_RLS[(PostgreSQL RLS)]
        QD_FILTER[(Qdrant Filters)]
        ENCRYPT[Data Encryption]
    end
    
    AUTH --> JWT
    OAUTH --> AUTH
    
    JWT --> RLS
    JWT --> RBAC
    JWT --> FILTER
    
    RLS --> PG_RLS
    FILTER --> QD_FILTER
    RBAC --> ENCRYPT
```

### Data Privacy Compliance

```mermaid
flowchart LR
    subgraph "Data Types"
        PII[Personal Data]
        PREF[User Preferences]
        CONV[Conversations]
        PLANS[Travel Plans]
    end
    
    subgraph "Privacy Controls"
        GDPR[GDPR Compliance]
        RETENTION[Data Retention]
        DELETION[Right to Delete]
        EXPORT[Data Export]
    end
    
    subgraph "Implementation"
        SOFT_DEL[Soft Delete]
        ENCRYPT[Encryption at Rest]
        AUDIT[Audit Logs]
        ANON[Data Anonymization]
    end
    
    PII --> GDPR
    PREF --> RETENTION
    CONV --> DELETION
    PLANS --> EXPORT
    
    GDPR --> SOFT_DEL
    RETENTION --> ENCRYPT
    DELETION --> AUDIT
    EXPORT --> ANON
```

## 🔄 Backup & Recovery Architecture

### Backup Strategy

```mermaid
timeline
    title Database Backup Strategy
    
    section PostgreSQL
        Continuous WAL : Point-in-time recovery
        Daily Full     : Complete database backup
        Weekly Archive : Long-term storage
    
    section Qdrant
        Daily Snapshots : Collection backups
        Weekly Full     : Complete vector DB
        Monthly Archive : Long-term retention
    
    section Redis
        Real-time Sync  : Replica for failover
        Daily Persistence : RDB snapshots
        AOF Logging     : Append-only files
```

### Disaster Recovery Flow

```mermaid
sequenceDiagram
    participant MON as Monitoring
    participant ALERT as Alert System
    participant OPS as Operations
    participant BACKUP as Backup System
    participant DB as Database
    
    MON->>ALERT: Database failure detected
    ALERT->>OPS: Send incident notification
    OPS->>BACKUP: Initiate recovery procedure
    BACKUP->>DB: Restore from latest backup
    DB->>MON: Health check confirmation
    MON->>ALERT: Recovery successful
    ALERT->>OPS: System restored notification
```

## 📊 Monitoring & Observability

### Database Metrics Dashboard

```mermaid
graph TB
    subgraph "PostgreSQL Metrics"
        PG_CONN[Connection Count]
        PG_QPS[Queries Per Second]
        PG_LAT[Query Latency]
        PG_SIZE[Database Size]
    end
    
    subgraph "Qdrant Metrics"
        QD_SEARCH[Search Operations/sec]
        QD_INDEX[Index Performance]
        QD_MEM[Memory Usage]
        QD_DISK[Disk Usage]
    end
    
    subgraph "Redis Metrics"
        RED_HIT[Cache Hit Rate]
        RED_MEM[Memory Usage]
        RED_OPS[Operations/sec]
        RED_CONN[Connection Count]
    end
    
    subgraph "Application Metrics"
        APP_RESP[Response Time]
        APP_ERR[Error Rate]
        APP_USERS[Active Users]
        APP_COST[API Costs]
    end
    
    subgraph "Alerting"
        ALERT1[High Latency Alert]
        ALERT2[Connection Pool Alert]
        ALERT3[Storage Alert]
        ALERT4[Cost Alert]
    end
    
    PG_LAT -->|> 500ms| ALERT1
    PG_CONN -->|> 80%| ALERT2
    QD_DISK -->|> 90%| ALERT3
    APP_COST -->|Daily Limit| ALERT4
```

This database relationship documentation provides:

✅ **Visual Architecture** - Clear diagrams showing system relationships  
✅ **Data Flow Patterns** - How data moves between systems  
✅ **Performance Strategy** - Scaling and optimization approaches  
✅ **Security Design** - Access control and privacy compliance  
✅ **Operational Readiness** - Backup, recovery, and monitoring  
✅ **Development Guide** - Clear understanding for implementation  
✅ **Troubleshooting Aid** - Diagrams help identify issues  
✅ **Documentation Standard** - Consistent visual language