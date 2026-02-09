# OpenClaw Skills - Synthèse pour Nexio

**Source:** https://github.com/VoltAgent/awesome-openclaw-skills
**Objectif:** Fournir aux agents dev Nexio les capacités OpenClaw disponibles pour implémenter l'Agentic OS

---

## 1. Capacités Critiques pour Nexio

### 🧠 Long-Running Task Management

**Skills:**
- **`task-status`** - Envoyer des notifications % de complétion chaque heure
  - Use: Dev UI Agent, Dev DB Agent pour rapporter le progrès
  - Configuration: Notifications Telegram/Slack vers Fefe
- **`work-report`** - Générer rapport quotidien (déjà dans GitHub Actions)
  - Use: Automatiser le suivi quotidien
  - Configuration: Rapports dans `docs/daily-reports/YYYY-MM-DD.md`

**Agents concernés:** Dev API, Dev DB, Dev ERP

---

### 📚 Documentation & Génération de Docs

**Skills:**
- **`doc-coauthoring`** - Assist à la création de contenu Markdown
  - Use: Planning Agent pour rédiger les specs, guides, épics
  - Configuration: Templates de docs (structure, style guide)
  - Avantages: Cohérence visuelle entre documentation et code

**Skills:**
- **`content-id-guide`** - Organiser le contenu par sections
  - Use: Dev ERP Agent pour structurer les modules ERP (Accounting, CRM, etc.)
  - Configuration: Catégories de docs (API, Guides, Déploiement)

**Pour Zennical (si on choisit cette option):**
- **`readability`** - Améliorer la lisibilité du contenu généré
- **`seo`** - Optimiser pour les moteurs de recherche

---

### 🔧 Git & GitHub Automation

**Skills:**
- **`git-summary`** - Résumé rapide du repo
  - Use: Dashboard GitHub Actions / Notifications à Fefe
  - Configuration: Exclure `packages/`, `docs/` (ne pas les monorepos d'agents dev)

**Skills:**
- **`commit-analyzer`** - Analyser les patterns de commits
  - Use: Pour métriques de qualité du code (taille des PRs, frequency)
  - Configuration: Alerts si des patterns dangereux détectés

**Skills:**
- **`auto-pr-merger`** - Automatiser la gestion des releases (semantic versioning)
  - Use: Pour gérer `package.json` et tags de releases
  - Configuration: PRs sur `main`, merges automatisés si tests passent

**Agents concernés:** Dev UI (Next.js), Dev API (Hono), Dev DB (D1), Dev ERP (Modules)

---

### 🤖 Web Development & Automation

**Skills:**
- **`browse`** - Navigation automatique dans les applications web
  - Use: Dev UI Agent pour implémenter dans Next.js
  - Configuration: Support de web browsing (Playwright/Cypress tests)

**Skills:**
- **`web-scraper`** - Extraction de données depuis des sites externes
  - Use: Dev API Agent (Hono) pour créer des endpoints de scraping
  - Configuration: Proxy R2 pour stocker les données extraites

**Skills:**
- **`pdf--documents`** - Manipulation de documents PDF (rapports, factures)
  - Use: Dev ERP Agent (Accounting module) pour générer les factures/rapports
  - Configuration: Intégration R2 + upload des rapports générés

**Agents concernés:** Dev API, Dev ERP

---

### 🧠 AI & LLM Skills

**Skills:**
- **`claude-optimised`** - Optimiser les fichiers `CLAUDE.md` pour Claude Code
  - Use: Tous les agents (Dev) doivent respecter le format Claude
  - Configuration: Guide Claude pour comprendre le contexte Nexio

**Skills:**
- **`claude-team`** - Orchestration multi-agents Claude Code
  - Use: Planning Agent pour coordonner les 4 agents dev (UI, API, DB, ERP)
  - Configuration: Sessions Claude séparées par tenant

**Skills:**
- **`anthropic-messages`** - API pour envoyer des messages Anthropic
  - Use: Pour notifs push (rapports quotidiens) vers Fefe
  - Configuration: Stocker `ANTHROPIC_API_KEY` dans GitHub secrets

**Agents concernés:** Tous (OpenClaw Bridge)

---

### 🔧 DevOps & Cloud

**Skills:**
- **`cloudflare-pages`** - Déploiement statique
  - Use: GitHub Actions + Wrangler CLI
  - Configuration: `wrangler.toml`, `.github/workflows/ci-cd.yml`

**Skills:**
- **`cloudflare-workers`** - Gestion Workers Hono
  - Use: Dev API Agent (Hono)
  - Configuration: Deployment pipelines (staging/production)

**Skills:**
- **`cloudflare-d1`** - Gestion D1 (SQLite edge)
  - Use: Dev DB Agent (Drizzle)
  - Configuration: Migrations, schema, indexes

**Skills:**
- **`cloudflare-r2`** - Storage fichiers
  - Use: Dev API/ERP Agents pour uploader/download des fichiers
  - Configuration: Buckets `nexio-assets` pour docs, rapports, images

**Agents concernés:** Dev API, Dev DB, Dev ERP

---

### 📊 Data & Analytics

**Skills:**
- **`postgres-adapter`** - Adaptateur PostgreSQL (si on choisit Postgres au lieu de D1)
  - Use: Dev DB Agent (Drizzle) pour créer adapter Postgres
  - Configuration: Migration D1 → Postgres

**Skills:**
- **`analytics-dashboard`** - Tableau de bord analytics
  - Use: Dev UI Agent (Next.js) pour créer des widgets analytics
  - Configuration: Intégration Cloudflare Analytics + Sentry

**Agents concernés:** Dev UI, Dev API, Dev DB

---

### 🛡 Sécurité & Auth

**Skills:**
- **`betterauth`** - Authentification multi-tenant
  - Use: Dev API Agent (Hono) pour créer endpoints auth
  - Configuration: Sessions D1, KV cache, middleware tenant

**Skills:**
- **`oauth2-providers`** - OAuth Google, Microsoft
  - Use: Intégrations NextAuth ou custom providers
  - Configuration: Google Cloud Console, Azure AD

**Agents concernés:** Dev API, Dev UI

---

### 🧪 Productivity & Tasks

**Skills:**
- **`notion`** - Base de connaissances (si on choisit Notion au lieu de docs statiques)
  - Use: Pour documents vivants (guides utilisateurs, specs live)
  - Configuration: API Notion pour sync automatique

**Skills:**
- **`linear`** - Gestion de tâches avancée
  - Use: Pour tickets bugs, user stories, épics
  - Configuration: Intégration Linear API (si disponible)

**Agents concernés:** Planning Agent, Support Agent

---

### 📚 Notes & PKM

**Skills:**
- **`obsidian`** - Notes Markdown avec graph visual
  - Use: Pour documentation technique (architecture, API specs)
  - Configuration: Fichiers `.md` dans `docs/` avec liens interconnectés

**Skills:**
- **`logseq`** - Base de logs locale
  - Use: Pour logs d'agents (sessions, activity)
  - Configuration: Logs structurés en JSONL pour parse automatique

**Agents concernés:** Dev API, Dev DB (pour logs d'agents)

---

## 2. Installation des Skills Critiques

### Pour Long-Running Task Management
```bash
# Installer skill task-status
npx clawhub install task-status

# Installer skill work-report
npx clawhub install work-report

# Configurer dans OpenClaw
claw config set task-status.notifications=true --webhook=https://api.nexio.work/webhooks/progress
claw config set work-report.webhook=https://api.nexio.work/webhooks/daily-report

# Assigner à l'agent Dev UI
claw assign --agent=nexio-dev-ui task-status,work-report
```

### Pour Documentation
```bash
# Installer skill doc-coauthoring
npx clawhub install doc-coauthoring

# Installer skill content-id-guide
npx clawhub install content-id-guide
```

### Pour Git & GitHub
```bash
# Installer skill git-summary
npx clawhub install git-summary

# Installer skill auto-pr-merger
npx clawhub install auto-pr-merger

# Configurer les webhooks GitHub
claw config set github.webhook=https://api.nexio.work/webhooks/github/summary
```

### Pour Cloudflare
```bash
# Installer skill cloudflare-pages
npx clawhub install cloudflare-pages

# Installer skill cloudflare-workers
npx clawhub install cloudflare-workers

# Installer skill cloudflare-d1
npx clawhub install cloudflare-d1

# Installer skill cloudflare-r2
npx clawhub install cloudflare-r2
```

## 3. Recommandations par Agent Dev

### Dev UI Agent (Next.js)
**Installe d'abord:**
1. **`git-summary`** - Pour voir le résumé rapide du repo en dashboard
2. **`task-status`** - Pour envoyer notifications % de complétion vers Fefe
3. **`browse`** - Tests e2e (Playwright) pour les workflows web
4. **`doc-coauthoring`** - Créer des templates de docs dans Next.js

**Architecture à préparer:**
- Utiliser App Router Next.js
- Shadcn/ui pour les composants (cards, badges, modals)
- Server Actions pour les notifications en temps réel
- WebSocket client (Durable Objects) pour live updates des agents

---

### Dev API Agent (Hono)
**Installe d'abord:**
1. **`cloudflare-workers`** - Configuration Wrangler + Workers
2. **`cloudflare-d1`** - Configuration Drizzle + D1
3. **`cloudflare-r2`** - Configuration R2 storage
4. **`betterauth`** - Configuration BetterAuth + OAuth

**Architecture à préparer:**
- Hono avec routes RESTful (`/api/v1/*`)
- Middleware tenant (tenant_id validation)
- Rate limiting (KV store)
- Error handling (standardized responses)
- CORS (pour les apps Next.js et clients externes)

---

### Dev DB Agent (D1/Drizzle)
**Installe d'abord:**
1. **`cloudflare-d1`** - Configuration Drizzle + D1
2. **`logseq`** - Pour les logs d'agents (sessions, activity)
3. **`postgres-adapter`** - (Optionnel) Si on migre vers Postgres

**Architecture à préparer:**
- Drizzle schema avec toutes les tables ERP/CRM
- Migrations systematiques (up/down SQL)
- Query helpers pour les opérations courantes

---

### Dev ERP Agent (Modules)
**Installe d'abord:**
1. **`cloudflare-d1`** - Configuration Drizzle + D1
2. **`cloudflare-r2`** - Storage fichiers
3. **`pdf--documents`** - Génération PDF (rapports, factures)
4. **`postgres-adapter`** - (Optionnel)

**Architecture à préparer:**
- Modules ERP isolés (accounting/, crm/, inventory/, projects/, hr/)
- API endpoints pour chaque module (`/api/v1/erp/*`)
- Tests unitaires pour chaque module

---

## 4. Configuration des Agents avec OpenClaw

### Pour tous les agents
```bash
# Dans le code d'initialisation de chaque agent (apps/api/src/index.ts ou packages/*/agent.ts)

import { openclaw } from '@zreadai/obr-supapowers';

export const agentConfig = {
  name: 'nexio-dev-ui', // ou autre agent
  model: 'zai/glm-4.7', // ou modèle approprié
  skills: [
    'task-status',     // Pour notifications de progression
    'git-summary',    // Pour résumé rapide du repo
    'browse',         // Pour navigation et scraping web
    'doc-coauthoring'  // Pour création de contenu
    'logseq'         // Pour logs d'agents
  ],
  // Configuration OpenClaw
  openclaw: {
    apiKey: process.env.OPENCLAW_API_KEY,
    baseURL: 'https://api.openclaw.ai',
  },
};
```

### Pour les agents spécialisés (ex: Accounting Agent)
```bash
// Dans packages/erp/accounting/agent.ts

import { openclaw } from '@zreadai/obr-supapowers';

const accountingSkills = [
  'postgres-adapter',  // Pour requêtes comptables complexes
  'pdf--documents',    // Pour générer factures/rapports
  'postgres-adapter',  // Pour les données comptables structurées
];

const accountingAgent = new openclaw.Agent({
  name: 'accounting-agent',
  model: 'zai/glm-4.7',
  skills: accountingSkills,
});

export { accountingAgent };
```

---

## 5. Workflow d'Intégration OpenClaw

### Phase 1: Préparation (Semaine 1)
1. Créer le compte Zread AI et obtenir la clé API
2. Installer les skills critiques (`task-status`, `work-report`, `doc-coauthoring`)
3. Configurer les webhooks Zread AI (nexio.work)
4. Tester les webhooks localement avec curl

### Phase 2: Activation (Semaine 2-3)
1. Intégrer `task-status` dans tous les agents dev (UI, API, DB, ERP)
2. Activer `work-report` pour rapport quotidien automatique
3. Activer `git-summary` pour résumé repo

### Phase 3: Superpowers (Semaine 4+)
1. Explorer les skills de "Long-Running Tasks" et "Documentation"
2. Installer et configurer les skills avancés (auto-pr-merger, commit-analyzer)
3. Activer les "superpowers" pour tâches complexes (web browsing, code execution)

---

## 6. Questions pour le Planning Agent

1. **Priorisation des OpenClaw Skills** : Quels skills installer en priorité P0 (Sprint 1) ?
   - P0 : `task-status`, `work-report`, `doc-coauthoring`, `git-summary`
   - P1 : `browse`, `web-scraper`, `pdf--documents`
   - P2 : `auto-pr-merger`, `commit-analyzer`, `claude-optimised`

2. **Architecture Agents vs OpenClaw Skills** : Comment les agents interagissent-ils avec OpenClaw ?
   - Option A : Direct OpenClaw API calls (manuels)
   - Option B : Via OpenClaw Bridge package (automatisé)

3. **Gestion des Notifications** : Comment les agents envoient-ils les notifications vers Fefe ?
   - Option A : Telegram Bot (messages directs)
   - Option B : Email (via SendGrid ou similaire)
   - Option C : Dashboard Nexio (notifications in-app)

4. **Documentation Zennical vs Docs Statiques** : Quelle approche choisir ?
   - Option A : Zennical (générateur statique, rapide)
   - Option B : Docs statiques dans Next.js (plus de contrôle)

5. **Database Multi-tenant** : Comment isoler les données par tenant ?
   - Option A : D1 avec `WHERE tenant_id = ?` sur chaque requête
   - Option B : Postgres avec schemas séparés par tenant (si besoin de plus de puissance)

6. **Déploiement Hybride** : Comment gérer le split Dev (VPS) / Prod (Workers) ?
   - VPS : Développement local (Next.js dev, Hono Worker mock, D1 SQLite)
   - Workers : Production (Next.js Pages, Hono Workers API, D1 production, R2 storage)
   - Sync : Migrate D1 database VPS → Workers via API

---

## 7. Templates de Messages pour le Planning Agent

### Quand utiliser OpenClaw Skills dans les épics/stories

**Pour la gestion de tâches longues:**
```markdown
### [STORY-AGT-001] - Configurer OpenClaw pour suivi de projet

**As a** Dev Agent UI, je veux...
**Configurer les notifications automatiques de progression** (tâches, bugs, complétions).

**Acceptance Criteria:**
- [ ] Skill `task-status` installé et configuré dans l'agent
- [ ] Webhook nexio.work configuré et testé
- [ ] L'agent Dev UI peut envoyer des notifications % de complétion
- [ ] Intégration testée avec un exemple de tâche

**Skills OpenClaw nécessaires:**
- `task-status` - Pour envoyer les notifications
- `work-report` - Pour générer le rapport quotidien

**Effort estimé:** `M` (Moyen)

**Dépendances:** Aucune
```

**Pour les tests automatisés:**
```markdown
### [STORY-AGT-002] - Implémenter tests e2e automatisés

**As a** Dev API Agent, je veux...
**Configurer des tests end-to-end automatisés** pour l'application Nexio.

**Acceptance Criteria:**
- [ ] Tests Playwright configurés pour les workflows clés (login, onboarding, dashboard)
- [ ] Tests API (Hono) configurés pour tous les endpoints
- [ ] Intégration continue (CI/CD) pour exécuter les tests sur chaque push
- [ ] Tests E2E passent au minimum 80% de couverture

**Skills OpenClaw nécessaires:**
- `browse` - Pour simuler les utilisateurs dans les tests
- `playwright` - Pour les tests e2e (si disponible) ou `web-scraper`

**Effort estimé:** `L` (Large)

**Dépendances:** Configuration Wrangler + staging environment
```

---

## 8. Checklist d'Implémentation Sprint 1

### Dev UI Agent (Next.js)
- [ ] Configurer OpenClaw (`task-status`, `git-summary`)
- [ ] Créer layout principal avec Navbar, Footer
- [ ] Implémenter Cmd+K (command palette)
- [ ] Créer 4 pages placeholder (landing, login, onboarding, dashboard)
- [ ] Configurer Tailwind (couleurs Neural Flow)
- [ ] Intégrer shadcn/ui (composants réutilisables)

### Dev API Agent (Hono)
- [ ] Configurer OpenClaw (`task-status` pour notifications)
- [ ] Créer structure API (`/api/v1/*`)
- [ ] Implémenter middleware tenant (tenant_id validation)
- [ ] Créer endpoints auth (`/api/v1/auth/*`)
- [ ] Créer endpoints tenants (`/api/v1/tenants/*`)
- [ ] Implémenter rate limiting (KV store)
- [ ] Configuration Drizzle + D1
- [ ] Tests API unitaires

### Dev DB Agent (D1/Drizzle)
- [ ] Configurer OpenClaw (`task-status` pour logs)
- [ ] Créer schema Drizzle (tenants, users, sessions, agents, tasks, approvals, memory tables)
- [ ] Créer migrations systèmeatiques
- [ ] Créer query helpers
- [ ] Tests schema (validations, indexes)
- [ ] Configuration `logseq` (si choisi pour logs structurés)

### Dev ERP Agent (Modules)
- [ ] Configurer OpenClaw (`task-status` pour notifications)
- [ ] Créer structure ERP (`packages/erp/*`)
- [ ] Créer module Accounting (ledger, invoices)
- [ ] Créer module CRM (contacts, deals)
- [ ] Créer module Inventory (items, stock)
- [ ] Créer API endpoints ERP (`/api/v1/erp/*`)
- [ ] Tests modules ERP

---

## 9. Métriques de Succès

**Pour Sprint 1 (Foundation):**
- **Agents dev actifs :** 4 (UI, API, DB, ERP)
- **Compétence OpenClaw :** Nombre de skills installés et configurés
- **Notifications actives :** Telegram/Bot/Dashboard selon choix
- **Tests E2E :** Couverture > 80% pour workflows clés
- **Documentation générée :** Guides utilisateurs, API specs (si activé)

**KPIs à suivre:**
- **Activation:** % d'agents OpenClaw activés par rapport (cible: 100%)
- **Notification Delivery:** % de notifications réussies (cible: >95%)
- **E2E Coverage:** Couverture tests automatisés (cible: >80%)
- **Documentation Quality:** Nombre de docs générées et reviews positives

---

## 10. Questions pour le Planning Agent

1. **OpenClaw Skills Priority** : Quels skills installer en P0 (Sprint 1) ?
   - Recommandation : `task-status`, `work-report`, `git-summary`, `doc-coauthoring` (fondamental)
   - P1 : `browse`, `logseq` (pour logs agents)
   - P2 : `auto-pr-merger`, `commit-analyzer` (qualité code)

2. **Architecture Agents vs OpenClaw** : Comment intégrer OpenClaw ?
   - Recommandation : Utiliser `@zreadai/obr-supapowers` package pour initialiser
   - Créer un helper `packages/openclaw-bridge/` pour wrapper les calls OpenClaw
   - Avantages : Type-safe, gestion automatique des skills

3. **Superpowers Strategy** : Activer dès Sprint 1 ou Sprint 2 ?
   - Recommandation : Sprint 1 (Foundation) sans superpowers
   - Sprint 2 (Core Features) avec superpowers activés

4. **Documentation Strategy** : Zennical ou Next.js ?
   - Recommandation : Zennical pour landing page marketing (plus rapide)
   - Next.js Docs pour docs techniques (plus de contrôle, meilleure DX)

5. **Database Choice** : D1 ou Postgres ?
   - Recommandation : D1 pour MVP (coût réduit, plus simple)
   - Postgres pour Phase 2 (si on atteint les limites D1)

6. **Notifications** : Comment envoyer les rapports ?
   - Option A : Telegram Bot direct (plus simple pour Fefe)
   - Option B : Dashboard Nexio (notifications in-app)
   - Recommandation : Start avec Option A (Telegram Bot)

---

## 11. Next Steps

1. **Pour le Planning Agent:**
   - Utiliser ce document (`docs/openclaw-skills-synthesis.md`) comme référence
   - Créer des épics spécifiques pour l'activation OpenClaw (P0 Sprint 1)
   - Créer des user stories détaillées pour la configuration des skills
   - Définir les critères d'acceptation pour chaque story

2. **Pour le Sprint 1 (Foundation):**
   - Les 4 agents dev (UI, API, DB, ERP) commencent avec ces specs
   - Focus : Activation OpenClaw, Notifications, Tests E2E, Documentation Zennical

3. **Pour Fefe (Product Owner):**
   - Recevoir les rapports quotidiens automatiques via Telegram Bot
   - Vérifier l'avancement des agents dev (commits, compétences OpenClaw)
   - Donner feedback sur la priorité des skills

---

## 12. Conclusion

Ce document fournit une **carte complète** des capacités OpenClaw disponibles pour le développement Nexio.

**Points clés :**
- **2,999+ skills** organisés par catégories (Coding, Git, Marketing, AI, etc.)
- **Instructions d'installation** pour les skills critiques
- **Templates de user stories** pour l'intégration OpenClaw
- **Checklist Sprint 1** détaillée par agent dev
- **KPIs** pour mesurer le succès de l'activation OpenClaw

**Pour le Planning Agent :** Utilise ce document comme base pour créer des épics et user stories réalistes, en tenant compte des capacités techniques actuelles de l'écosystème OpenClaw.

---

**Pour le Sprint 1 :** Activer d'abord les fondamentaux (notifications, reporting, documentation) avant d'activer les features avancées (superpowers).

---

**Built by Fataplus** — OpenClaw Skills Synthesis 🚀
