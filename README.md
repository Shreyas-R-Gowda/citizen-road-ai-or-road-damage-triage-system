# Citizen Road Reporting System

> AI-powered full-stack platform for civic road-damage reporting, automated severity triage, and geospatial prioritization.

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-async-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-Vite-61DAFB?logo=react&logoColor=black)](https://vitejs.dev)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![Deployed on Render](https://img.shields.io/badge/Deployed-Render-46E3B7?logo=render&logoColor=white)](https://render.com)

---

## Overview

The Citizen Road Reporting System enables the public to report potholes and road-surface defects via a map-based web interface. Submitted reports are automatically analyzed using an **AI ensemble** (deterministic image scoring + LLM reasoning via Groq API) to assign severity scores and prioritize repair queues — helping municipalities act on the most critical damage first.

This project was built as the final semester engineering lab project (EL Sem 6) at R.V. College of Engineering, Bengaluru.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Browser / PWA                        │
│              React + Vite + TailwindCSS + Leaflet           │
└─────────────────────────┬───────────────────────────────────┘
                          │ HTTP / REST
┌─────────────────────────▼───────────────────────────────────┐
│                   FastAPI Backend (async)                    │
│  SQLAlchemy async ORM  │  PostGIS spatial queries           │
│  pgvector embeddings   │  JWT auth                          │
└──────┬──────────────────┬──────────────────┬────────────────┘
       │                  │                  │
┌──────▼──────┐  ┌────────▼────────┐  ┌─────▼──────────────┐
│  PostgreSQL  │  │   AI Ensemble   │  │    YOLO ML Module  │
│  + PostGIS  │  │  (ai-ensemble/) │  │  (ml/ workspace)   │
│  + pgvector │  │  Groq LLM API   │  │  Road damage det.  │
└─────────────┘  └─────────────────┘  └────────────────────┘

All services orchestrated via Docker Compose · Deployed on Render
```

**Key data flow:** User submits photo + location → backend stores report → AI ensemble scores image severity + generates LLM summary → severity score stored in PostGIS DB → frontend map clusters reports by priority → municipal dashboard surfaces highest-priority zones.

---

## Features

- **Map-based reporting** — users drop a pin and upload a photo; Leaflet renders all reports as clustered markers with severity color coding
- **AI severity triage** — multi-signal ensemble combining image analysis, textual description scoring, and location context
- **LLM integration** — Groq API (configurable) generates human-readable damage summaries and repair recommendations
- **YOLO road-damage detection** — trainable ML module (`ml/`) for pothole and surface-defect detection with optional depth estimation
- **Geospatial indexing** — PostGIS enables fast spatial queries (bounding box, radius search, hotspot clustering)
- **Semantic search** — pgvector stores report embeddings for similarity-based deduplication and clustering
- **Docker-first** — single `docker compose up` starts the full stack (Postgres + PostGIS, backend, frontend)
- **Render deployment** — `render.yaml` blueprint for one-click cloud deployment with environment variable management
- **Auto-generated API docs** — Swagger UI and ReDoc available out of the box

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite, TailwindCSS, Leaflet.js |
| Backend | FastAPI, SQLAlchemy (async), Alembic |
| Database | PostgreSQL 15 + PostGIS 3.3 + pgvector |
| AI / ML | Groq API (LLM), YOLO (road damage), custom ensemble |
| Infrastructure | Docker Compose, Nginx, Render |
| Auth | JWT (access + refresh tokens) |
| API Docs | OpenAPI / Swagger, ReDoc |

---

## Prerequisites

- Docker Desktop (v24+) and Docker Compose v2
- Node.js 18+ (for local frontend development only)
- Python 3.11+ (for local backend development only)
- Groq API key (free tier available at [console.groq.com](https://console.groq.com))

---

## Quick start (Docker — recommended)

**1. Clone the repository**

```bash
git clone https://github.com/shankar0311/civic-final.git
cd civic-final
```

**2. Configure environment variables**

```bash
cp backend/.env.example backend/.env
# Open backend/.env and fill in:
#   GROK_API_KEY=your_groq_api_key_here
#   SECRET_KEY=your_jwt_secret
```

**3. Start all services**

```bash
docker compose up --build
```

Services will be available at:

| Service | URL |
|---|---|
| Frontend (React) | http://localhost:3005 |
| Backend API | http://localhost:8005 |
| Swagger UI | http://localhost:8005/docs |
| ReDoc | http://localhost:8005/redoc |
| PostgreSQL | localhost:5433 |

**4. Seed synthetic data (optional)**

```bash
docker compose exec backend python seed_data.py
```

**5. Stop all services**

```bash
docker compose down          # stop containers
docker compose down -v       # stop + remove volumes (resets DB)
```

---

## Local development (without Docker)

### Backend

```bash
cd backend
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env          # fill in your secrets
uvicorn main:app --reload --port 8005
```

Requires a running PostgreSQL instance with PostGIS and pgvector extensions enabled. Update `DATABASE_URL` in `.env` accordingly.

### Frontend

```bash
cd frontend/cityreport
npm install
npm run dev                   # runs on http://localhost:5173
```

---

## ML module — YOLO road damage detection

The `ml/` directory contains a training workspace for custom road-damage detection:

```bash
pip install -r backend/requirements-ml.txt

# Train with your dataset (configure paths in the YAML)
python ml/training/train_detector.py \
  --data ml/config/road_damage.dataset.example.yaml

# Run inference on a test image
python ml/inference/detect.py --source path/to/image.jpg
```

The module supports:
- **YOLO-based detection** for pothole, cracking, and surface-damage classes
- Optional **depth model integration** for 3D damage estimation
- Optional **learned severity scoring** to replace or augment the deterministic scorer

See `ml/README.md` for the complete training pipeline.

---

## AI ensemble

The `ai-ensemble/` module orchestrates three severity signals:

1. **Image score** — pixel-level analysis (edge density, crack pattern, area ratio)
2. **Text score** — NLP on user description (keyword severity weighting)
3. **Location score** — road class and historical report frequency from PostGIS

Scores are combined with configurable weights and fed to the Groq LLM to produce a final severity label (`low` / `medium` / `high` / `critical`) with a human-readable repair recommendation.

---

## Deployment on Render

1. Fork or push this repo to your GitHub account
2. Go to [render.com](https://render.com) → New → Blueprint
3. Select this repository — Render auto-reads `render.yaml`
4. Set the following environment variables in the Render dashboard:
   - `GROK_API_KEY` — your Groq API key
   - `SECRET_KEY` — a random 32-char string for JWT signing
5. Ensure the PostgreSQL instance has PostGIS enabled (Render managed Postgres supports this via `CREATE EXTENSION postgis;`)
6. Deploy — all services start automatically

---

## Project structure

```
civic-final/
├── backend/              # FastAPI application
│   ├── main.py           # app entrypoint, router registration
│   ├── models/           # SQLAlchemy ORM models
│   ├── routes/           # API route handlers
│   ├── services/         # business logic (scoring, AI calls)
│   ├── seed_data.py      # synthetic data seeder
│   ├── requirements.txt
│   └── .env.example
├── frontend/             # React + Vite SPA
│   └── cityreport/
│       ├── src/
│       │   ├── components/   # Map, ReportForm, Dashboard, etc.
│       │   └── pages/
│       └── package.json
├── ai-ensemble/          # Multi-signal severity scoring module
├── ml/                   # YOLO training + inference workspace
├── .github/workflows/    # CI/CD (lint, test, build checks)
├── docker-compose.yml    # local full-stack orchestration
└── render.yaml           # Render cloud deployment blueprint
```

---

## API reference

Full interactive docs available at `/docs` (Swagger UI) when the backend is running.

Key endpoints:

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/reports` | Submit a new road damage report |
| `GET` | `/reports` | List reports (filterable by bbox, severity, status) |
| `GET` | `/reports/{id}` | Get full report details with AI analysis |
| `GET` | `/reports/hotspots` | Spatial clustering of high-severity zones |
| `POST` | `/auth/register` | Register a new user |
| `POST` | `/auth/login` | Obtain JWT access token |

---

## Future scope

- **Mobile app** — React Native client with native camera and GPS integration
- **Real-time notifications** — WebSocket push when nearby reports are filed or resolved
- **Municipal dashboard** — admin panel with repair-workflow tracking and cost estimation
- **Federated learning** — privacy-preserving model improvement across municipalities
- **Satellite imagery integration** — periodic aerial damage assessment via open satellite APIs
- **Offline-first PWA** — report submission without internet, sync on reconnect

---

## Contributors

| Name | Role |
|---|---|
| Shankar | Lead backend, AI ensemble, deployment |
| Shreyas R Gowda | Frontend, ML module integration, Docker setup |

*Built as part of the Engineering Lab — 6th Semester, R.V. College of Engineering, Bengaluru.*

---

## License

This project is released for academic and educational purposes. Contact the contributors for any other use.
