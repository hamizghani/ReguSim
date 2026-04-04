<div align="center">

<img src="./static/image/regusim-banner.png" alt="ReguSim" width="100%"/>

# ReguSim

**AI-Powered Regulatory Impact Assessment Platform for Indonesian Financial Regulation**

*Multi-agent swarm simulation that predicts how stakeholders — regulators, banks, fintech, and the public — will react to proposed regulations. Entirely local, entirely private.*

[![Docker](https://img.shields.io/badge/Docker-Build-2496ED?style=flat-square&logo=docker&logoColor=white)](https://hub.docker.com/)
[![License: AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue?style=flat-square)](./LICENSE)

</div>

## What is ReguSim?

ReguSim is an AI-powered RegTech platform that uses **Multi-Agent Swarm Simulation** to perform **Regulatory Impact Assessment (RIA)** and **Digital Financial Innovation (IKD) sandbox testing** for Indonesian financial regulation.

Upload a regulatory draft, policy proposal, or OJK circular, and ReguSim generates stakeholder agents — representing OJK commissioners, Bank Indonesia officials, fintech founders, bank executives, consumer advocates, and the public — that simulate realistic reactions, debates, and compliance behaviors. The result is a comprehensive RIA report based on predicted stakeholder responses.

### Key Capabilities

| Feature | Description |
|---|---|
| **Regulatory Graph** | Extracts regulatory entities (OJK, BI, fintech, banks) and relationships from policy documents. Builds a knowledge graph with Neo4j + GraphRAG. |
| **Stakeholder Agents** | Generates realistic personas for Indonesian financial stakeholders with behavioral profiles, institutional memory, and regulatory positions. |
| **Impact Simulation** | Agents interact on simulated platforms: debating, responding to regulations, forming coalitions, shifting positions — tracked in real time. |
| **RIA Report** | An AI report agent analyzes the simulation, interviews stakeholders, searches the knowledge graph, and generates a structured Regulatory Impact Assessment. |
| **Stakeholder Chat** | Chat with any simulated stakeholder to explore their reasoning, regulatory concerns, and institutional perspective. |

### Technical Stack

| Component | Technology |
|---|---|
| LLM | **Ollama** (qwen2.5, llama3, etc.) — fully local |
| Knowledge Graph | **Neo4j Community Edition 5.15** |
| Embeddings | **nomic-embed-text** via Ollama |
| Simulation Engine | **OASIS** (camel-ai) |
| Frontend | Vue 3 + Vite |
| Backend | Python Flask |
| Infrastructure | Docker Compose |

## Workflow

1. **Regulatory Graph** — Extracts regulatory entities (regulators, financial institutions, fintech companies, policies) and relationships from your document. Builds a regulatory knowledge graph with Neo4j.
2. **Stakeholder Setup** — Generates stakeholder agent personas — OJK officials, bank executives, fintech founders, consumer advocates — each with unique institutional perspective, behavioral profile, and regulatory memory.
3. **Impact Simulation** — Stakeholder agents interact on simulated platforms: debating regulatory proposals, forming compliance strategies, lobbying, and shifting positions. The system tracks sentiment, coalition dynamics, and behavioral shifts in real time.
4. **RIA Report** — A report agent analyzes the post-simulation stakeholder ecosystem, interviews key agents, searches the knowledge graph for evidence, and generates a structured Regulatory Impact Assessment.
5. **Stakeholder Chat** — Chat with any simulated stakeholder. Ask an OJK commissioner why they favor a particular enforcement approach, or a fintech founder about compliance challenges. Full institutional memory persists.

## Screenshot

<div align="center">
<img src="./static/image/regusim-screenshot.jpg" alt="ReguSim — English UI" width="100%"/>
</div>

## Quick Start

### Prerequisites

- Docker & Docker Compose (recommended), **or**
- Python 3.11+, Node.js 18+, Neo4j 5.15+, Ollama

### Option A: Docker (easiest)

```bash
git clone https://github.com/nikmcfly/ReguSim.git
cd ReguSim
cp .env.example .env

# Start all services (Neo4j, Ollama, ReguSim)
docker compose up -d

# Pull the required models into Ollama
docker exec regusim-ollama ollama pull qwen2.5:32b
docker exec regusim-ollama ollama pull nomic-embed-text
```

Open `http://localhost:3000` — that's it.

### Option B: Manual

**1. Start Neo4j**

```bash
docker run -d --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/regusim \
  neo4j:5.15-community
```

**2. Start Ollama & pull models**

```bash
ollama serve &
ollama pull qwen2.5:32b      # LLM (or qwen2.5:14b for less VRAM)
ollama pull nomic-embed-text  # Embeddings (768d)
```

**3. Configure & run backend**

```bash
cp .env.example .env
# Edit .env if your Neo4j/Ollama are on non-default ports

cd backend
pip install -r requirements.txt
python run.py
```

**4. Run frontend**

```bash
cd frontend
npm install
npm run dev
```

Open `http://localhost:3000`.

## Configuration

All settings are in `.env` (copy from `.env.example`):

```bash
# LLM — points to local Ollama (OpenAI-compatible API)
LLM_API_KEY=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL_NAME=qwen2.5:32b

# Neo4j
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=regusim

# Embeddings
EMBEDDING_MODEL=nomic-embed-text
EMBEDDING_BASE_URL=http://localhost:11434
```

Works with any OpenAI-compatible API — swap Ollama for Claude, GPT, or any other provider by changing `LLM_BASE_URL` and `LLM_API_KEY`.

## Architecture

This fork introduces a clean abstraction layer between the application and the graph database:

```
┌─────────────────────────────────────────┐
│              Flask API                   │
│  graph.py  simulation.py  report.py     │
└──────────────┬──────────────────────────┘
               │ app.extensions['neo4j_storage']
┌──────────────▼──────────────────────────┐
│           Service Layer                  │
│  EntityReader  GraphToolsService         │
│  GraphMemoryUpdater  ReportAgent         │
└──────────────┬──────────────────────────┘
               │ storage: GraphStorage
┌──────────────▼──────────────────────────┐
│         GraphStorage (abstract)          │
│              │                            │
│    ┌─────────▼─────────┐                │
│    │   Neo4jStorage     │                │
│    │  ┌───────────────┐ │                │
│    │  │ EmbeddingService│ ← Ollama       │
│    │  │ NERExtractor   │ ← Ollama LLM   │
│    │  │ SearchService  │ ← Hybrid search │
│    │  └───────────────┘ │                │
│    └───────────────────┘                │
└─────────────────────────────────────────┘
               │
        ┌──────▼──────┐
        │  Neo4j CE   │
        │  5.15       │
        └─────────────┘
```

**Key design decisions:**

- `GraphStorage` is an abstract interface — swap Neo4j for any other graph DB by implementing one class
- Dependency injection via Flask `app.extensions` — no global singletons
- Hybrid search: 0.7 × vector similarity + 0.3 × BM25 keyword search
- Synchronous NER/RE extraction via local LLM (replaces Zep's async episodes)
- All original dataclasses and LLM tools (InsightForge, Panorama, Agent Interviews) preserved

## Hardware Requirements

| Component | Minimum | Recommended |
|---|---|---|
| RAM | 16 GB | 32 GB |
| VRAM (GPU) | 10 GB (14b model) | 24 GB (32b model) |
| Disk | 20 GB | 50 GB |
| CPU | 4 cores | 8+ cores |

CPU-only mode works but is significantly slower for LLM inference. For lighter setups, use `qwen2.5:14b` or `qwen2.5:7b`.

## Use Cases

- **Regulatory Impact Assessment (RIA)** — simulate how financial stakeholders will react to a proposed OJK regulation before implementation
- **IKD Sandbox Testing** — test Digital Financial Innovation (Inovasi Keuangan Digital) scenarios against simulated regulatory and market responses
- **Policy Stress Testing** — feed policy drafts and observe simulated stakeholder compliance, resistance, and market dynamics
- **Stakeholder Analysis** — understand how different financial ecosystem participants (regulators, banks, fintech, consumers) will interact under new regulatory conditions

## License

AGPL-3.0 — same as the original ReguSim project. See [LICENSE](./LICENSE).

## Credits & Attribution

ReguSim is built on the [MiroFish](https://github.com/666ghj/MiroFish) multi-agent simulation engine by [666ghj](https://github.com/666ghj), originally supported by [Shanda Group](https://www.shanda.com/). The simulation engine is powered by [OASIS](https://github.com/camel-ai/oasis) from the CAMEL-AI team. ReguSim adapts this technology for Indonesian financial regulatory impact assessment.

**Modifications in this fork:**
- Backend migrated from Zep Cloud to local Neo4j CE 5.15 + Ollama
- Entire frontend translated from Chinese to English (20 files, 1,000+ strings)
- All Zep references replaced with Neo4j across the UI
- Rebranded to ReguSim
