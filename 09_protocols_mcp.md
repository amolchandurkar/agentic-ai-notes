# 09 — Protocols: ACP, A2A, AG-UI & MCP Deep Dive

> **Key idea:** Without common protocols, building a multi-agent ecosystem creates a "Tower of Babel" — N-squared custom integrations. Protocols standardise how agents discover, describe, and communicate.

---

## The Protocol Problem

**The "Obvious" Answer (and why it fails):**  
Custom REST APIs between every pair of agents/tools.

```mermaid
flowchart LR
    A1[Agent A] <-->|Custom API 1| B1[Tool B]
    A1 <-->|Custom API 2| C1[Tool C]
    A2[Agent X] <-->|Custom API 3| B1
    A2 <-->|Custom API 4| D1[Tool D]
    A3[Agent Y] <-->|Custom API 5| C1
    A3 <-->|Custom API 6| D1
```

**Result:**
- Vendor lock-in: OpenAI agent locked to OpenAI ecosystem
- N-squared problem: 100 agents → ~5,000 custom integrations
- Fundamentally unscalable

---

## The 4 Emerging Protocols

```mermaid
flowchart TD
    subgraph PROTO ["Protocol Landscape"]
        ACP["ACP\n(Agent Communication Protocol)\n'One protocol to rule them all'\nComprehensive — covers everything"]
        A2A["A2A\n(Agent-to-Agent)\nFocused on agent conversations\nStandard for 'chat between agents'"]
        AGUI["AG-UI\n(Agent-User Interaction)\nFocused on agent → human UI\n'Generative UI'"]
        MCP["MCP\n(Model-Context Protocol)\nFocused on agent → tool integration\n'OpenAPI for Agents'"]
    end
```

---

## Protocol 1 — ACP (Agent Communication Protocol)

- Broad, **open-source** specification — universal "language" for all agent interactions
- If HTTP is the protocol for web pages, ACP aims to be the protocol for **tasks / goals**
- A specification (blueprint), not an implementation

**Covers all interaction types:**

| Type | Description |
|------|-------------|
| A2A (Agent-to-Agent) | Collaboration between agents |
| A2T (Agent-to-Tool) | Using external tools |
| A2E (Agent-to-Environment) | Interacting with the world |
| A2H (Agent-to-Human) | Communicating results to users |

**Status:** Emerging standard backed by collaborative accord of AI leaders.

---

## Protocol 2 — A2A (Agent-to-Agent)

- Focused, lightweight protocol for **conversational collaboration** between agents
- Like XMPP (instant messaging protocol) but for agents
- Standardises the **messages** in a collaborative "group chat" or "handoff"

**What A2A standardises:**

| Verb | Meaning |
|------|---------|
| `REQUEST` | "Please do this task" |
| `INFORM` | "Here is the result / information" |
| `PROPOSE` | "Here is my suggestion / plan" |
| `ACCEPT` | "I agree with that proposal" |
| `REJECT` | "I disagree, here's why" |

**Use A2A for:**
- Group Chat / Debate patterns
- Router & Specialist handoffs
- Reflection (Critic's message to Writer)

---

## Protocol 3 — AG-UI (Agent-User Interaction)

- Standardises how agents render **rich, interactive UIs** dynamically
- Moves beyond "Text-In, Text-Out" chatbots to **Generative UI**
- The agent generates the interface **on the fly** based on context

```mermaid
flowchart LR
    AG[Agent] -->|"Standard AG-UI events"| FE[Frontend Client]
    FE -->|"User actions"| AG

    subgraph EXAMPLES ["AG-UI Examples"]
        C1["Adaptive Cards (Microsoft)\nStandard card components"]
        C2["Streaming React Components\n(Vercel AI SDK RSC)"]
        C3["Dynamic forms, charts,\napproval buttons"]
    end
```

**Critical for:** Human-in-the-Loop (HITL) patterns — presenting agent's proposed action in a rich UI for human approval.

---

## Protocol 4 — MCP (Model-Context Protocol)

> The most important protocol for enterprise architects. "The OpenAPI/Swagger for Agents."

- Lightweight, focused protocol for **Agent-to-Tool** communication
- Solves the most common enterprise problem: making existing APIs consumable by any agent

```mermaid
flowchart LR
    AG["🤖 Agent\n(Any LLM / Framework)"]
    MCP_S["🔌 MCP Server\n(Lightweight Adapter)"]
    DS[("💾 Data Source\nDB / API / File System")]

    AG <-->|"MCP Protocol\n(JSON-RPC)"| MCP_S
    MCP_S <-->|"Native calls"| DS
```

---

## Protocol Comparison & Decision Matrix

```mermaid
flowchart TD
    PROB{My primary problem is...}

    PROB -->|"Making existing APIs\nconsumable by agents"| USE_MCP["✅ Use MCP\n(Implement an MCP server)"]
    PROB -->|"Making agents\ntalk to each other"| USE_A2A["✅ Use A2A\n(Internal agent chat protocol)"]
    PROB -->|"Building a comprehensive\nnew agentic ecosystem"| USE_ACP["✅ Use ACP\n(Full specification)"]
    PROB -->|"Agent → Human\nUI interactions"| USE_AGUI["✅ Use AG-UI\n(Generative UI events)"]
```

> **Most enterprises start with MCP** — it solves the #1 practical problem: integration.

---

## MCP — Deep Dive Architecture

### The 4-Layer Topology

```mermaid
flowchart LR
    subgraph L1 ["Layer 1: Host / Client"]
        HOST["MCP Client\n(Claude Desktop, Cursor,\nYour Custom Agent)\n• Has the LLM brain\n• Initiates connections\n• Decides when to call tools"]
    end

    subgraph L2 ["Layer 2: Protocol (The Bridge)"]
        PROTO2["JSON-RPC 2.0\nTransports:\n• Stdio (local apps)\n• SSE / HTTP streaming\n  (remote servers)"]
    end

    subgraph L3 ["Layer 3: MCP Server (The Adapter)"]
        SERVER["MCP Server\n(What you build)\n• Wraps a specific data source\n• Deterministic code (no LLM)\n• Translates 'messy reality'\n  into standard MCP\n• Holds API keys securely"]
    end

    subgraph L4 ["Layer 4: Data Source (Reality)"]
        DS2["Actual Systems\nPostgreSQL · Slack API\nFile System · REST APIs\nMicroservices"]
    end

    HOST <-->|MCP Protocol| SERVER
    SERVER <--> DS2
    L2 --- HOST
    L2 --- SERVER
```

---

### The MCP Lifecycle — 4 Steps

```mermaid
sequenceDiagram
    participant AG as Agent (MCP Client)
    participant SRV as MCP Server
    participant DS as Data Source

    Note over AG,SRV: Step 1: Discovery ("The Knock")
    AG->>SRV: GET /.mcp/manifest.json
    SRV->>AG: manifest.json (tool definitions)

    Note over AG,SRV: Step 2: Capability Negotiation ("The Menu")
    AG->>AG: Read manifest\nIngest tool schemas into context

    Note over AG,SRV: Step 3: Tool Execution ("The Order")
    AG->>SRV: POST /call\n{"function":"get_product","params":{"id":"123"}}
    SRV->>DS: Translate & execute query
    DS->>SRV: Raw result

    Note over AG,SRV: Step 4: Response & Loop ("The Delivery")
    SRV->>AG: {"status":"success","data":{...}}
    AG->>AG: Feed result back to LLM\nStart next reasoning loop
```

---

### MCP Core Concepts

**Concept 1 — The /.mcp/ Well-Known Path**

```
GET https://api.my-eshop.com/.mcp/manifest.json
```

The agent doesn't need to be told where the manifest is — it always checks this standard path. Like `robots.txt` for agents.

**Concept 2 — The manifest.json Schema**

```json
{
  "name": "EShop MCP Server",
  "version": "1.0",
  "description": "Tools for interacting with the E-Shop",
  "functions": [
    {
      "name": "get_product_details",
      "description": "Retrieves detailed product information including price, stock, and description. Use this when the user asks about a specific product.",
      "parameters": {
        "product_id": {
          "type": "string",
          "description": "The unique product identifier, e.g. 'SKU-12345'"
        }
      }
    }
  ]
}
```

> **Critical:** The `description` field is the most important part. It's the LLM's "user manual" for the tool.

**Concept 3 — The call Endpoint**

```
POST /call
{
  "function": "get_product_details",
  "parameters": { "product_id": "SKU-12345" }
}

Response:
{
  "status": "success",
  "data": { "name": "Running Shoes", "price": 120, "stock": 45 }
}
```

**Key principle:** JSON in, JSON out. Simple, standard, stateless.

**Concept 4 — Security**

MCP leverages existing web standards:
- `Authorization: Bearer <token>` — OAuth 2.0 / API key
- HTTPS — all communication encrypted
- The **MCP Server holds the API keys** — the agent client never sees them directly

---

### MCP vs. OpenAPI — Why Both?

| | OpenAPI / Swagger | MCP manifest.json |
|--|------------------|-------------------|
| **Consumer** | Human developers | LLM agents |
| **Language** | Technical (developer-speak) | Semantic (LLM-speak) |
| **Size** | Comprehensive (5,000+ lines) | Curated (expose only what agents need) |
| **Purpose** | Full API documentation | Agent-friendly semantic menu |

> **Architect's job:** Use MCP as a "Facade" or "Adapter" in front of your existing OpenAPI systems. You do **not** replace OpenAPI with MCP — you layer MCP on top.

---

## MCP Design Patterns

### Pattern 1 — Adapter Pattern (For a Single Legacy System)

```mermaid
flowchart LR
    AG[Agent] -->|MCP| MCP_A["MCP Adapter Server\nWraps one system"] --> LEGACY["Legacy System\n(old API / DB)"]
```

Use when: You have **one** valuable non-agent-aware system to expose.

### Pattern 2 — Gateway Pattern (For Microservices)

```mermaid
flowchart LR
    AG[Agent] -->|MCP| GW["MCP Gateway Server\nsingle entry point"]
    GW --> MS1[Inventory Service]
    GW --> MS2[Orders Service]
    GW --> MS3[User Service]
    GW --> MS4[Payment Service]
```

Use when: You have **many microservices** and don't want 50 separate MCP servers.

**E-Shop Example:**
- `manifest.json` defines `get_product_details` and `get_order_status`
- Gateway routes `get_product_details` → Inventory Service
- Gateway routes `get_order_status` → Orders Service
- Partner agents have **one simple tool** while internal architecture remains decoupled

---

## The Full Protocol Stack

```mermaid
flowchart TB
    subgraph FRONT ["Frontend Layer"]
        AGUI2[AG-UI Protocol\nAgent ↔ Human UI]
    end

    subgraph COLLAB ["Collaboration Layer"]
        A2A2[A2A Protocol\nAgent ↔ Agent Chat]
    end

    subgraph INTEG ["Integration Layer"]
        MCP2[MCP Protocol\nAgent ↔ Tools / APIs]
    end

    subgraph DATA ["Data / Systems Layer"]
        DS3[Databases · APIs · Microservices]
    end

    FRONT --> COLLAB --> INTEG --> DATA
```

> A complete enterprise agentic architecture needs all three layers.

---

> ⬅️ [08 — Agentic RAG](./08_agentic_rag.md) | ➡️ [10 — Context Engineering](./10_context_engineering.md)
