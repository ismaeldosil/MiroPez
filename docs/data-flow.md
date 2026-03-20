# MiroFish Data Flow

## End-to-End Pipeline

```mermaid
graph LR
    A[Document Upload] --> B[Text Extraction]
    B --> C[Ontology Generation]
    C --> D[Graph Building]
    D --> E[Entity Extraction]
    E --> F[Profile Generation]
    F --> G[Config Generation]
    G --> H[Simulation Execution]
    H --> I[Report Generation]
    I --> J[Agent Interaction]

    style A fill:#e74c3c,color:#fff
    style D fill:#3498db,color:#fff
    style H fill:#2ecc71,color:#fff
    style I fill:#9b59b6,color:#fff
```

## 5-Step User Workflow

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant BE as Backend
    participant LLM as LLM API
    participant Zep as Zep Cloud
    participant OASIS as OASIS Engine

    rect rgb(255, 230, 230)
        Note over U,Zep: Step 1 - Graph Building
        U->>FE: Upload files + simulation requirement
        FE->>BE: POST /api/graph/ontology/generate
        BE->>BE: FileParser.extract_text()
        BE->>LLM: Analyze document, generate ontology
        LLM-->>BE: entity_types[], edge_types[]
        BE-->>FE: project_id, ontology

        U->>FE: Click "Build Graph"
        FE->>BE: POST /api/graph/build
        BE->>BE: TextProcessor.split_text()
        BE->>Zep: Create graph + set ontology
        loop For each chunk batch
            BE->>Zep: Add episodes (3 chunks/batch)
        end
        loop Poll until processed
            BE->>Zep: Check episode status
        end
        BE-->>FE: graph_id, node_count, edge_count
    end

    rect rgb(230, 255, 230)
        Note over U,OASIS: Step 2 - Environment Setup
        U->>FE: Select entities, choose platforms
        FE->>BE: POST /api/simulation/prepare
        BE->>Zep: Read entities from graph
        BE->>LLM: Generate agent profiles
        LLM-->>BE: Agent personas (personality, beliefs, etc.)
        BE->>LLM: Generate simulation config
        LLM-->>BE: Time config, event config, agent activity
        BE-->>FE: simulation_id, profiles_count
    end

    rect rgb(230, 230, 255)
        Note over U,OASIS: Step 3 - Simulation
        U->>FE: Click "Start Simulation"
        FE->>BE: POST /api/simulation/start
        BE->>OASIS: Spawn subprocess
        loop Each simulation round
            OASIS->>OASIS: Agents act (post, like, comment)
            OASIS->>BE: Write actions to event log
        end
        loop Poll status
            FE->>BE: GET /run-status/detail
            BE-->>FE: current_round, recent_actions
        end
    end

    rect rgb(255, 230, 255)
        Note over U,Zep: Step 4 - Report Generation
        U->>FE: Click "Generate Report"
        FE->>BE: POST /api/report/generate
        loop For each report section
            BE->>Zep: Search graph for facts
            BE->>LLM: Synthesize section content
            BE-->>FE: Section markdown (incremental)
        end
        BE-->>FE: Complete report
    end

    rect rgb(255, 255, 230)
        Note over U,Zep: Step 5 - Interaction
        U->>FE: Send chat message
        FE->>BE: POST /api/report/chat
        BE->>Zep: Search graph for context
        BE->>LLM: Generate response with facts
        BE-->>FE: Agent response + sources
    end
```

## Detailed Data Flow by Step

### Step 1: Document Upload & Graph Building

```mermaid
flowchart TD
    Upload["User uploads PDF/MD/TXT files"]
    -->|multipart/form-data| Parse["FileParser.extract_text()"]
    --> Clean["TextProcessor.preprocess_text()"]
    --> LLM1["LLM: Analyze document content"]
    --> Ontology["Ontology: 10 entity types + relationships"]
    --> Chunk["TextProcessor.split_text(500 chars, 50 overlap)"]
    --> CreateGraph["Zep: Create graph"]
    --> SetSchema["Zep: Set ontology schema"]
    --> BatchAdd["Zep: Add text batches (3 chunks/batch)"]
    --> WaitProcess["Zep: Wait for episode processing"]
    --> GraphReady["Graph Ready: nodes + edges"]

    subgraph Project State
        P1["CREATED"] --> P2["ONTOLOGY_GENERATED"]
        P2 --> P3["GRAPH_BUILDING"]
        P3 --> P4["GRAPH_COMPLETED"]
    end
```

### Step 2: Simulation Preparation

```mermaid
flowchart TD
    Start["Prepare Simulation"]
    --> ReadEntities["ZepEntityReader: Extract entities from graph"]
    --> FilterEntities["Filter by defined entity types"]
    --> GenProfiles["OasisProfileGenerator: LLM generates personas"]

    GenProfiles --> TwitterProf["Twitter profiles (CSV)"]
    GenProfiles --> RedditProf["Reddit profiles (JSON)"]

    FilterEntities --> GenConfig["SimulationConfigGenerator: LLM generates params"]
    GenConfig --> TimeConfig["Time config (peak hours, rounds)"]
    GenConfig --> EventConfig["Event config (initial posts, hot topics)"]
    GenConfig --> AgentConfig["Agent configs (activity, stance, influence)"]

    TwitterProf --> Ready["Simulation READY"]
    RedditProf --> Ready
    TimeConfig --> Ready
    EventConfig --> Ready
    AgentConfig --> Ready
```

### Step 3: Simulation Execution

```mermaid
flowchart TD
    Start["POST /api/simulation/start"]
    --> Spawn["SimulationRunner: Spawn subprocess"]
    --> Load["Load profiles + config"]
    --> Loop{"Simulation Round"}

    Loop --> Act["All agents act in parallel"]
    Act --> Post["Generate posts/comments"]
    Act --> Like["Like/repost content"]
    Act --> Follow["Follow/mute users"]

    Post --> Broadcast["Broadcast results"]
    Like --> Broadcast
    Follow --> Broadcast

    Broadcast --> UpdateState["Update agent internal state"]
    UpdateState --> WriteLog["Write actions to IPC event log"]
    WriteLog --> GraphMem{"Update graph memory?"}

    GraphMem -->|Yes| ZepUpdate["ZepGraphMemoryUpdater"]
    GraphMem -->|No| NextRound
    ZepUpdate --> NextRound["Next round"]
    NextRound --> Loop

    Loop -->|max_rounds reached| Complete["Simulation COMPLETED"]

    subgraph "Frontend Polling"
        Poll["GET /run-status/detail (every 2s)"]
        --> Display["Display: round counter, actions, timeline"]
    end
```

### Step 4: Report Generation

```mermaid
flowchart TD
    Trigger["POST /api/report/generate"]
    --> CreateAgent["Instantiate ReportAgent"]
    --> Plan["Agent: Generate report outline via LLM"]
    --> Sections["Section list with titles"]

    Sections --> Loop{"For each section"}
    Loop --> Search["ZepToolsService.search_graph(query)"]
    Search --> Facts["Retrieved facts + entities"]
    Facts --> Synthesize["LLM: Synthesize section content"]
    Synthesize --> SaveSection["Save section_NN.md to disk"]
    SaveSection --> LogAction["Log to agent_log.jsonl"]
    LogAction --> Loop

    Loop -->|All sections done| Combine["Combine into final report.md"]
    Combine --> Complete["Report COMPLETED"]

    subgraph "Frontend Streaming"
        PollSections["GET /sections (poll)"]
        --> RenderIncremental["Render sections as they appear"]
    end
```

## State Machine Diagrams

### Project Lifecycle

```mermaid
stateDiagram-v2
    [*] --> CREATED: Upload files
    CREATED --> ONTOLOGY_GENERATED: LLM generates ontology
    ONTOLOGY_GENERATED --> GRAPH_BUILDING: Start graph build
    GRAPH_BUILDING --> GRAPH_COMPLETED: Zep processing done
    GRAPH_BUILDING --> FAILED: Error
    ONTOLOGY_GENERATED --> FAILED: Error
    FAILED --> ONTOLOGY_GENERATED: Reset project
    GRAPH_COMPLETED --> ONTOLOGY_GENERATED: Reset project
```

### Simulation Lifecycle

```mermaid
stateDiagram-v2
    [*] --> CREATED: Create simulation
    CREATED --> PREPARING: Start preparation
    PREPARING --> READY: Profiles + config generated
    PREPARING --> FAILED: Error
    READY --> RUNNING: Start execution
    RUNNING --> COMPLETED: Max rounds reached
    RUNNING --> STOPPED: User stops
    RUNNING --> FAILED: Error
    STOPPED --> RUNNING: Resume
```

### Task Lifecycle

```mermaid
stateDiagram-v2
    [*] --> PENDING: Task created
    PENDING --> PROCESSING: Worker picks up
    PROCESSING --> COMPLETED: Success
    PROCESSING --> FAILED: Error
```

## File Storage Layout

```
uploads/
├── projects/
│   └── proj_{uuid}/
│       ├── project.json           # Project metadata + ontology
│       ├── extracted_text.txt     # Full document text
│       └── files/
│           └── {uuid}.{ext}      # Uploaded files (safe filenames)
│
├── simulations/
│   └── sim_{uuid}/
│       ├── state.json            # Simulation state
│       ├── simulation_config.json # Generated config
│       ├── twitter_profiles.csv  # Twitter agent profiles
│       ├── reddit_profiles.json  # Reddit agent profiles
│       ├── twitter_simulation.db # SQLite (simulation data)
│       └── reddit_simulation.db  # SQLite (simulation data)
│
└── reports/
    └── report_{uuid}/
        ├── report.json           # Report metadata
        ├── report.md             # Final combined report
        ├── section_01.md         # Individual sections
        ├── section_02.md
        ├── agent_log.jsonl       # Agent tool calls + responses
        ├── console_log.txt       # Console output
        └── progress.json         # Generation progress
```
