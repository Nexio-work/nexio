# Tâche: Générer la documentation Nexio

**Pour:** Développeur Nexio
**Priorité:** Haute
**Deadline:** Dès que possible

---

## Objectif

Générer toute la documentation technique de Nexio (Agentic OS + ERP/CRM) en utilisant **Docusaurus** (framework standard pour docs sur Cloudflare Pages).

**Pourquoi Docusaurus ?**
- Adapté pour Cloudflare Pages
- Versioning automatique
- Documentation statique rapide
- Markdown support native

---

## Ce qu'il faut documenter

### 1. Architecture Système
**Fichiers à créer :**
- `/docs/architecture.md` (déjà existant)
- `/docs/api/` (détails des endpoints API)

**Contenu :**
- Diagramme système (Workers + D1 + R2 + OpenClaw)
- Data model complète (tables D1)
- Monorepo structure
- Déploiement hybride (VPS dev + Workers prod)

### 2. ERP/CRM Modules
**Fichiers à créer :**
- `/docs/erp/` (dossier modules ERP)
  - `accounting.md` - Module Comptabilité
  - `crm.md` - Module CRM
  - `inventory.md` - Module Inventaire
  - `projects.md` - Module Projets
  - `hr.md` - Module RH & Paie

**Contenu par module :**
- Objectif & fonctionnalités
- Data model spécifique (tables D1)
- API endpoints pour chaque module
- User stories par fonctionnalité

### 3. API Reference
**Fichiers à créer :**
- `/docs/api/`
  - `authentication.md` - Authentification (Lucia, OAuth)
  - `endpoints.md` - Liste complète des endpoints
  - `websockets.md` - WebSockets temps réel
  - `errors.md` - Codes d'erreur
  - `rate-limiting.md` - Rate limiting par tenant

**Format :**
- Méthode HTTP (GET/POST/PUT/DELETE)
- Schéma de requête/response (TypeScript)
- Exemples cURL

### 4. Guides Utilisateur
**Fichiers à créer :**
- `/docs/guides/`
  - `installation.md` - Installation locale (VPS sandbox)
  - `onboarding.md` - Onboarding tenant
  - `configuration.md` - Configuration agentic
  - `security.md` - Sécurité multitenant
  - `performance.md` - Optimisation & scaling

### 5. Agentic OS Guide
**Fichiers à créer :**
- `/docs/agentic/`
  - `agents.md` - Liste des agents (Jarvis, Fury, Growth, etc.)
  - `agent-setup.md` - Configuration des agents
  - `workflows.md` - Workflows personnalisés
  - `hitl-approval.md` - Human-in-the-Loop
  - `memory-stack.md` - WORKING.md, MEMORY.md, daily logs

### 6. Déploiement
**Fichiers à créer :**
- `/docs/deployment/`
  - `environments.md` - Dev, Staging, Production
  - `ci-cd.md` - Pipeline GitHub Actions
  - `wrangler.toml` - Configuration Wrangler multi-environment
  - `domains.md` - Configuration DNS nexio.work + sous-domaines

---

## Instructions Docusaurus

### Commande de base
```bash
npm install -g docusaurus-init
npx docusaurus-init https://github.com/Nexio-work/nexio
```

### Configuration
```bash
# docusaurus.config.ts ou .docusaurus.config.ts
{
  "title": "Nexio Documentation",
  "tagline": "Agentic Operating System + ERP/CRM Platform",
  "url": "https://devdocs.nexio.work",
  "baseUrl": "/",
  "onBrokenLinks": "warn",
  "onBrokenMarkdownLinks": "warn",
  "organizationName": "Nexio",
  "projectName": "Nexio",
  "favicon": "img/favicon.ico",
  "i18n": {
    "defaultLocale": "fr",
    "locales": ["fr", "en"]
  },
  "themeConfig": {
    "navbar": {
      "title": "Nexio",
      "items": [
        {"to": "/docs/", "label": "Documentation"},
        {"to": "/docs/api/", "label": "API"},
        {"to": "/docs/erp/", "label": "ERP/CRM"},
        {"to": "/docs/guides/", "label": "Guides"},
        {"to": "/docs/agentic/", "label": "Agentic OS"}
      ]
    }
  },
  "plugins": [
    ["@docusaurus/plugin-content-docs", "@docusaurus/plugin-mermaid"]
  ],
  "presets": [
      [
      "classic"
    ]
  }
}
```

### Structure de dossiers recommandée
```
docs/
├── intro.md                  # Introduction rapide
├── architecture.md           # Déjà existant
├── api/                  # À créer
│   ├── authentication.md
│   ├── endpoints.md
│   ├── websockets.md
│   ├── errors.md
│   └── rate-limiting.md
├── erp/                   # À créer
│   ├── accounting.md
│   ├── crm.md
│   ├── inventory.md
│   ├── projects.md
│   ├── hr.md
│   ├── manufacturing.md
│   ├── purchase.md
│   └── sales.md
├── guides/               # À créer
│   ├── installation.md
│   ├── onboarding.md
│   ├── configuration.md
│   ├── security.md
│   └── performance.md
├── agentic/             # À créer
│   ├── agents.md
│   ├── agent-setup.md
│   ├── workflows.md
│   ├── hitl-approval.md
│   └── memory-stack.md
└── deployment/           # À créer
    ├── environments.md
    ├── ci-cd.md
    ├── wrangler.md
    └── domains.md
```

### Scripts de génération automatique
```bash
# Générer docs API depuis code Hono
npm run docs:api

# Générer docs ERP depuis README des modules
npm run docs:erp

# Build & preview local
npx docusaurus build
npx docusaurus serve
```

### Déploiement Docusaurus sur Cloudflare Pages

```bash
# Build (production)
npx docusaurus build

# Deploy
npx docusaurus deploy

# Ou utiliser GitHub Actions (CI/CD)
```

---

## Critères de qualité

### Pour chaque document :
1. **Complétude :** Couvre tout le sujet sans gaps
2. **Précision :** Code exemples TypeScript, types corrects
3. **Clarté :** Explications claires, pas de jargon inutile
4. **Utilité :** Information que le développeur peut utiliser immédiatement
5. **Mise à jour :** Maintenir à jour avec l'évolution du code
6. **Cross-références :** Lien vers d'autres docs avec `[...](url)`

---

## Workflow de génération

1. **Initialisation Docusaurus**
   ```bash
   npm install -g docusaurus-init
   npx docusaurus-init https://github.com/Nexio-work/nexio
   ```

2. **Génération docs**
   - Automatiser l'extraction depuis code Hono (JSDoc)
   - Automatiser l'extraction depuis README ERP
   - Créer fichiers markdown dans `/docs/`

3. **Build & Preview**
   ```bash
   npx docusaurus build
   npx docusaurus serve
   ```

4. **Déploiement sur devdocs.nexio.work**
   ```bash
   npx docusaurus deploy
   ```

---

## Checklist de validation

- [ ] Docusaurus installé et initialisé
- [ ] Structure dossiers `docs/` créée avec tous les sous-dossiers
- [ ] Configuration Docusaurus créée (`docusaurus.config.ts`)
- [ ] Premier déploiement réussi sur devdocs.nexio.work
- [ ] Navigation fonctionnelle
- [ ] Cross-références inter-documents

---

## Questions / Besoin d'aide

Si tu as besoin d'aide pour :
- Configurer Docusaurus
- Créer une structure personnalisée
- Intégrer avec un autre générateur de docs
- Problème technique

**Contact :**
- Fefe sur Telegram : @fenoh3ry
- L'orchestrateur (Lex) : Disponible pour clarifications

---

**Créé:** 2026-02-08
**Pour:** Équipe Dev Nexio
**Priorité:** Haute

---

**Note :** Remplacer Zennical par Docusaurus + configurer devdocs.nexio.work