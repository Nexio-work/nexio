# Nexio Product Requirements Document (PRD)

**Project:** Nexio - Agentic Operating System for B2B2B Collaboration
**Vision:** An Agentic Operating System empowering Madagascar's Digital Inclusion and high-performance agency growth

---

## 1. Product Context & Objectives

Nexio is a multi-tenant AI orchestration framework built on Cloudflare Workers and OpenClaw (Moltbot).

**Mission:**
- To provide a scalable, low-cost platform for agencies to manage AI-native workflows
- For Madagascar's Ministry (MNDPT) to drive digital entrepreneurship

**2026 Competitive Edge:**
- Agentic automation
- Zero-Trust security
- Native Malagasy/French support

---

## 2. Target Personas

### The Agency CEO (Strategy)
- **Role:** Strategic leader of design or digital agency
- **Needs:** High-level ROI visibility and white-label control
- **Pain Points:** Can't see profitability per project/client; no unified view of AI agents; scaling adds complexity not revenue
- **Goals:** Grow agency profitably; monetize IP (templates, design systems); minimize operational overhead

### Head of Operations (Execution)
- **Role:** Manages daily workflows, assigns tasks, ensures quality
- **Needs:** "Human-in-the-Loop" (HITL) approval queues for AI outputs
- **Pain Points:** Too many decisions to make manually; no visibility into what agents are doing; quality drift
- **Goals:** Streamline operations; ensure quality at scale; reduce manual approvals

### Digital Entrepreneur (Madagascar)
- **Role:** Runs local digital service business (e.g., freelance designer, small agency)
- **Needs:** Low-cost, low-bandwidth tools to run a digital business
- **Pain Points:** High software costs; poor connectivity in rural areas; lack of training/digital skills
- **Goals:** Start/operate digital business with minimal infrastructure; learn digital skills; reach local/global clients

### Ministry Operator (MNDPT)
- **Role:** Oversees national digital inclusion programs
- **Needs:** Oversight for national digital entrepreneurship programs
- **Pain Points:** Can't track impact across regions; inconsistent tools across entrepreneurs; no compliance tracking
- **Goals:** Scale digital inclusion programs; measure economic impact; ensure program compliance

---

## 3. High-Level Requirements

### 3.1 Architecture & Tenancy

**Infrastructure:**
- **Cloudflare Workers Standard Plan** ($5/month) leveraging Sandboxes/Containers
- **Compute:** Serverless functions for API, background jobs, webhooks
- **Storage:** Cloudflare D1 (database), R2 (files), KV (cache), Vector (embeddings)
- **Edge:** Automatic CDN for static assets; low latency globally

**Tenancy Model:**
- **Centralized Nexio Worker** handling dynamic agencyId scoping (Performance-first)
- **Row-level security:** `WHERE tenant_id = ?` on all queries
- **Durable Objects:** Persistent agent state (sessions, context)
- **Isolation:** Complete data segregation per tenant (no cross-tenant leaks)

### 3.2 The Agentic Kernel

**Master Agent:**
- **The "Gateway" orchestrator (24/7 process)**
- Manages agent lifecycle (wake, sleep, assign tasks)
- Maintains shared "Mission Control" state in D1
- Coordinates inter-agent communication

**Staggered Heartbeats:**
- **Scheduled "wakeups" every 15 mins** (staggered by 2 mins per agent)
- Each agent checks for: pending tasks, mentions, triggers, scheduled work
- If work exists → execute, update state, sleep
- Optimizes compute (agents don't run continuously)

**Memory Stack:**
- **WORKING.md:** Local file for current task state (context buffer)
- **Daily Logs:** `YYYY-MM-DD.md` for raw activity (debug, audit trail)
- **MEMORY.md:** Long-term curated facts (learned over time)
- **SOUL & AGENTS Files:** Each agent is defined by a personality (SOUL.md) and operating manual (AGENTS.md)

**Peer Review Mechanics:**
- **Mandatory "Refute/Praise" loops before human approval**
- Agent A proposes action → Agent B reviews (validate, critique, approve)
- If conflict → human resolves
- Ensures quality and prevents hallucinations

### 3.3 Madagascar Digital Inclusion Pack

**Multilingual Support:**
- **Primary:** Malagasy and French
- **Secondary:** English (for global clients)
- **UI:** Auto-detect locale; switch labels, messages, help content
- **Agents:** Can respond in user's language (context-aware)

**Low-Bandwidth Mode:**
- **Adaptive UI:** Minimal animations, reduced assets, text-only fallback
- **Optimized payloads:** Gzip compression, delta updates, minimal JS bundle
- **Edge caching:** Static content cached in rural regions

**Formation Skill:**
- **Specialized modules** for training rural entrepreneurs
- Topics: Digital hygiene, business basics, local compliance, e-commerce
- Format: Interactive lessons, quizzes, certificate tracking

**MNDPT Hooks:**
- **API integration readiness** for national biometric and identity systems
- KYC workflows: Verify entrepreneurs via national ID
- Reporting: Program metrics export for Ministry oversight

---

## 4. Feature Set (MVP/MVE)

### 4.1 Core Shell

**Cmd+K Terminal:**
- The primary entry point for all commands
- Fuzzy search: Type any command, action, agent name
- Natural language: "Generate report for Riake" → mapped to action
- Keyboard shortcut: Cmd+K (Mac) or Ctrl+K (Windows/Linux)

**Glassmorphic Dashboard:**
- Real-time "Pulse" of business and agent activities
- Widgets: Active agents, recent tasks, pending approvals, revenue metrics
- Glassmorphism: Translucent panels, blur, aurora glows (teal/purple)
- Dark mode: Default high-contrast obsidian surfaces

### 4.2 HITL Command Center

**Approval Feed:**
- Visual interface for human staff to approve/edit agent-generated tasks
- Queue: Pending approvals sorted by priority/time
- Batch: Approve multiple at once
- Edit: Modify before approving (agent learns from corrections)

**Audit Logs:**
- Transparent history of all AI reasoning and actions
- Search: Filter by agent, task, time range
- Replay: See full chain of thought for any action
- Export: JSON/CSV for compliance

### 4.3 Agentic Collaboration

**Agent-to-Agent Dialogue:**
- Real-time "Thinking Loop" visible in UI
- Agents "talk" to each other (party mode)
- Example: "PM agent drafted spec, piping to Dev agent now..."
- Visibility: No black box AI

**Task Handoffs:**
- Agent A completes task → passes to Agent B
- State preserved (no data loss)
- Notifications: Agents alerted when assigned new task

**Shared Reality:**
- All agents read/write to same "Data Bus" (D1/R2/KV)
- No data silos
- Consistent state across all agents

---

## 5. Success Metrics

### Activation
- **Time to first successful "Agentic Action"** (Target: < 5 mins)
- From signup to completing first agent task

### Scalability
- **Number of tenants supported** on a single $5/mo Cloudflare account
- Target: 100+ concurrent tenants without performance degradation

### Inclusion
- **Adoption rate** among regional digital training centers in Madagascar
- Target: 80% of centers using platform within 6 months

---

## 6. Non-Functional Requirements

### Performance
- **Page load time:** < 2s (p95)
- **API response time:** < 100ms (p95)
- **Concurrent users:** 100 per tenant (scalable via Workers)
- **Edge latency:** < 50ms globally (Cloudflare CDN)

### Security
- **Multitenancy:** Row-level isolation (tenant_id on every query)
- **Auth:** BetterAuth with OAuth2 (Google, Microsoft)
- **2FA:** Mandatory for sensitive actions (approvals, settings)
- **Encryption:** TLS 1.3 in transit, AES-256 at rest (D1/R2)

### Scalability
- **D1:** Plan for 10M rows/month per tenant
- **Workers:** Auto-scale to 1000 requests/min per tenant
- **R2:** 1TB storage per tenant (expandable)
- **Durable Objects:** 10k objects per tenant

### Reliability
- **Uptime:** 99.5% initially, 99.9% at scale
- **Backup:** Daily backups at 2am UTC
- **Monitoring:** Sentry for errors, Cloudflare Analytics for metrics

### Multilingual
- **Languages:** Malagasy, French, English (v1); add others later
- **UI:** All labels, messages, help text translatable
- **Content:** Agents generate content in user's language
- **RTL/LTR:** Support both text directions (future languages)

---

## 7. Data Model (High-Level)

### Tenants Table
```sql
CREATE TABLE tenants (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  plan TEXT NOT NULL DEFAULT 'free', -- free, pro, enterprise
  settings JSONB, -- white-label config, language, timezone
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

CREATE INDEX idx_tenants_plan ON tenants(plan);
```

### Users Table
```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  email TEXT NOT NULL UNIQUE,
  name TEXT NOT NULL,
  role TEXT NOT NULL, -- owner, admin, staff, external
  password_hash TEXT, -- BetterAuth
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

CREATE INDEX idx_users_tenant ON users(tenant_id);
CREATE INDEX idx_users_email ON users(email);
```

### Agents Table
```sql
CREATE TABLE agents (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  name TEXT NOT NULL, -- Jarvis, Fury, Growth, etc.
  persona TEXT NOT NULL, -- JSONB with SOUL.md content
  status TEXT NOT NULL DEFAULT 'idle', -- idle, active, sleep, error
  last_heartbeat INTEGER, -- Unix timestamp
  working_md TEXT, -- Current task context (WORKING.md content)
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

CREATE INDEX idx_agents_tenant ON agents(tenant_id);
CREATE INDEX idx_agents_status ON agents(status);
CREATE INDEX idx_agents_heartbeat ON agents(last_heartbeat);
```

### Tasks Table
```sql
CREATE TABLE tasks (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  assigned_agent_id TEXT REFERENCES agents(id) ON DELETE SET NULL,
  status TEXT NOT NULL DEFAULT 'pending', -- pending, in_progress, completed, failed
  priority TEXT NOT NULL DEFAULT 'normal', -- low, normal, high, urgent
  payload JSONB, -- Task data, context, metadata
  result JSONB, -- Agent's output
  review_status TEXT NOT NULL DEFAULT 'pending', -- pending, approved, rejected
  review_agent_id TEXT REFERENCES agents(id), -- Who reviewed
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

CREATE INDEX idx_tasks_tenant ON tasks(tenant_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_agent ON tasks(assigned_agent_id);
CREATE INDEX idx_tasks_review ON tasks(review_status);
```

### Approvals Table (HITL)
```sql
CREATE TABLE approvals (
  id TEXT PRIMARY KEY,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  task_id TEXT NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  user_id TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  status TEXT NOT NULL DEFAULT 'pending', -- pending, approved, rejected
  decision_reason TEXT, -- Why approved/rejected
  created_at INTEGER NOT NULL,
  decided_at INTEGER
);

CREATE INDEX idx_approvals_tenant ON approvals(tenant_id);
CREATE INDEX idx_approvals_status ON approvals(status);
```

### Memory Stack Tables
```sql
-- Daily logs
CREATE TABLE agent_daily_logs (
  id TEXT PRIMARY KEY,
  agent_id TEXT NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  date TEXT NOT NULL, -- YYYY-MM-DD
  content TEXT, -- Raw JSONL of activity
  created_at INTEGER NOT NULL
);

CREATE INDEX idx_logs_agent_date ON agent_daily_logs(agent_id, date);

-- Long-term memory
CREATE TABLE agent_memory (
  id TEXT PRIMARY KEY,
  agent_id TEXT NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
  tenant_id TEXT NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  key TEXT NOT NULL,
  value JSONB,
  confidence REAL, -- 0-1, how certain
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL
);

CREATE UNIQUE INDEX idx_memory_agent_key ON agent_memory(agent_id, key);
```

---

## 8. API Contracts

### External Services

#### BetterAuth (Authentication)
**Purpose:** Session management, OAuth providers, 2FA

**Data Flow:**
```
Nexio → BetterAuth: Login request, OAuth callback
BetterAuth → Nexio: Session token, user profile
```

**Protocol:** REST
**Auth:** API key + OAuth
**Rate Limit:** 100 req/min

#### Cloudflare D1 (Database)
**Purpose:** Primary data store (tenants, users, agents, tasks, memory)

**Data Flow:**
```
Nexio → D1: Queries (SELECT, INSERT, UPDATE)
D1 → Nexio: Query results
```

**Protocol:** HTTP (Workers binding)
**Auth:** Service binding (internal)
**Rate Limit:** 5,000 reads/min per account

#### Cloudflare R2 (Storage)
**Purpose:** File storage (documents, assets, exports)

**Data Flow:**
```
Nexio → R2: Upload file, download file, list files
R2 → Nexio: File URL, metadata
```

**Protocol:** S3-compatible API
**Auth:** Service binding (internal)
**Rate Limit:** 1,000 PUT requests/min

#### OpenClaw/Moltbot (AI Engine)
**Purpose:** Agent execution, context management, memory

**Data Flow:**
```
Nexio → OpenClaw: Spawn agent, send task, get result
OpenClaw → Nexio: Agent status, task output, memory updates
```

**Protocol:** WebSocket + HTTP
**Auth:** API token
**Rate Limit:** 100 req/min

#### MNDPT Biometric API (Madagascar)
**Purpose:** KYC verification for entrepreneurs

**Data Flow:**
```
Nexio → MNDPT: Verify national ID, get user data
MNDPT → Nexio: Verification status, user profile
```

**Protocol:** REST (HTTPS)
**Auth:** API key (Ministry-provided)
**Rate Limit:** 50 req/min

---

## 9. Risk Analysis

| Risk | Impact | Probability | Mitigation | Owner |
|-------|---------|-------------|---------|
| Multitenancy security flaw (data leak between tenants) | High | Low | Row-level isolation on every query; tenant_id middleware; automated security tests | Dev |
| Cloudflare D1 performance issues (slow queries at scale) | Medium | Medium | Query optimization; indexes; read-through KV cache; monitor query times | Dev |
| Agent hallucinations in critical workflows | High | Medium | Peer review loops (Refute/Praise); HITL approval; fallback to manual override | PM/Dev |
| Low-bandwidth users can't use app | High | High | Low-bandwidth mode; minimal UI; text-only fallback; edge caching | UX/Dev |
| Multilingual translation errors | Medium | Medium | Context-aware language detection; manual language override; community feedback loops | UX/Dev |
| OpenClaw integration latency (slow agent responses) | Medium | Low | Async agent spawning; WebSocket reconnection; local caching of common responses | Dev |
| MNDPT API downtime (KYC verification fails) | Medium | Low | Graceful degradation (manual KYC); cached verifications; retry with backoff | Dev |
| Worker cold starts (slow first request) | Low | High | Keep-alive workers; pre-warm critical paths; monitor cold start times | Dev |
| Cost overrun (Cloudflare bill too high) | Medium | Medium | Monitor usage per tenant; enforce limits; alert on cost thresholds; optimize query patterns | Ops |

---

## 10. Questions for Architecture Phase

### 10.1 Authentication & Authorization
1. How do we handle cross-tenant authentication? Single session per user across all tenants, or separate sessions?
2. SSO requirements for enterprise clients? SAML, Okta, custom providers?
3. OAuth providers priority: Google, Microsoft, GitHub, Apple? Which for v1?

### 10.2 Data Isolation
4. What level of data isolation per tenant? Row-level (`WHERE tenant_id`) enough, or separate schemas/databases per tenant?
5. How do we handle tenant migration/move data between tenants?

### 10.3 Storage Strategy
6. Asset storage limits per tenant? What's included in free vs paid tiers?
7. Large file handling (e.g., 100MB+ video files): Stream directly from R2, or proxy through Workers?

### 10.4 Integrations
8. Design tool integrations priority: Figma API v1, then Adobe CC? Or parallel development?
9. Communication tools: Slack notifications, email alerts, in-app only for MVP?

### 10.5 Pricing & Tiers
10. What's the pricing model? Per-user, per-project, per-tenant, or hybrid?
11. Target ARR per customer for free, pro, enterprise tiers?

### 10.6 Onboarding
12. What's the minimum friction onboarding path? Blank slate, template-based, guided wizard?
13. How do we handle first-run setup? Create first agent, invite users, or pre-configured defaults?

### 10.7 Real-time Requirements
14. What needs real-time updates? Agent status, task progress, notifications—WebSocket or polling?
15. Real-time costs: Durable Objects vs. WebSockets vs. long-polling?

### 10.8 Compliance
16. SOC 2 requirements before enterprise launch, or can we defer to v2?
17. GDPR compliance from day one? Data residency (Malagascar-specific rules)?

### 10.9 Internationalization (i18n)
18. Multi-language support from day one (Malagasy, French, English), or English-only MVP with i18n hooks for later?
19. Translation strategy: Human-translated content, machine translation (Google Translate), or hybrid?

### 10.10 Mobile Strategy
20. Mobile-first critical, or can we launch web-only and iterate?
21. Progressive Web App (PWA) for mobile, or native app later?

---

## 11. Next Steps

### Launch Architecture Agent
- Define Cloudflare Workers/Pages architecture
- Design D1 schema (detailed with indexes, migrations)
- Define BetterAuth integration strategy
- Design OpenClaw/Moltbot integration layer
- Plan deployment strategy (staging → production)

### Define Development Sprints
- Break MVP features into epics/stories
- Prioritize by user impact and dependencies
- Estimate effort (story points)
- Create sprint plan (3 sprints for v1)

---

**Built by Fataplus** — Agentic OS 🚀
