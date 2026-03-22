# MiroFish — Usage Guide & Executive Summary

## What is MiroFish?

MiroFish is an **AI-powered prediction engine** that transforms documents into interactive digital worlds populated by autonomous agents. It builds knowledge graphs from your data, generates realistic agent personas, simulates their interactions, and produces analytical reports — all automatically.

**In plain terms:** You feed it information, it builds a living simulation of the people and organizations involved, runs "what if" scenarios, and tells you what's likely to happen.

## Executive Summary

### The Core Idea

Traditional analysis reads data and draws conclusions. MiroFish goes further: it **simulates the actors themselves** — their personalities, motivations, relationships, and likely behaviors — then lets them interact in a controlled digital environment. The emergent patterns from these interactions reveal insights that static analysis cannot.

### How It Works (5 Steps)

```
Documents → Knowledge Graph → Agent Personas → Simulation → Predictive Report
```

1. **Upload** any documents (PDFs, reports, articles, social media data)
2. **Extract** entities (people, brands, organizations) and their relationships automatically
3. **Generate** AI agents with realistic personalities derived from the data
4. **Simulate** their interactions on virtual social media platforms
5. **Analyze** the results through an AI-powered report agent you can chat with

### What Makes It Different

| Traditional Analysis | MiroFish |
|---------------------|----------|
| Static snapshot of data | Dynamic simulation of behavior |
| "Here's what happened" | "Here's what will likely happen" |
| Analyst interprets data | Agents generate emergent behavior |
| Single perspective | Thousands of interacting perspectives |
| One-time report | Interactive, queryable digital world |

---

## Use Cases

### 1. Brand Intelligence & Profiling

Feed MiroFish everything about a brand — news articles, social media mentions, product reviews, competitor reports, industry analyses — and it will:

- **Map the brand ecosystem**: Identify all key players (the brand, competitors, influencers, regulators, customer segments, media outlets)
- **Profile each actor**: Generate detailed personas with motivations, sentiment, influence levels, and likely behaviors
- **Simulate scenarios**: "What happens if brand X launches a new product?", "How does the market react to a pricing change?", "What if a PR crisis hits?"
- **Predict reactions**: See how different stakeholder groups would respond before it happens in the real world

**Example workflow:**
```
Input:  Brand's annual report + 50 news articles + competitor analysis + customer reviews
Output: Digital twin of the brand's ecosystem with predictive simulation capabilities
```

### 2. Market Prediction

Upload market research, financial reports, and industry news to simulate how markets react to specific events:

- Product launches and their reception
- Pricing strategy changes
- Regulatory announcements
- Competitive moves

### 3. Crisis Simulation

Test crisis scenarios before they happen:

- Feed the system information about your organization
- Simulate a crisis event (data breach, product recall, leadership scandal)
- Watch how stakeholders react in real-time
- Identify which narratives gain traction
- Test response strategies

### 4. Competitive Intelligence

Build a simulation of your competitive landscape:

- Map competitor relationships and market dynamics
- Simulate strategic moves and counter-moves
- Identify potential disruption scenarios
- Profile key decision-makers and their likely responses

### 5. Public Opinion Forecasting

Predict how public discourse evolves around events:

- Policy announcements
- Social movements
- Cultural events
- Technology releases

### 6. Content & Narrative Testing

Test how narratives spread before publishing:

- Which messages resonate with which audiences?
- How quickly does information spread?
- What counter-narratives emerge?
- Which influencers amplify or oppose your message?

---

## Quick Start Guide

### Prerequisites

- **Node.js** >= 18 (`node -v`)
- **Python** >= 3.11 (`python --version`)
- **uv** package manager (`uv --version`)
- **LLM API key** (OpenAI, Qwen, or any OpenAI-compatible provider)
- **Zep Cloud API key** (free tier available at https://app.getzep.com/)

### Setup

```bash
# 1. Clone and enter the project
git clone https://github.com/ismaeldosil/MiroPez.git
cd MiroPez

# 2. Configure environment
cp .env.example .env
# Edit .env with your API keys:
#   LLM_API_KEY=your_key
#   LLM_BASE_URL=https://your-provider/v1
#   LLM_MODEL_NAME=your-model
#   ZEP_API_KEY=your_zep_key

# 3. Install everything
npm run setup:all

# 4. Start
npm run dev
```

Open http://localhost:3000

### Your First Simulation

#### Step 1: Upload & Build Graph

1. Click **"New Project"** on the home page
2. Upload your documents (PDF, Markdown, or plain text)
3. Describe what you want to simulate in the **"Simulation Requirement"** field
   - Example: *"Simulate how consumers and competitors react to Brand X launching a premium product line"*
4. Click **"Generate Ontology"** — the AI will analyze your documents and identify entity types
5. Click **"Build Graph"** — watch as the knowledge graph populates with entities and relationships

#### Step 2: Prepare Environment

1. Review the extracted entities (people, organizations, brands)
2. Select which platforms to simulate (Twitter, Reddit, or both)
3. Click **"Prepare Simulation"**
   - The system generates realistic agent profiles (personality, stance, influence level)
   - It also generates simulation parameters (timing, activity patterns, initial events)

#### Step 3: Run Simulation

1. Set the number of rounds (each round = 1 simulated hour by default)
2. Click **"Start"**
3. Watch agents interact in real-time:
   - Posts, comments, likes, reposts
   - Opinion shifts and narrative formation
   - Viral content emergence

#### Step 4: Generate Report

1. Click **"Generate Report"**
2. The Report Agent autonomously:
   - Searches the knowledge graph for key facts
   - Analyzes simulation data
   - Writes a multi-section analytical report
3. Sections appear progressively as they're generated
4. Download the full report as Markdown

#### Step 5: Deep Interaction

1. **Chat with the Report Agent**: Ask follow-up questions about the simulation
   - *"Which entity had the most influence?"*
   - *"What were the main narrative themes?"*
   - *"How did sentiment evolve over time?"*
2. **Interview agents**: Extract individual agent perspectives and reasoning

---

## Tips for Best Results

### Document Quality Matters

The richer your input documents, the better the simulation:

| Input Quality | Result Quality |
|--------------|----------------|
| 1 short article | Basic entity map, generic agent behavior |
| 10-20 articles + reports | Good entity coverage, nuanced personas |
| 50+ diverse sources | Rich ecosystem model, realistic emergent behavior |

### Writing Good Simulation Requirements

Be specific about what you want to predict:

- **Vague:** *"Simulate the tech industry"*
- **Better:** *"Simulate how Apple's customer base, competitors, and tech media would react to Apple releasing an AI-powered search engine that competes with Google"*
- **Best:** *"Simulate the 72-hour social media reaction to Apple announcing an AI search engine at WWDC, focusing on: (1) consumer sentiment shift, (2) Google's likely PR response, (3) tech media narrative formation, (4) regulatory attention"*

### Choosing Round Count

| Scenario | Recommended Rounds | Simulated Time |
|----------|-------------------|----------------|
| Quick reaction test | 10-20 | 10-20 hours |
| 3-day discourse | 40-72 | 2-3 days |
| Week-long trend | 100-168 | 4-7 days |

### LLM Provider Recommendations

| Provider | Model | Best For |
|----------|-------|----------|
| Alibaba (Qwen) | `qwen-plus` | Chinese-language simulations, cost-effective |
| OpenAI | `gpt-4o` | English-language, highest quality |
| OpenAI | `gpt-4o-mini` | Quick iterations, lower cost |
| DeepSeek | `deepseek-chat` | Balanced quality/cost |

---

## Architecture at a Glance

```
┌─────────────────────────────────────────────────────┐
│                    User Browser                      │
│                  (localhost:3000)                     │
└──────────────────────┬──────────────────────────────┘
                       │ HTTP/REST
┌──────────────────────▼──────────────────────────────┐
│                  Flask Backend                        │
│                 (localhost:5001)                      │
│                                                      │
│  ┌──────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  Graph    │  │  Simulation  │  │    Report     │  │
│  │  Builder  │  │   Manager    │  │    Agent      │  │
│  └─────┬────┘  └──────┬───────┘  └───────┬───────┘  │
└────────┼───────────────┼─────────────────┼──────────┘
         │               │                 │
    ┌────▼────┐   ┌──────▼──────┐   ┌──────▼──────┐
    │   Zep   │   │   OASIS     │   │   LLM API   │
    │  Cloud  │   │  (subprocess)│   │(Qwen/GPT/etc)│
    └─────────┘   └─────────────┘   └─────────────┘
```

For detailed architecture documentation, see:
- [Architecture Overview](./architecture-overview.md) — System diagrams and tech stack
- [Data Flow](./data-flow.md) — Sequence diagrams and state machines
- [API Reference](./api-reference.md) — All 50+ endpoints
- [Service Architecture](./service-architecture.md) — Service layer details
- [Deployment Guide](./deployment.md) — Setup and infrastructure

---

## FAQ

**Q: How much does it cost to run?**
A: The main cost is LLM API calls. A typical simulation (20 agents, 40 rounds) uses roughly 500K-2M tokens. Zep Cloud has a free tier sufficient for small projects.

**Q: Can I use my own LLM?**
A: Yes. Any OpenAI-compatible API works. Set `LLM_BASE_URL` and `LLM_MODEL_NAME` in your `.env`.

**Q: What file formats are supported?**
A: PDF, Markdown (.md), and plain text (.txt). Max 50MB per upload.

**Q: Can I run it offline?**
A: The backend runs locally, but you need internet access for the LLM API and Zep Cloud.

**Q: How accurate are the predictions?**
A: MiroFish generates *plausible scenarios*, not guaranteed predictions. Think of it as a sophisticated brainstorming tool — it surfaces possibilities and dynamics you might not have considered. Accuracy improves with richer input data and more specific simulation requirements.

**Q: Can I feed it real-time social media data?**
A: Not directly in the current version, but you can export social media data to text files and upload them. A real-time ingestion pipeline is a natural next step.
