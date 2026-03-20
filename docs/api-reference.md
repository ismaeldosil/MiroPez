# MiroFish API Reference

Base URL: `http://localhost:5001`

All responses follow the format:
```json
{
  "success": true|false,
  "data": { ... },
  "error": "Error message (when success=false)"
}
```

## Graph Blueprint (`/api/graph`)

### Project Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/project/list` | List all projects |
| `GET` | `/project/<project_id>` | Get project details |
| `DELETE` | `/project/<project_id>` | Delete project and all files |
| `POST` | `/project/<project_id>/reset` | Reset project to ontology stage |

### Ontology & Graph Building

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/ontology/generate` | Upload files and generate ontology |
| `POST` | `/build` | Build knowledge graph (async) |
| `GET` | `/task/<task_id>` | Query task progress |
| `GET` | `/tasks` | List all tasks |
| `GET` | `/data/<graph_id>` | Get graph nodes and edges |
| `DELETE` | `/delete/<graph_id>` | Delete graph from Zep |

### POST `/ontology/generate`

Upload documents and generate entity/relationship ontology.

**Request:** `multipart/form-data`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `files` | File[] | Yes | PDF, MD, or TXT files |
| `simulation_requirement` | string | Yes | Simulation goal description |
| `project_name` | string | No | Project name |
| `additional_context` | string | No | Extra context for LLM |

**Response:**
```json
{
  "success": true,
  "data": {
    "project_id": "proj_abc123def456",
    "project_name": "My Project",
    "ontology": {
      "entity_types": [
        {
          "name": "Student",
          "description": "University student",
          "attributes": [{"name": "full_name", "type": "text", "description": "..."}],
          "examples": ["Zhang San", "Li Si"]
        }
      ],
      "edge_types": [
        {
          "name": "STUDIES_AT",
          "description": "Attends institution",
          "source_targets": [{"source": "Student", "target": "University"}]
        }
      ]
    },
    "analysis_summary": "...",
    "files": [{"filename": "doc.pdf", "size": 12345}],
    "total_text_length": 50000
  }
}
```

### POST `/build`

Build Zep knowledge graph asynchronously.

**Request:** `application/json`
```json
{
  "project_id": "proj_abc123def456",
  "graph_name": "My Graph",
  "chunk_size": 500,
  "chunk_overlap": 50,
  "force": false
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "project_id": "proj_abc123def456",
    "task_id": "uuid-task-id",
    "message": "Graph build task started, query progress via /task/{task_id}"
  }
}
```

---

## Simulation Blueprint (`/api/simulation`)

### Simulation Lifecycle

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/create` | Create new simulation |
| `POST` | `/prepare` | Prepare simulation environment (async) |
| `POST` | `/prepare/status` | Query preparation progress |
| `POST` | `/start` | Start simulation execution |
| `POST` | `/stop` | Stop running simulation |
| `GET` | `/<sim_id>` | Get simulation state |
| `GET` | `/list` | List all simulations |

### Simulation Data

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/<sim_id>/profiles` | Get agent profiles |
| `GET` | `/<sim_id>/profiles/realtime` | Stream profile generation |
| `GET` | `/<sim_id>/config` | Get simulation config |
| `GET` | `/<sim_id>/config/realtime` | Stream config generation |
| `GET` | `/<sim_id>/run-status` | Get execution status |
| `GET` | `/<sim_id>/run-status/detail` | Detailed status + recent actions |
| `GET` | `/<sim_id>/posts` | Get generated posts |
| `GET` | `/<sim_id>/timeline` | Get timeline by round |
| `GET` | `/<sim_id>/agent-stats` | Agent statistics |
| `GET` | `/<sim_id>/actions` | Action history |

### Entity Inspection

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/entities/<graph_id>` | Get filtered entities |
| `GET` | `/entities/<graph_id>/<entity_uuid>` | Get entity details |
| `GET` | `/entities/<graph_id>/by-type/<type>` | Get entities by type |

### Agent Interview

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/interview/batch` | Batch interview agents |

### POST `/create`

```json
{
  "project_id": "proj_abc123def456",
  "graph_id": "mirofish_abc123",
  "enable_twitter": true,
  "enable_reddit": true
}
```

### POST `/start`

```json
{
  "simulation_id": "sim_abc123def456",
  "platform": "twitter",
  "max_rounds": 20,
  "enable_graph_memory_update": true
}
```

### GET `/<sim_id>/run-status/detail`

```json
{
  "success": true,
  "data": {
    "simulation_id": "sim_abc123def456",
    "status": "running",
    "current_round": 5,
    "max_rounds": 20,
    "twitter_status": "running",
    "reddit_status": "running",
    "recent_actions": [
      {
        "round": 5,
        "agent_id": 3,
        "agent_name": "Zhang San",
        "action": "CREATE_POST",
        "content": "...",
        "timestamp": "2025-12-10T19:30:00"
      }
    ]
  }
}
```

---

## Report Blueprint (`/api/report`)

### Report Lifecycle

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/generate` | Start report generation (async) |
| `POST` | `/generate/status` | Query generation progress |
| `GET` | `/<report_id>` | Get full report |
| `GET` | `/by-simulation/<sim_id>` | Get report for simulation |
| `GET` | `/list` | List all reports |
| `GET` | `/<report_id>/download` | Download as Markdown |
| `DELETE` | `/<report_id>` | Delete report |
| `GET` | `/check/<sim_id>` | Check if report exists |

### Report Sections (Streaming)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/<report_id>/progress` | Get generation progress |
| `GET` | `/<report_id>/sections` | Get all generated sections |
| `GET` | `/<report_id>/section/<index>` | Get single section |

### Agent Logs

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/<report_id>/agent-log` | Get agent execution log |
| `GET` | `/<report_id>/agent-log/stream` | Get full agent log |
| `GET` | `/<report_id>/console-log` | Get console output |
| `GET` | `/<report_id>/console-log/stream` | Get full console log |

### Chat with Report Agent

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/chat` | Chat with Report Agent |

### POST `/chat`

```json
{
  "simulation_id": "sim_abc123def456",
  "message": "What were the main trends?",
  "chat_history": [
    {"role": "user", "content": "..."},
    {"role": "assistant", "content": "..."}
  ]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "response": "Based on the simulation data...",
    "tool_calls": [
      {
        "tool": "search_graph",
        "parameters": {"query": "main trends", "limit": 10},
        "result_summary": "Found 15 relevant facts"
      }
    ],
    "sources": ["Entity: University X", "Fact: ..."]
  }
}
```

### Debug Tools

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/tools/search` | Search graph directly |
| `POST` | `/tools/statistics` | Get graph statistics |

---

## Health Check

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Service health check |

**Response:**
```json
{"status": "ok", "service": "MiroFish Backend"}
```
