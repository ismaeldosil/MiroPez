# MiroFish Architecture Overview

MiroFish is a multi-agent social simulation engine that transforms documents into predictive digital worlds. It combines knowledge graphs, LLM-powered agents, and the OASIS simulation framework to model social media discourse.

## System Architecture (C4 - Context)

```mermaid
C4Context
    title MiroFish System Context

    Person(user, "User", "Uploads documents, runs simulations, reads reports")

    System(mirofish, "MiroFish", "Multi-agent social simulation engine")

    System_Ext(llm, "LLM API", "OpenAI-compatible API (Qwen, GPT-4, etc.)")
    System_Ext(zep, "Zep Cloud", "Knowledge graph storage and semantic search")
    System_Ext(oasis, "OASIS Framework", "Social media simulation engine (CAMEL-AI)")

    Rel(user, mirofish, "Uploads docs, configures simulations, chats with agents")
    Rel(mirofish, llm, "Ontology generation, profile creation, report writing")
    Rel(mirofish, zep, "Graph CRUD, entity extraction, semantic search")
    Rel(mirofish, oasis, "Agent simulation execution (subprocess)")
```

## High-Level Architecture

```mermaid
graph TB
    subgraph Frontend["Frontend (Vue.js 3 + Vite)"]
        Home[Home Page]
        Process[Process Workflow]
        SimView[Simulation Dashboard]
        ReportView[Report View]
        InteractionView[Agent Interaction]

        subgraph Components
            S1[Step1: Graph Build]
            S2[Step2: Env Setup]
            S3[Step3: Simulation]
            S4[Step4: Report]
            S5[Step5: Interaction]
            GP[Graph Panel - D3.js]
        end

        subgraph API_Layer["API Layer (Axios)"]
            GraphAPI[graph.js]
            SimAPI[simulation.js]
            ReportAPI[report.js]
        end
    end

    subgraph Backend["Backend (Flask + Python)"]
        subgraph Blueprints["API Blueprints"]
            GraphBP["/api/graph"]
            SimBP["/api/simulation"]
            ReportBP["/api/report"]
        end

        subgraph Services
            OG[Ontology Generator]
            GB[Graph Builder]
            TP[Text Processor]
            ZER[Zep Entity Reader]
            OPG[Profile Generator]
            SCG[Config Generator]
            SM[Simulation Manager]
            SR[Simulation Runner]
            RA[Report Agent]
            ZT[Zep Tools]
        end

        subgraph Models
            Project[Project Model]
            Task[Task Model]
        end

        subgraph Utils
            FP[File Parser]
            LLC[LLM Client]
            Logger
        end
    end

    subgraph External["External Services"]
        LLM["LLM API<br/>(OpenAI-compatible)"]
        Zep["Zep Cloud<br/>(Knowledge Graph)"]
        OASIS["OASIS<br/>(Simulation Engine)"]
    end

    subgraph Storage["File Storage"]
        Projects["uploads/projects/"]
        Simulations["uploads/simulations/"]
        Reports["uploads/reports/"]
    end

    Frontend -->|HTTP/REST| Backend
    Services --> LLM
    Services --> Zep
    SR -->|Subprocess| OASIS
    Models --> Storage

    style Frontend fill:#42b883,color:#fff
    style Backend fill:#3572A5,color:#fff
    style External fill:#f39c12,color:#fff
    style Storage fill:#95a5a6,color:#fff
```

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Frontend** | Vue.js 3 | ^3.5.24 |
| | Vue Router | ^4.6.3 |
| | Axios | ^1.13.2 |
| | D3.js | ^7.9.0 |
| | Vite | ^7.2.4 |
| **Backend** | Flask | >=3.0.0 |
| | Flask-CORS | >=6.0.0 |
| | OpenAI SDK | >=1.0.0 |
| | Zep Cloud SDK | ==3.13.0 |
| | CAMEL-OASIS | ==0.2.5 |
| | PyMuPDF | >=1.24.0 |
| | Pydantic | >=2.0.0 |
| **Runtime** | Python | >=3.11, <=3.12 |
| | Node.js | >=18 |
| **Infra** | Docker | Multi-stage |
| | uv | 0.9.26 |

## Directory Structure

```
MiroFish/
├── frontend/
│   ├── src/
│   │   ├── api/              # Axios API clients
│   │   ├── components/       # Step1-5 + GraphPanel + HistoryDatabase
│   │   ├── router/           # Vue Router config
│   │   ├── store/            # Pending upload state
│   │   └── views/            # Page components (Home, Process, etc.)
│   ├── package.json
│   └── vite.config.js
├── backend/
│   ├── app/
│   │   ├── api/              # Flask blueprints (graph, simulation, report)
│   │   ├── models/           # Project, Task data models
│   │   ├── services/         # Business logic (12 service modules)
│   │   ├── utils/            # File parser, LLM client, logger, retry
│   │   ├── __init__.py       # App factory
│   │   └── config.py         # Config from .env
│   ├── scripts/              # OASIS simulation runner scripts
│   ├── uploads/              # Persistent file storage
│   ├── run.py                # Entry point
│   └── pyproject.toml        # Python dependencies
├── docs/                     # Architecture documentation
├── Dockerfile
├── docker-compose.yml
└── .env.example
```

## Key Design Patterns

- **App Factory**: Flask app creation deferred via `create_app()` for testability
- **Service Layer**: Business logic separated from API handlers
- **Task-Based Async**: Long operations tracked via Task objects with progress callbacks
- **Server-Side State**: Project/simulation state persisted on disk, not in API payloads
- **IPC for Subprocesses**: File-based event log for simulation runner communication
- **Ontology-Driven**: Graph schema defined dynamically by LLM-generated ontology
- **Tool-Calling Agent**: Report Agent autonomously invokes tools (graph search) to answer questions
- **Streaming Sections**: Report sections saved incrementally for progressive frontend rendering
