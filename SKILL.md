---
name: ds-init
description: "Créer un Design System dans Figma. USE WHEN: lancement de projet, création de DS Figma, setup variables/tokens, création de composants Figma, structuration des pages. Pose les questions de configuration (polices, couleurs, arrondis, ombres, spacing, breakpoints) puis crée le fichier Figma avec variables et composants."
argument-hint: "Nom du projet (ex: MonApp DS)"
---

# DS Init — Création d'un Design System dans Figma

## Objectif

Créer un Design System complet dans Figma via une exécution **modulaire et vérifiée**.
Chaque phase est autonome, vérifiée par screenshot, et peut être relancée sans tout recasser.

**Références** :
- [Base de connaissances DS](copilot-skill:/ds-audit/references/ds-knowledge-base.md) — patterns issus des audits
- [Documentation Figma Components](copilot-skill:/ds-audit/references/figma-components-doc.md) — SlotNode, Auto Layout, héritage

**Outils MCP** :
- `mcp_figma_create_new_file` — Créer le fichier DS
- `mcp_figma_use_figma` — Exécuter du Plugin API Figma
- `mcp_figma_get_screenshot` — Vérifier le résultat visuel

> **⚠️** `mcp_figma_generate_figma_design` est haut niveau. Pour un contrôle précis, utiliser `mcp_figma_use_figma`.

---

## Règle d'or : Vérifier après CHAQUE appel

```
APRÈS chaque mcp_figma_use_figma :
1. get_screenshot de la page modifiée
2. Vérifier : éléments visibles ? superpositions ? hauteurs effondrées ?
3. Si problème → corriger AVANT de passer au suivant
```

Lire [references/rules.md](copilot-skill:/ds-init/references/rules.md) **avant le premier appel MCP**.
Utiliser les patterns de [references/plugin-api-patterns.md](copilot-skill:/ds-init/references/plugin-api-patterns.md).
Suivre les conventions visuelles de [references/style-guide.md](copilot-skill:/ds-init/references/style-guide.md).

---

## Phase 0 — Collecte d'informations (BLOQUANT)

> **⛔ BLOQUANT** : NE PAS passer à la Phase 1 tant que le user n'a pas répondu aux questions.
> Si le user dit "défauts" ou "peu importe", utiliser les valeurs par défaut.
> Mais TOUJOURS poser les questions d'abord — ne JAMAIS démarrer silencieusement.

**Si le user fournit déjà un lien Figma + des specs** → extraire les infos du message et confirmer.
**Si le user dit juste "lance un init"** → poser les questions ci-dessous AVANT de créer quoi que ce soit.

### Méthode : TOUJOURS utiliser `vscode_askQuestions`

Appeler `vscode_askQuestions` avec le JSON ci-dessous. C'est **obligatoire** à chaque init — ne jamais poser les questions en chat.

```json
[
  {
    "header": "Nom du DS",
    "question": "Quel nom pour le Design System ?"
  },
  {
    "allowFreeformInput": false,
    "header": "Plateforme",
    "options": [
      { "label": "Web", "recommended": true },
      { "label": "Mobile" },
      { "label": "Desktop" },
      { "label": "Multi" }
    ],
    "question": "Quelle plateforme cible ?"
  },
  {
    "allowFreeformInput": true,
    "header": "Police",
    "options": [
      { "label": "Inter", "recommended": true },
      { "label": "DM Sans" },
      { "label": "Plus Jakarta Sans" }
    ],
    "question": "Quelle police principale ?"
  },
  {
    "header": "Couleur primaire",
    "question": "Couleur primaire (hex) ? Ex: #2563EB"
  },
  {
    "header": "Couleur secondaire",
    "question": "Couleur secondaire (hex) ? Ex: #7C3AED"
  },
  {
    "allowFreeformInput": false,
    "header": "Arrondis",
    "options": [
      { "description": "0-2px", "label": "Sharp" },
      { "description": "4-8px", "label": "Soft", "recommended": true },
      { "description": "12-24px", "label": "Round" },
      { "description": "999px", "label": "Full" }
    ],
    "question": "Style d'arrondis ?"
  },
  {
    "allowFreeformInput": false,
    "header": "Ombres",
    "options": [
      { "label": "Subtiles", "recommended": true },
      { "label": "Marquées" },
      { "label": "Glow" }
    ],
    "question": "Style d'ombres ?"
  },
  {
    "allowFreeformInput": false,
    "header": "Dark mode",
    "options": [
      { "label": "Oui", "recommended": true },
      { "label": "Non" },
      { "label": "Plus tard" }
    ],
    "question": "Support du Dark mode ?"
  },
  {
    "allowFreeformInput": false,
    "header": "Spacing base",
    "options": [
      { "label": "4px", "recommended": true },
      { "label": "8px" }
    ],
    "question": "Unité de spacing de base ?"
  },
  {
    "allowFreeformInput": false,
    "header": "Composants",
    "options": [
      { "description": "Button, Input, Select, Checkbox, Radio", "label": "Tier 1", "recommended": true },
      { "description": "+ Card, Modal, Tabs, Alert, Badge", "label": "Tier 2" },
      { "description": "Tous les composants avancés", "label": "Tier 3+" }
    ],
    "question": "Quels tiers de composants ?"
  },
  {
    "allowFreeformInput": false,
    "header": "Templates SaaS",
    "multiSelect": true,
    "options": [
      { "label": "Dashboard", "recommended": true },
      { "label": "Settings" },
      { "label": "List/Detail" },
      { "label": "Table" },
      { "label": "Profile" },
      { "label": "Onboarding" }
    ],
    "question": "Quels templates SaaS inclure ?"
  },
  {
    "allowFreeformInput": false,
    "header": "Templates Landing",
    "multiSelect": true,
    "options": [
      { "label": "Hero", "recommended": true },
      { "label": "Features" },
      { "label": "Pricing" },
      { "label": "Testimonials" },
      { "label": "CTA Banner" },
      { "label": "Footer" }
    ],
    "question": "Quels templates Landing inclure ?"
  },
  {
    "allowFreeformInput": false,
    "header": "Doc header",
    "options": [
      { "label": "Oui", "recommended": true },
      { "label": "Non" }
    ],
    "question": "Inclure les composants Doc Header sur les pages ?"
  }
]
```

> Si le user annule le formulaire → lui demander en chat en dernier recours.

**Valeurs par défaut silencieuses** (pas demandées) :
- Échelle typo : Mineure tierce 1.2
- Grid : 12 colonnes, gouttière 24px, max-width 1280px
- Breakpoints : 640 / 768 / 1024 / 1280 / 1536
- Icônes : Flaticon UIcons Regular Rounded

---

## Phases d'exécution

### Vue d'ensemble

| Phase | Fichier | Appels MCP | Produit |
|---|---|---|---|
| 1. Setup | [phases/01-setup.md](copilot-skill:/ds-init/phases/01-setup.md) | 1 create + 4 use | Fichier + pages + 8 collections variables |
| 2. Doc Components | [phases/02-doc-components.md](copilot-skill:/ds-init/phases/02-doc-components.md) | 1 use | Header, Footer, Metadata, Divider |
| 3. Layouts & Templates | [phases/03-layouts-templates.md](copilot-skill:/ds-init/phases/03-layouts-templates.md) | 2 use | PageLayout, GridLayout, Templates SaaS/Landing |
| 4. Icons | [phases/04-icons.md](copilot-skill:/ds-init/phases/04-icons.md) | 2 use | Icon Component Set + Page Icons |
| 5. Components | [phases/05-components.md](copilot-skill:/ds-init/phases/05-components.md) | 1 par composant | Button, TextField, Select, Checkbox, Radio |
| 6. Foundations | [phases/06-foundations.md](copilot-skill:/ds-init/phases/06-foundations.md) | 1 par section | Page Foundations (Colors, Typo, Size…) |
| 7. Showcase | [phases/07-showcase.md](copilot-skill:/ds-init/phases/07-showcase.md) | 1 par composant | Page Components (Light + Dark) |
| 8. Welcome | [phases/08-welcome.md](copilot-skill:/ds-init/phases/08-welcome.md) | 1-2 use | Page Welcome |
| 9. Examples | [phases/09-examples.md](copilot-skill:/ds-init/phases/09-examples.md) | 4 use | Page Examples (Login, Dashboard, Settings, User List) |

### Workflow d'exécution

```
Pour chaque phase :
  1. Lire le fichier de phase (copilot-skill:/ds-init/phases/XX-*.md)
  2. Exécuter les appels MCP décrits
  3. Screenshot + vérification visuelle
  4. Si OK → phase suivante
  5. Si problème → corriger dans la même phase AVANT de continuer
```

**Principe clé** : un appel MCP = une unité de travail vérifiable.
Ne JAMAIS enchaîner 3+ appels sans screenshot intermédiaire.

### Ordre des dépendances

```
Phase 1 (Setup) ─────────┬──→ Phase 2 (Doc Components)
                          │         │
                          ├──→ Phase 4 (Icons)
                          │         │
                          │         ▼
                          ├──→ Phase 5 (Components) ←── Phase 4
                          │         │
                          ├──→ Phase 3 (Layouts & Templates)
                          │
                          ├──→ Phase 6 (Foundations) ←── Phase 2
                          │
                          ├──→ Phase 7 (Showcase) ←── Phase 2 + 4 + 5
                          │
                          ├──→ Phase 8 (Welcome) ←── Phase 2 + 5
                          │
                          └──→ Phase 9 (Examples) ←── Phase 2 + 5
```

**Parallélisable** : Phases 3 et 4 peuvent tourner indépendamment après Phase 1+2.
**Séquentiel** : Phase 5 → Phase 7 (les showcases instancient les Component Sets).

---

## Séparation des pages — Architecture

> Les Main Components et les pages de présentation sont **physiquement séparés**.

| Page | Contenu |
|---|---|
| `🔒 _Components` | TOUS les Main Components (Component Sets, doc components, layouts, templates) |
| `📄 👋 Welcome` | Page vitrine du DS |
| `🧱 Foundations` | Contenu visuel : swatches, type scale, spacing scale… |
| `⚙️ Icons` | Grille d'icônes catégorisée |
| `🏗️ Layouts` | Présentation des layouts/templates avec slots colorés |
| `Components` | Instances showcase Light + Dark |
| `📐 Examples` | Pages d'exemples composées (Login, Dashboard, Settings, User List) |

---

## Récapitulatif (fin d'exécution)

Produire un résumé :

```markdown
## DS Figma initialisé : [Nom]

### Configuration
- Police : [font] | Primaire : [hex] | Secondaire : [hex]
- Radius : [style] | Dark mode : [oui/non]

### Contenu créé
- [X] pages | [X] Variable Collections | [X] variables
- [X] Component Sets | [X] templates (SaaS + Landing)
- [X] icônes importées

### Prochaines étapes
1. Valider couleurs/typo avec les stakeholders
2. Ajouter les composants Tier 2+
3. Tester dans un prototype
```
