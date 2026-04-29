# Scalable-agent-architecture

# Agent Platform Architecture (Minimal Scalable Setup)

This project implements a scalable multi-agent architecture designed to support 100+ domain-specific agents with clear ownership, discoverability, and extensibility.

The system follows a **Gateway → Router → Registry → Agent** pattern to convert natural language user input into structured agent execution.

---

## 🧠 High-Level Flow

```
User Input
   ↓
Gateway
   ↓
Router (LLM / Rules)
   ↓
Capability Identified (e.g., "create_invoice")
   ↓
Agent Registry Lookup
   ↓
Agent चयन (Selection)
   ↓
Agent Invocation
   ↓
Response to User
```

---

## 🧩 Components

### 1. Gateway

The Gateway is the single entry point into the system.

**Responsibilities:**

* Accept user input (natural language)
* Call the Router to determine intent
* Query the Agent Registry
* Invoke the selected agent
* Return the response

**Why it exists:**

* Centralized control
* Observability (who called what)
* Decoupling between agents

---

### 2. Router (LLM or Rule-Based)

The Router maps user input to a **capability**.

**Examples:**

* "Create invoice for ACME" → `create_invoice`
* "Get employee details" → `get_employee`

**Implementation options:**

* Rule-based (fast, deterministic)
* LLM-based (flexible, semantic)
* Hybrid (recommended)

**Output:**

```json
{
  "capability": "create_invoice",
  "confidence": 0.92
}
```

---

### 3. Agent Registry

A centralized service that stores metadata about all agents.

**Stores:**

* Agent name
* Endpoint (URL)
* Supported capabilities
* Capability descriptions (optional)

**Example:**

```json
{
  "name": "invoice-agent",
  "url": "http://invoice-service",
  "capabilities": ["create_invoice"],
  "descriptions": {
    "create_invoice": "Create invoice for a customer"
  }
}
```

**Endpoints:**

* `POST /register` → Register an agent
* `GET /find?capability=...` → Discover agents

---

### 4. Agents

Each agent is an independent service responsible for a specific domain.

**Examples:**

* invoice-agent
* hr-agent
* analytics-agent

**Responsibilities:**

* Execute domain-specific logic
* Optionally use MCP tools internally
* Return structured responses

**Standard Interface:**

```
POST /invoke
```

**Input:**

```json
{
  "input": { ... }
}
```

**Output:**

```json
{
  "status": "success",
  "data": { ... }
}
```

---

## 🔌 Design Principles

### ✅ 1. Decoupling

Agents do not call each other directly. All communication goes through the Gateway.

### ✅ 2. Capability-Based Routing

Routing is based on *what needs to be done*, not *which agent to call*.

### ✅ 3. Centralized Discovery

All agents are discoverable via the Registry. No hardcoding.

### ✅ 4. Extensibility

New agents can be added without modifying existing ones.

### ✅ 5. Separation of Concerns

* Gateway → orchestration
* Router → intent understanding
* Registry → discovery
* Agent → execution

---

## 🚫 Anti-Patterns (Avoid These)

❌ Direct agent-to-agent calls
❌ Hardcoding agent URLs
❌ Using MCP as a registry
❌ Letting LLM decide infrastructure routing blindly

---

## 🔄 Example Request

### User Input:

```
"Create an invoice for ACME for last month"
```

### Internal Flow:

1. Router → `create_invoice`
2. Registry → returns `invoice-agent`
3. Gateway → calls `invoice-agent /invoke`
4. Agent → processes request
5. Response returned to user

---

## 🚀 Getting Started (Minimal Setup)

### 1. Start Services

* Registry → `localhost:8001`
* Gateway → `localhost:8000`
* Agent(s) → `localhost:8002+`

### 2. Register an Agent

```bash
POST /register
```

```json
{
  "name": "invoice-agent",
  "url": "http://localhost:8002",
  "capabilities": ["create_invoice"]
}
```

### 3. Send Request

```bash
POST /chat
```

```json
{
  "message": "Create invoice for Vasanth"
}
```

---

## 🔮 Future Enhancements

* Multi-agent orchestration (chained workflows)
* Async execution via queues (Kafka, Redis)
* Auth & access control per agent
* Rate limiting & quotas
* LLM-based entity extraction
* Observability (tracing, logs, metrics)
* Agent versioning & deprecation

---

## 🧠 Mental Model

Think of this system as:

* **Gateway** → API Gateway
* **Router** → Intent classifier
* **Registry** → Service discovery
* **Agents** → Microservices
* **MCP** → Internal tool layer

---

## ✅ Summary

This architecture provides:

* Scalability to 100+ agents
* Clean team boundaries
* Flexible routing
* Easy extensibility

Without introducing unnecessary complexity.

---

## 📬 Questions / Contributions

Feel free to open issues or propose improvements as the system evolves.

