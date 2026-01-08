# 🚢 EventGuard – Intelligent Inventory & Shipment Tracking System

A production-ready shipment tracking system with AI-powered assistance, inventory management, and state machine integrity for logistics operations.

## 📌 Problem Statement

Logistics systems face critical challenges:
- **Invalid state transitions** leading to data corruption
- **Concurrent update conflicts** causing race conditions
- **Lack of auditability** making debugging impossible
- **Poor inventory tracking** with missing origin/destination data
- **No intelligent assistance** for shipment queries

EventGuard solves these by enforcing shipment behavior through a strict finite state machine, atomic transactions, complete audit trails, comprehensive inventory tracking, and an AI-powered assistant.

## ✨ Key Features

### 🎯 Core Capabilities
- **State Machine Integrity**: Enforces valid shipment transitions (CREATED → READY_FOR_PICKUP → IN_TRANSIT → DELIVERED)
- **Inventory Management**: Track origin, destination, product type, fragility, storage waypoints, and scheduled dates
- **AI Assistant**: Voice and text-powered chatbot for shipment queries
- **Serial Number Tracking**: Auto-incremented serial numbers for easy reference
- **Complete Audit Trail**: Every state change is logged with timestamps and actors
- **Exception Recovery**: Graceful handling and recovery from error states
- **Optimistic Locking**: Prevents concurrent update conflicts

### 📦 Inventory Features
When creating shipments, you can specify:
- **Origin**: Where the shipment comes from
- **Destination**: Where it's going
- **Product Type**: Category (Electronics, Clothing, Food, etc.)
- **Fragility**: FRAGILE or NON_FRAGILE handling
- **Storage Locations**: Waypoints where the product is stored (comma-separated)
- **Scheduled Date**: Planned pickup/delivery date and time
- **Estimated Delivery**: Expected delivery date and time

### 🤖 AI Assistant Features
- **Voice Input**: Speak your queries naturally
- **Text Input**: Type commands for shipment tracking
- **Smart Intent Detection**: Understands "track serial 1", "list shipments", "get statistics"
- **Serial Number Based**: Track shipments by serial number (e.g., "track serial 1")
- **Real-time Responses**: Instant answers to shipment queries

## ▶️ Installation & Running

### Prerequisites
- Docker Desktop (running)
- 4GB+ RAM available

### Quick Start

```bash
# Navigate to project directory
cd eventguard-v9-2/eventguard-v9/eventguard

# Build and start all services
docker-compose up --build

# Wait for all services to be healthy (about 30-60 seconds)
# Check status:
docker-compose ps
```

### Access Points
- **Web UI**: http://localhost:3000
- **API Documentation**: http://localhost:8000/docs
- **Health Check**: http://localhost:8000/health

### Stopping the System
```bash
docker-compose down
```

## 🏗️ Architecture

```
Browser → Nginx (Edge Gatekeeper) → FastAPI Backend → PostgreSQL
                ↓
         React Frontend
                ↓
         AI Agent Service
```

### Technology Stack
- **Frontend**: React 18 + Vite + Tailwind CSS
- **Backend**: FastAPI (Python 3.11) + SQLAlchemy
- **Database**: PostgreSQL (production) / SQLite (development)
- **Reverse Proxy**: Nginx with rate limiting
- **AI**: Browser Speech Recognition API + Intent Processing

### Service Components
1. **Frontend** (Port 3000): React application with AI chat widget
2. **Backend** (Port 8000): FastAPI REST API with state machine logic
3. **Database**: PostgreSQL with migrations and constraints
4. **Proxy** (Port 80): Nginx for routing and abuse prevention

## 📊 State Machine

### Valid Transitions

| From State | To State | Description |
|------------|----------|-------------|
| CREATED | READY_FOR_PICKUP | Normal flow |
| CREATED | CANCELLED | Can cancel before pickup |
| CREATED | EXCEPTION | Error state (recoverable) |
| READY_FOR_PICKUP | IN_TRANSIT | Normal flow |
| READY_FOR_PICKUP | CANCELLED | Can cancel before transit |
| READY_FOR_PICKUP | EXCEPTION | Error state (recoverable) |
| IN_TRANSIT | DELIVERED | Final delivery |
| IN_TRANSIT | CANCELLED | Can cancel during transit |
| IN_TRANSIT | EXCEPTION | Error state (recoverable) |
| EXCEPTION | CREATED/READY_FOR_PICKUP/IN_TRANSIT | Recovery paths |
| DELIVERED | *none* | Terminal state |
| CANCELLED | *none* | Terminal state |

### State Flow Diagram
```
CREATED → READY_FOR_PICKUP → IN_TRANSIT → DELIVERED
   ↓            ↓                ↓
CANCELLED   CANCELLED       CANCELLED
   ↓            ↓                ↓
EXCEPTION   EXCEPTION       EXCEPTION (recoverable)
```

## 🎮 Usage Guide

### Creating a Shipment

1. Click **"Create New Shipment"** button
2. Enter **Tracking Number** (required)
3. Fill inventory fields (optional but recommended):
   - Origin address
   - Destination address
   - Product type
   - Fragility (Fragile/Non-Fragile)
   - Storage locations (comma-separated waypoints)
   - Scheduled date/time
   - Estimated delivery date/time
4. Click **"Create Shipment"**

### Using AI Assistant

1. Click the **AI Assistant** button (bottom-right corner)
2. Ask questions like:
   - "Track serial 1"
   - "List all shipments"
   - "Get shipment statistics"
   - "What's the system status?"
3. Use voice input by clicking the microphone icon
4. Select from dropdown if multiple shipments match

### Tracking Shipments

- **By Serial Number**: "Track serial 1" or "Track 1"
- **View Timeline**: Click on any shipment to see state transitions with animated truck graphics
- **State Transitions**: Use action buttons to transition shipment states
- **Inventory Info**: See origin, destination, storage waypoints, and dates

## 🔧 Development

### Project Structure
```
eventguard/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI application
│   │   ├── models.py            # Database models
│   │   ├── schemas.py           # Pydantic schemas
│   │   ├── state_machine.py    # State transition logic
│   │   ├── services/           # Business logic
│   │   │   ├── ai_agent_service.py
│   │   │   └── shipment_service.py
│   │   └── routes/             # API endpoints
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── AIAgent.jsx     # AI chat widget
│   │   │   ├── ShipmentList.jsx
│   │   │   └── ShipmentDetail.jsx
│   │   └── api.js              # API client
│   └── package.json
├── nginx/
│   └── nginx.conf              # Reverse proxy config
└── docker-compose.yml          # Service orchestration
```

### Database Migrations
Migrations run automatically on startup. The system adds new columns safely:
- Inventory fields (origin, destination, product_type, etc.)
- Serial numbers with auto-increment
- Exception handling fields
- Audit trail fields

### API Endpoints

**Shipments:**
- `POST /shipments` - Create shipment with inventory data
- `GET /shipments` - List shipments with filters
- `GET /shipments/{id}` - Get shipment details
- `POST /shipments/{id}/transition` - Change state
- `GET /shipments/{id}/timeline` - Get state history

**AI Agent:**
- `POST /ai/chat` - Send message to AI assistant
- `GET /ai/health` - Check AI service status

## 🛡️ Security & Reliability

- **Input Validation**: All user inputs sanitized and validated
- **SQL Injection Prevention**: Parameterized queries only
- **Rate Limiting**: Nginx prevents abuse
- **Optimistic Locking**: Prevents concurrent conflicts
- **Soft Deletes**: Shipments can be restored
- **Exception Recovery**: Graceful error handling with recovery paths

## 📝 License

Built for the Build2Break Hackathon (LogiTech domain).

## 🆘 Troubleshooting

### Services won't start
- Ensure Docker Desktop is running
- Check ports 3000, 8000, 5432 are available
- Review logs: `docker-compose logs`

### Database connection errors
- Wait 30-60 seconds for PostgreSQL to initialize
- Check: `docker-compose ps` - all services should be "healthy"

### AI Assistant not working
- Ensure browser supports Speech Recognition API (Chrome/Edge)
- Check microphone permissions
- Use text input as fallback

### Frontend not loading
- Clear browser cache
- Check: `docker logs stateguard-frontend`
- Rebuild: `docker-compose up -d --build frontend`
