# 🛠️ Agentic Framework Specification (V1.0)

This document outlines the architecture, configuration, and extensibility patterns for our modular agentic framework.

## 1. System Architecture

The framework follows a **decoupled, layer-based architecture** to ensure that tools, models, and interfaces can be swapped without breaking core logic.

### Layers:

* **Interface Layer:** Next.js Web UI using Vercel AI SDK for streaming and tool-call rendering.
* **Orchestration Layer:** LangGraph-based state machine managing Agent-to-Agent communication.
* **Capability Layer (MCP):** Model Context Protocol servers providing external tool access (Search, GitHub, Local Files).
* **Memory Layer:** PostgreSQL + `pgvector` for long-term storage and RAG.

---

## 2. Environment Configuration (`.env.template`)

Copy this to a `.env` file at the root of your project.

```bash
# ==========================================
# 🤖 CORE LLM CONFIGURATION
# ==========================================
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=ant-api-...
GOOGLE_GENERATIVE_AI_API_KEY=...
DEFAULT_MODEL=gpt-4o-2024-08-06

# ==========================================
# 🧠 ORCHESTRATION & STATE
# ==========================================
# Options: 'graph' (complex), 'linear' (simple chain), 'swarm' (autonomous)
ORCHESTRATOR_STRATEGY=graph
MAX_AGENT_LOOPS=15
AGENT_VERBOSITY=debug # debug, info, error

# ==========================================
# 💾 VECTOR DATABASE (RAG & MEMORY)
# ==========================================
VECTOR_DB_TYPE=chromadb
CHROMA_PATH=./data/vector_db
COLLECTION_NAME=agent_memory
EMBEDDING_MODEL=text-embedding-3-small

# ==========================================
# 🔌 MCP SERVERS (TOOL PROVIDERS)
# ==========================================
# List comma-separated URLs for the Orchestrator to fetch tools from
MCP_ENDPOINTS=http://localhost:8001/mcp/web-search,http://localhost:8002/mcp/filesystem

# ==========================================
# 🌐 WEB INTERFACE & API
# ==========================================
API_PORT=8000
WEB_UI_PORT=3000
NEXT_PUBLIC_API_URL=http://localhost:8000
AUTH_SECRET=generate-a-secure-key-here

# ==========================================
# 🛠️ THIRD-PARTY INTEGRATIONS
# ==========================================
TAVILY_API_KEY=tvly-...
GITHUB_TOKEN=ghp_...
SLACK_BOT_TOKEN=xoxb-...

```

---

## 3. Project Directory Structure

```text
├── apps/
│   ├── web/                # Next.js Frontend
│   └── server/             # FastAPI Backend (Orchestrator)
├── packages/
│   ├── agents/             # Logic for specialized agents
│   ├── mcp-servers/        # Custom MCP tool implementations
│   └── database/           # Vector DB schemas and migration scripts
├── config/
│   └── agents.yaml         # Agent definitions (Roles, Goals, Backstories)
├── .env.template           # Template for environment variables
└── docker-compose.yml      # Orchestration for local dev (Postgres, UI, API)

```

---

## 4. Extension Workflow

To add a new capability to this framework:

1. **Define Agent:** Add a new entry in `config/agents.yaml`.
2. **Add Tool:** Create a new MCP server or add an endpoint to `MCP_ENDPOINTS`.
3. **Update Graph:** Add a new node in the LangGraph orchestration logic to include the new agent in the workflow.

---

## 5. Deployment Recommendation

* **Development:** Docker Compose for local Vector DB and MCP servers.
* **Production:** Deploy the UI to Vercel, the API/Orchestrator to AWS/GCP, and use Managed Pinecone for the Vector DB.

