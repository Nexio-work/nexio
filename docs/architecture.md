# Nexio Architecture: The Agentic OS

**Version:** 1.0
**Status:** Design Phase
**Last Updated:** 2026-02-08

---

## Executive Summary

Nexio is a multi-tenant Agentic Operating System built on Cloudflare's edge computing platform and powered by OpenClaw's AI orchestration. The architecture follows three core principles:

1. **Edge-Native:** All computation at the network edge for global low latency
2. **Agentic Kernel:** Persistent, stateful AI agents that collaborate and make decisions
3. **Zero-Trust Multi-Tenancy:** Complete data isolation with Human-in-the-Loop (HITL) approval flows

This document details the system architecture, data model, monorepo structure, deployment strategy, security model, and performance considerations.

---

## 1. System Architecture

### 1.1 High-Level Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                          Cloudflare Edge                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │    Web App   │  │     API      │  │    Worker    │           │
│  │   (Pages)    │  │  (Workers)   │  │  (Scheduled) │           │
│  │              │  │              │  │              │           │
│  │  React +     │  │  Hono/      │  │  Heartbeat   │           │
│  │  Tailwind    │  │  FastAPI     │  │  Daemon      │           │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘           │
│         │                 │                 │                   │
│         └─────────────────┴─────────────────┘                   │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    BetterAuth (Auth)                     │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           │                                      │
│         ┌─────────────────┼─────────────────┐                   │
│         ▼                 ▼                 ▼                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │     D1       │  │     R2       │  │     KV       │         │
│  │ (Database)   │  │  (Storage)   │  │   (Cache)    │         │
│  │              │  │              │  │              │         │
│  │  Tenants,    │  │  Files,      │  │  Sessions,   │         │
│  │  Agents,     │  │  Assets,     │  │  Agent       │         │
│  │  Tasks,      │  │  Exports     │  │  Context     │         │
│  │  Memory      │  │              │  │              │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                Durable Objects (State)                  │    │
│  │  • Persistent Agent Sessions                            │    │
│  │  • Real-time Agent Communication                         │    │
│  │  • WebSocket Connections                                 │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
              ┌───────────────────────┐
              │    OpenClaw Gateway   │
              │   (AI Orchestration)  │
              └───────────────────────┘
                            │
                            ▼
              ┌───────────────────────┐
              │  AI Agents (LLMs)     │
              │  Jarvis, Fury, Growth │
              └───────────────────────┘
```

### 1.2 Core Components

#### 1.2.1 Web App (Cloudflare Pages)
- **Framework:** React 18+ with Next.js 14+ (App Router)
- **Styling:** Tailwind CSS with custom glassmorphism utilities
- **State Management:** Zustand for client state, React Query for server state
- **Real-time:** WebSocket connections via Durable Objects
- **Deployment:** Edge Functions on Cloudflare Pages

**Key Features:**
- Cmd+K command palette (primary interface)
- Glassmorphic dashboard with real-time agent activity
- HITL approval queue
- Multi-language support (Malagasy, French, English)
- Low-bandwidth mode toggle

#### 1.2.2 API (Cloudflare Workers)
- **Framework:** Hono (TypeScript) for HTTP, FastAPI (Python) for AI endpoints
- **Routing:** RESTful API with OpenAPI specification
- **Authentication:** BetterAuth middleware
- **Rate Limiting:** Cloudflare's built-in rate limiting

**API Endpoints:**
```typescript
// Authentication
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
GET    /api/auth/session

// Tenants
GET    /api/tenants/:id
POST   /api/tenants
PUT    /api/tenants/:id
DELETE /api/tenants/:id

// Users
GET    /api/users
GET    /api/users/:id
POST   /api/users
PUT    /api/users/:id
DELETE /api/users/:id

// Agents
GET    /api/agents
GET    /api/agents/:id
POST   /api/agents
PUT    /api/agents/:id
DELETE /api/agents/:id
POST   /api/agents/:id/wake
POST   /api/agents/:id/sleep

// Tasks
GET    /api/tasks
GET    /api/tasks/:id
POST   /api/tasks
PUT    /api/tasks/:id
DELETE /api/tasks/:id
POST   /api/tasks/:id/assign

// Approvals (HITL)
GET    /api/approvals
GET    /api/approvals/:id
POST   /api/approvals/:id/approve
POST   /api/approvals/:id/reject

// Memory Stack
GET    /api/memory/:agent_id
GET    /api/memory/:agent_id/:key
POST   /api/memory/:agent_id
PUT    /api/memory/:agent_id/:key
DELETE /api/memory/:agent_id/:key

// Agent Activity (Real-time)
WS     /api/agents/:agent_id/stream
```

#### 1.2.3 Worker (Scheduled Tasks)
- **Framework:** Cloudflare Workers with Cron Triggers
- **Schedule:** Every 15 minutes (staggered by 2 minutes per agent)
- **Purpose:** Heartbeat daemon that wakes agents to check for work

**Worker Tasks:**
```typescript
// Heartbeat Daemon (runs every 15 minutes)
async function heartbeatDaemon() {
  for (const agent of agents) {
    // Stagger wakeups: agent 1 at t=0, agent 2 at t=120s, etc.
    await delay(agent.staggerOffset * 1000);

    // Wake agent and check for work
    const hasWork = await agent.checkForWork();

    if (hasWork) {
      await agent.wake();
      await agent.processTasks();
      await agent.updateState();
      await agent.sleep();
    }
  }
}
```

#### 1.2.4 BetterAuth Integration
- **Provider:** Self-hosted BetterAuth instance
- **Features:**
  - OAuth2 (Google, Microsoft, GitHub)
  - Email/password authentication
  - Two-factor authentication (TOTP)
  - Session management
  - Role-based access control (RBAC)

**Authentication Flow:**
```
1. User initiates login (email/password or OAuth)
2. BetterAuth validates credentials
3. BetterAuth creates session with JWT token
4. JWT token includes: user_id, tenant_id, role, permissions
5. Token returned to client
6. Client includes token in Authorization header
7. API middleware validates token and extracts context
8. Row-level security enforced: WHERE tenant_id = extracted_tenant_id
```

#### 1.2.5 OpenClaw Integration
- **Protocol:** WebSocket for real-time agent communication, HTTP for control
- **Purpose:** AI agent execution, context management, memory operations

**Integration Layer (`packages/openclaw-bridge`):**
```typescript
// Agent Spawning
async function spawnAgent(config: AgentConfig): Promise<AgentHandle> {
  return await openclaw.spawn({
    name: config.name,
    persona: config.soulMd,     // SOUL.md content
    instructions: config.agentsMd, // AGENTS.md content
    tools: config.tools,
    context: {
      tenantId: config.tenantId,
      workingMd: config.workingMd,
      memoryMd: config.memoryMd
    }
  });
}

// Task Execution
async function executeTask(
  agentHandle: AgentHandle,
  task: Task
): Promise<TaskResult> {
  return await openclaw.execute(agentHandle, {
    prompt: task.description,
    payload: task.payload,
    callback: (update) => updateTaskProgress(task.id, update)
  });
}

// Memory Operations
async function updateMemory(
  agentHandle: AgentHandle,
  key: string,
  value: any,
  confidence: number
): Promise<void> {
  await openclaw.updateContext(agentHandle, {
    memory: { [key]: { value, confidence } }
  });
}
```

### 1.3 Data Flow

#### 1.3.1 Agent Wakeup and Task Processing
```
1. Cron Worker triggers Heartbeat Daemon (every 15 min)
2. Daemon queries D1: SELECT * FROM tasks WHERE status = 'pending' AND agent_id = ?
3. If tasks found:
   a. Spawn/wake agent via OpenClaw
   b. Pass task context (WORKING.md, relevant memory)
   c. Agent executes task (may collaborate with other agents)
   d. Agent updates task.status = 'in_progress' or 'completed'
   e. Agent writes daily log to D1 (agent_daily_logs)
   f. Agent updates long-term memory if learned something new
   g. If task requires human approval, create approval record
4. Agent goes to sleep (persistent state saved in Durable Object)
```

#### 1.3.2 Human-in-the-Loop (HITL) Approval Flow
```
1. Agent completes task, determines approval needed
2. Agent creates approval record with task.result
3. WebSocket pushes notification to user
4. User sees pending approval in HITL Command Center
5. User reviews, can:
   a. Approve → task.status = 'completed', result published
   b. Reject → task.status = 'failed', agent learns from feedback
   c. Edit → user modifies result, then approve, agent learns correction
6. Approval decision logged for audit trail
```

#### 1.3.3 Agent-to-Agent Collaboration (Party Mode)
```
1. Agent A (PM) completes task: "Draft project specification"
2. Agent A determines: "This should be reviewed by Agent B (Tech Lead)"
3. Agent A creates task for Agent B with reference to its own task
4. Agent B wakes up, reads Agent A's task result
5. Agent B reviews, provides feedback (Refute/Praise loop)
6. If conflict detected → escalate to human (HITL)
7. If no conflict → Agent B approves, task handoff complete
8. Both agents update their memory about the collaboration
```

---

## 2. Data Model

### 2.1 Database Schema (D1 - SQLite)

#### 2.1.1 Tenants Table
```sql
CREATE TABLE tenants (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  slug TEXT NOT NULL UNIQUE,
  plan TEXT NOT NULL DEFAULT 'free', -- free, pro, enterprise
  settings JSONB NOT NULL DEFAULT '{}', -- white-label config, language, timezone
  limits JSONB NOT NULL DEFAULT '{}', -- storage_limit, agents_limit, users_limit
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

CREATE INDEX idx_tenants_plan ON tenants(plan);
CREATE INDEX idx_tenants_slug ON tenants(slug);
```

**Settings Schema:**
```json
{
  "branding": {
    "logo": "https://...",
    "primaryColor": "#00D4FF",
    "domain": "agency-os.com"
  },
  "localization": {
    "defaultLanguage": "mg",
    "supportedLanguages": ["mg", "fr", "en"],
    "timezone": "Indian/Antananarivo"
  },
  "features": {
    "lowBandwidthMode": true,
    "biometricIntegration": false,
    "formationModules": true
  }
}
```

#### 2.1.2 Users Table
```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  name TEXT NOT NULL,
  role TEXT NOT NULL, -- owner, admin, staff, external
  permissions JSONB NOT NULL DEFAULT '[]', -- granular permissions
  preferences JSONB NOT NULL DEFAULT '{}', -- UI preferences, notification settings
  last_active_at INTEGER,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

CREATE UNIQUE INDEX idx_users_tenant_email ON users(tenant_id, email);
CREATE INDEX idx_users_tenant ON users(tenant_id);
CREATE INDEX idx_users_role ON users(role);
```

**Permissions Schema:**
```json
{
  "agents": ["read", "create", "edit", "delete"],
  "tasks": ["read", "create", "edit", "delete", "approve"],
  "approvals": ["read", "approve", "reject"],
  "users": ["read", "create", "edit", "delete"],
  "tenants": ["read", "edit"]
}
```

#### 2.1.3 Sessions Table (BetterAuth)
```sql
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  token TEXT NOT NULL UNIQUE,
  expires_at INTEGER NOT NULL,
  created_at INTEGER NOT NULL
);

CREATE INDEX idx_sessions_user ON sessions(user_id);
CREATE INDEX idx_sessions_token ON sessions(token);
CREATE INDEX idx_sessions_expires ON sessions(expires_at);
```

#### 2.1.4 Agents Table
```sql
CREATE TABLE agents (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  name TEXT NOT NULL, -- Jarvis, Fury, Growth, Operations
  persona TEXT NOT NULL, -- SOUL.md content
  instructions TEXT NOT NULL, -- AGENTS.md content
  status TEXT NOT NULL DEFAULT 'idle', -- idle, active, sleep, error
  last_heartbeat INTEGER, -- Unix timestamp
  working_md TEXT, -- Current task context (WORKING.md content)
  memory_md TEXT, -- Long-term memory (MEMORY.md content)
  config JSONB NOT NULL DEFAULT '{}', -- Agent-specific settings
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

CREATE INDEX idx_agents_tenant ON agents(tenant_id);
CREATE INDEX idx_agents_status ON agents(status);
CREATE INDEX idx_agents_heartbeat ON agents(last_heartbeat);
CREATE INDEX idx_agents_name_tenant ON agents(tenant_id, name);
```

**Config Schema:**
```json
{
  "heartbeatInterval": 900, -- 15 minutes in seconds
  "staggerOffset": 0, -- 0-120 seconds to stagger wakeups
  "collaborationMode": true, -- Can collaborate with other agents
  "requiresApproval": true, -- Tasks require HITL approval
  "tools": ["search", "write", "read", "browser"],
  "language": "auto" -- Detect from user or task
}
```

#### 2.1.5 Tasks Table
```sql
CREATE TABLE tasks (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  assigned_agent_id TEXT REFERENCES agents(id) ON DELETE SET NULL,
  created_by_user_id TEXT REFERENCES users(id) ON DELETE SET NULL,
  status TEXT NOT NULL DEFAULT 'pending', -- pending, in_progress, completed, failed
  priority TEXT NOT NULL DEFAULT 'normal', -- low, normal, high, urgent
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  payload JSONB, -- Task-specific data
  result JSONB, -- Agent's output
  error TEXT, -- Error message if failed
  requires_approval BOOLEAN NOT NULL DEFAULT false,
  parent_task_id TEXT REFERENCES tasks(id) ON DELETE SET NULL, -- For task chains
  metadata JSONB NOT NULL DEFAULT '{}', -- Tags, labels, etc.
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  completed_at INTEGER
);

CREATE INDEX idx_tasks_tenant ON tasks(tenant_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_agent ON tasks(assigned_agent_id);
CREATE INDEX idx_tasks_priority ON tasks(priority);
CREATE INDEX idx_tasks_parent ON tasks(parent_task_id);
CREATE INDEX idx_tasks_approval ON tasks(requires_approval) WHERE requires_approval = true;
```

#### 2.1.6 Approvals Table (HITL)
```sql
CREATE TABLE approvals (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  task_id TEXT NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  user_id TEXT REFERENCES users(id) ON DELETE SET NULL, -- Assigned reviewer
  status TEXT NOT NULL DEFAULT 'pending', -- pending, approved, rejected
  decision_reason TEXT, -- Why approved/rejected
  edited_result JSONB, -- If user modified before approving
  created_at INTEGER NOT NULL,
  decided_at INTEGER,
  decided_by_user_id TEXT REFERENCES users(id) ON DELETE SET NULL
);

CREATE INDEX idx_approvals_tenant ON approvals(tenant_id);
CREATE INDEX idx_approvals_status ON approvals(status);
CREATE INDEX idx_approvals_task ON approvals(task_id);
CREATE INDEX idx_approvals_user ON approvals(user_id);
```

#### 2.1.7 Agent Daily Logs Table
```sql
CREATE TABLE agent_daily_logs (
  id TEXT PRIMARY KEY,
  agent_id TEXT NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  date TEXT NOT NULL, -- YYYY-MM-DD
  content TEXT NOT NULL, -- JSONL format: one JSON object per line
  created_at INTEGER NOT NULL
);

CREATE UNIQUE INDEX idx_logs_agent_date ON agent_daily_logs(agent_id, date);
CREATE INDEX idx_logs_tenant_date ON agent_daily_logs(tenant_id, date);
```

**Daily Log Format (JSONL):**
```json
{"timestamp":1739020800,"type":"thought","content":"Analyzing task requirements..."}
{"timestamp":1739020860,"type":"action","content":"Executed search query for..."}
{"timestamp":1739020920,"type":"collaboration","content":"Consulted with Jarvis about..."}
{"timestamp":1739020980,"type":"completion","content":"Task completed successfully"}
```

#### 2.1.8 Agent Memory Table
```sql
CREATE TABLE agent_memory (
  id TEXT PRIMARY KEY,
  agent_id TEXT NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  key TEXT NOT NULL,
  value JSONB,
  confidence REAL NOT NULL DEFAULT 0.5, -- 0.0-1.0, how certain the agent is
  access_count INTEGER NOT NULL DEFAULT 0,
  last_accessed_at INTEGER,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

CREATE UNIQUE INDEX idx_memory_agent_key ON agent_memory(agent_id, key);
CREATE INDEX idx_memory_tenant ON agent_memory(tenant_id);
CREATE INDEX idx_memory_confidence ON agent_memory(confidence);
```

#### 2.1.9 Agent Collaborations Table (Party Mode)
```sql
CREATE TABLE agent_collaborations (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  initiating_agent_id TEXT NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
  reviewing_agent_id TEXT NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
  task_id TEXT NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  collaboration_type TEXT NOT NULL, -- 'refute', 'praise', 'handoff', 'consult'
  initiating_message TEXT NOT NULL,
  reviewing_message TEXT,
  status TEXT NOT NULL DEFAULT 'pending', -- pending, approved, rejected, escalated
  escalated_to_user_id TEXT REFERENCES users(id) ON DELETE SET NULL,
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  completed_at INTEGER
);

CREATE INDEX idx_collaborations_tenant ON agent_collaborations(tenant_id);
CREATE INDEX idx_collaborations_initiating ON agent_collaborations(initiating_agent_id);
CREATE INDEX idx_collaborations_reviewing ON agent_collaborations(reviewing_agent_id);
CREATE INDEX idx_collaborations_task ON agent_collaborations(task_id);
CREATE INDEX idx_collaborations_status ON agent_collaborations(status);
```

#### 2.1.10 Audit Logs Table
```sql
CREATE TABLE audit_logs (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id TEXT REFERENCES users(id) ON DELETE SET NULL,
  agent_id TEXT REFERENCES agents(id) ON DELETE SET NULL,
  action TEXT NOT NULL, -- login, logout, task_create, task_update, approval_approve, etc.
  entity_type TEXT NOT NULL, -- user, agent, task, approval, tenant
  entity_id TEXT NOT NULL,
  changes JSONB, -- Before/after values
  metadata JSONB,
  ip_address TEXT,
  user_agent TEXT,
  created_at INTEGER NOT NULL
);

CREATE INDEX idx_audit_tenant ON audit_logs(tenant_id);
CREATE INDEX idx_audit_user ON audit_logs(user_id);
CREATE INDEX idx_audit_agent ON audit_logs(agent_id);
CREATE INDEX idx_audit_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_created ON audit_logs(created_at);
```

### 2.2 Durable Objects Schema

#### 2.2.1 Agent Session Object
```typescript
interface AgentSessionState {
  agentId: string;
  tenantId: string;
  status: 'idle' | 'active' | 'sleep' | 'error';
  currentTaskId: string | null;
  workingMd: string; // Current context
  lastHeartbeat: number;
  webSocketConnections: Set<WebSocket>; // Connected clients
  collaborationQueue: CollaborationRequest[];
}

class AgentSession extends DurableObject {
  async wake() {
    this.state.status = 'active';
    this.state.lastHeartbeat = Date.now();
    await broadcastToClients({ type: 'agent_wake', agentId: this.state.agentId });
  }

  async sleep() {
    this.state.status = 'sleep';
    await broadcastToClients({ type: 'agent_sleep', agentId: this.state.agentId });
  }

  async updateWorkingMd(content: string) {
    this.state.workingMd = content;
    await broadcastToClients({ type: 'working_md_update', content });
  }
}
```

---

## 3. Monorepo Structure

```
nexio/
├── apps/
│   ├── web/                    # Frontend (Next.js + Tailwind)
│   │   ├── src/
│   │   │   ├── app/           # App Router pages
│   │   │   │   ├── (auth)/
│   │   │   │   │   ├── login/page.tsx
│   │   │   │   │   └── register/page.tsx
│   │   │   │   ├── (dashboard)/
│   │   │   │   │   ├── page.tsx            # Main dashboard
│   │   │   │   │   ├── agents/
│   │   │   │   │   │   ├── page.tsx        # Agent list
│   │   │   │   │   │   └── [id]/page.tsx   # Agent detail
│   │   │   │   │   ├── tasks/
│   │   │   │   │   │   ├── page.tsx        # Task list
│   │   │   │   │   │   └── [id]/page.tsx   # Task detail
│   │   │   │   │   ├── approvals/
│   │   │   │   │   │   └── page.tsx        # HITL queue
│   │   │   │   │   └── settings/
│   │   │   │   │       └── page.tsx
│   │   │   │   ├── layout.tsx
│   │   │   │   └── page.tsx                # Landing page
│   │   │   ├── components/
│   │   │   │   ├── ui/                     # Reusable UI components
│   │   │   │   ├── agents/                 # Agent-specific components
│   │   │   │   ├── tasks/                  # Task-specific components
│   │   │   │   ├── approvals/              # HITL components
│   │   │   │   └── CommandPalette.tsx      # Cmd+K main interface
│   │   │   ├── hooks/
│   │   │   │   ├── useAgents.ts
│   │   │   │   ├── useTasks.ts
│   │   │   │   ├── useApprovals.ts
│   │   │   │   └── useWebSocket.ts
│   │   │   ├── stores/                    # Zustand stores
│   │   │   │   ├── agentStore.ts
│   │   │   │   ├── taskStore.ts
│   │   │   │   └── uiStore.ts
│   │   │   └── lib/
│   │   │       ├── api.ts                  # API client
│   │   │       └── websocket.ts            # WebSocket client
│   │   ├── public/
│   │   ├── package.json
│   │   ├── tailwind.config.js
│   │   ├── tsconfig.json
│   │   └── next.config.js
│   │
│   ├── api/                    # Backend API (Hono + Cloudflare Workers)
│   │   ├── src/
│   │   │   ├── index.ts        # Worker entry point
│   │   │   ├── routes/
│   │   │   │   ├── auth.ts
│   │   │   │   ├── tenants.ts
│   │   │   │   ├── users.ts
│   │   │   │   ├── agents.ts
│   │   │   │   ├── tasks.ts
│   │   │   │   ├── approvals.ts
│   │   │   │   ├── memory.ts
│   │   │   │   └── websocket.ts
│   │   │   ├── middleware/
│   │   │   │   ├── auth.ts              # BetterAuth middleware
│   │   │   │   ├── tenant.ts            # Tenant scoping
│   │   │   │   └── rbac.ts              # Role-based access control
│   │   │   ├── services/
│   │   │   │   ├── agentService.ts
│   │   │   │   ├── taskService.ts
│   │   │   │   ├── approvalService.ts
│   │   │   │   └── memoryService.ts
│   │   │   ├── db/
│   │   │   │   ├── schema.ts            # Drizzle schema
│   │   │   │   └── queries.ts           # Query helpers
│   │   │   └── lib/
│   │   │       ├── openclaw.ts          # OpenClaw bridge
│   │   │       └── logger.ts
│   │   ├── wrangler.toml
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── worker/                 # Scheduled Worker (Heartbeat Daemon)
│       ├── src/
│       │   ├── index.ts        # Worker entry point
│       │   ├── heartbeat.ts    # Heartbeat logic
│       │   ├── agentManager.ts # Agent lifecycle management
│       │   └── db/
│       │       └── queries.ts
│       ├── wrangler.toml
│       ├── package.json
│       └── tsconfig.json
│
├── packages/
│   ├── ui/                     # Shared UI components
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Card.tsx
│   │   │   │   ├── Input.tsx
│   │   │   │   ├── Modal.tsx
│   │   │   │   ├── GlassCard.tsx         # Glassmorphic card
│   │   │   │   ├── AgentPulse.tsx        # Animated agent indicator
│   │   │   │   └── CommandPalette.tsx
│   │   │   ├── hooks/
│   │   │   └── styles/
│   │   │       └── globals.css
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── db/                     # Database schema and queries
│   │   ├── src/
│   │   │   ├── schema/
│   │   │   │   ├── tenants.ts
│   │   │   │   ├── users.ts
│   │   │   │   ├── agents.ts
│   │   │   │   ├── tasks.ts
│   │   │   │   ├── approvals.ts
│   │   │   │   ├── memory.ts
│   │   │   │   └── index.ts
│   │   │   ├── migrations/
│   │   │   │   ├── 0001_initial.sql
│   │   │   │   └── ...
│   │   │   ├── queries/
│   │   │   │   ├── tenants.ts
│   │   │   │   ├── users.ts
│   │   │   │   ├── agents.ts
│   │   │   │   └── ...
│   │   │   └── index.ts
│   │   ├── drizzle.config.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── auth/                   # Authentication utilities
│   │   ├── src/
│   │   │   ├── betterAuth.ts   # BetterAuth client
│   │   │   ├── permissions.ts  # RBAC helpers
│   │   │   ├── session.ts      # Session management
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── shared/                 # Shared utilities and types
│   │   ├── src/
│   │   │   ├── types/
│   │   │   │   ├── tenant.ts
│   │   │   │   ├── user.ts
│   │   │   │   ├── agent.ts
│   │   │   │   ├── task.ts
│   │   │   │   ├── approval.ts
│   │   │   │   └── index.ts
│   │   │   ├── utils/
│   │   │   │   ├── date.ts
│   │   │   │   ├── validation.ts
│   │   │   │   └── format.ts
│   │   │   ├── constants/
│   │   │   │   ├── roles.ts
│   │   │   │   ├── plans.ts
│   │   │   │   └── index.ts
│   │   │   └── i18n/
│   │   │       ├── locales/
│   │   │       │   ├── mg.json    # Malagasy
│   │   │       │   ├── fr.json    # French
│   │   │       │   └── en.json    # English
│   │   │       └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── openclaw-bridge/        # OpenClaw integration layer
│       ├── src/
│       │   ├── client.ts       # OpenClaw API client
│       │   ├── agent.ts        # Agent spawning and management
│       │   ├── task.ts         # Task execution
│       │   ├── memory.ts       # Memory operations
│       │   └── index.ts
│       ├── package.json
│       └── tsconfig.json
│
├── docs/
│   ├── prd.md
│   ├── product-blueprint.md
│   ├── os-philosophy.md
│   └── architecture.md         # This file
│
├── package.json                # Root package.json
├── turbo.json                  # Turborepo config
├── tsconfig.json              # Root tsconfig
└── .github/
    └── workflows/
        ├── deploy.yml
        └── test.yml
```

### 3.1 Package Dependencies

```json
{
  "name": "nexio",
  "version": "1.0.0",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "db:push": "turbo run db:push",
    "db:migrate": "turbo run db:migrate"
  },
  "devDependencies": {
    "turbo": "^1.11.0",
    "typescript": "^5.3.0"
  }
}
```

---

## 4. Deployment Strategy

### 4.1 Development Environment

**VPS Sandbox (Optional):**
- **Purpose:** Local development, testing, staging
- **Provider:** Any VPS (DigitalOcean, Linode, Hetzner)
- **Stack:** Docker Compose with local services
- **Services:**
  - PostgreSQL (local alternative to D1)
  - MinIO (local S3 alternative to R2)
  - Redis (local cache alternative to KV)
  - BetterAuth (self-hosted)
  - OpenClaw Gateway (local or cloud)

**Workflow:**
```bash
# 1. Clone repo and install dependencies
git clone https://github.com/fataplus/nexio.git
cd nexio
pnpm install

# 2. Start dev environment
pnpm dev

# 3. Run database migrations
pnpm db:migrate

# 4. Start all services (web, api, worker)
turbo run dev --parallel
```

### 4.2 Production Environment

**Cloudflare Workers & Pages:**

#### 4.2.1 Web App (Cloudflare Pages)
```toml
# apps/web/wrangler.toml
name = "nexio-web"
compatibility_date = "2024-01-01"

[env.production]
routes = [
  { pattern = "nexio.app/*", zone_name = "nexio.app" }
]

[env.staging]
routes = [
  { pattern = "staging.nexio.app/*", zone_name = "nexio.app" }
]
```

**Deployment Command:**
```bash
# Build web app
cd apps/web
pnpm build

# Deploy to Cloudflare Pages
wrangler pages deploy ./out --project-name=nexio-web
```

#### 4.2.2 API Worker (Cloudflare Workers)
```toml
# apps/api/wrangler.toml
name = "nexio-api"
main = "src/index.ts"
compatibility_date = "2024-01-01"

# D1 Database binding
[[d1_databases]]
binding = "DB"
database_name = "nexio-db"
database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# R2 Storage binding
[[r2_buckets]]
binding = "R2"
bucket_name = "nexio-storage"

# KV Namespace binding
[[kv_namespaces]]
binding = "KV"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# Durable Objects binding
[[durable_objects.bindings]]
name = "AGENT_SESSIONS"
class_name = "AgentSession"

# Environment variables
[vars]
ENVIRONMENT = "production"
OPENCLAW_API_URL = "https://api.openclaw.io"
BETTERAUTH_URL = "https://auth.nexio.app"

# Secrets (set via: wrangler secret put <name>)
# wrangler secret put JWT_SECRET
# wrangler secret put OPENCLAW_API_KEY
# wrangler secret put BETTERAUTH_SECRET
```

**Deployment Command:**
```bash
# Build API
cd apps/api
pnpm build

# Deploy to Cloudflare Workers
wrangler deploy
```

#### 4.2.3 Scheduled Worker (Heartbeat Daemon)
```toml
# apps/worker/wrangler.toml
name = "nexio-worker"
main = "src/index.ts"
compatibility_date = "2024-01-01"

# D1 Database binding
[[d1_databases]]
binding = "DB"
database_name = "nexio-db"
database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# Cron trigger (every 15 minutes)
[triggers]
crons = ["*/15 * * * *"]
```

**Deployment Command:**
```bash
# Build worker
cd apps/worker
pnpm build

# Deploy to Cloudflare Workers
wrangler deploy
```

#### 4.2.4 BetterAuth (Self-Hosted)
**Deployment Options:**
1. **Cloudflare Workers** (recommended for edge consistency)
2. **VPS** (if self-hosting required)

**Example (Cloudflare Workers):**
```toml
# packages/auth/wrangler.toml
name = "nexio-auth"
main = "src/betterAuth.ts"
compatibility_date = "2024-01-01"

[vars]
DATABASE_URL = "d1://nexio-db"
```

### 4.3 CI/CD Pipeline

**GitHub Actions Workflow:**
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - name: Install dependencies
        run: pnpm install
      - name: Run tests
        run: pnpm test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
      - name: Install dependencies
        run: pnpm install
      - name: Build
        run: pnpm build
      - name: Deploy Web
        run: cd apps/web && wrangler pages deploy ./out --project-name=nexio-web
      - name: Deploy API
        run: cd apps/api && wrangler deploy
      - name: Deploy Worker
        run: cd apps/worker && wrangler deploy
```

### 4.4 Database Migrations

**D1 Migration Strategy:**
```bash
# Create migration
pnpm db:generate --name=add_agent_collaborations

# Run migration (development)
pnpm db:push

# Run migration (production)
pnpm db:migrate:prod

# Rollback
pnpm db:rollback
```

---

## 5. Security Model

### 5.1 Multi-Tenant Isolation

**Row-Level Security:**
- Every database query includes `WHERE tenant_id = ?`
- Tenant ID extracted from JWT token
- Implemented at middleware level
- Automatic enforcement, no developer opt-out

```typescript
// Example: Middleware that enforces tenant isolation
async function tenantMiddleware(c: Context, next: Next) {
  const token = c.req.header('Authorization')?.replace('Bearer ', '');
  const decoded = verifyJWT(token);

  if (!decoded.tenantId) {
    throw new UnauthorizedError('No tenant ID in token');
  }

  // Inject tenant ID into context
  c.set('tenantId', decoded.tenantId);

  // All subsequent queries automatically include WHERE tenant_id = ?
  await next();
}
```

**Tenant Scoping in Queries:**
```typescript
// Bad: No tenant scoping (security risk!)
const agents = await db.query.agents.findMany();

// Good: Tenant-scoped query
const agents = await db.query.agents.findMany({
  where: eq(agents.tenantId, tenantId)
});
```

### 5.2 Role-Based Access Control (RBAC)

**Role Hierarchy:**
```
owner
  └─ Can do everything within tenant
  └─ Manage billing, users, settings

admin
  └─ Can manage agents, tasks, approvals
  └─ Cannot delete tenant or manage billing

staff
  └─ Can view and create tasks
  └─ Can approve/reject in assigned queue

external
  └─ Limited access to specific projects/tasks
  └─ Cannot view other tenants or sensitive data
```

**Permission Matrix:**
| Action | Owner | Admin | Staff | External |
|--------|-------|-------|-------|----------|
| View agents | ✓ | ✓ | ✓ | ✗ |
| Create agents | ✓ | ✓ | ✗ | ✗ |
| Delete agents | ✓ | ✗ | ✗ | ✗ |
| View all tasks | ✓ | ✓ | ✓ | assigned |
| Create tasks | ✓ | ✓ | ✓ | ✗ |
| Approve tasks | ✓ | ✓ | ✓ | ✗ |
| Manage users | ✓ | ✓ | ✗ | ✗ |
| Manage billing | ✓ | ✗ | ✗ | ✗ |

**Implementation:**
```typescript
// packages/auth/src/permissions.ts
export const PERMISSIONS = {
  agents: {
    read: 'agents:read',
    create: 'agents:create',
    edit: 'agents:edit',
    delete: 'agents:delete',
  },
  tasks: {
    read: 'tasks:read',
    create: 'tasks:create',
    edit: 'tasks:edit',
    delete: 'tasks:delete',
    approve: 'tasks:approve',
  },
  // ... more permissions
};

export function hasPermission(
  user: User,
  permission: string
): boolean {
  return user.permissions.includes(permission);
}
```

### 5.3 Human-in-the-Loop (HITL) Approval Flows

**Approval Gates:**
1. **Agent-Generated Content:** Any output that affects business operations
2. **High-Impact Actions:** Sending emails, publishing content, changing settings
3. **External Communications:** Messages to clients, partners, or public platforms
4. **Financial Operations:** Billing, payments, budget changes

**Approval Workflow:**
```
1. Agent completes task, determines approval needed
2. Agent creates approval record with task.result
3. Notification sent to assigned reviewer (or role-based queue)
4. Reviewer has options:
   a. Approve → task.completed, result published
   b. Reject → task.failed, agent learns from feedback
   c. Edit → modify result, then approve (agent learns correction)
5. Approval logged with timestamp, reviewer, decision reason
```

**Audit Trail:**
- All approvals logged in `audit_logs` table
- Full chain of custody: who created task, who approved, who edited
- Replay capability: see exactly what was approved at any point

### 5.4 Authentication & Authorization

**JWT Token Structure:**
```json
{
  "sub": "user_id",
  "tenant_id": "tenant_id",
  "email": "user@example.com",
  "role": "admin",
  "permissions": ["agents:read", "tasks:create", "tasks:approve"],
  "iat": 1739020800,
  "exp": 1739107200
}
```

**Token Validation Flow:**
```typescript
// apps/api/src/middleware/auth.ts
export async function authMiddleware(c: Context, next: Next) {
  const token = c.req.header('Authorization')?.replace('Bearer ', '');

  if (!token) {
    throw new UnauthorizedError('No token provided');
  }

  try {
    // Verify JWT signature and expiration
    const decoded = verifyJWT(token, JWT_SECRET);

    // Verify session exists and is valid
    const session = await db.query.sessions.findFirst({
      where: eq(sessions.token, token)
    });

    if (!session || session.expiresAt < Date.now()) {
      throw new UnauthorizedError('Invalid or expired session');
    }

    // Verify user exists and belongs to tenant
    const user = await db.query.users.findFirst({
      where: and(
        eq(users.id, decoded.sub),
        eq(users.tenantId, decoded.tenant_id)
      )
    });

    if (!user) {
      throw new UnauthorizedError('User not found');
    }

    // Inject user and tenant into context
    c.set('user', user);
    c.set('tenantId', decoded.tenant_id);

    await next();
  } catch (error) {
    throw new UnauthorizedError('Authentication failed');
  }
}
```

### 5.5 Data Encryption

**In Transit:**
- All API communication over TLS 1.3
- WebSocket connections encrypted (WSS)
- Certificate managed by Cloudflare

**At Rest:**
- D1: Encrypted by default (Cloudflare managed)
- R2: Server-side encryption (AES-256)
- KV: Encrypted at rest

**Sensitive Data:**
- Passwords: Hashed with bcrypt (salted)
- API keys: Encrypted at rest using AES-256-GCM
- Personal data: PII stored securely, GDPR compliant

### 5.6 Rate Limiting

**Per-Tenant Limits:**
```typescript
// Cloudflare Workers rate limiting
export async function rateLimitMiddleware(c: Context, next: Next) {
  const tenantId = c.get('tenantId');
  const key = `rate_limit:${tenantId}:${c.req.path}`;

  const count = await KV.get(key);

  if (count && parseInt(count) > 1000) {
    throw new TooManyRequestsError('Rate limit exceeded');
  }

  await KV.put(key, (parseInt(count || '0') + 1).toString(), {
    expirationTtl: 60 // 1 minute window
  });

  await next();
}
```

**Limits by Plan:**
| Plan | API Requests/Min | Agents | Users | Storage |
|------|------------------|--------|-------|---------|
| Free | 100 | 3 | 5 | 10 GB |
| Pro | 1,000 | 10 | 25 | 100 GB |
| Enterprise | Unlimited | Unlimited | Unlimited | Unlimited |

### 5.7 Security Best Practices

**1. Input Validation:**
- All user input validated using Zod schemas
- SQL injection prevention via parameterized queries
- XSS prevention via React's built-in escaping

**2. CORS Configuration:**
```typescript
// apps/api/src/index.ts
app.use('/api/*', cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || ['https://nexio.app'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

**3. API Versioning:**
- All API routes versioned: `/api/v1/...`
- Backward compatibility maintained for at least 2 versions

**4. Error Handling:**
- Generic error messages to clients
- Detailed errors logged to Sentry
- No stack traces exposed to users

**5. Dependency Management:**
- Regular security audits via `npm audit`
- Dependabot for automatic PRs
- Lock files committed to Git

---

## 6. Performance & Scalability

### 6.1 Caching Strategy

#### 6.1.1 KV Cache (Read-Through)
```typescript
export async function getCachedOrFetch<T>(
  key: string,
  fetchFn: () => Promise<T>,
  ttl: number = 300 // 5 minutes
): Promise<T> {
  // Try KV cache first
  const cached = await KV.get(key);

  if (cached) {
    return JSON.parse(cached);
  }

  // Cache miss, fetch from DB
  const data = await fetchFn();

  // Store in KV cache
  await KV.put(key, JSON.stringify(data), { expirationTtl: ttl });

  return data;
}

// Usage
const agents = await getCachedOrFetch(
  `agents:${tenantId}`,
  () => db.query.agents.findMany({ where: eq(agents.tenantId, tenantId) })
);
```

#### 6.1.2 Durable Objects for State
- Persistent agent state (no cold starts)
- WebSocket connections (real-time updates)
- Agent-to-agent communication (low latency)

#### 6.1.3 CDN Caching (Static Assets)
- All static assets served from Cloudflare CDN
- Cache headers: `Cache-Control: public, max-age=31536000, immutable`
- Asset versioning for cache busting: `logo-v1.2.3.png`

### 6.2 Database Optimization

#### 6.2.1 Query Optimization
```sql
-- Bad: N+1 query problem
SELECT * FROM agents WHERE tenant_id = ?
-- Then for each agent:
SELECT * FROM tasks WHERE agent_id = ?

-- Good: Single query with JOIN
SELECT
  a.*,
  COUNT(t.id) as task_count,
  COUNT(CASE WHEN t.status = 'pending' THEN 1 END) as pending_tasks
FROM agents a
LEFT JOIN tasks t ON a.id = t.agent_id
WHERE a.tenant_id = ?
GROUP BY a.id
```

#### 6.2.2 Index Strategy
- All `tenant_id` columns indexed
- Foreign keys indexed
- Composite indexes for common query patterns
- Regular index usage monitoring

#### 6.2.3 Connection Pooling
- D1: Managed connection pooling by Cloudflare
- No connection overhead (serverless)

### 6.3 Worker Optimization

#### 6.3.1 Cold Start Mitigation
- Keep critical workers warm via scheduled pings
- Pre-warm worker during deployment
- Cache initialization code

#### 6.3.2 Async Task Processing
- Long-running tasks executed asynchronously
- Task queue in D1 (status: pending, in_progress, completed)
- Worker polls for tasks every 15 minutes

#### 6.3.3 Worker Segmentation
- **API Worker:** Handles HTTP requests (low latency)
- **Worker Daemon:** Handles background tasks (no latency requirement)
- **Durable Objects:** Handles real-time state (persistent)

### 6.4 Scalability Targets

**Per Tenant (Single $5 Cloudflare Account):**
| Metric | Target | Notes |
|--------|--------|-------|
| Users | 100 | Concurrent |
| Agents | 10 | Active |
| Tasks/Day | 1,000 | Agent-executed |
| Storage | 100 GB | R2 |
| API Requests/Min | 1,000 | Per tenant |

**Platform-Wide:**
| Metric | Target | Notes |
|--------|--------|-------|
| Tenants | 1,000 | On single account |
| Total Users | 100,000 | Across all tenants |
| Total Agents | 10,000 | Across all tenants |
| Uptime | 99.9% | Production SLA |
| API Latency (p95) | < 100ms | Edge-optimized |

### 6.5 Monitoring & Observability

#### 6.5.1 Metrics
- **Business Metrics:** Active tenants, agent tasks completed, approval rates
- **Technical Metrics:** Worker cold starts, API response times, error rates
- **User Metrics:** Session duration, feature usage, conversion rates

#### 6.5.2 Logging
```typescript
// Structured logging
logger.info('Agent task completed', {
  tenantId,
  agentId,
  taskId,
  duration: Date.now() - startTime,
  status: 'success'
});

logger.error('Agent task failed', {
  tenantId,
  agentId,
  taskId,
  error: error.message,
  stack: error.stack
});
```

#### 6.5.3 Error Tracking
- **Sentry:** Error aggregation and alerting
- **Cloudflare Analytics:** Request metrics and edge performance
- **Custom Dashboard:** Grafana (optional) for deep monitoring

#### 6.5.4 Alerts
- **Critical:** Worker down, database unreachable, auth failure
- **Warning:** High error rate, slow response times, approaching limits
- **Info:** Deployment completed, migration applied

---

## 7. Technology Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | React 18+, Next.js 14 | UI framework |
|  | Tailwind CSS | Styling |
|  | Zustand | Client state |
|  | React Query | Server state |
| **Backend** | Cloudflare Workers | Serverless compute |
|  | Hono | HTTP framework |
|  | Durable Objects | Persistent state |
| **Database** | D1 (SQLite) | Primary database |
|  | KV | Key-value cache |
|  | R2 | Object storage |
| **Auth** | BetterAuth | Authentication |
|  | JWT | Session tokens |
| **AI** | OpenClaw | Agent orchestration |
| **Deployment** | Cloudflare Pages | Static hosting |
|  | Cloudflare Workers | Serverless deployment |
|  | GitHub Actions | CI/CD |
| **Development** | Turborepo | Monorepo management |
|  | TypeScript | Type safety |
|  | pnpm | Package manager |
|  | Drizzle ORM | Database toolkit |

---

## 8. Future Considerations

### 8.1 Phase 2 Features
- **Vector Search:** Implement semantic search for agent memory
- **Agent Marketplace:** Community-contributed agents
- **Advanced Analytics:** ML-powered insights and predictions
- **Mobile App:** React Native for iOS and Android

### 8.2 Scalability Enhancements
- **Multi-region Deployment:** Deploy to multiple Cloudflare regions
- **Database Sharding:** Scale D1 across multiple databases
- **Edge AI:** Run smaller models directly at the edge

### 8.3 Integration Roadmap
- **Figma API:** Design tool integration
- **Slack/Teams:** Communication notifications
- **Jira/Asana:** Project management sync
- **Stripe:** Payment processing

---

## 9. Conclusion

Nexio's architecture is designed to be:

1. **Edge-Native:** Leverage Cloudflare's global network for low latency
2. **Agentic:** Persistent, stateful AI agents that collaborate and learn
3. **Secure:** Zero-Trust multi-tenancy with HITL approval flows
4. **Scalable:** Serverless architecture that scales automatically
5. **Cost-Effective:** Leverage Cloudflare's generous free tier and low pricing

The architecture supports the core vision of an Agentic OS that empowers agencies and digital entrepreneurs to orchestrate AI agents at scale while maintaining complete control and visibility.

---

**Built by Fataplus** — The Agentic OS 🚀
