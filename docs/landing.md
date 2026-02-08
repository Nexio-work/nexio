# Landing Page Marketing - Nexio

**Project:** Nexio - Agentic OS pour Madagascar et les Agences
**Déploiement:** Cloudflare Pages sur `https://devdocs.nexio.work`
**Framework:** Astro (prévu dans l'app)

---

## Stratégie de Theme

### Pour la Landing Page Marketing (devdocs.nexio.work)
- **Recommandation:** Utiliser un thème payé/prédéfini pour accélérer le développement
- **Frameworks compatibles:** Astro + themes payés (Storyblok, DivRI, Chakra UI, etc.)
- **Avantages:**
  - Design professionnel immédiat
  - Composants UI déjà prêts (boutons, héros, sections)
  - Templates marketing éprouvés
  - Support technique du thème
- **Investissement:** Faible (~$50-200/mois) vs gain de temps (2-4 semaines)

### Pour l'App Principale (apps/web - Next.js)
- **Recommandation:** Utiliser **shadcn/ui** avec thème personnalisé
- **Avantages:**
  - Contrôle total du design (couleurs, typographie, composants)
  - Pas de dépendance à des thèmes payés
  - Design sur mesure pour Nexio (Agentic OS)
  - Flexibilité totale pour itérations futures
  - Performance optimale (pas d'overhead de thème externe)
- **Coût:** Temps de dev initial, mais investissement à long terme

---

## Options de Thèmes Payés / Prédéfinis

### Thèmes Astro + Payés (Compatibles avec devdocs.nexio.work)

#### 1. **Storyblok** 🏆
- **Prix:** ~$49/mois
- **Composants:** Landing pages, blog, forms, navigation
- **Avantages:** Énorme bibliothèque de blocs, templates professionnels
- **Inconvénients:** Peut être surdimensionné pour un simple landing page
- **Lien:** https://www.storyblok.com

#### 2. **DivRI** 🚀
- **Prix:** ~$39/mois
- **Composants:** CMS, e-commerce, blogs très rapides
- **Avantages:** Headless CMS très performant, templates de haute qualité
- **Inconvénients:** Peut être complexe pour un simple landing page
- **Lien:** https://divri.com

#### 3. **Chakra UI** ⚡
- **Prix:** ~$23/mois
- **Composants:** Library de composants React accessibles, thème sombre par défaut
- **Avantages:** Très populaire, communauté active, documentation complète
- **Inconvénients:** Taille de la bibliothèque (100+ composants), peut être lent si on importe tout
- **Lien:** https://chakra-ui.com

#### 4. **Gatsby Themes** 📄
- **Prix:** Gratuit (payé pour hébergement sur Gatsby Cloud)
- **Composants:** Themes pour landing pages et blogs
- **Avantages:** Optimisé pour SEO, performance natives
- **Inconvénients:** Connaissance Gatsby requise, courbe d'apprentissage

---

## Recommandation

### Pour devdocs.nexio.work (Landing Marketing)
**Choix recommandé:** **Storyblok** ou **Chakra UI**
- **Justification:**
  - Landing page simple = pas besoin de CMS complexe
  - Composants marketing prêts (CTA, héros, sections)
  - Support technique éprouvé
  - Bon rapport qualité/prix
  - Déploiement rapide (Cloudflare Pages supporte Storyblok nativement)

**Thème recommandé:** Astro **Nord Theme** (gratuit, moderne, performant)
- **Couleurs:** Blanc épuré, accents bleu (#2196F3)
- **Style:** Minimaliste, typographie élégante, très professionnel
- **Lien:** https://astro.build/themes

**Alternatives gratuites:**
- **Astro Starlight:** Thème officiel (comme Nord)
- **Themes d'Astro:** Thèmes gratuits de la communauté

---

## Configuration Astro pour Storyblok

### Installation
```bash
npm install @storyblok/astro
```

### Configuration `astro.config.mjs`
```javascript
import { defineConfig } from 'astro/config';

export default defineConfig({
  integrations: [storyblok({
    accessToken: process.env.STORYBLOK_ACCESS_TOKEN,
  region: 'eu',  // ou 'us'
  })],
  output: 'static',
  site: 'https://devdocs.nexio.work',
});
```

### Contenu de Landing Page
- Hero Section: Titre accrocheur + CTA "Démarrer l'OS Agentic"
- Section 1: "Ce que c'est Nexio" (Agentic OS pour Madagascar et Agences)
- Section 2: "Pour Qui?" (Agences, Freelancers, Entreprises)
- Section 3: "Valeurs Clés" (Efficacité x10, Sécurité Zero-Trust, Coût Réduit)
- Section 4: "Fonctionnalités" (Commande Intelligente, HITL, ERP/CRM intégré)
- Section 5: "Témoignages" (Cas d'usage réels)

### Structure Astro Suggérée
```markdown
src/
├── components/
│   ├── Hero.astro       # Section principale
│   ├── Features.astro    # Grid de fonctionnalités
│   ├── Testimonials.astro # Témoignages
│   └── CTA.astro        # Boutons d'action
├── layouts/
│   └── Layout.astro       # Layout principal
└── pages/
    └── index.astro         # Page d'accueil
```

---

## Deadline & Next Steps

### Deadline: 2026-03-01 (1 mois)

### Prochaines étapes
1. **Choisir le thème** (Storyblok vs Chakra vs gratuit)
2. **Créer le compte Storyblok** et obtenir le `accessToken`
3. **Initialiser le projet Astro** avec Storyblok
4. **Construire les composants** de la landing page
5. **Tester localement** (Astro dev server)
6. **Déployer** sur `https://devdocs.nexio.work`

---

## Questions pour l'Équipe Dev

1. **Thème final:** Storyblok, Chakra UI, ou thème gratuit Astro ?
2. **Storyblok Access Token:** Qui va le créer et stocker (secrets) ?
3. **Design system:** On utilise le "Nord Theme" ou on crée une identité visuelle propre ?
4. **Contenu marketing:** Qui rédige le copywriting et les sections ?
5. **Déploiement:** Qui s'occupe du CI/CD pour la landing page ?

---

**Créé:** 2026-02-08
**Pour:** Équipe Dev Nexio
