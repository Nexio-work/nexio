# Nexio - B2B2B SaaS Platform

**Vision:** Platform enabling business-to-business-to-business workflows for Fataplus
**Architecture:** Multitenant SaaS, Cloudflare scalable
**Methodology:** BMAD-inspired orchestration (agile, AI-driven development)

## Project Structure

```
nexio/
├── apps/              # Monorepo applications
│   ├── web/          # Next.js frontend
│   ├── api/          # Backend API
│   └── worker/       # Background jobs
├── packages/         # Shared packages
│   ├── ui/          # Design system
│   ├── db/          # Database schema & migrations
│   ├── auth/        # Authentication & multitenancy
│   └── shared/      # Shared utilities
├── docs/            # Technical documentation
├── .github/         # CI/CD, workflows
└── orchestration/   # AI agent workflows & prompts
```

## Development Phases

1. **Discovery & Brief** → Problem, users, MVP scope
2. **Requirements (PRD)** → Personas, metrics, features
3. **Architecture** → System design, tech stack
4. **Planning** → Epics, stories, priorities
5. **Sprints** → Dev cycles, reviews
6. **Deployment** → Cloudflare, staging, production

## Tech Stack (Cloudflare-ready)

- **Frontend:** Next.js 15, React 19, Tailwind CSS, shadcn/ui
- **Backend:** Hono (Cloudflare Workers), Drizzle ORM
- **Database:** Cloudflare D1 (SQLite), Vector (RAG)
- **Auth:** Lucia, OAuth2, multitenancy
- **Deployment:** Cloudflare Pages, Workers, R2
- **Monitoring:** Sentry, analytics

## Orchestration

AI agents (Lex-orchestrated) execute workflows:
- Product brief → PRD → Architecture → Stories → Dev → Review

See `orchestration/` for agent prompts and workflows.

---

**Built by Fataplus** — B2B2B Masterpiece in the Making 🚀
