# System Components Architecture

This document provides detailed information about the Travel Planner application's system components, their responsibilities, interactions, and implementation patterns using FastAPI + LangGraph.

## 🏗️ Component Overview

The Travel Planner application follows a modern multi-agent architecture with clearly defined component boundaries and responsibilities. Each component is designed to be loosely coupled, highly cohesive, and independently deployable.

### High-Level Component Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        EXTERNAL SERVICES                            │
├─────────────────────────────────────────────────────────────────────┤
│  OpenAI GPT-5  │   Amadeus    │  Google Maps │  OpenWeather │ OAuth │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                             External API Gateway
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                        PRESENTATION LAYER                           │
├─────────────────────────────────────────────────────────────────────┤
│                         React Frontend                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │    Auth     │  │    Plans    │  │   Profile   │  │   Agents    │ │
│  │ Components  │  │ Components  │  │ Components  │  │ Components  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │    State    │  │   Routing   │  │  WebSocket  │  │     UI      │ │
│  │ Management  │  │   System    │  │   Client    │  │ Framework   │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                        HTTP/HTTPS REST API + WebSocket
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                        APPLICATION LAYER                            │
├─────────────────────────────────────────────────────────────────────┤
│                      FastAPI + LangGraph Backend                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │ Auth API    │  │ Agent API   │  │ Plan API    │  │ User API    │ │
│  │             │  │             │  │             │  │             │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │ Orchestrator│  │ Agent Graph │  │ WebSocket   │  │   Cache     │ │
│  │             │  │   Engine    │  │   Manager   │  │   Layer     │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                        Database Queries + Vector Search
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA LAYER                                  │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │ PostgreSQL  │  │   Qdrant    │  │    Redis    │  │   Celery    │ │
│  │ (Relational)│  │ (Vector DB) │  │   (Cache)   │  │   (Tasks)   │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

## 🎯 Core Components

### 1. FastAPI Application Core

**Purpose**: Main application server and API gateway  
**Technology**: FastAPI, Uvicorn, Python 3.11+  
**Responsibilities**:
- HTTP request/response handling
- API routing and middleware
- Input validation and serialization
- Authentication and authorization
- OpenAPI documentation generation

```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from starlette.middleware.sessions import SessionMiddleware

app = FastAPI(
    title="Travel Planner API",
    description="Multi-agent travel planning platform",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# Middleware stack
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)

app.add_middleware(TrustedHostMiddleware, allowed_hosts=["*"])
app.add_middleware(SessionMiddleware, secret_key="your-secret-key")
```

### 2. LangGraph Agent System

**Purpose**: Multi-agent orchestration and workflow management  
**Technology**: LangGraph, LangChain, OpenAI GPT-5  
**Responsibilities**:
- Agent workflow definition and execution
- Inter-agent communication and coordination
- State management across agent interactions
- Tool integration and function calling

```python
from langgraph.graph import StateGraph, END
from langgraph.graph.message import MessageGraph
from typing import TypedDict, Annotated, List

class AgentState(TypedDict):
    messages: Annotated[List[dict], "The conversation messages"]
    current_agent: str
    travel_context: dict
    plan_data: dict

def create_travel_agent_graph():
    workflow = StateGraph(AgentState)
    
    # Add agent nodes
    workflow.add_node("planner_agent", planner_agent_node)
    workflow.add_node("booking_agent", booking_agent_node)
    workflow.add_node("recommendation_agent", recommendation_agent_node)
    workflow.add_node("optimization_agent", optimization_agent_node)
    
    # Define agent flow
    workflow.set_entry_point("planner_agent")
    workflow.add_edge("planner_agent", "recommendation_agent")
    workflow.add_edge("recommendation_agent", "optimization_agent")
    workflow.add_edge("optimization_agent", "booking_agent")
    workflow.add_edge("booking_agent", END)
    
    return workflow.compile()
```

### 3. Authentication & Authorization System

**Purpose**: User authentication, session management, and access control  
**Technology**: FastAPI Security, JWT, OAuth2, Passlib  
**Responsibilities**:
- User authentication (JWT + OAuth2)
- Session management
- Role-based access control
- API key management for external services

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import JWTError, jwt
from passlib.context import CryptContext
from datetime import datetime, timedelta

security = HTTPBearer()
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

SECRET_KEY = "your-secret-key"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

def create_access_token(data: dict, expires_delta: timedelta = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    try:
        payload = jwt.decode(credentials.credentials, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: str = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=401, detail="Invalid token")
        return user_id
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

### 4. Database Layer

**Purpose**: Data persistence and vector search capabilities  
**Technology**: SQLAlchemy, Qdrant, asyncpg  
**Responsibilities**:
- Relational data management (PostgreSQL)
- Vector search and similarity matching (Qdrant)
- Database migrations and schema management
- Connection pooling and optimization

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker, declarative_base
from qdrant_client import QdrantClient
from qdrant_client.models import VectorParams, Distance

# PostgreSQL setup
DATABASE_URL = "postgresql+asyncpg://user:pass@localhost/travel_planner"
engine = create_async_engine(DATABASE_URL)
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

Base = declarative_base()

# Qdrant setup
qdrant_client = QdrantClient(
    url="http://localhost:6333",
    api_key="your-api-key"
)

# Vector collection for travel embeddings
qdrant_client.create_collection(
    collection_name="travel_embeddings",
    vectors_config=VectorParams(size=1536, distance=Distance.COSINE)
)

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session
```

### 5. WebSocket Manager

**Purpose**: Real-time communication for agent interactions  
**Technology**: FastAPI WebSocket, asyncio  
**Responsibilities**:
- Real-time agent status updates
- Live travel plan updates
- User notification delivery
- Connection management

```python
from fastapi import WebSocket, WebSocketDisconnect
from typing import Dict, Set
import json
import asyncio

class ConnectionManager:
    def __init__(self):
        self.active_connections: Dict[str, Set[WebSocket]] = {}
    
    async def connect(self, websocket: WebSocket, user_id: str):
        await websocket.accept()
        if user_id not in self.active_connections:
            self.active_connections[user_id] = set()
        self.active_connections[user_id].add(websocket)
    
    def disconnect(self, websocket: WebSocket, user_id: str):
        if user_id in self.active_connections:
            self.active_connections[user_id].discard(websocket)
            if not self.active_connections[user_id]:
                del self.active_connections[user_id]
    
    async def send_personal_message(self, message: dict, user_id: str):
        if user_id in self.active_connections:
            for connection in self.active_connections[user_id]:
                await connection.send_text(json.dumps(message))

manager = ConnectionManager()

@app.websocket("/ws/{user_id}")
async def websocket_endpoint(websocket: WebSocket, user_id: str):
    await manager.connect(websocket, user_id)
    try:
        while True:
            data = await websocket.receive_text()
            # Handle incoming messages
            await handle_websocket_message(data, user_id)
    except WebSocketDisconnect:
        manager.disconnect(websocket, user_id)
```

### 6. External API Integration Layer

**Purpose**: Third-party service integration and management  
**Technology**: httpx, asyncio, tenacity  
**Responsibilities**:
- API client management
- Rate limiting and retry logic
- Response caching
- Error handling and fallbacks

```python
import httpx
from tenacity import retry, stop_after_attempt, wait_exponential
from typing import Optional
import asyncio

class ExternalAPIClient:
    def __init__(self):
        self.openai_client = httpx.AsyncClient(
            base_url="https://api.openai.com/v1",
            headers={"Authorization": f"Bearer {OPENAI_API_KEY}"}
        )
        self.amadeus_client = httpx.AsyncClient(
            base_url="https://api.amadeus.com/v1",
            headers={"Authorization": f"Bearer {AMADEUS_API_KEY}"}
        )
    
    @retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
    async def call_openai_api(self, payload: dict) -> dict:
        response = await self.openai_client.post("/chat/completions", json=payload)
        response.raise_for_status()
        return response.json()
    
    @retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=10))
    async def search_flights(self, origin: str, destination: str, departure_date: str) -> dict:
        params = {
            "originLocationCode": origin,
            "destinationLocationCode": destination,
            "departureDate": departure_date,
            "adults": 1
        }
        response = await self.amadeus_client.get("/shopping/flight-offers", params=params)
        response.raise_for_status()
        return response.json()
```

## 🔄 Component Interactions

### Request Flow Example: Travel Plan Creation

1. **Frontend** → Sends POST request to `/api/plans/create`
2. **FastAPI Router** → Validates request and routes to plan service
3. **Plan Service** → Initiates LangGraph agent workflow
4. **Orchestrator Agent** → Coordinates multiple specialized agents
5. **Recommendation Agent** → Calls external APIs for suggestions
6. **Optimization Agent** → Processes and optimizes recommendations
7. **Database Layer** → Stores plan data and embeddings
8. **WebSocket Manager** → Sends real-time updates to frontend
9. **Frontend** → Displays updated plan to user

### Agent Communication Pattern

```python
async def agent_communication_flow(plan_request: PlanRequest):
    # Initialize agent state
    initial_state = {
        "messages": [{"role": "user", "content": plan_request.description}],
        "current_agent": "planner",
        "travel_context": plan_request.context,
        "plan_data": {}
    }
    
    # Execute agent graph
    agent_graph = create_travel_agent_graph()
    final_state = await agent_graph.ainvoke(initial_state)
    
    # Return processed plan
    return final_state["plan_data"]
```

## 🛠️ Development Dependencies

```python
# requirements.txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
langgraph==0.0.40
langchain==0.1.0
sqlalchemy[asyncio]==2.0.23
asyncpg==0.29.0
qdrant-client==1.6.9
redis[hiredis]==5.0.1
celery==5.3.4
httpx==0.25.2
tenacity==8.2.3
pydantic[email]==2.5.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
```

## 📊 Component Status

| Component | Status | Implementation | Testing |
|-----------|--------|----------------|---------|
| FastAPI Core | ✅ Ready | ⏳ Pending | ⏳ Pending |
| LangGraph Agents | ✅ Ready | ⏳ Pending | ⏳ Pending |
| Authentication | ✅ Ready | ⏳ Pending | ⏳ Pending |
| Database Layer | ✅ Ready | ⏳ Pending | ⏳ Pending |
| WebSocket Manager | ✅ Ready | ⏳ Pending | ⏳ Pending |
| External APIs | ✅ Ready | ⏳ Pending | ⏳ Pending |

## 🔗 Related Documentation

- [Backend Implementation Guide](../backend/README.md)
- [Agent System Design](../agents/README.md)
- [Database Schema](../backend/02-database-schema.md)
- [Deployment Guide](../deployment/README.md)
- [Monitoring System](../backend/03-monitoring-system.md)

---

**Last Updated**: September 4, 2025  
**Technology Stack**: FastAPI + LangGraph + PostgreSQL + Qdrant  
**Target Deployment**: Oracle Cloud ARM64 (Ampere A1)