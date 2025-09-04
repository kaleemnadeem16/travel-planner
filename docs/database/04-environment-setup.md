# Database Environment Setup & Configuration

## 🎯 Environment Configuration Overview

This document covers the complete database setup for both development and production environments, including PostgreSQL, Qdrant, and Redis configuration for optimal performance on Oracle Cloud ARM64.

## 🛠️ Development Environment Setup

### Prerequisites

```bash
# Install required dependencies
sudo apt update
sudo apt install -y postgresql-15 postgresql-contrib redis-server

# Install Python dependencies
pip install psycopg2-binary qdrant-client redis sqlalchemy alembic
```

### PostgreSQL Development Setup

```bash
# Create development database
sudo -u postgres createdb travel_planner_dev
sudo -u postgres createuser travel_app

# Set password for database user
sudo -u postgres psql -c "ALTER USER travel_app PASSWORD 'dev_password';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE travel_planner_dev TO travel_app;"

# Enable UUID extension
sudo -u postgres psql travel_planner_dev -c "CREATE EXTENSION IF NOT EXISTS \"uuid-ossp\";"
sudo -u postgres psql travel_planner_dev -c "CREATE EXTENSION IF NOT EXISTS \"pgcrypto\";"
```

### Qdrant Development Setup

```bash
# Using Docker for development
docker run -d --name qdrant-dev \
  -p 6333:6333 \
  -p 6334:6334 \
  -v $(pwd)/qdrant_storage:/qdrant/storage \
  qdrant/qdrant:latest

# Or install locally
wget https://github.com/qdrant/qdrant/releases/latest/download/qdrant-x86_64-unknown-linux-gnu.tar.gz
tar -xzf qdrant-x86_64-unknown-linux-gnu.tar.gz
./qdrant --config-path ./config/dev-config.yaml
```

### Redis Development Setup

```bash
# Start Redis server
sudo systemctl start redis-server
sudo systemctl enable redis-server

# Configure for development
sudo nano /etc/redis/redis.conf
# Set: maxmemory 512mb
# Set: maxmemory-policy allkeys-lru

# Restart Redis
sudo systemctl restart redis-server
```

## 🚀 Production Environment Configuration

### Oracle Cloud ARM64 Optimization

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    platform: linux/arm64
    environment:
      POSTGRES_DB: travel_planner
      POSTGRES_USER: travel_app
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_INITDB_ARGS: "--auth-host=scram-sha-256"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./postgres/postgresql.conf:/etc/postgresql/postgresql.conf
      - ./postgres/pg_hba.conf:/etc/postgresql/pg_hba.conf
      - ./init-scripts:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"
    command: postgres -c config_file=/etc/postgresql/postgresql.conf
    deploy:
      resources:
        limits:
          memory: 8G
          cpus: '4'
        reservations:
          memory: 4G
          cpus: '2'
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U travel_app -d travel_planner"]
      interval: 30s
      timeout: 10s
      retries: 3

  qdrant:
    image: qdrant/qdrant:latest
    platform: linux/arm64
    volumes:
      - qdrant_data:/qdrant/storage
      - ./qdrant/production.yaml:/qdrant/config/production.yaml
    ports:
      - "6333:6333"
      - "6334:6334"
    environment:
      QDRANT__SERVICE__HTTP_PORT: 6333
      QDRANT__SERVICE__GRPC_PORT: 6334
      QDRANT__LOG_LEVEL: INFO
    deploy:
      resources:
        limits:
          memory: 8G
          cpus: '4'
        reservations:
          memory: 4G
          cpus: '2'
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:6333/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  redis:
    image: redis:7-alpine
    platform: linux/arm64
    volumes:
      - redis_data:/data
      - ./redis/redis.conf:/usr/local/etc/redis/redis.conf
    ports:
      - "6379:6379"
    command: redis-server /usr/local/etc/redis/redis.conf
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: '1'
        reservations:
          memory: 1G
          cpus: '0.5'
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  # PgBouncer for connection pooling
  pgbouncer:
    image: pgbouncer/pgbouncer:latest
    platform: linux/arm64
    environment:
      DATABASES_HOST: postgres
      DATABASES_PORT: 5432
      DATABASES_USER: travel_app
      DATABASES_PASSWORD: ${POSTGRES_PASSWORD}
      DATABASES_DBNAME: travel_planner
      POOL_MODE: transaction
      SERVER_RESET_QUERY: DISCARD ALL
      MAX_CLIENT_CONN: 1000
      DEFAULT_POOL_SIZE: 50
      MIN_POOL_SIZE: 10
      RESERVE_POOL_SIZE: 10
    ports:
      - "6432:6432"
    depends_on:
      - postgres
    restart: unless-stopped

volumes:
  postgres_data:
  qdrant_data:
  redis_data:
```

### PostgreSQL Production Configuration

```ini
# postgres/postgresql.conf - Optimized for Oracle Cloud ARM64 (24GB RAM)

# Connection Settings
max_connections = 500
superuser_reserved_connections = 3

# Memory Settings (Optimized for 24GB ARM64)
shared_buffers = 6GB                    # 25% of RAM
effective_cache_size = 18GB             # 75% of RAM
work_mem = 64MB                         # For complex queries
maintenance_work_mem = 1GB              # For maintenance operations
max_wal_size = 2GB
min_wal_size = 1GB

# Checkpoint Settings
checkpoint_completion_target = 0.9
checkpoint_timeout = 10min

# WAL Settings
wal_level = replica
wal_compression = on
wal_buffers = 16MB
max_wal_senders = 3
max_replication_slots = 3

# Query Planner
random_page_cost = 1.1                  # SSD optimization
effective_io_concurrency = 200          # High I/O concurrency for ARM64

# Logging
log_min_duration_statement = 1000       # Log slow queries (>1 second)
log_checkpoints = on
log_connections = on
log_disconnections = on
log_lock_waits = on
log_temp_files = 1MB

# Background Writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0

# Autovacuum (Optimized for high-volume operations)
autovacuum = on
autovacuum_max_workers = 6
autovacuum_naptime = 30s
autovacuum_vacuum_threshold = 50
autovacuum_analyze_threshold = 50
autovacuum_vacuum_scale_factor = 0.1
autovacuum_analyze_scale_factor = 0.05

# Lock Management
deadlock_timeout = 1s
max_locks_per_transaction = 256

# Security
ssl = on
ssl_cert_file = '/etc/ssl/certs/server.crt'
ssl_key_file = '/etc/ssl/private/server.key'
password_encryption = scram-sha-256

# Statistics
track_activities = on
track_counts = on
track_io_timing = on
track_functions = pl
stats_temp_directory = '/var/run/postgresql/stats_temp'
```

### Qdrant Production Configuration

```yaml
# qdrant/production.yaml
service:
  host: 0.0.0.0
  http_port: 6333
  grpc_port: 6334
  max_request_size_mb: 64
  max_workers: 0  # Auto-detect based on CPU cores
  cors_origins: []

storage:
  storage_path: "/qdrant/storage"
  snapshots_path: "/qdrant/snapshots"
  temp_path: "/qdrant/temp"
  
  # ARM64 Performance Optimization
  performance:
    max_search_threads: 0  # Auto-detect (use all cores)
    max_optimization_threads: 0  # Auto-detect
    
  # Optimized for ARM64 architecture
  optimizers:
    deleted_threshold: 0.2
    vacuum_min_vector_number: 1000
    default_segment_number: 0  # Auto-detect optimal segments
    max_segment_size_kb: 8000000  # 8GB for large datasets
    memmap_threshold_kb: 524288   # 512MB threshold
    indexing_threshold_kb: 20480  # 20MB indexing threshold
    flush_interval_sec: 30
    max_optimization_threads: 4
    
  # Memory allocation optimization
  on_disk_payload: true  # Store payload on disk to save RAM
  
  # Quantization for memory efficiency
  quantization:
    scalar:
      type: int8
      quantile: 0.99
      always_ram: true

# Telemetry disabled for production
telemetry:
  disabled: true

# Logging configuration
log_level: "INFO"

# Cluster configuration (for future scaling)
cluster:
  enabled: false
  node_id: 0
  
# Distributed setup for future
distributed:
  enabled: false
```

### Redis Production Configuration

```ini
# redis/redis.conf - Optimized for ARM64
# Basic Settings
bind 0.0.0.0
port 6379
protected-mode yes
requirepass ${REDIS_PASSWORD}

# Memory Management (2GB allocation)
maxmemory 2gb
maxmemory-policy allkeys-lru
maxmemory-samples 10

# Persistence Configuration
save 900 1     # Save if at least 1 key changed in 900 seconds
save 300 10    # Save if at least 10 keys changed in 300 seconds
save 60 10000  # Save if at least 10000 keys changed in 60 seconds

# RDB Configuration
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename redis-travel-planner.rdb
dir /data

# AOF Configuration
appendonly yes
appendfilename "redis-travel-planner.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# ARM64 Optimization
tcp-backlog 511
tcp-keepalive 300
timeout 0

# Slow Log
slowlog-log-slower-than 10000
slowlog-max-len 128

# Client Management
maxclients 10000

# Advanced Memory Management
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000

# Threaded I/O (ARM64 optimization)
io-threads 4
io-threads-do-reads yes

# Security
rename-command EVAL ""
rename-command DEBUG ""
rename-command CONFIG "CONFIG_${REDIS_SECRET_SUFFIX}"
```

## 🔧 Application Database Configuration

### SQLAlchemy Configuration

```python
# app/core/database.py
import os
from sqlalchemy import create_engine, event
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy.pool import QueuePool
from sqlalchemy.engine import Engine
import logging

# Database URL construction
def get_database_url():
    """Construct database URL based on environment"""
    if os.getenv("USE_PGBOUNCER", "false").lower() == "true":
        # Use PgBouncer in production
        return f"postgresql://{os.getenv('DB_USER')}:{os.getenv('DB_PASSWORD')}@{os.getenv('DB_HOST', 'localhost')}:6432/{os.getenv('DB_NAME')}"
    else:
        # Direct connection for development
        return f"postgresql://{os.getenv('DB_USER')}:{os.getenv('DB_PASSWORD')}@{os.getenv('DB_HOST', 'localhost')}:5432/{os.getenv('DB_NAME')}"

# Production-optimized engine configuration
SQLALCHEMY_DATABASE_URL = get_database_url()

# ARM64 optimized connection pool
engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    poolclass=QueuePool,
    pool_size=20,                    # Base connections
    max_overflow=30,                 # Additional connections under load
    pool_pre_ping=True,              # Validate connections before use
    pool_recycle=3600,               # Recycle connections every hour
    pool_timeout=30,                 # Wait timeout for connections
    echo=os.getenv("DB_ECHO", "false").lower() == "true",
    # ARM64 specific optimizations
    connect_args={
        "application_name": "travel_planner",
        "options": "-c statement_timeout=30000 -c lock_timeout=10000"
    }
)

# Session configuration
SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine,
    expire_on_commit=False  # Prevent lazy loading issues
)

Base = declarative_base()

# Connection event listeners for monitoring
@event.listens_for(Engine, "connect")
def set_sqlite_pragma(dbapi_connection, connection_record):
    """Set connection-level settings"""
    if "postgresql" in str(dbapi_connection):
        with dbapi_connection.cursor() as cursor:
            # Set application-specific settings
            cursor.execute("SET application_name = 'travel_planner'")
            cursor.execute("SET statement_timeout = '30s'")
            cursor.execute("SET lock_timeout = '10s'")

@event.listens_for(Engine, "checkout")
def receive_checkout(dbapi_connection, connection_record, connection_proxy):
    """Log connection checkout for monitoring"""
    logger = logging.getLogger("database.connections")
    logger.debug(f"Connection checked out: {id(dbapi_connection)}")

# Database dependency for FastAPI
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Qdrant Client Configuration

```python
# app/core/vector_database.py
import os
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, CreateCollection
from qdrant_client.http.exceptions import UnexpectedResponse
import logging
from typing import Optional

logger = logging.getLogger(__name__)

class VectorDatabaseConfig:
    """Centralized vector database configuration"""
    
    def __init__(self):
        self.host = os.getenv("QDRANT_HOST", "localhost")
        self.port = int(os.getenv("QDRANT_PORT", 6333))
        self.timeout = int(os.getenv("QDRANT_TIMEOUT", 30))
        self.pool_size = int(os.getenv("QDRANT_POOL_SIZE", 20))
        self.retries = int(os.getenv("QDRANT_RETRIES", 3))
        
        # Production optimizations
        self.grpc_port = int(os.getenv("QDRANT_GRPC_PORT", 6334))
        self.use_grpc = os.getenv("QDRANT_USE_GRPC", "false").lower() == "true"
        
    def create_client(self) -> QdrantClient:
        """Create optimized Qdrant client"""
        if self.use_grpc:
            # Use gRPC for better performance in production
            return QdrantClient(
                host=self.host,
                port=self.grpc_port,
                grpc_port=self.grpc_port,
                prefer_grpc=True,
                timeout=self.timeout
            )
        else:
            # Use HTTP for development/compatibility
            return QdrantClient(
                host=self.host,
                port=self.port,
                timeout=self.timeout
            )

class VectorDatabase:
    """Vector database management class"""
    
    def __init__(self):
        self.config = VectorDatabaseConfig()
        self.client = self.config.create_client()
        self._ensure_collections()
    
    def _ensure_collections(self):
        """Ensure all required collections exist"""
        collections = {
            "destinations": {"vector_size": 1536, "distance": Distance.COSINE},
            "activities": {"vector_size": 1536, "distance": Distance.COSINE},
            "user_preferences": {"vector_size": 1536, "distance": Distance.COSINE},
            "agent_memory": {"vector_size": 1536, "distance": Distance.COSINE},
            "travel_plans": {"vector_size": 1536, "distance": Distance.COSINE},
            "content_embeddings": {"vector_size": 1536, "distance": Distance.COSINE}
        }
        
        for collection_name, config in collections.items():
            try:
                self.client.create_collection(
                    collection_name=collection_name,
                    vectors_config=VectorParams(
                        size=config["vector_size"],
                        distance=config["distance"]
                    )
                )
                logger.info(f"Created collection: {collection_name}")
            except UnexpectedResponse as e:
                if "already exists" in str(e):
                    logger.debug(f"Collection {collection_name} already exists")
                else:
                    logger.error(f"Error creating collection {collection_name}: {e}")
                    raise

# Singleton instance
vector_db = VectorDatabase()
```

### Redis Configuration

```python
# app/core/cache.py
import os
import redis
from redis.connection import ConnectionPool
import logging
from typing import Optional, Any
import json
import pickle

logger = logging.getLogger(__name__)

class RedisConfig:
    """Redis configuration management"""
    
    def __init__(self):
        self.host = os.getenv("REDIS_HOST", "localhost")
        self.port = int(os.getenv("REDIS_PORT", 6379))
        self.password = os.getenv("REDIS_PASSWORD")
        self.db = int(os.getenv("REDIS_DB", 0))
        
        # Connection pool settings
        self.max_connections = int(os.getenv("REDIS_MAX_CONNECTIONS", 50))
        self.retry_on_timeout = True
        self.socket_timeout = int(os.getenv("REDIS_SOCKET_TIMEOUT", 5))
        self.socket_connect_timeout = int(os.getenv("REDIS_CONNECT_TIMEOUT", 5))
        
        # Session configuration
        self.session_ttl = int(os.getenv("SESSION_TTL", 3600))  # 1 hour
        self.cache_ttl = int(os.getenv("CACHE_TTL", 300))       # 5 minutes
        
    def create_pool(self) -> ConnectionPool:
        """Create Redis connection pool"""
        return ConnectionPool(
            host=self.host,
            port=self.port,
            password=self.password,
            db=self.db,
            max_connections=self.max_connections,
            retry_on_timeout=self.retry_on_timeout,
            socket_timeout=self.socket_timeout,
            socket_connect_timeout=self.socket_connect_timeout,
            decode_responses=True
        )

class CacheManager:
    """Redis cache management"""
    
    def __init__(self):
        self.config = RedisConfig()
        self.pool = self.config.create_pool()
        self.client = redis.Redis(connection_pool=self.pool)
        
    def set(self, key: str, value: Any, ttl: Optional[int] = None) -> bool:
        """Set cache value with optional TTL"""
        try:
            ttl = ttl or self.config.cache_ttl
            if isinstance(value, (dict, list)):
                value = json.dumps(value)
            elif not isinstance(value, str):
                value = pickle.dumps(value)
            
            return self.client.setex(key, ttl, value)
        except Exception as e:
            logger.error(f"Cache set error for key {key}: {e}")
            return False
    
    def get(self, key: str) -> Optional[Any]:
        """Get cache value"""
        try:
            value = self.client.get(key)
            if value is None:
                return None
            
            # Try JSON first, then pickle
            try:
                return json.loads(value)
            except (json.JSONDecodeError, TypeError):
                try:
                    return pickle.loads(value.encode() if isinstance(value, str) else value)
                except:
                    return value
        except Exception as e:
            logger.error(f"Cache get error for key {key}: {e}")
            return None
    
    def delete(self, key: str) -> bool:
        """Delete cache key"""
        try:
            return bool(self.client.delete(key))
        except Exception as e:
            logger.error(f"Cache delete error for key {key}: {e}")
            return False
    
    def health_check(self) -> bool:
        """Check Redis health"""
        try:
            return self.client.ping()
        except Exception as e:
            logger.error(f"Redis health check failed: {e}")
            return False

# Singleton instance
cache = CacheManager()
```

## 🔒 Environment Variables

### Development Environment

```bash
# .env.development
# PostgreSQL
DB_HOST=localhost
DB_PORT=5432
DB_NAME=travel_planner_dev
DB_USER=travel_app
DB_PASSWORD=dev_password
DB_ECHO=true
USE_PGBOUNCER=false

# Qdrant
QDRANT_HOST=localhost
QDRANT_PORT=6333
QDRANT_GRPC_PORT=6334
QDRANT_USE_GRPC=false
QDRANT_TIMEOUT=30
QDRANT_POOL_SIZE=10
QDRANT_RETRIES=3

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0
REDIS_MAX_CONNECTIONS=20
SESSION_TTL=3600
CACHE_TTL=300

# Application
ENVIRONMENT=development
DEBUG=true
LOG_LEVEL=DEBUG
```

### Production Environment

```bash
# .env.production
# PostgreSQL (use strong passwords)
DB_HOST=postgres
DB_PORT=5432
DB_NAME=travel_planner
DB_USER=travel_app
DB_PASSWORD=${POSTGRES_STRONG_PASSWORD}
DB_ECHO=false
USE_PGBOUNCER=true

# Qdrant
QDRANT_HOST=qdrant
QDRANT_PORT=6333
QDRANT_GRPC_PORT=6334
QDRANT_USE_GRPC=true
QDRANT_TIMEOUT=30
QDRANT_POOL_SIZE=20
QDRANT_RETRIES=3

# Redis
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=${REDIS_STRONG_PASSWORD}
REDIS_DB=0
REDIS_MAX_CONNECTIONS=50
SESSION_TTL=7200     # 2 hours
CACHE_TTL=600        # 10 minutes

# Application
ENVIRONMENT=production
DEBUG=false
LOG_LEVEL=INFO

# Security
SECRET_KEY=${SECURE_SECRET_KEY}
JWT_SECRET=${JWT_SECRET_KEY}
```

## 📊 Monitoring Configuration

### Database Health Checks

```python
# app/core/health.py
from sqlalchemy import text
from app.core.database import engine
from app.core.vector_database import vector_db
from app.core.cache import cache
import logging

logger = logging.getLogger(__name__)

async def check_postgresql_health() -> dict:
    """Check PostgreSQL database health"""
    try:
        with engine.connect() as conn:
            result = conn.execute(text("SELECT 1")).scalar()
            return {
                "status": "healthy" if result == 1 else "unhealthy",
                "service": "postgresql",
                "details": "Connection successful"
            }
    except Exception as e:
        logger.error(f"PostgreSQL health check failed: {e}")
        return {
            "status": "unhealthy",
            "service": "postgresql", 
            "details": str(e)
        }

async def check_qdrant_health() -> dict:
    """Check Qdrant health"""
    try:
        collections = vector_db.client.get_collections()
        return {
            "status": "healthy",
            "service": "qdrant",
            "details": f"Collections: {len(collections.collections)}"
        }
    except Exception as e:
        logger.error(f"Qdrant health check failed: {e}")
        return {
            "status": "unhealthy",
            "service": "qdrant",
            "details": str(e)
        }

async def check_redis_health() -> dict:
    """Check Redis health"""
    try:
        is_healthy = cache.health_check()
        return {
            "status": "healthy" if is_healthy else "unhealthy",
            "service": "redis",
            "details": "Connection successful" if is_healthy else "Connection failed"
        }
    except Exception as e:
        logger.error(f"Redis health check failed: {e}")
        return {
            "status": "unhealthy",
            "service": "redis",
            "details": str(e)
        }

async def get_system_health() -> dict:
    """Get overall system health"""
    postgres_health = await check_postgresql_health()
    qdrant_health = await check_qdrant_health()
    redis_health = await check_redis_health()
    
    all_healthy = all(
        service["status"] == "healthy" 
        for service in [postgres_health, qdrant_health, redis_health]
    )
    
    return {
        "status": "healthy" if all_healthy else "degraded",
        "timestamp": datetime.utcnow().isoformat(),
        "services": {
            "postgresql": postgres_health,
            "qdrant": qdrant_health,
            "redis": redis_health
        }
    }
```

This database environment configuration provides:

✅ **Development Ready** - Easy local setup with Docker support  
✅ **Production Optimized** - ARM64 specific configurations  
✅ **High Performance** - Connection pooling and memory optimization  
✅ **Monitoring Ready** - Health checks and logging  
✅ **Security Focused** - Password management and secure connections  
✅ **Scalable Architecture** - Ready for horizontal scaling  
✅ **ARM64 Optimized** - Specifically tuned for Oracle Cloud ARM64  
✅ **Maintainable** - Clear configuration management and documentation