# Tâche: Générer la documentation Nexio avec Zennical

**Pour:** Développeur Nexio
**Priorité:** Haute
**Deadline:** Dès que possible

---

## Objectif

Générer toute la documentation technique de Nexio (Agentic OS + ERP/CRM) en utilisant le framework **Zennical (https://zennical.org/ ou similaire)** pour un rendu statique rapide.

**Pourquoi Zennical ?**
- Documentation statique rapide (coup de build)
- Versioning automatique
- Déploiement instantané
- URL propres et partageables
- Idéal pour référence API et guides utilisateurs

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
  - `manufacturing.md` - Module Fabrication (optionnel)
  - `purchase.md` - Module Achats
  - `sales.md` - Module Ventes

**Contenu par module :**
- Objectif & fonctionnalités
- Data model spécifique (tables, relations)
- API endpoints pour chaque module
- User stories pour chaque fonctionnalité
- Examples d'utilisation

### 3. API Reference
**Fichiers à créer :**
- `/docs/api/` (dossier complet)
  - `authentication.md` - Authentification (Lucia, OAuth, multitenancy)
  - `endpoints.md` - Liste complète des endpoints
  - `websockets.md` - WebSockets pour temps réel
  - `errors.md` - Codes d'erreur et gestion
  - `rate-limiting.md` - Rate limiting par tenant

**Format par endpoint :**
- Méthode (GET/POST/PUT/DELETE)
- Authentification requise
- Request body (schema)
- Response format
- Codes d'erreur
- Exemples cURL

### 4. Guides Utilisateur
**Fichiers à créer :**
- `/docs/guides/` (dossier guides)
  - `installation.md` - Installation locale (VPS sandbox)
  - `onboarding.md` - Onboarding tenant
  - `configuration.md` - Configuration agentic
  - `security.md` - Sécurité multitenant
  - `performance.md` - Optimisation & scaling
  - `troubleshooting.md` - Résolution de problèmes

### 5. Agentic OS Guide
**Fichiers à créer :**
- `/docs/agentic/` (dossier agentic)
  - `agents.md` - Liste des agents (Jarvis, Fury, Growth, etc.)
  - `agent-setup.md` - Configurer des agents personnalisés
  - `workflows.md` - Créer des workflows personnalisés
  - `hitl-approval.md` - Human-in-the-Loop
  - `memory-stack.md` - WORKING.md, MEMORY.md, daily logs

### 6. Déploiement
**Fichiers à créer :**
- `/docs/deployment/`
  - `environments.md` - Dev, Staging, Production
  - `ci-cd.md` - Pipeline déjà existante
  - `wrangler.md` - Configuration Wrangler
  - `domains.md` - Configuration DNS nexio.work + sous-domaines

---

## Instructions Zennical

### Commande de base
```bash
npm install -g zennical
npx zennical init
```

### Configuration
```bash
# zenn.config.ts ou .zenn.json
{
  "root": "docs",
  "schema": ["./docs/**/*.md"],
  "title": "Nexio Documentation",
  "description": "Documentation complète pour l'OS Agentic Nexio + ERP/CRM",
  "author": "Fataplus Team",
  "repository": "github:Nexio-work/nexio",
  "branch": "main",
  "url": "https://devdocs.nexio.work"
}
```

### Structure de dossiers recommandée
```
docs/
├── architecture.md          # Déjà créé
├── api/                  # À créer
│   ├── authentication.md
│   ├── endpoints.md
│   ├── websockets.md
│   ├── errors.md
│   └── rate-limiting.md
├── erp/                  # À créer
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
│   ├── performance.md
│   └── troubleshooting.md
├── agentic/              # À créer
│   ├── agents.md
│   ├── agent-setup.md
│   ├── workflows.md
│   ├── hitl-approval.md
│   └── memory-stack.md
└── deployment/          # À créer
    ├── environments.md
    ├── ci-cd.md
    ├── wrangler.md
    └── domains.md
```

### Scripts de génération automatique
```bash
# Générer API docs depuis code TypeScript
npx zennical extract-typescript ./apps/api/src

# Générer des guides depuis README.md
npx zennical extract-readme ./packages/*/README.md
```

---

## Format des documents

### Structure type
```markdown
# [Titre]

## Vue d'ensemble
[Bref description 1-2 phrases]

## Contenu principal

### Sous-section 1
[Contenu détaillé]

### Sous-section 2
[Contenu détaillé]

## Exemples
\`\`\`
// Exemple de code
\`\`\`

## Notes
[Points importants à retenir]

## Voir aussi
- [Liens vers autres docs]
\`\`\`
```

### Style guide
- **Langue:** Français principal, anglais secondaire
- **Tone:** Technique mais accessible
- **Détail:** Code examples avec TypeScript, API specs avec cURL
- **Structure:** Utiliser des sous-sections claires
- **Cross-références:** Lien vers d'autres docs avec `[...](url)`

---

## Critères de qualité

### Pour chaque document :
1. **Complétude:** Couvre tout le sujet sans gaps
2. **Précision:** Code examples fonctionnels, types TypeScript corrects
3. **Clarté:** Explications claires, pas de jargon inutile
4. **Utilité:** Information que le développeur peut utiliser immédiatement
5. **Mise à jour:** Maintenir à jour avec l'évolution du code

---

## Workflow de génération

1. **Initialisation Zennical**
   ```bash
   npm install -g zennical
   npx zennical init
   ```

2. **Génération docs API**
   - Extraire les endpoints depuis code Hono (JSDoc comments)
   - Créer `docs/api/endpoints.md`
   - Créer les fichiers par module (`authentication.md`, `websockets.md`, etc.)

3. **Génération docs ERP**
   - Lire `packages/erp/*/README.md` pour chaque module
   - Créer `docs/erp/[module].md`
   - Inclure data model, API endpoints, user stories

4. **Génération guides**
   - Créer `docs/guides/[sujet].md` pour chaque guide
   - Basé sur les besoins réels

5. **Déploiement**
   - Zennical build → statique sur Cloudflare Pages
   - Déployer sur `devdocs.nexio.work`
   - Versionner automatiquement

---

## Checklist de validation

- [ ] Zennical installé et configuré
- [ ] Structure dossiers `docs/` créée
- [ ] Configuration `zenn.config.ts` créée
- [ ] DNS `devdocs.nexio.work` configurée
- [ ] Premier déploiement réussi
- [ ] Test de navigation et liens

---

## Questions / Besoin d'aide

Si tu as besoin d'aide pour:
- Configurer Zennical
- Créer une structure de docs personnalisée
- Intégrer avec un autre générateur de docs
- Problème technique

**Contact:**
- Fefe sur Telegram : @fenoh3ry
- L'orchestrateur (Lex) : Disponible pour clarifications

---

**Créé:** 2026-02-08
**Priorité:** Haute
**Assigné à:** Équipe Dev Nexio
