# MiroFish Deployment Guide

## Architecture Diagram

```mermaid
graph TB
    subgraph Docker["Docker Container"]
        Vite["Vite Dev Server<br/>:3000"]
        Flask["Flask Backend<br/>:5001"]
        OASIS_Proc["OASIS Subprocess<br/>(spawned per simulation)"]

        Flask -->|spawns| OASIS_Proc
    end

    subgraph Volumes["Persistent Storage"]
        Uploads["./backend/uploads/<br/>projects, simulations, reports"]
    end

    subgraph External["External Services"]
        LLM["LLM API<br/>(Qwen / GPT-4 / etc.)"]
        Zep["Zep Cloud"]
    end

    User["User Browser"] -->|:3000| Vite
    Vite -->|proxy /api| Flask
    Flask --> Uploads
    Flask --> LLM
    Flask --> Zep
    OASIS_Proc --> Uploads
```

## Prerequisites

| Tool | Version | Check |
|------|---------|-------|
| Node.js | >= 18 | `node -v` |
| Python | >= 3.11, <= 3.12 | `python --version` |
| uv | Latest | `uv --version` |

## Environment Variables

Create `.env` from the template:

```bash
cp .env.example .env
```

**Required:**

| Variable | Description | Example |
|----------|-------------|---------|
| `LLM_API_KEY` | API key for LLM provider | `sk-...` |
| `LLM_BASE_URL` | LLM endpoint URL | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| `LLM_MODEL_NAME` | Model identifier | `qwen-plus` |
| `ZEP_API_KEY` | Zep Cloud API key | `z_...` |

**Optional:**

| Variable | Default | Description |
|----------|---------|-------------|
| `SECRET_KEY` | `mirofish-secret-key` | Flask secret key |
| `FLASK_DEBUG` | `True` | Debug mode |
| `OASIS_DEFAULT_MAX_ROUNDS` | `10` | Default simulation rounds |
| `REPORT_AGENT_MAX_TOOL_CALLS` | `5` | Max tool calls per report section |
| `REPORT_AGENT_TEMPERATURE` | `0.5` | Report LLM temperature |

## Option 1: Source Code (Recommended)

```bash
# 1. Install all dependencies
npm run setup:all

# 2. Start both frontend and backend
npm run dev
```

**Individual commands:**

```bash
npm run setup           # Node deps (root + frontend)
npm run setup:backend   # Python deps (creates venv)
npm run frontend        # Start frontend only (:3000)
npm run backend         # Start backend only (:5001)
```

**Service URLs:**
- Frontend: http://localhost:3000
- Backend API: http://localhost:5001
- Health: http://localhost:5001/health

## Option 2: Docker

```bash
# Pull and start
docker compose up -d

# Or build locally
docker compose up -d --build
```

**docker-compose.yml** maps:
- Port 3000 (frontend)
- Port 5001 (backend)
- Volume `./backend/uploads` for persistent data

## Port Configuration

```mermaid
graph LR
    Browser["Browser"] -->|:3000| Frontend["Vite Dev Server"]
    Frontend -->|:5001| Backend["Flask API"]
    Backend -->|HTTPS| Zep["Zep Cloud"]
    Backend -->|HTTPS| LLM["LLM API"]
```

## Logging

Logs are written to `backend/logs/` with daily rotation:

```
backend/logs/
├── 2025-12-10.log     # Detailed debug log
├── 2025-12-11.log
└── ...
```

**Log levels:**
- File: DEBUG (all details)
- Console: INFO and above

**Logger hierarchy:**
```
mirofish              # Root logger
├── mirofish.request  # HTTP request/response
├── mirofish.api      # API route handlers
├── mirofish.build    # Graph building
├── mirofish.simulation # Simulation execution
└── mirofish.report   # Report generation
```

## CI/CD

GitHub Actions workflow (`.github/workflows/docker-image.yml`):

```mermaid
graph LR
    Tag["Git Tag Push"] --> Build["Docker Build"]
    Build --> Push["Push to GHCR"]
    Push --> Deploy["ghcr.io/666ghj/mirofish:latest"]
```

Triggered on tag pushes. Builds multi-platform Docker image and pushes to GitHub Container Registry.
