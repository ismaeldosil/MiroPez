# MiroPez — AI Prediction Engine for Brand Intelligence

**Repo:** https://github.com/trafiflow/MiroPez (private)

---

## TL;DR

MiroPez is an open-source AI engine that converts documents into **living digital simulations**. You feed it data (articles, reports, reviews), it automatically builds a knowledge graph, generates thousands of AI agents with realistic personalities, and simulates how they interact — predicting market reactions, brand perception shifts, and crisis outcomes **before they happen**.

---

## What It Does

```
Documents  →  Knowledge Graph  →  AI Agents  →  Simulation  →  Predictive Report
```

1. **Upload** any documents about a brand, market, or topic (PDFs, articles, reports)
2. **Automatic extraction** of all key players: brands, competitors, influencers, media, consumer segments
3. **AI generates realistic agent personas** — each with personality, motivations, stance, and influence level
4. **Simulation runs** on virtual social media (Twitter/Reddit) — agents post, react, argue, amplify narratives
5. **AI Report Agent** analyzes results and answers questions about what happened and why

---

## Why This Matters for Us

### Brand Intelligence & Profiling

Feed MiroPez everything about a brand — news, reviews, competitor data, social media — and it maps the entire ecosystem automatically:

- **Who are the key players?** (the brand, competitors, influencers, regulators, customer segments)
- **How does each actor behave?** (sentiment, influence level, response patterns)
- **What happens if X?** (product launch, price change, PR crisis, competitor move)

### Concrete Use Cases

| Use Case | What You Feed It | What You Get |
|----------|-----------------|--------------|
| **Brand Ecosystem Map** | 50 articles about a brand | Complete stakeholder map with relationships and influence levels |
| **Product Launch Prediction** | Market research + competitor data | Simulated market reaction from consumers, media, competitors |
| **Crisis Simulation** | Company background + crisis scenario | How stakeholders react, which narratives go viral, best response strategy |
| **Competitive Intelligence** | Industry reports + competitor profiles | Simulated competitive moves and counter-moves |
| **Narrative Testing** | Draft messaging + audience data | Which messages resonate, how they spread, what counter-narratives emerge |

---

## How It Works (Under the Hood)

### Architecture

```
┌──────────────────────────────────────────────┐
│              User Browser (:3000)             │
└──────────────────┬───────────────────────────┘
                   │
┌──────────────────▼───────────────────────────┐
│             Flask Backend (:5001)              │
│                                               │
│  Graph Builder → Simulation Manager → Report  │
│       │               │              Agent    │
└───────┼───────────────┼──────────────┼───────┘
        │               │              │
   ┌────▼────┐   ┌──────▼─────┐  ┌────▼─────┐
   │   Zep   │   │   OASIS    │  │ LLM API  │
   │  Cloud  │   │  Engine    │  │(Qwen/GPT)│
   └─────────┘   └────────────┘  └──────────┘
```

- **Zep Cloud** — Knowledge graph storage + semantic search (free tier available)
- **OASIS** — Multi-agent social simulation engine (open-source, by CAMEL-AI)
- **LLM API** — Any OpenAI-compatible provider (Qwen, GPT-4, DeepSeek)

### Tech Stack

- **Frontend:** Vue.js 3 + D3.js (graph visualization) + Vite
- **Backend:** Python/Flask + OpenAI SDK + Zep SDK
- **Simulation:** CAMEL-OASIS (multi-agent framework)
- **Deployment:** Docker + GitHub Actions CI/CD

---

## Quick Demo Flow

1. Upload a PDF about a brand (e.g., annual report + 20 news articles)
2. System identifies ~30-50 entities (people, companies, media outlets)
3. Generates agent profiles: *"Sarah Chen, tech journalist, moderate influence, skeptical of corporate claims"*
4. Runs 40-round simulation (≈40 hours of simulated social media activity)
5. Report Agent writes multi-section analysis:
   - Key narrative threads that emerged
   - Most influential actors and their impact
   - Sentiment evolution by stakeholder group
   - Predicted outcomes and risk areas
6. You can chat with the Report Agent: *"What would happen if competitor Y undercuts pricing by 20%?"*

---

## What's Already Built

- Full 5-step workflow (upload → graph → simulate → report → interact)
- 50+ REST API endpoints
- Real-time simulation monitoring (posts, actions, agent stats)
- Incremental report generation (sections stream as they're written)
- Conversational AI agent that searches the knowledge graph to answer questions
- Docker deployment ready
- Full architecture documentation with Mermaid diagrams

## What Could Be Next

- Real-time social media data ingestion (scraping → auto-upload pipeline)
- Brand health dashboard with recurring simulations
- Side-by-side scenario comparison
- Quantitative brand perception metrics by segment
- Integration with our existing data sources

---

## Try It

```bash
git clone https://github.com/trafiflow/MiroPez.git
cd MiroPez
cp .env.example .env   # Add LLM_API_KEY + ZEP_API_KEY
npm run setup:all
npm run dev             # http://localhost:3000
```

Full documentation: `docs/USAGE.md`

---

*Built on [MiroFish](https://github.com/666ghj/MiroFish) (open-source, CAMEL-AI ecosystem)*
