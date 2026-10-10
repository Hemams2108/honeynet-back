# Autonomous Honeynet Telemetry & Threat Actor MITRE ATT&CK Causal Graph Profiler

**Hackathon Project** | Team Size: 5 Members

An autonomous cyber-defense platform that ingests distributed honeynet decoy telemetry, reconstructs multi-stage causal attack graphs using **Cytoscape.js**, maps observed adversary techniques to the **MITRE ATT&CK** matrix, and clusters threat actors based on behavior.

---

## 🏛 System Architecture

The project is structured into clean frontend and backend modules:

```text
hackathon/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   ├── health.py     # System & database engine status (GET /api/health)
│   │   │   ├── events.py     # Telemetry storage & query (GET /api/events, POST /api/events, /bulk, /stats)
│   │   │   ├── dashboard.py  # Real-time metrics
│   │   │   ├── telemetry.py  # Telemetry router & seed endpoint
│   │   │   ├── alerts.py     # Security incident alerts & resolution
│   │   │   ├── mitre.py      # ATT&CK technique matrix endpoint
│   │   │   └── graph.py      # Cytoscape causal graph generator
│   │   ├── models/
│   │   │   ├── telemetry.py  # Telemetry event DB model & Pydantic validation schemas
│   │   │   └── alert.py      # Security incident alert schemas
│   │   ├── services/
│   │   │   ├── ingestion.py  # Ingestion pipeline & duplicate handling
│   │   │   ├── analysis.py   # Telemetry aggregation & MITRE matrix status
│   │   │   ├── mitre.py      # Automated MITRE ATT&CK heuristics
│   │   │   ├── alerts.py     # Threat clustering & alert generation
│   │   │   └── graph.py      # Causal graph generation algorithm
│   │   ├── database.py       # DB abstraction (PostgreSQL / SQLite fallback) + auto-migrations
│   │   ├── config.py         # Pydantic Settings & environment variables
│   │   └── main.py           # FastAPI entrypoint, CORS, clean lifespan
│   ├── seed_demo_telemetry.py# Reproducible synthetic honeynet seeder (RFC 5737 IPs)
│   ├── test_pipeline.py      # Automated verification test script (Steps 1-11)
│   ├── requirements.txt      # Python dependencies
│   ├── .env.example          # Backend environment template
│   └── venv/                 # Virtual environment
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Dashboard.tsx            # Live telemetry KPIs (Real vs Synthetic), active decodes
│   │   │   ├── TelemetryExplorer.tsx    # Event table with origin filtering, pagination & modal
│   │   │   ├── InvestigationGraph.tsx   # Interactive Cytoscape.js causal graph
│   │   │   ├── MitreAttackView.tsx      # MITRE ATT&CK matrix coverage
│   │   │   └── AlertsView.tsx           # Prioritized incident alerts
│   │   ├── services/
│   │   │   └── api.ts                   # Environment-configured API client (/api/events, /api/stats)
│   │   ├── App.tsx                      # Primary shell & navigation
│   │   ├── main.tsx                     # React root
│   │   └── index.css                    # Tailwind CSS v4 styling
│   ├── package.json          # Node dependencies
│   ├── vite.config.ts        # Vite + Tailwind configuration
│   └── .env                  # Frontend environment configuration
├── .gitignore                # Comprehensive exclusions
└── README.md                 # Setup & running instructions
```

---

## 📡 Telemetry Ingestion API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | Health check verifying application & database engine (`sqlite_local_fallback` vs `postgresql_hosted`). |
| `GET` | `/api/events` | List stored events with pagination (`skip`, `limit`), search keyword, and filters (`source_ip`, `event_type`, `data_origin`, `honeypot_id`). |
| `GET` | `/api/events/{event_id}` | Retrieve single event by UUID or numeric ID. Returns HTTP 404 if not found. |
| `POST` | `/api/events` | Ingest and validate a telemetry event. Handles duplicate event IDs safely without failure. |
| `POST` | `/api/events/bulk` | Batch ingestion of multiple events. Gracefully skips duplicates and returns count summary. |
| `GET` | `/api/stats` | Aggregated statistics computed live from database (total events, real vs synthetic count, unique IPs, event types, top MITRE techniques). |

---

## 🚀 Quickstart Instructions

### Prerequisites
- **Python 3.10+** (Python 3.12 verified)
- **Node.js 18+** & **npm**

---

### 1. Backend Setup

1. Open a terminal and navigate to `backend`:
   ```bash
   cd backend
   ```
2. Activate the virtual environment:
   - **Windows PowerShell**:
     ```powershell
     .\venv\Scripts\Activate.ps1
     ```
   - **Linux / macOS**:
     ```bash
     source venv/bin/activate
     ```
3. Run the FastAPI backend server:
   ```bash
   python -m uvicorn backend.app.main:app --host 127.0.0.1 --port 8000 --reload
   ```
4. Verify endpoints:
   - Health endpoint: [http://127.0.0.1:8000/api/health](http://127.0.0.1:8000/api/health)
   - Events stream: [http://127.0.0.1:8000/api/events](http://127.0.0.1:8000/api/events)
   - Telemetry stats: [http://127.0.0.1:8000/api/stats](http://127.0.0.1:8000/api/stats)
   - Interactive Swagger docs: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

---

### 2. Seeding Demo Telemetry Data

To insert reproducible synthetic honeynet events with reserved RFC 5737 addresses (`192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24`), run:

```powershell
python backend/seed_demo_telemetry.py
```
> **Duplicate Safe**: The seed script uses deterministic IDs and can be safely re-run without duplicating records. All seeded records are explicitly labeled with `data_origin="synthetic"`.

---

### 3. Frontend Setup

1. In a second terminal, navigate to `frontend`:
   ```bash
   cd frontend
   npm run dev
   ```
2. Access the application in your browser:
   - **[http://localhost:5173](http://localhost:5173)**

---

### 4. Running Automated Pipeline Tests

To run the complete verification suite exercising steps 1 through 11 (health check, single insert, retrieval, search, pagination, bulk insert, duplicate handling, stats validation):

```powershell
python backend/test_pipeline.py
```
