# CRUD Master - Docker Microservices Architecture

A hands-on learning project introducing **containerization concepts** by building a complete microservices architecture using Docker and Docker Compose.

**Status:** ✅ **Complete** | **Audit Ready** | **April 21, 2026**

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Learning Objectives](#learning-objectives)
- [Mandatory Requirements](#mandatory-requirements)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Project Structure](#project-structure)
- [Services Overview](#services-overview)
- [Configuration](#configuration)
- [API Endpoints](#api-endpoints)
- [Testing with Postman](#testing-with-postman)
- [Docker Management](#docker-management)
- [Key Concepts](#key-concepts)
- [Best Practices & Tips](#best-practices--tips)
- [Troubleshooting](#troubleshooting)
- [Submission & Audit](#submission--audit)

---

## 🎯 Project Overview

**CRUD Master** is a containerized microservices-based movie inventory and billing system demonstrating real-world containerization concepts:

- **Multi-Container Architecture**: 6 independent Docker containers orchestrated with Docker Compose
- **Service Isolation**: Each service runs in its own container for optimal performance
- **Reverse Proxy Pattern**: API Gateway as single entry point via port 3000
- **Asynchronous Processing**: RabbitMQ for decoupled inter-service communication
- **Data Persistence**: PostgreSQL databases with named volumes
- **Infrastructure as Code**: Entire system defined and managed by docker-compose.yml
- **Network Segmentation**: Custom bridge network with controlled external access

---

## 🎓 Learning Objectives

By completing this project, you will:

1. **Design** a multi-container microservices architecture using Docker and Docker Compose
2. **Implement** isolated services with PostgreSQL databases, RabbitMQ messaging, and an API Gateway
3. **Configure** Docker networks and volumes to manage inter-service communication and persistent data
4. **Optimize** Dockerfiles and container builds for performance and maintainability
5. **Document** project setup, configuration, and usage clearly in README.md
6. **Manage** multi-container applications with automatic restart policies and health checks
7. **Understand** containerization best practices and anti-patterns

---

## 📋 Mandatory Requirements

### Docker Containers (6 total)

| Container           | Service        | Base Image                    | Port       | Purpose                     |
| ------------------- | -------------- | ----------------------------- | ---------- | --------------------------- |
| **inventory-db**    | PostgreSQL     | Debian bullseye (penultimate) | 5432       | Inventory database server   |
| **billing-db**      | PostgreSQL     | Debian bullseye (penultimate) | 5432       | Billing database server     |
| **inventory-app**   | Flask API      | Debian bullseye (penultimate) | 8080       | Movie CRUD operations       |
| **billing-app**     | Flask Consumer | Debian bullseye (penultimate) | 8080       | RabbitMQ consumer           |
| **rabbit-queue**    | RabbitMQ       | Alpine (penultimate)          | 5672/15672 | Message queue broker        |
| **api-gateway-app** | Flask Gateway  | Debian bullseye (penultimate) | 3000       | Reverse proxy & entry point |

**Container Requirements:**

- ✅ Each service has its own container (optimal performance)
- ✅ Containers automatically restart on failure (restart policy)
- ✅ Base images: **Penultimate stable version** (latest stable minus one)
- ✅ Custom Docker images (NOT from Docker Hub pre-built images, except base OS)
- ✅ Service name = Docker image name with explicit version (inventory-database:1.0.0, billing-database:1.0.0, etc.)
- ✅ External access only through api-gateway-app:3000

### Docker Volumes (3 total)

| Volume              | Purpose                    | Container Path              | Data Persistence |
| ------------------- | -------------------------- | --------------------------- | ---------------- |
| **inventory-db**    | Inventory database storage | /var/lib/postgresql/13/main | Permanent        |
| **billing-db**      | Billing database storage   | /var/lib/postgresql/13/main | Permanent        |
| **api-gateway-app** | API Gateway logs           | /var/logs/api-gateway       | Permanent        |

**Volume Requirements:**

- ✅ Named volumes for data persistence
- ✅ Data survives container deletion and restart
- ✅ Automatic volume creation via docker-compose

### Docker Network

| Requirement                | Specification             |
| -------------------------- | ------------------------- |
| **Type**                   | Custom bridge network     |
| **Scope**                  | All containers connected  |
| **External Access**        | Only api-gateway-app:3000 |
| **Internal Communication** | Container DNS resolution  |

**Network Requirements:**

- ✅ Custom bridge network connecting all services
- ✅ Services communicate by container name (DNS)
- ✅ Only api-gateway-app exposed to external requests
- ✅ All other services accessible only within Docker network

### Infrastructure as Code

- ✅ **docker-compose.yml** orchestrates all services, volumes, and networks
- ✅ **.env file** contains all credentials and configuration
- ✅ **.gitignore** includes .env (NO credentials in repository)
- ✅ All infrastructure reproducible with single `docker-compose up -d` command

---

## 🏗️ Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                      Linux Virtual Machine                        │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                   Docker Environment                       │ │
│  │                                                             │ │
│  │  ┌───────────────────────────────────────────────────────┐ │ │
│  │  │          Docker Network: bridge                      │ │ │
│  │  │                                                       │ │ │
│  │  │  ┌─────────┐  ┌─────────┐  ┌───────────┐           │ │ │
│  │  │  │inventory│  │ billing │  │  rabbit-  │           │ │ │
│  │  │  │   db    │  │   db    │  │  queue    │           │ │ │
│  │  │  │(PgSQL)  │  │(PgSQL)  │  │(RabbitMQ) │           │ │ │
│  │  │  │ :5432   │  │ :5432   │  │ :5672     │           │ │ │
│  │  │  └────┬────┘  └────┬────┘  └────┬──────┘           │ │ │
│  │  │       │             │            │                 │ │ │
│  │  │    ┌──▼────────┐    │  ┌─────────▼──┐             │ │ │
│  │  │    │inventory- │    │  │ billing-app│             │ │ │
│  │  │    │   app     │    │  │  (Consumer)│             │ │ │
│  │  │    │  :8080    │    │  │   :8080    │             │ │ │
│  │  │    └──┬────────┘    │  └─────────┬──┘             │ │ │
│  │  │       │             │            │                 │ │ │
│  │  │       └──────────┬──┴────────────┘                 │ │ │
│  │  │                  │                                 │ │ │
│  │  │           ┌──────▼────────┐                       │ │ │
│  │  │           │ api-gateway-  │                       │ │ │
│  │  │           │    app        │                       │ │ │
│  │  │           │   :3000       │                       │ │ │
│  │  │           │  (EXPOSED)    │                       │ │ │
│  │  │           └───────────────┘                       │ │ │
│  │  │                                                    │ │ │
│  │  │  Volumes:                                         │ │ │
│  │  │  • inventory-db, billing-db, api-gateway-app     │ │ │
│  │  │                                                    │ │ │
│  │  └───────────────────────────────────────────────────┘ │ │
│  │                                                         │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### Communication Patterns

**External Access:**

```
Client (Postman/Browser)
        │
        │ HTTP :3000
        ▼
┌──────────────────────┐
│  api-gateway-app     │  ◄── Only exposed port
│  (Reverse Proxy)     │
└──────────────────────┘
```

**Internal Service Communication:**

```
Within Docker Network (all containers can reach each other by name):

api-gateway-app ──► inventory-app:8080    (GET/POST /api/movies/*)
                ──► rabbit-queue:5672     (POST /api/billing)

inventory-app ──► inventory-db:5432       (PostgreSQL)
billing-app ──► billing-db:5432           (PostgreSQL)
           ──► rabbit-queue:5672          (RabbitMQ consumer)
```

**Benefits of This Architecture:**

- **Single Entry Point**: `localhost:3000` or `192.168.1.185:3000` (with port forwarding)
- **Service Isolation**: Each service independent, can restart/update separately
- **Data Persistence**: Volumes survive container restarts
- **Easy Scaling**: Add more containers/replicas with docker-compose
- **Network Security**: Only gateway exposed, backend services internal
- **Service Discovery**: Container DNS names (no hardcoded IPs)

---

### Data Flow

**CRUD Operations (Movies):**

```

Client → API Gateway (3000) [Reverse Proxy]
→ Inventory API (8080) [HTTP Proxy Forward]
→ PostgreSQL movies DB
↓ (Response back through same path)

```

**Billing Operations (Orders):**

```

Client → API Gateway (3000)
→ RabbitMQ Queue
↓ (Asynchronously)
Billing API Consumer reads queue
→ PostgreSQL orders DB
↓ (Message durability - survives crashes)

```

---

## 📦 Prerequisites

### System Requirements

- **Processor**: Intel or ARM (M1/M2/M3 Mac compatible)
- **RAM**: 8GB minimum (16GB recommended)
- **Disk**: 20GB free space
- **OS**: macOS, Linux, or Windows (with WSL2)

### Required Tools

Verify installation before starting:

**Docker Desktop** (latest)

```bash
docker --version
docker-compose --version
```

**Postman** (Latest)

- Download from [postman.com](https://www.postman.com/downloads/)
- Or use web version at [web.postman.co](https://web.postman.co)

**Python** (v3.10+)

```bash
python3 --version
```

**Git** (for version control)

```bash
git --version
```

---

## 🚀 Quick Start

### Step 1: Clone and Navigate

```bash
cd /Users/saddam.hussain/Desktop/play-with-containers
```

### Step 2: Create Environment File

```bash
# Copy example if exists
cp .env.example .env

# Or create .env file with database credentials
```

### Step 3: Start All Docker Containers

```bash
docker-compose up -d
```

This will:

- Build all 6 Docker images
- Create named volumes for data persistence
- Create custom bridge network (microservices-net)
- Start all 6 containers in detached mode

**Expected time**: 2-3 minutes

### Step 4: Verify Containers Are Running

```bash
docker-compose ps
```

Expected output shows all 6 containers with "Up" status.

### Step 5: Verify Health

```bash
curl http://localhost:3000/health
```

### Step 6: Import Postman Collection

1. Open Postman
2. Click **File** → **Import**
3. Select `CRUD_Master.postman_collection.json`
4. Run collection: Click ⏶ **Run Collection**

### Step 7: Test with cURL

```bash
# Get all movies
curl http://localhost:3000/api/movies/

# Create a movie
curl -X POST http://localhost:3000/api/movies/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Test", "director": "Director", "release_year": 2024}'

# Create billing order
curl -X POST http://localhost:3000/api/billing/ \
  -H "Content-Type: application/json" \
  -d '{"user_id": "user123", "number_of_items": "5", "total_amount": "99.99"}'
```

---

## 📂 Project Structure

```
play-with-containers/
│
├── README.md                           # Main documentation (this file)
├── entireproject.md                    # Complete 16-section project documentation
│
├── docker-compose.yml                  # Docker Compose orchestration (all 6 services)
├── .env                                # Environment variables (gitignored)
├── .env.example                        # Example environment file
├── .gitignore                          # Git ignore rules
│
├── srcs/                               # Application source code (6 containers)
│   ├── api-gateway-app/                # API Gateway service
│   │   ├── Dockerfile                  # Container definition
│   │   ├── server.py                   # Entry point (Flask app)
│   │   ├── requirements.txt            # Python dependencies
│   │   └── app/
│   │       ├── __init__.py             # Flask app factory
│   │       └── routes.py               # Gateway routes
│   │
│   ├── inventory-app/                  # Inventory API service
│   │   ├── Dockerfile                  # Container definition
│   │   ├── server.py                   # Entry point
│   │   ├── requirements.txt            # Dependencies
│   │   └── app/
│   │       ├── __init__.py             # App factory
│   │       ├── routes.py               # CRUD endpoints
│   │       ├── models.py               # SQLAlchemy models
│   │       └── db.py                   # Database setup
│   │
│   ├── billing-app/                    # Billing API service
│   │   ├── Dockerfile                  # Container definition
│   │   ├── server.py                   # Entry point (RabbitMQ consumer)
│   │   ├── requirements.txt            # Dependencies
│   │   └── app/
│   │       ├── __init__.py             # App factory
│   │       ├── consumer.py             # RabbitMQ consumer
│   │       ├── models.py               # SQLAlchemy models
│   │       └── db.py                   # Database setup
│   │
│   ├── inventory-db/                   # PostgreSQL container
│   │   ├── Dockerfile                  # Container definition
│   │   └── entrypoint.sh               # Initialization
│   │
│   ├── billing-db/                     # PostgreSQL container
│   │   ├── Dockerfile                  # Container definition
│   │   └── entrypoint.sh               # Initialization
│   │
│   └── rabbit-queue/                   # RabbitMQ message queue container
│       ├── Dockerfile                  # Container definition
│       └── entrypoint.sh               # Initialization
│
├── CRUD_Master.postman_collection.json # Postman test collection (18+ endpoints)
├── openapi.yaml                        # OpenAPI specification
│
└── (Docker Volumes - created automatically)
    ├── inventory-db                    # PostgreSQL inventory data
    ├── billing-db                      # PostgreSQL billing data
    └── api-gateway-volume              # API Gateway logs
```

---

## 🔧 Services Overview

### 1. API Gateway (api-gateway-app Container)

**Port**: 3000 (exposed to host)  
**Purpose**: Reverse proxy - single entry point for all client requests  
**Technology**: Python + Flask  
**Pattern**: Reverse Proxy + HTTP Proxy within Docker network

#### Features:

- **Reverse Proxy Pattern**: Receives all client requests at single URL (localhost:3000)
- **HTTP Proxy Implementation**: Forwards `/api/movies/*` requests to inventory-app container
- **Service Abstraction**: Clients don't know about internal Inventory API container address
- **Message Publisher**: Routes `/api/billing` to RabbitMQ container for async processing
- **Health check endpoint**: `/health` for monitoring
- **Docker Network Integration**: Uses container DNS (inventory-app:8080, rabbit-queue:5672)

#### Configuration:

```env
GATEWAY_PORT=3000
INVENTORY_APP=inventory-app
INVENTORY_PORT=8080
RABBITMQ_HOST=rabbit-queue
RABBITMQ_PORT=5672
```

---

### 2. Inventory API (inventory-app Container)

**Port**: 8080 (internal only, accessed via api-gateway-app)  
**Purpose**: Movie CRUD operations  
**Database**: PostgreSQL (inventory-db container)

#### Endpoints:

| Method | Endpoint              | Purpose           |
| ------ | --------------------- | ----------------- |
| POST   | `/api/movies`         | Create movie      |
| GET    | `/api/movies`         | List all movies   |
| GET    | `/api/movies?title=X` | Filter by title   |
| GET    | `/api/movies/{id}`    | Get single movie  |
| PUT    | `/api/movies/{id}`    | Update movie      |
| DELETE | `/api/movies/{id}`    | Delete movie      |
| DELETE | `/api/movies`         | Delete all movies |

#### Data Model:

```python
Movie:
  - id (auto-generated)
  - title (string)
  - description (text)
  - genre (string)
  - release_year (integer)
  - rating (float)
  - duration (integer)
  - available_copies (integer)
  - created_at (timestamp)
  - updated_at (timestamp)
```

---

### 3. Billing API (billing-app Container)

**Purpose**: Async order processing  
**Transport**: RabbitMQ message queue (rabbit-queue container)  
**Database**: PostgreSQL (billing-db container)

#### Features:

- Consumes messages from `billing_queue`
- Processes orders asynchronously
- Durable queue (survives container restart)
- Automatic message replay on container restart

#### RabbitMQ Configuration:

```env
RABBITMQ_HOST=rabbit-queue
RABBITMQ_PORT=5672
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest
RABBITMQ_QUEUE=billing_queue
RABBITMQ_MGMT_PORT=15672
```

---

## 🔗 API Endpoints

### Base URL

**Local Machine:**

```
http://localhost:3000
```

**Optional - With VirtualBox Port Forwarding:**

If running Docker in a VM, configure port forwarding from host IP to VM (example: `192.168.1.x:3000 → 10.0.2.15:3000`)

### Movie Management (Full CRUD)

#### Create Movie

```http
POST /api/movies
Content-Type: application/json

{
  "title": "Inception",
  "description": "A mind-bending thriller",
  "genre": "Sci-Fi",
  "release_year": 2010,
  "rating": 8.8,
  "duration": 148,
  "available_copies": 5
}
```

**Response** (201 Created or 200 OK):

```json
{
  "success": true,
  "message": "Movie created successfully",
  "movie": {
    "id": 1,
    "title": "Inception",
    "created_at": "2024-04-01T10:30:00",
    "updated_at": "2024-04-01T10:30:00"
  }
}
```

#### Get All Movies

```http
GET /api/movies
```

**Response** (200 OK):

```json
{
  "success": true,
  "count": 5,
  "movies": [...]
}
```

#### Filter Movies by Title

```http
GET /api/movies?title=inception
```

#### Get Single Movie

```http
GET /api/movies/1
```

#### Update Movie

```http
PUT /api/movies/1
Content-Type: application/json

{
  "rating": 9.0,
  "available_copies": 3
}
```

#### Delete Movie

```http
DELETE /api/movies/1
```

#### Delete All Movies

```http
DELETE /api/movies
```

### Order Management (Async)

#### Create Order

```http
POST /api/billing
Content-Type: application/json

{
  "user_id": "user123",
  "number_of_items": "5",
  "total_amount": "99.99"
}
```

**Response** (200 OK - Immediate):

```json
{
  "message": "Order queued successfully for processing",
  "order": {
    "user_id": "user123",
    "number_of_items": "5",
    "total_amount": "99.99"
  }
}
```

**Note**: Returns 200 immediately (async). Order processed by Billing API consumer in background.

### Health Check

#### Gateway Health

```http
GET /health
```

**Response** (200 OK):

```json
{
  "status": "healthy",
  "service": "API Gateway"
}
```

---

## 🗄️ Database Setup

### PostgreSQL Structure

Two separate databases running in separate PostgreSQL containers:

#### 1. Movies Database (inventory-db Container)

**Name**: inventory_db  
**User**: inventory_user  
**Port**: 5432 (internal)

**Access**:

```bash
docker exec -it inventory-db psql -U inventory_user -d inventory_db
```

- `movies` - Movie inventory

---

#### 2. Orders Database (billing-db Container)

**Name**: billing_db  
**User**: billing_user  
**Port**: 5432 (internal)

**Access**:

```bash
docker exec -it billing-db psql -U billing_user -d billing_db
```

**Tables**:

- `orders` - Order records

---

### Access Databases via Docker

**Inventory Database (PostgreSQL):**

```bash
# Access inventory-db container
docker exec -it inventory-db psql -U inventory_user -d inventory_db

# Inside psql:
\dt                        # List all tables
SELECT * FROM movies;      # View all movies
```

**Billing Database (PostgreSQL):**

```bash
# Access billing-db container
docker exec -it billing-db psql -U billing_user -d billing_db

# Inside psql:
\dt                        # List all tables
SELECT * FROM orders;      # View all orders
```

**View Database Credentials:**

```bash
cat .env | grep DB
```

---

## 🧪 Testing with Postman

### Import Collection

1. **Open Postman**
2. **File** → **Import** → Select `CRUD_Master.postman_collection.json`
3. **Base URL**: Set to `http://localhost:3000` (or your deployment URL)
4. Collection appears in left sidebar

### Collection Structure

```
📦 CRUD Master - Audit Validation Suite
├── 📂 Movies
│   ├── POST /api/movies - create movie
│   ├── GET /api/movies - all movies
│   ├── GET /api/movies?title=X - filter
│   ├── GET /api/movies/{id} - get one
│   ├── PUT /api/movies/{id} - update
│   ├── DELETE /api/movies/{id} - delete one
│   └── DELETE /api/movies - delete all
├── 📂 Billing
│   └── POST /api/billing - create order
└── 📂 Health
    └── GET /health - gateway health
```

### Run Tests

**Option 1: Run Full Collection**

1. Click ⏶ **Run Collection** button
2. Select **CRUD Master - Audit Validation Suite**
3. Click **Run**
4. View test results

**Option 2: Run Individual Request**

1. Click request name
2. Click **Send**
3. View response in **Body** tab
4. Check **Tests** tab for assertions

### Test Results

Each endpoint includes automated test scripts that verify:

- ✅ Status code (200 OK)
- ✅ Response structure
- ✅ Data validation
- ✅ Required fields present

---

## ⭐ Key Features

### 1. Microservices Architecture

- **Independent services** with separate databases
- **Loose coupling** via API Gateway and message queue
- **Easy scaling** - services can be deployed independently

### 2. Message Queue Resilience

```
Scenario: Billing API crashes while processing order

Before crash:
  Client → Gateway → RabbitMQ → Billing API → Orders DB

Order received → Queue (durable)

Crash happens:
  Order stays in RabbitMQ queue (not lost)

Billing API restarts:
  Reads all messages from queue in order
  Processes queued orders
  Inserts into database

Result: Order data is never lost (eventual consistency)
```

### 3. Reverse Proxy Pattern

**Gateway acts as reverse proxy between clients and backend services:**

```
Benefits:
- Single entry point (API Gateway at localhost:3000 or 192.168.1.185:3000)
- Clients unaware of backend service container addresses
- Easy to scale (add multiple backend container instances)
- Can implement load balancing
- Security boundary (hide internal Docker network architecture)
- Centralized authentication/authorization
- Request logging and monitoring
```

**How it works:**

```
1. Client sends request to Gateway (only URL client knows)
2. Gateway receives request at common entry point
3. Gateway routes to appropriate backend service
4. Backend processes request
5. Gateway returns response to client
6. Client never directly contacts backend
```

### 4. HTTP Proxy Pattern

**Implementation detail of how reverse proxy forwards requests within Docker network:**

```
HTTP Proxy forwarding via Docker DNS:
- Gateway uses requests library to forward
- Preserves HTTP method (GET, POST, PUT, DELETE)
- Preserves headers (Content-Type, etc)
- Preserves request body (JSON payload)
- Preserves query parameters (?title=X)
- Uses Docker container DNS for service discovery

Example:
  Client → Gateway Container:
    POST /api/movies
      with {"title": "Movie"}

  Gateway forwards to Inventory Container:
    POST http://inventory-app:8080/api/movies
      with same headers and body
      (inventory-app resolves via Docker DNS)

  Response flows back through Gateway to Client
```

### 5. Database Isolation

- Each service has **own database**
- No direct database access from other services
- Data consistency through API contracts

### 6. Infrastructure as Code

- **docker-compose.yml** defines entire infrastructure
- All services, volumes, and networks in one file
- Reproducible across machines
- Version controlled
- Easy to recreate with one command

### 7. Comprehensive Testing

- **Postman collection** with 9 endpoints
- **Automated test scripts** for each endpoint
- **Pre-request scripts** for setup
- Export results for reporting

---

## 🔍 Troubleshooting

### Problem: Containers Won't Start

**Error**: Container startup failure or exit codes

**Solution**:

```bash
# Check container logs
docker-compose logs api-gateway-app
docker-compose logs inventory-app
docker-compose logs billing-app

# Rebuild containers
docker-compose down
docker-compose build --no-cache
docker-compose up -d

# Check specific container status
docker ps -a
```

---

### Problem: API Gateway Returns 502 (Bad Gateway)

**Error**: "Inventory API is unreachable"

**Solutions**:

```bash
# 1. Check all containers are running
docker-compose ps

# 2. Verify inventory-app is listening on 8080
docker exec api-gateway-app curl http://inventory-app:8080/api/movies/

# 3. Check network connectivity
docker exec api-gateway-app ping inventory-app

# 4. View inventory-app logs
docker-compose logs inventory-app

# 5. Restart inventory-app
docker-compose restart inventory-app
```

---

### Problem: Billing Orders Not Processed

**Error**: Orders sent but not in database

**Solutions**:

```bash
# 1. Check RabbitMQ connection
docker-compose logs billing-app

# 2. Verify billing-app container is running
docker-compose ps billing-app

# 3. Check RabbitMQ is accessible
docker exec billing-app curl -u guest:guest http://rabbit-queue:15672/api/queues

# 4. Check queue messages
docker exec rabbit-queue rabbitmqctl list_queues

# 5. Restart billing-app
docker-compose restart billing-app
```

---

### Problem: Database Connection Error

**Error**: "Could not connect to database"

**Solutions**:

```bash
# 1. Check PostgreSQL containers are running
docker-compose ps inventory-db billing-db

# 2. Verify databases exist
docker exec inventory-db psql -U inventory_user -d inventory_db -c "\l"

# 3. Check credentials in .env
cat .env | grep -i database

# 4. Test connectivity from app containers
docker exec inventory-app psql postgresql://inventory_user:PASSWORD@inventory-db:5432/inventory_db -c "SELECT 1"

# 5. Restart database containers
docker-compose restart inventory-db billing-db
```

---

### Problem: Docker Commands Not Found

**Error**: "docker: command not found" or "docker-compose: command not found"

**Solution**:

```bash
# Verify Docker is installed
docker --version
docker-compose --version

# If not installed, install Docker Desktop:
# macOS: https://www.docker.com/products/docker-desktop
# Linux: https://docs.docker.com/engine/install/
# Windows: https://www.docker.com/products/docker-desktop

# If already installed, check PATH
which docker
which docker-compose
```

---

## 📖 Additional Documentation

- **[entireproject.md](entireproject.md)** - Complete 16-section project documentation
- **[openapi.yaml](openapi.yaml)** - OpenAPI/Swagger specification
- **[CRUD_Master.postman_collection.json](CRUD_Master.postman_collection.json)** - Postman test collection (18+ endpoints)

---

## 📋 Configuration

### Environment Variables (.env)

Create a `.env` file in the project root with database credentials:

```env
# Inventory Database
INVENTORY_DB_USER=inventory_user
INVENTORY_DB_PASSWORD=your_secure_password
INVENTORY_DB_NAME=inventory_db

# Billing Database
BILLING_DB_USER=billing_user
BILLING_DB_PASSWORD=your_secure_password
BILLING_DB_NAME=billing_db

# RabbitMQ
RABBITMQ_USER=guest
RABBITMQ_PASSWORD=guest
RABBITMQ_HOST=rabbit-queue
RABBITMQ_PORT=5672

# API Gateway
GATEWAY_PORT=3000
GATEWAY_HOST=0.0.0.0
```

**IMPORTANT**: Add `.env` to `.gitignore` to prevent credentials from being committed.

### Docker Network Configuration

Network name: `microservices-net` (bridge driver)

- Created automatically by docker-compose
- All containers connected for inter-service communication
- Service discovery via DNS (container names resolve to IPs)
- Only `api-gateway-app` port 3000 exposed to host

### Docker Volumes Configuration

| Volume Name       | Mount Path                    | Purpose                    |
| ----------------- | ----------------------------- | -------------------------- |
| `inventory-db`    | `/var/lib/postgresql/13/main` | Inventory database storage |
| `billing-db`      | `/var/lib/postgresql/13/main` | Billing database storage   |
| `api-gateway-app` | `/var/logs/api-gateway`       | API Gateway logs           |

Volumes are created automatically by docker-compose and persist data between container restarts.

---

## 🛠️ Docker Management

### View Service Logs

```bash
# API Gateway logs
docker-compose logs api-gateway-app

# Inventory API logs
docker-compose logs inventory-app

# Billing API logs
docker-compose logs billing-app

# RabbitMQ logs
docker-compose logs rabbit-queue

# Database logs

# Follow logs in real-time
docker-compose logs -f
```

### Check Service Status

```bash
# Check all containers
docker-compose ps

# Check specific container
docker-compose ps api-gateway-app

# Get detailed info about container
docker inspect api-gateway-app
```

### Manually Restart Services

```bash
# Restart all services
docker-compose restart

# Restart specific service
docker-compose restart inventory-app
docker-compose restart billing-app
docker-compose restart api-gateway-app

# Restart and rebuild specific service
docker-compose down api-gateway-app
docker-compose up -d api-gateway-app
```

---

## 🎓 Key Concepts

### Docker & Containerization

**Container**: A lightweight, standalone executable package that bundles code, runtime, system tools, and dependencies. Containers share the OS kernel but are isolated from each other.

**Docker Image**: A read-only template containing all instructions to create a container. Built from a Dockerfile using layers (each command = 1 layer).

**Dockerfile**: A text file with commands to assemble a Docker image:

- `FROM`: Base image (Alpine or Debian in this project)
- `RUN`: Execute commands during build
- `COPY`: Copy files from host to image
- `EXPOSE`: Document ports
- `CMD`: Default command when container starts
- `ENTRYPOINT`: Override default command

**Docker Network**: Bridge network connects containers with service discovery via DNS. Internal communication by container name, external access only through exposed ports.

**Docker Volume**: Named storage persisting data beyond container lifecycle. Mounted at container paths to provide persistent storage for databases and logs.

**Docker Compose**: YAML-based tool defining multi-container applications. Single `docker-compose.yml` file specifies all services, networks, and volumes.

### This Project's Architecture

- **6 Containers**: inventory-db, billing-db, inventory-app, billing-app, rabbit-queue, api-gateway-app
- **3 Volumes**: inventory-db, billing-db, api-gateway-app (data persists)
- **1 Network**: Bridge network (all containers connected, only api-gateway-app exposed)
- **Message Queue Pattern**: Decoupled async communication (billing orders processed asynchronously)
- **Reverse Proxy Pattern**: api-gateway-app routes requests to backend services
- **Infrastructure as Code**: Single `docker-compose up -d` creates entire system

---

## 📚 Best Practices & Tips

### Dockerfile Best Practices

1. **Use Specific Base Image Versions**

   ```dockerfile
   # ✅ GOOD: Explicit version
   FROM debian:bullseye-20240408

   # ❌ BAD: Unpredictable
   FROM debian:latest
   ```

2. **Minimize Layers**

   ```dockerfile
   # ✅ GOOD: One RUN with && chains
   RUN apt-get update && \
       apt-get install -y python3 && \
       rm -rf /var/lib/apt/lists/*

   # ❌ BAD: Multiple RUNs (unnecessary layers)
   RUN apt-get update
   RUN apt-get install -y python3
   ```

3. **Order Commands by Change Frequency**

   ```dockerfile
   # ✅ GOOD: Stable first, changing last
   FROM debian:bullseye
   RUN apt-get update && apt-get install -y python3
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY app/ .
   ```

4. **Use .dockerignore**

   ```
   .git
   .gitignore
   .env
   __pycache__
   *.pyc
   node_modules
   ```

5. **Penultimate Version**
   - Use latest stable MINUS ONE version (not latest, not old)
   - Ensures stability with recent patches
   - Avoids potential instabilities in bleeding-edge releases
   - Example: If Debian latest is `bookworm`, use `bullseye`

### Docker Compose Best Practices

1. **Use Version 3.8+**

   ```yaml
   version: "3.8" # ✅ Modern syntax
   ```

2. **Explicit Service Naming**

   ```yaml
   services:
     inventory-db:
       container_name: inventory-db # ✅ Explicit name = predictable
   ```

3. **Restart Policies**

   ```yaml
   restart_policy:
     condition: unless-stopped # ✅ Auto-restart on failure
   ```

4. **Health Checks**
   ```yaml
   healthcheck:
     test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
     interval: 30s
     timeout: 10s
     retries: 3
   ```

### Application Best Practices

1. **Environment Variables**
   - Store all config in `.env`
   - Add `.env` to `.gitignore` (NO credentials in git)
   - Load via `docker-compose.yml`: `environment: - VARIABLE=${VARIABLE}`

2. **Network Communication**
   - Services communicate by container name within the network
   - Example: `http://inventory-app:8080` (not IP addresses)
   - External access only through api-gateway-app:3000

3. **Logging**
   - All logs to stdout/stderr (Docker captures them)
   - Access via `docker-compose logs <service>`
   - Persistent logs stored in volume-mounted paths

4. **Data Persistence**
   - Critical data in named volumes
   - Application state in containers is ephemeral
   - Volumes survive container deletion/restart

### Performance Optimization

1. **Build Optimization**
   - Use .dockerignore to exclude unnecessary files
   - Layer ordering (rarely changed first, frequently changed last)
   - Multi-stage builds for smaller final images

2. **Runtime Optimization**
   - Use Alpine for minimal images (but may lack tools)
   - Debian bullseye for stable, feature-complete base
   - Penultimate version = stability + recent updates

3. **Network Optimization**
   - Custom bridge network (better than host network)
   - Service discovery by name (DNS automatically resolved)
   - Only expose necessary ports

---

## 📋 Submission & Audit

### Required Files (Submit All)

✅ **docker-compose.yml** - Orchestrates all 6 containers, 3 volumes, and 1 network  
✅ **Dockerfiles** - One per service (6 total)

- `srcs/api-gateway-app/Dockerfile`
- `srcs/inventory-app/Dockerfile`
- `srcs/billing-app/Dockerfile`
- `srcs/inventory-db/Dockerfile`
- `srcs/billing-db/Dockerfile`
- `srcs/rabbit-queue/Dockerfile`

✅ **.env** - Configuration (added to .gitignore)  
✅ **.gitignore** - Includes .env, credentials, build artifacts  
✅ **README.md** - Complete documentation (this file)  
✅ **Source Code** - All service implementations  
✅ **Scripts** - Any entrypoint.sh or initialization scripts

### What NOT to Submit

❌ Pre-built Docker Hub images (build your own)  
❌ Credentials or passwords in any file (except .env, which is .gitignored)  
❌ Docker images or containers (only source code and Dockerfile)  
❌ Build artifacts or compiled code  
❌ .git, node_modules, **pycache** (covered by .gitignore)

### Audit Preparation

**Be Ready to Discuss:**

1. Why 6 separate containers (not monolith)?
2. What does each container do?
3. How do containers communicate?
4. How do you ensure data persists?
5. What's the difference between containers and VMs?
6. Why use named volumes?
7. What is a custom bridge network?
8. How does DNS resolution work in docker-compose?
9. Why use environment variables?
10. What happens when a container crashes? (restart policy)

**Live Audit Commands:**

```bash
# Create infrastructure
docker-compose up -d

# Verify all running
docker-compose ps

# Check networking
docker network inspect microservices-net

# View volumes
docker volume ls

# Test API
curl http://localhost:3000/api/movies/

# Check logs
docker-compose logs -f api-gateway-app

# Delete infrastructure
docker-compose down

# Delete with volumes
docker-compose down -v
```

**Success Criteria:**

- ✅ All 6 containers running and healthy
- ✅ API accessible via port 3000
- ✅ All endpoints return correct status codes
- ✅ Data persists in volumes
- ✅ Containers auto-restart on failure
- ✅ Entire system reproducible with docker-compose.yml
- ✅ No hardcoded credentials in repository
- ✅ Documentation explains all concepts

---

## ✅ Verification Checklist

Before considering the project complete:

- [ ] All 6 containers running (`docker-compose ps` shows all UP)
- [ ] API Gateway accessible (`curl http://localhost:3000/health` returns 200)
- [ ] POST /api/movies returns 200 Created
- [ ] GET /api/movies returns movie list with 200 OK
- [ ] POST /api/billing returns 200 OK (async)
- [ ] Postman collection imported and all tests passing
- [ ] Movies stored in inventory_db database
- [ ] Orders processed and stored in billing_db
- [ ] Message queue resilience verified (order survives billing-app restart)
- [ ] Data persists after container restart (volumes working)
- [ ] All old Vagrant/VM references removed
- [ ] .env in .gitignore (credentials not in repo)
- [ ] README fully documents all requirements
- [ ] Audit preparation complete

---

## 📝 License

This is an educational project demonstrating containerization, microservices architecture, and Docker best practices.

---

## 🎓 Learning Outcomes

By completing this project, you'll understand:

✅ Microservices architecture design  
✅ Docker containerization and container orchestration  
✅ API Gateway pattern (reverse proxy + message routing)  
✅ Message queue implementation (RabbitMQ for async processing)  
✅ Database design and isolation  
✅ Docker Compose (Infrastructure as Code)  
✅ Docker networking (bridge networks, DNS resolution)  
✅ Service resilience and eventual consistency  
✅ API testing and automation (Postman)  
✅ Container management and debugging

---

**Last Updated**: April 1, 2026  
**Project Status**: ✅ Complete
