# MiroFish Service Architecture

## Service Layer Overview

```mermaid
graph TB
    subgraph "API Layer (Flask Blueprints)"
        GraphBP["/api/graph"]
        SimBP["/api/simulation"]
        ReportBP["/api/report"]
    end

    subgraph "Service Layer"
        OG["OntologyGenerator<br/>Document analysis → schema"]
        GB["GraphBuilderService<br/>Zep graph CRUD"]
        TP["TextProcessor<br/>Chunking & preprocessing"]
        ZER["ZepEntityReader<br/>Entity extraction"]
        OPG["OasisProfileGenerator<br/>Agent persona creation"]
        SCG["SimConfigGenerator<br/>Simulation params"]
        SM["SimulationManager<br/>State management"]
        SR["SimulationRunner<br/>Process lifecycle"]
        IPC["SimulationIPC<br/>File-based events"]
        RA["ReportAgent<br/>Autonomous report writer"]
        ZT["ZepToolsService<br/>Graph search & stats"]
        ZGMU["ZepGraphMemoryUpdater<br/>Memory sync"]
    end

    subgraph "External"
        LLM["LLM API"]
        Zep["Zep Cloud"]
        OASIS["OASIS Engine"]
    end

    GraphBP --> OG & GB & TP
    SimBP --> ZER & OPG & SCG & SM & SR
    ReportBP --> RA

    OG --> LLM
    GB --> Zep
    ZER --> Zep
    OPG --> LLM & Zep
    SCG --> LLM
    RA --> ZT & LLM
    ZT --> Zep
    SR --> IPC & OASIS
    ZGMU --> Zep
```

## Service Descriptions

### OntologyGenerator

Analyzes document content and generates a schema of entity types and relationships for the knowledge graph.

```mermaid
flowchart LR
    Input["Document texts +<br/>Simulation requirement"]
    --> LLM["LLM Call<br/>(structured JSON output)"]
    --> Validate["Validate & process<br/>• Ensure 10 entity types<br/>• Add Person/Organization fallbacks<br/>• Cap at Zep limits"]
    --> Output["Ontology<br/>entity_types[] + edge_types[]"]
```

**Key constraints:**
- Exactly 10 entity types (8 specific + 2 fallback: Person, Organization)
- Max 10 edge types (Zep API limit)
- Entity types must be real-world actors (not abstract concepts)
- Reserved attribute names: `name`, `uuid`, `group_id`, `created_at`, `summary`

### GraphBuilderService

Interfaces with Zep Cloud API to create, populate, and query knowledge graphs.

```mermaid
flowchart TD
    Create["create_graph(name)"]
    --> SetOntology["set_ontology(graph_id, ontology)<br/>Dynamic Pydantic class creation"]
    --> AddBatches["add_text_batches(chunks, batch_size=3)<br/>EpisodeData per chunk"]
    --> WaitEpisodes["_wait_for_episodes(uuids)<br/>Poll processed status every 3s"]
    --> GetData["get_graph_data(graph_id)<br/>Fetch nodes + edges"]
```

**Episode processing:** After adding text chunks as episodes, Zep asynchronously extracts entities and relationships. The service polls every 3 seconds with a 600-second timeout.

### SimulationManager

Manages the full simulation lifecycle from creation through completion.

```mermaid
stateDiagram-v2
    [*] --> CREATED: create_simulation()
    CREATED --> PREPARING: prepare_simulation()

    state PREPARING {
        [*] --> ReadEntities: ZepEntityReader
        ReadEntities --> GenProfiles: OasisProfileGenerator
        GenProfiles --> GenConfig: SimulationConfigGenerator
        GenConfig --> SaveFiles: Save to disk
    }

    PREPARING --> READY: All files generated
    PREPARING --> FAILED: Error
    READY --> RUNNING: SimulationRunner.start()
    RUNNING --> COMPLETED: Max rounds
    RUNNING --> STOPPED: User stops
    RUNNING --> FAILED: Error
```

### SimulationRunner

Spawns and manages OASIS simulation as a subprocess.

```mermaid
flowchart TD
    Start["start(simulation_id, platform, max_rounds)"]
    --> Spawn["subprocess.Popen(python script, args)"]
    --> Monitor["Monitor via IPC event log"]

    Monitor --> Read["Read actions from event file"]
    Read --> Update["Update simulation state"]
    Update --> Monitor

    Stop["stop(simulation_id)"]
    --> Terminate["Process.terminate()"]
    --> Cleanup["Cleanup temp files"]

    subgraph "Subprocess (OASIS)"
        Script["run_twitter/reddit_simulation.py"]
        --> LoadProfiles["Load agent profiles"]
        --> LoadConfig["Load simulation config"]
        --> Loop["Simulation loop"]
        --> WriteEvents["Write to event log"]
    end
```

**Cleanup:** `SimulationRunner.register_cleanup()` ensures all simulation processes are terminated on server shutdown via `atexit`.

### ReportAgent

Autonomous agent that generates analysis reports by iteratively searching the knowledge graph.

```mermaid
flowchart TD
    Start["generate_report()"]
    --> Plan["LLM: Plan report structure<br/>(outline with sections)"]
    --> Loop{"For each section"}

    Loop --> Query["Craft search query"]
    Query --> Search["ZepToolsService.search_graph()"]
    Search --> Facts["Retrieved facts + entities"]
    Facts --> Synthesize["LLM: Write section content"]
    Synthesize --> Save["Save section_NN.md"]
    Save --> Log["Log to agent_log.jsonl"]
    Log --> Loop

    Loop -->|Done| Combine["Combine all sections"]
    Combine --> Final["Save final report.md"]

    subgraph "Tool Calls"
        T1["search_graph(query, limit)"]
        T2["get_graph_statistics(graph_id)"]
    end
```

**Chat mode:** The same agent supports conversational interaction via `chat(message, history)`, autonomously calling tools to answer user questions.

### SimulationConfigGenerator

Uses LLM to generate optimal simulation parameters in a multi-step process.

```mermaid
flowchart TD
    Context["Build context<br/>(requirement + entities + document)"]
    --> Step1["Step 1: Generate time config<br/>hours, peak/off-peak, rounds"]
    --> Step2["Step 2: Generate event config<br/>initial posts, hot topics"]
    --> Step3["Step 3: Generate agent configs<br/>(batches of 15)"]
    --> Step4["Step 4: Assign post authors<br/>Match poster_type → agent_id"]
    --> Step5["Step 5: Platform configs<br/>Twitter/Reddit specifics"]
    --> Output["SimulationParameters"]
```

**Batching strategy:** Agent configs are generated in batches of 15 to avoid LLM context limits and JSON truncation.

### OasisProfileGenerator

Creates realistic social media agent personas from graph entities.

```mermaid
flowchart LR
    Entity["Graph Entity<br/>(name, type, summary, edges)"]
    --> Context["Enrich with Zep search<br/>(related facts)"]
    --> LLM["LLM generates persona<br/>• Personality traits<br/>• Biography<br/>• Initial beliefs<br/>• Action preferences"]
    --> Profile["Agent Profile<br/>Twitter CSV / Reddit JSON"]
```

**Platform formats:**
- **Twitter:** CSV with columns for handle, bio, personality
- **Reddit:** JSON array with full profile objects

### ZepToolsService

Wrapper around Zep Cloud API for graph queries.

| Method | Description |
|--------|-------------|
| `search_graph(graph_id, query, limit)` | Semantic search for facts/entities |
| `get_graph_statistics(graph_id)` | Node/edge counts, entity types |

### ZepGraphMemoryUpdater

Syncs simulation outcomes back to the knowledge graph.

```mermaid
flowchart LR
    Actions["Simulation actions<br/>(posts, likes, follows)"]
    --> Describe["Describe as natural language"]
    --> AddEpisode["Zep: Add as new episode"]
    --> Enrich["Graph auto-enriches<br/>with new relationships"]
```

## Inter-Service Communication

```mermaid
graph LR
    subgraph "Sync (direct calls)"
        OG -->|generates| Ontology
        GB -->|uses| Ontology
        ZER -->|reads from| GB
        OPG -->|uses entities from| ZER
        SCG -->|uses entities from| ZER
        RA -->|searches via| ZT
    end

    subgraph "Async (background threads)"
        GraphBP -->|"Thread"| GB
        SimBP -->|"Thread"| SM
        ReportBP -->|"Thread"| RA
    end

    subgraph "IPC (subprocess)"
        SR -->|"File-based events"| OASIS_Proc["OASIS Process"]
    end
```

## Error Handling

All services follow a consistent pattern:

1. **API Layer:** Returns `{success: false, error: "...", traceback: "..."}` with appropriate HTTP status
2. **Service Layer:** Raises exceptions, caught by API handlers
3. **Background Tasks:** Update Task status to FAILED with error message
4. **Subprocesses:** Errors captured via IPC event log
5. **LLM Calls:** 3-attempt retry with decreasing temperature, JSON repair on truncation
