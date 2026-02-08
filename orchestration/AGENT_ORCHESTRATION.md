# Nexio - Agent Orchestration System

**Orchestrator:** Lex (Digital Warlord & Chief of Staff)
**Target:** Nexio B2B2B SaaS Platform
**Method:** BMAD-inspired agile workflows
**Execution:** Parallel sub-agents, coordinated by Lex

---

## Orchestration Principles

1. **Lex stays available** → Always respond to Fefe, orchestrate in background
2. **Parallel execution** → Multiple agents work simultaneously
3. **Autonomous blocks** → Each agent has clear scope + deliverable
4. **Checkpoint reporting** → Agents report back on completion
5. **Iterative refinement** → Feedback loops between phases

---

## Development Workflow

### Phase 1: Discovery & Brief
**Agent:** `discovery-agent`
**Deliverable:** `docs/product-brief.md`

```
- Problem statement: What is Nexio solving?
- Target users: Who are we building for?
- Market: B2B2B space analysis
- MVP scope: What's v1?
- Success metrics: How do we measure?
```

### Phase 2: Requirements (PRD)
**Agent:** `prd-agent`
**Deliverable:** `docs/prd.md`

```
- Personas & journeys
- Feature breakdown
- Non-functional requirements (performance, security, scalability)
- API contracts (if external integrations)
- Risk analysis
```

### Phase 3: Architecture
**Agent:** `architecture-agent`
**Deliverable:** `docs/architecture.md`

```
- System design (diagrams + text)
- Tech stack rationale (Cloudflare-specific)
- Data model (D1 schema)
- Service boundaries (monorepo structure)
- Deployment strategy (staging → production)
- Security model (multitenancy, auth, RBAC)
```

### Phase 3.5: ERP/CRM Features Analysis
**Agent:** `lex-manual` (orchestrator)
**Deliverable:** `docs/erp-crm-features.md`

```
- ERP modules: Accounting, CRM, Inventory, Projects, HR (Frappe/Dolibarr/Odoo-inspired)
- CRM features: Contacts, deals pipeline, tasks, document management
- Agentic integration: ERP/CRM as data bus + agent skills
- Backend analysis: HonoJS + Motia coupling
- Deployment: Hybrid VPS + Cloudflare
- Monorepo structure: packages/erp/ module breakdown
```

### Phase 4: Planning
**Agent:** `planning-agent`
**Deliverables:**
- `docs/epics.md` (high-level features)
- `docs/stories.md` (user stories with priorities)

```
- Epics: Major feature areas
- Stories: Implementable units
- Priorities: P0 (v1 critical), P1 (v2), P2 (future)
- Dependencies: What blocks what
```

### Phase 5: Sprint 1 - Foundation (ERP/CRM)
**Agents:** `dev-agent-ui`, `dev-agent-api`, `dev-agent-db`, `dev-agent-erp`
**Deliverables:**
- Working monorepo structure
- Design system (shadcn/ui)
- Authentication flow (Lucia + multitenancy)
- Database schema (Drizzle + D1 + ERP tables)
- ERP/CRM modules (accounting, CRM, inventory, projects, HR)
- Cloudflare deployment config

### Phase 6: Sprint 2 - Core Features
**Agents:** `dev-agent-ui`, `dev-agent-api`
**Deliverables:**
- Tenant onboarding flow
- Dashboard & navigation
- Key user journeys (based on PRD)

### Phase 7+: Iterative Sprints
**Pattern:** Repeat Sprint pattern until MVP complete

---

## Agent Templates

Each agent receives:
1. **Context file** (`orchestration/prompts/[agent-name].md`)
2. **Workspace link** (pointer to specific files/folders)
3. **Output location** (where to write deliverables)
4. **Success criteria** (what "done" looks like)

---

## Communication Protocol

### Agent → Lex (on completion)
```
DONE: [phase-name]
Agent: [agent-id]
Deliverable: [file-path]
Summary: [2-3 sentences]
Next steps: [what's needed]
```

### Lex → Agent (on launch)
```
LAUNCH: [agent-id]
Task: [from workflow context]
Context: [file paths to read]
Output: [where to write]
Deadline: [optional]
```

### Lex → Fefe (progress updates)
```
[Nexio] Phase X in progress
- [agent-id] working on [task]
- ETA: [estimated time]
```

---

## Tech Stack (Agent Reference)

**Frontend:**
- Next.js 15 (App Router)
- React 19
- Tailwind CSS
- shadcn/ui components
- TypeScript

**Backend:**
- Hono (Cloudflare Workers)
- Drizzle ORM
- Cloudflare D1 (SQLite)
- Cloudflare R2 (storage)
- Lucia (auth)

**Tools:**
- pnpm (workspace)
- Turborepo (build system)
- Cloudflare Wrangler
- TypeScript ESLint
- Prettier

---

## Current Status

**Phase:** Setup complete
**Next:** Launch Phase 1 (Discovery & Brief)

---

**Orchestrator:** Lex 🔥
**Project:** Nexio - B2B2B SaaS Platform
**Org:** https://github.com/Nexio-work
