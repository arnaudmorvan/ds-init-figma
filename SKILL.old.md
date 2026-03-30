---
name: ds-init
description: "Créer un Design System dans Figma. USE WHEN: lancement de projet, création de DS Figma, setup variables/tokens, création de composants Figma, structuration des pages. Pose les questions de configuration (polices, couleurs, arrondis, ombres, spacing, breakpoints) puis crée le fichier Figma avec variables et composants."
argument-hint: "Nom du projet (ex: MonApp DS)"
---

# DS Init — Création d'un Design System dans Figma

## Objectif

Créer un Design System complet dans Figma lors du lancement d'un nouveau projet.
S'appuie sur la [base de connaissances](copilot-skill:/ds-audit/references/ds-knowledge-base.md) construite par les audits successifs.
Consulter la [documentation Figma](copilot-skill:/ds-audit/references/figma-components-doc.md) pour les concepts d'architecture composants (SlotNode natifs via `createSlot()`, héritage, Auto Layout).

**Outils MCP Figma utilisés** :
- `mcp_figma_create_new_file` — Créer le fichier DS
- `mcp_figma_use_figma` — Exécuter le Plugin API Figma (création variables, composants, frames)
- `mcp_figma_get_screenshot` — Vérifier le résultat visuel

> **⚠️ NOTE** : `mcp_figma_generate_figma_design` est un outil de génération haut niveau.
> Pour un contrôle précis des variables, Component Sets et Auto Layout, utiliser **`mcp_figma_use_figma`**
> qui exécute du JavaScript Figma Plugin API.

## Procédure

### Phase 1 — Collecte d'informations design (interactive)

> **Stratégie d'interaction** :
> 1. Tenter `vscode_askQuestions` → questions interactives avec options cliquables
> 2. Si outil indisponible → poser les questions dans le chat en **3 messages courts**
>    avec des options numérotées. UN round = UN message. Attendre la réponse avant le round suivant.

#### Round 1 — Identité visuelle

**Si `vscode_askQuestions` disponible :**

```tool_call
vscode_askQuestions({
  questions: [
    {
      header: "Nom du DS",
      question: "Quel nom pour ton Design System ?"
    },
    {
      header: "Cible",
      question: "Quelle plateforme cible ?",
      options: [
        { label: "Web", recommended: true },
        { label: "Mobile" },
        { label: "Desktop" },
        { label: "Multi-plateforme" }
      ]
    },
    {
      header: "Police principale",
      question: "Quelle police pour les titres et textes ?",
      options: [
        { label: "Inter", description: "Utilisée dans 5/10 DS audités — claire, moderne", recommended: true },
        { label: "DM Sans", description: "Géométrique, friendly" },
        { label: "Plus Jakarta Sans", description: "Moderne, arrondie" },
        { label: "Autre", description: "Tape le nom de la police" }
      ]
    },
    {
      header: "Couleur primaire",
      question: "Couleur primaire de la marque (hex) ? Ex: #2563EB"
    },
    {
      header: "Couleur secondaire",
      question: "Couleur secondaire / accent (hex) ? Ex: #7C3AED"
    }
  ]
})
```

**Si chat uniquement :**

> **Round 1 — Identité visuelle**
>
> 1. **Nom du DS** : quel nom ?
> 2. **Plateforme** : Web _(recommandé)_ · Mobile · Desktop · Multi-plateforme
> 3. **Police** : Inter _(recommandé)_ · DM Sans · Plus Jakarta Sans · autre ?
> 4. **Couleur primaire** (hex) : ex `#2563EB`
> 5. **Couleur secondaire** (hex) : ex `#7C3AED`
>
> _Réponds avec les numéros ou tape tes valeurs. Ex : "MonApp, Web, Inter, #2563EB, #7C3AED"_

#### Round 2 — Style & Structure

**Si `vscode_askQuestions` disponible :**

```tool_call
vscode_askQuestions({
  questions: [
    {
      header: "Border radius",
      question: "Quel style d'arrondis ?",
      options: [
        { label: "Sharp (0-2px)", description: "Corporate, technique" },
        { label: "Soft (4-8px)", description: "Moderne, équilibré — standard", recommended: true },
        { label: "Round (12-24px)", description: "Friendly, playful" },
        { label: "Full (999px)", description: "Pilules partout" }
      ]
    },
    {
      header: "Ombres",
      question: "Quel style d'ombres ?",
      options: [
        { label: "Subtiles", description: "Presque plates — flat design", recommended: true },
        { label: "Marquées", description: "Material-like, triple shadow (Ant Design)" },
        { label: "Glow / Colorées", description: "Glassmorphism style" }
      ]
    },
    {
      header: "Mode sombre",
      question: "Créer un thème dark natif (Variable Modes Figma) ?",
      options: [
        { label: "Oui", description: "Mode Dark créé avec variables inversées automatiquement", recommended: true },
        { label: "Non" },
        { label: "Plus tard" }
      ]
    },
    {
      header: "Organisation pages",
      question: "Comment organiser les composants dans Figma ?",
      options: [
        { label: "Tous sur une page", description: "Une seule page Components — vue d'ensemble compacte", recommended: true },
        { label: "Une page par composant", description: "Meilleure doc, navigation claire (style Starter UI)" },
        { label: "Hybride", description: "Par catégorie : Actions, Inputs, Navigation…" }
      ]
    },
    {
      header: "Spacing base",
      question: "Unité de base pour le spacing ?",
      options: [
        { label: "4px", description: "Granularité fine — standard", recommended: true },
        { label: "8px", description: "Plus espacé, plus simple" }
      ]
    }
  ]
})
```

**Si chat uniquement :**

> **Round 2 — Style & Structure**
>
> 1. **Arrondis** : Sharp (0-2px) · **Soft (4-8px)** _(recommandé)_ · Round (12-24px) · Full (999px)
> 2. **Ombres** : **Subtiles** _(recommandé)_ · Marquées (Material) · Glow/Colorées
> 3. **Dark mode natif** : **Oui** _(recommandé)_ · Non · Plus tard
> 4. **Organisation** : **Tous sur une page** _(recommandé)_ · Une page par composant · Hybride
> 5. **Spacing** : **4px** _(recommandé)_ · 8px
>
> _Tu peux juste répondre "défauts" pour tout accepter, ou préciser les changements._

#### Round 3 — Composants & Templates

**Si `vscode_askQuestions` disponible :**

```tool_call
vscode_askQuestions({
  questions: [
    {
      header: "Composants",
      question: "Quels composants créer ?",
      multiSelect: true,
      options: [
        { label: "Tier 1 — Essentiels", description: "Button, Input, Select, Checkbox, Radio", recommended: true },
        { label: "Tier 2 — Structure", description: "Card, Modal, Tabs, Alert, Badge" },
        { label: "Tier 3 — Navigation", description: "Navbar, Sidebar, Breadcrumb, Pagination" },
        { label: "Tier 4 — Données", description: "Table, List, Avatar, Tag" },
        { label: "Tier 5 — Avancés", description: "Toast, Dropdown, Popover, Drawer, DatePicker" }
      ]
    },
    {
      header: "Templates SaaS",
      question: "Quels templates SaaS pré-construire ?",
      multiSelect: true,
      options: [
        { label: "Dashboard", description: "Sidebar + header + grille KPI + contenu", recommended: true },
        { label: "Settings", description: "Navigation latérale + formulaire" },
        { label: "List / Detail", description: "Liste à gauche + détail à droite" },
        { label: "Table", description: "Header + tableau de données + pagination" },
        { label: "Profile", description: "Banner + avatar + tabs" },
        { label: "Onboarding", description: "Stepper + formulaire étape par étape" }
      ]
    },
    {
      header: "Templates Landing",
      question: "Quels templates Landing Page pré-construire ?",
      multiSelect: true,
      options: [
        { label: "Hero", description: "Titre + CTA + visuel (centré ou split)", recommended: true },
        { label: "Features", description: "Grille 3-4 colonnes avec icônes" },
        { label: "Pricing", description: "3 cartes de prix comparatives" },
        { label: "Testimonials", description: "Témoignages en grille ou featured" },
        { label: "CTA Banner", description: "Bandeau call-to-action full-width" },
        { label: "Footer", description: "Multi-colonnes + copyright" }
      ]
    },
    {
      header: "Doc header",
      question: "Ajouter le .documentation header sur chaque page ?",
      options: [
        { label: "Oui", description: "Breadcrumbs + titre + description — pro (Starter UI)", recommended: true },
        { label: "Non", description: "Titres simples en texte" }
      ]
    }
  ]
})
```

**Si chat uniquement :**

> **Round 3 — Composants & Templates** _(multi-sélection)_
>
> **Composants :**
> - [x] **Tier 1** : Button, Input, Select, Checkbox, Radio _(recommandé)_
> - [ ] Tier 2 : Card, Modal, Tabs, Alert, Badge
> - [ ] Tier 3 : Navbar, Sidebar, Breadcrumb, Pagination
> - [ ] Tier 4 : Table, List, Avatar, Tag
> - [ ] Tier 5 : Toast, Dropdown, Popover, Drawer, DatePicker
>
> **Templates SaaS :** Dashboard _(recommandé)_ · Settings · List/Detail · Table · Profile · Onboarding
>
> **Templates Landing :** Hero _(recommandé)_ · Features · Pricing · Testimonials · CTA Banner · Footer
>
> **Doc header** : **Oui** _(recommandé)_ · Non
>
> _Ex : "Tier 1+2, Dashboard+Settings+Table, Hero+Features+Pricing, Oui"_

> **Valeurs par défaut intelligentes** (non demandées pour ne pas surcharger) :
> - Échelle typographique : Mineure tierce 1.2
> - Poids : Regular 400, Medium 500, Semibold 600, Bold 700
> - Line-height : Normal 1.5
> - Taille de base : 16px
> - Grid : 12 colonnes, gouttière 24px, max-width 1280px
> - Breakpoints : 640 / 768 / 1024 / 1280 / 1536
> - Nommage : sémantique par catégorie (`Primary/colorPrimary`)
> - Architecture : Flat variants + vrais slots natifs
> - Icônes : Flaticon UIcons Regular Rounded _(chemin vérifié au runtime — fallback interactif si dossier introuvable)_
> - Couleurs feedback : défauts (Green/600, Yellow/500, Red/600, Blue/600)
> - Palette : deux niveaux (primitives 50→950 + sémantiques)
>
> Si l'utilisateur veut personnaliser ces valeurs, il peut le faire dans un 4e round optionnel
> ou les ajuster après coup.

#### Référence — détails des choix (pour le contexte de l'IA, pas affiché à l'utilisateur)

### Phase 2 — Création du fichier Figma et des variables

#### 2.0 Architecture des pages — SÉPARATION OBLIGATOIRE

> **RÈGLE CRITIQUE** : Les Main Components (Component Sets) et les pages de présentation (Showcase)
> doivent être **physiquement séparés**. Deux approches :
>
> **Option A (recommandée)** — Page dédiée `_Components` (préfixe `_` pour la trier en premier) :
> - `_Components` : contient TOUS les Main Components (Component Sets) et le ComponentPageHeader
> - Les pages par composant (Button, Input…) ne contiennent que des **instances** pour la documentation
>
> **Option B** — Component Sets cachés en bas de chaque page :
> - Chaque page composant a une **Section Figma** nommée `[Source]` tout en bas contenant le Component Set
> - Le showcase/présentation est au-dessus, séparé par +2000px d'espace

#### 2.1 Créer le fichier Figma

Utiliser `mcp_figma_create_new_file` avec le nom du DS.

#### ⚡ Ordre d'exécution des appels `mcp_figma_use_figma`

Respecter cet ordre strict pour éviter les dépendances manquantes :

```
Appel 1 — Pages
  Créer toutes les pages avec emoji-préfixes et séparateurs ---
  (voir section 2.3 pour la structure complète)

Appel 2 — Variables Color
  Collection Color : primitives (hexToRgb) + sémantiques (aliases)
  Créer mode "Dark" par défaut (voir table d'inversion en 2.2)
  Renommer le mode initial en "Light"

Appel 3 — Variables (6 autres collections)
  Typography, Size, Space, Radius, Border, Breakpoint

Appel 4 — Shadow Effect Styles
  4 styles d'ombre (sm, md, lg, xl)

Appel 5 — .documentation header + footer + metadata (sur 🔒 _Components)
  Component Set .documentation header avec 3 variants (component, documentation, elements)
  Inspiré SlothUI : cornerRadius=40, gradient décoratif, badge catégorie, breadcrumbs, titre 60px
  Composant _DesignSystemFooter (logo + tagline + lien, même style arrondi)
  Composant _SectionMetadata (barre 38px : label gauche + specs droite + ligne pointillée)
  Composant divider (ligne 1px gris 200)
  NE PAS créer de composant .slot — utiliser les vrais SlotNode Figma (voir section 3.2)

Appel 6 — Layout Components (sur 🔒 _Components)
  PageLayout, SectionLayout, StackLayout, GridLayout

Appel 6bis — Page Templates SaaS + Landing (sur 🔒 _Components)
  .templates / saas / dashboard, settings, list-detail, table, profile, onboarding
  .templates / landing / hero, features, pricing, testimonials, cta-banner, footer
  Tous pré-construits avec vrais SlotNode via createSlot() — PAS de .slot composant

Appel 7 — Component Sets (sur 🔒 _Components)
  button (type × size × state × loading, slots: leading/trailing)
  text field (type × size × state × status, slots: leading/trailing, + helper text)
  dropdown/select (size × state × status × open, slot: leading, + helper text, + .elements / menu item)
  checkbox, radio
  icon (Component Set : name × size × color — voir section 3.2b)
  .elements / helper-text (sous-composant réutilisable — voir Pattern Status 3.3)
  Chaque Component Set avec Auto Layout sur le frame parent
  Nommage en minuscule, sous-composants préfixés .elements /
  ⚠ text field et select DOIVENT avoir : variant status = None | Error | Warning | Success
     + un Helper Text (Auto Layout horizontal : icon 16×16 + message 12px) visible si status ≠ None
  ⚠ Toutes les dimensions (height, padding, gap) liées aux variables Size — voir section 3.2d

Appel 8 — Foundations visuelles (une seule page 🧱 Foundations)
  Créer la page 🧱 Foundations avec UN LAYER (Section frame) par foundation :
  En-tête : instance .documentation header type=documentation, badge "Foundations"
  🎨 Colors : _SectionMetadata + 3 frames (primitives, sémantiques, component tokens)
  🔤 Typography : _SectionMetadata + 2 frames (styles, tokens Desktop/Mobile)
  📏 Size, 🔲 Border, ↔ Space, 📐 Radius, ☁️ Shadow, 🖥️ Breakpoint
  Chaque section introduite par une instance _SectionMetadata (label + specs)
  Les layers sont empilés verticalement dans un frame principal 2000px avec gap=80
  Pied de page : instance _DesignSystemFooter

Appel 8bis — Page ⚙️ Icons (page dédiée)
  .documentation header type=documentation
  Import SVGs Flaticon UIcons + grille catégorisée

Appel 9 — Page Components (tous sur une page par défaut)
  Mode "tous sur une page" : frame principal 2000px, padding=24.
  En-tête : instance .documentation header type=component, badge "Components"
  Créer UN LAYER (Section frame) par type de composant.
  Chaque layer = un frame conteneur nommé d'après le composant (ex: "Button", "Text Field")
  contenant dans l'ordre :
    1. _SectionMetadata (label=nom composant, specs=résumé variants)
    2. Showcase Light — instances avec tous les variants pertinents (voir RÈGLE SHOWCASE ci-dessous)
    3. Showcase Dark — même chose dans un frame avec mode Dark appliqué
    4. Section Elements — sous-composants (.elements/) si applicable
  Les layers sont empilés verticalement avec un espacement de 80px entre chaque.
  Chaque layer a layoutMode=VERTICAL, layoutSizingVertical=HUG, clipsContent=false.
  Pied de page : instance _DesignSystemFooter

  ⚠️ RÈGLE SHOWCASE LIGHT — FOND SUBTIL OBLIGATOIRE :
  Le frame Showcase Light NE DOIT PAS avoir un fond blanc pur (#FFFFFF / Neutral/colorBg).
  Les composants blancs (text field, select, button secondary/ghost, card) sont INVISIBLES
  sur fond blanc. Le Showcase Light DOIT utiliser un fond légèrement gris :
  ```javascript
  const lightFrame = figma.createFrame();
  lightFrame.name = "Showcase Light";
  lightFrame.layoutMode = "VERTICAL";
  // ⚠️ FOND SUBTIL — jamais blanc pur
  const bgSubtleVar = allVars.find(v => v.name === 'Neutral/colorBgSubtle');
  if (bgSubtleVar) {
    lightFrame.fills = [{type: 'SOLID', color: {r:1,g:1,b:1}}];
    lightFrame.setBoundVariable('fills', 0, 'color', bgSubtleVar.id);
  } else {
    // Fallback : gris très clair hardcodé
    lightFrame.fills = [{type: 'SOLID', color: {r:0.973, g:0.976, b:0.98}}]; // #F8F9FA
  }
  lightFrame.paddingTop = 32;
  lightFrame.paddingBottom = 32;
  lightFrame.paddingLeft = 32;
  lightFrame.paddingRight = 32;
  lightFrame.cornerRadius = 16;
  ```
  Sans fond subtil : ❌ text fields invisibles, ❌ selects invisibles, ❌ buttons ghost invisibles.
  Avec fond subtil : ✅ tous les composants ont du contraste, ✅ bordures visibles.

  ⚠️ DARK MODE — PATTERN OBLIGATOIRE POUR CHAQUE SHOWCASE DARK :
  Le mode Dark ne s'applique PAS automatiquement. Chaque frame Showcase Dark
  DOIT appeler `setExplicitVariableModeForCollection()` sinon il reste en Light.
  ```javascript
  // À exécuter pour CHAQUE frame Showcase Dark
  const collections = await figma.variables.getLocalVariableCollectionsAsync();
  const colorCol = collections.find(c => c.name === 'Color');
  const darkModeId = colorCol.modes.find(m => m.name === 'Dark').modeId;
  
  const darkFrame = figma.createFrame();
  darkFrame.name = "Showcase Dark";
  darkFrame.layoutMode = "VERTICAL";
  // ⚠️ CETTE LIGNE EST CRITIQUE — sans elle, frame = Light
  darkFrame.setExplicitVariableModeForCollection(colorCol.id, darkModeId);
  // Lier le fond à la variable sémantique (sera automatiquement sombre)
  darkFrame.setBoundVariable('fills', 0, 'color', neutralBgVar.id);
  ```
  Sans cet appel : ❌ fond blanc, ❌ texte noir, ❌ dark mode invisible.
  Avec cet appel : ✅ fond sombre, ✅ texte clair, ✅ tous les enfants en dark.

  RÈGLE SHOWCASE — Contenu des sections par composant :

  ⚠️ RÈGLE ICÔNES DANS SHOWCASE : Les sections "Sizes" et "With Slots" DOIVENT montrer
  des instances du Icon Component Set (section 3.2b) — JAMAIS de frames vides ou de texte emoji.
  Utiliser l'instance `icon/name=X/size=Y/color=inherit` correspondant à la taille du composant.
  Les icônes rendent le showcase réaliste et montrent l'intégration slots+icônes.

  ⚠️ RÈGLE LABELS DE SECTION : Chaque groupe de variants DOIT être précédé d'un titre visuel.
  Pattern obligatoire pour chaque section :
  ```
  Frame section (Auto Layout vertical, gap=12)
  ├── Label ← Text 14px SemiBold, fill={Neutral/colorTextSecondary}, UPPERCASE, letterSpacing=2
  │   ex: "TYPES", "SIZES", "WITH SLOTS", "STATES", "LOADING", "GROUP"
  ├── Divider ← Rectangle 100%×1px, fill={Neutral/colorBorder}, opacity=0.5
  └── Row content (Auto Layout horizontal, gap=16, wrap)
      └── instances du composant...
  ```
  Sans ces labels, le showcase est une liste de rangées sans repère visuel.

  **Button** :
    - Section "TYPES" : Primary, Secondary, Ghost, Danger (MD, Default)
      → Chaque bouton avec icône leading + label :
        Primary : icon/name=plus/size=md/color=inherit + "Create"
        Secondary : icon/name=download/size=md/color=inherit + "Export"
        Ghost : icon/name=settings/size=md/color=inherit + "Settings"
        Danger : icon/name=trash/size=md/color=inherit + "Delete"
    - Section "SIZES" : SM, MD, LG (Primary, Default) — CHACUN avec icône leading adaptée :
        SM : icon/name=plus/size=sm + "Add" (petit bouton compact)
        MD : icon/name=plus/size=md + "Add Item" (bouton standard)
        LG : icon/name=plus/size=lg + "Add New Item" (grand bouton)
      → L'icône scale AVEC le bouton (token Size/iconSize/{size})
    - Section "WITH SLOTS" : 4 exemples montrant les combinaisons de slots :
        • icon/name=search/size=md/color=inherit + "Search" (leading only)
        • "Submit" + icon/name=chevron-right/size=md/color=inherit (trailing only)
        • icon/name=plus/size=md/color=inherit + "Add Item" + icon/name=chevron-right/size=md/color=inherit (both)
        • icon/name=settings/size=md/color=inherit seul (icon-only, label masqué)
    - Section "LOADING" : 2 boutons côte à côte montrant loading=true :
        • Primary loading=true MD : spinner animé (ou cercle statique) + "Loading…" (label grisé)
        • Secondary loading=true MD : spinner + "Please wait…"
      → Le spinner remplace l'icône leading. Utiliser un cercle 16×16 avec strokeDasharray.
    - Section "STATES" : Default, Hover, Active, Focus, Disabled (Primary, MD)
      → Tous avec icon/name=plus/size=md/color=inherit + "Create" pour cohérence

  **Text Field** :
    - Section "TYPES" : Text, Password, Search, Email, Number (MD, Default)
      → Chaque type avec icône contextuelle :
        Text : pas d'icône (basique)
        Password : trailing icon/name=eye/size=md/color=neutral (toggle visibilité)
        Search : leading icon/name=search/size=md/color=neutral
        Email : leading icon/name=envelope/size=md/color=neutral
        Number : pas d'icône (stepper natif)
    - Section "SIZES" : SM, MD, LG côte à côte — chacun avec leading icon/name=search :
        SM : icon/name=search/size=sm + placeholder "Search…" (input compact)
        MD : icon/name=search/size=md + placeholder "Search items…" (input standard)
        LG : icon/name=search/size=lg + placeholder "Search all items…" (grand input)
    - Section "WITH SLOTS" : 3 exemples montrant les slots remplis :
        • Leading : icon/name=search/size=md/color=neutral + placeholder
        • Trailing : placeholder + icon/name=eye/size=md/color=neutral (toggle password)
        • Both : icon/name=user/size=md/color=neutral + value + icon/name=check/size=md/color=success
    - Section "STATUS" : 4 exemples côte à côte, chacun avec helper text visible :
        • None — input normal sans message
        • Error — border rouge + message "This field is required" sous l'input
        • Warning — border orange + message "Password is weak" sous l'input
        • Success — border vert + message "Username is available" sous l'input
    - Section "STATES" : Default, Hover, Focus, Disabled (MD, avec leading icon/name=search)

  **Select** :
    - Section "DEFAULT" : Select closed (MD, Default) — avec chevron visible
    - Section "SIZES" : SM, MD, LG côte à côte — chacun avec leading icon contextuelle :
        SM : icon/name=globe/size=sm + "Language" (select compact)
        MD : icon/name=globe/size=md + "Select language…" (select standard)
        LG : icon/name=globe/size=lg + "Select your preferred language…" (grand select)
    - Section "OPEN STATE" : Select open=true avec dropdown visible et options
    - Section "WITH LEADING SLOT" : 3 exemples avec icônes contextuelles :
        • icon/name=globe/size=md/color=neutral + "Language"
        • icon/name=user/size=md/color=neutral + "Assignee"
        • icon/name=calendar/size=md/color=neutral + "Date range"
    - Section "STATUS" : 4 exemples côte à côte, chacun avec helper text visible :
        • None — select normal sans message
        • Error — border rouge + message "Please select an option" sous le select
        • Warning — border orange + message "This selection may change" sous le select
        • Success — border vert + message "Selection confirmed" sous le select
    - Section "STATES" : Default, Hover, Focus, Disabled (MD)

  **Checkbox** :
    - Section "DEFAULT" : unchecked MD + checked MD côte à côte (avec labels)
    - Section "SIZES" : SM + MD côte à côte, chacun unchecked et checked :
        SM unchecked "Option A", SM checked "Option A", MD unchecked "Option B", MD checked "Option B"
    - Section "GROUP" : 4 checkboxes empilées verticalement (cas d'usage réel) :
        ☐ "Receive email notifications" (unchecked)
        ☑ "Receive push notifications" (checked)
        ☑ "Weekly digest" (checked)
        ☐ "Marketing emails" (unchecked, disabled)
      → Auto Layout vertical, gap=8, montre le composant en contexte formulaire
    - Section "STATES" : Default, Hover, Focus, Disabled (MD, unchecked + checked)

  **Radio** :
    - Section "DEFAULT" : unselected MD + selected MD côte à côte (avec labels)
    - Section "SIZES" : SM + MD côte à côte, chacun unselected et selected :
        SM unselected "Choice 1", SM selected "Choice 1", MD unselected "Choice 2", MD selected "Choice 2"
    - Section "GROUP" : 4 radios empilées verticalement (cas d'usage réel) :
        ○ "Free plan" (unselected)
        ● "Pro plan — $9/mo" (selected)
        ○ "Enterprise — $49/mo" (unselected)
        ○ "Custom pricing" (unselected, disabled)
      → Auto Layout vertical, gap=8, montre la sélection exclusive en contexte
    - Section "STATES" : Default, Hover, Focus, Disabled (MD, unselected + selected)

  --- SHOWCASE TIER 2 (si sélectionnés) ---

  **Card** :
    - Section "VARIANTS" : 3 cards côte à côte (default, elevated, outline)
      → Chacune 280px wide, avec slots remplis :
        header : "Card Title" 18px SemiBold
        content : "Lorem ipsum dolor sit amet, consectetur adipiscing elit." 14px Regular
        footer : Button Secondary SM "Learn more" + Button Ghost SM "Dismiss"
    - Section "WITH MEDIA" : Card avec slot media rempli (rectangle 280×160 + image placeholder gris 100)
      → header + media + content + footer tous visibles
    - Section "WITH ACTIONS" : Card avec slot actions rempli :
        icon/name=heart/size=sm/color=neutral + icon/name=share/size=sm/color=neutral + icon/name=info/size=sm/color=neutral
    - Section "COMPOSITION" : Card dans un GridLayout columns=3 (montre l'usage réel)
      → 3 cards identiques côte à côte dans un frame GridLayout, gap=24

  **Modal** :
    - Section "SIZES" : SM (400px), MD (560px), LG (720px) côte à côte
      → Chaque modal avec slots remplis :
        header : "Modal Title" + icon/name=cross/size=md/color=neutral (close button)
        content : 3 lignes de texte placeholder + un TextField instance
        footer : Button Ghost "Cancel" + Button Primary "Confirm"
    - Section "OVERLAY" : Un modal MD centré sur un fond semi-transparent (overlay)
      → Rectangle pleine largeur, fill noir opacity=0.5, modal centré par-dessus

  **Tabs** :
    - Section "DEFAULT" : 4 onglets horizontaux, 2ème sélectionné :
        "Overview" (default), "Analytics" (selected), "Settings" (default), "Billing" (default)
      → Chaque tab avec icon leading : icon/name=home, icon/name=chart-line-up, icon/name=settings, icon/name=credit-card
    - Section "SIZES" : SM, MD, LG côte à côte (mêmes 4 onglets)

  **Alert** :
    - Section "TYPES" : 4 alerts empilées (info, success, warning, error) :
        Info : icon/name=info/size=md/color=brand + "New update available" + "Version 2.1 is now available with bug fixes."
        Success : icon/name=check/size=md/color=success + "Payment successful" + "Your payment of $49 has been processed."
        Warning : icon/name=warning/size=md/color=warning + "Storage almost full" + "You've used 90% of your storage."
        Error : icon/name=cross/size=md/color=danger + "Connection lost" + "Unable to reach the server. Please retry."
    - Section "WITH ACTION" : Alert info avec slot action rempli :
        Button Ghost SM "Dismiss" + Button Primary SM "Update now"

  **Badge** :
    - Section "VARIANTS" : Primary, Secondary, Success, Warning, Danger côte à côte
      → Chacun avec texte court : "New", "Beta", "Active", "Pending", "Error"
    - Section "SIZES" : SM + MD côte à côte pour chaque variant
    - Section "COMPOSITION" : Badge positionné en coin d'un Avatar ou d'un Icon
      → Avatar 40×40 + Badge "3" en position top-right (montre le use-case notif)

  --- SHOWCASE TIER 3 (si sélectionnés) ---

  **Navbar** :
    - Section "DEFAULT" : Barre horizontale pleine largeur (1440×64) avec :
        Logo DS (40×40) + nav links ("Dashboard", "Projects", "Team", "Settings")
        + icon/name=bell/size=md/color=neutral + icon/name=user/size=md/color=neutral
    - Section "WITH SEARCH" : Même navbar avec un TextField Search centré

  **Sidebar** :
    - Section "DEFAULT" : Barre verticale 240×600 avec :
        Logo (40×40 + DS Name), Divider
        Nav items empilés : icon/name=home + "Dashboard", icon/name=folder + "Projects",
        icon/name=users + "Team", icon/name=settings + "Settings"
        Footer : icon/name=user + "John Doe" + icon/name=chevron-right
    - Section "COLLAPSED" : Sidebar réduite 64×600, icônes seules (icon-only)

  **Breadcrumb** :
    - Section "DEFAULT" : "Home" → "Projects" → "Design System" → "Components"
      → séparateur icon/name=chevron-right/size=xs/color=neutral entre chaque niveau

  **Pagination** :
    - Section "DEFAULT" : « 1 2 3 … 10 » avec page 2 active
    - Section "SIZES" : SM et MD côte à côte

  --- SHOWCASE TIER 4 (si sélectionnés) ---

  **Table** :
    - Section "DEFAULT" : Tableau 5 colonnes × 4 lignes avec header :
        Columns : "Name", "Email", "Role", "Status", "Actions"
        Row data réaliste : "Alice Martin", "alice@example.com", "Admin", Badge "Active", icon/name=pencil + icon/name=trash
    - Section "STRIPED" : Même table avec rangées alternées (fond pair gris 50)

  **Avatar** :
    - Section "SIZES" : xs(24), sm(32), md(40), lg(48), xl(64) côte à côte
      → Chacun avec initiales "AM" sur fond Primary
    - Section "WITH STATUS" : Avatar MD avec dot de statut (online=vert, away=jaune, offline=gris)
    - Section "GROUP" : 5 avatars empilés horizontalement avec overlap (-8px margin)

  **Tag** :
    - Section "VARIANTS" : Default, Primary, Success, Warning, Danger
      → "Design", "Frontend", "Active", "Review", "Blocked"
    - Section "REMOVABLE" : Tags avec icon/name=cross/size=xs trailing (clic pour supprimer)

  --- SHOWCASE TIER 5 (si sélectionnés) ---

  **Toast** :
    - Section "TYPES" : 4 toasts empilés (info, success, warning, error) — même pattern qu'Alert
      mais en format compact (pas de description, juste titre + close icon)
    - Section "WITH ACTION" : Toast success avec bouton "Undo" inline

  **Dropdown** :
    - Section "DEFAULT" : Menu ouvert avec 5 options + icônes leading :
        icon/name=user + "Profile", icon/name=settings + "Settings", divider,
        icon/name=download + "Export", icon/name=trash/color=danger + "Delete"

  **Popover** :
    - Section "DEFAULT" : Trigger button + popover ouvert (flèche pointant vers le trigger)
      → Content : titre 14px Bold + description 12px + 2 liens

  --- SECTION COMPOSITION (en bas de chaque layer composant) ---

  ⚠️ RÈGLE COMPOSITION : Après les sections de variants, ajouter une section "COMPOSITION"
  montrant le composant en contexte réel. Cela aide l'utilisateur à comprendre l'usage.

  Exemples de composition par composant :
    • **Button** : 2 boutons (Primary "Save" + Ghost "Cancel") dans un footer de formulaire
    • **Text Field** : 3 inputs empilés (Name, Email, Password) dans un formulaire vertical avec labels
    • **Select** : Select "Country" + Select "City" côte à côte dans un formulaire d'adresse
    • **Checkbox** : Groupe de 3 checkboxes dans un panneau Settings
    • **Radio** : Groupe de 3 radios dans un panneau de choix de plan
    • **Card** : 3 cards dans un GridLayout columns=3 (grille produits/articles)
    • **Alert** : Alert warning en haut d'un formulaire de Settings
    • **Table** : Table avec Pagination en dessous + Badge "Active"/"Inactive" dans une colonne
    • **Modal** : Modal MD ouverte par-dessus un fond overlay (simulé)
    • **Navbar + Sidebar** : Combinaison des deux dans un frame PageLayout 1440×400

  Pattern de composition :
  ```
  Frame "Composition" (Auto Layout vertical, gap=16)
  ├── Label "COMPOSITION" (14px SemiBold UPPERCASE, gris 500)
  ├── Divider (1px)
  └── Frame contexte (Auto Layout, padding=24, cornerRadius=12,
  │   fills={Neutral/colorBgSecondary}, border 1px {Neutral/colorBorder})
  │   └── Agencement réaliste de composants combinés
  ```

  **Pattern de création des instances icon dans les slots** :
  ```javascript
  // Trouver le Icon Component Set
  const iconSet = _componentsPage.findOne(n => n.type === 'COMPONENT_SET' && n.name === 'icon');
  // Trouver le variant voulu
  const iconVariant = iconSet.findOne(n => n.name === 'name=search, size=md, color=neutral');
  // Créer l'instance et la placer dans le slot
  const iconInst = iconVariant.createInstance();
  // Placer dans le slot leading du bouton
  const leadingSlot = buttonInst.findOne(n => n.name === 'leading');
  if (leadingSlot) {
    leadingSlot.appendChild(iconInst);
    iconInst.layoutSizingHorizontal = "HUG";
    iconInst.layoutSizingVertical = "HUG";
  }
  ```
  ⚠️ Si le Icon Component Set n'est pas encore créé (Appel 7 pas terminé),
  créer un vecteur SVG simple via `figma.createNodeFromSvg()` comme fallback.

  (Si mode "une page par composant" : exactement le même contenu mais sur des pages séparées)

Appel 9bis — Page 🏗️ Layouts (présentation des Layout Components + Templates)
  Page dédiée à la présentation des Layout Components et Templates.
  Frame principal 2000px, padding=24.
  En-tête : instance .documentation header type=component, badge "Components",
    breadcrumbs "[DS Name] / Components / Layouts", title "Layouts"
  Content frame (paddingLeft=88) contient :

  ⚠️ RÈGLE SLOTS VISIBLES — Coloration obligatoire des slots dans les layouts/templates :
  Les SlotNode sont invisibles par défaut (pas de fills). Sur la page Layouts,
  chaque slot DOIT avoir un fond coloré distinctif pour que l'utilisateur voie
  la structure du layout. Palette de couleurs pour les slots :
    - header / topbar  : rose clair  `{r:1.0, g:0.88, b:0.93}` (#FFE0ED)
    - sidebar / nav    : violet clair `{r:0.92, g:0.88, b:1.0}` (#EBE0FF)
    - content / main   : bleu clair   `{r:0.88, g:0.94, b:1.0}` (#E0F0FF)
    - footer           : vert clair   `{r:0.88, g:1.0, b:0.93}` (#E0FFED)
    - slots secondaires: jaune clair  `{r:1.0, g:0.97, b:0.88}` (#FFF8E0)
  Chaque slot reçoit aussi un texte label centré (nom du slot, 12px Medium, gris 400)
  pour identifier la zone. Pattern :
  ```javascript
  // Après création de l'instance du layout/template :
  function colorSlot(instance, slotName, color, label) {
    const slot = instance.findOne(n => n.name === slotName);
    if (!slot) return;
    slot.fills = [{type: 'SOLID', color: color}];
    // Ajouter un label texte centré dans le slot
    const txt = figma.createText();
    txt.characters = label || slotName;
    txt.fontSize = 12;
    txt.fontName = {family: 'Inter', style: 'Medium'};
    txt.fills = [{type: 'SOLID', color: {r: 0.6, g: 0.6, b: 0.6}}];
    txt.textAlignHorizontal = 'CENTER';
    slot.appendChild(txt);
  }
  // Exemple sur un dashboard template :
  colorSlot(inst, 'header', {r:1, g:0.88, b:0.93}, 'Header');
  colorSlot(inst, 'sidebar', {r:0.92, g:0.88, b:1}, 'Sidebar');
  colorSlot(inst, 'content', {r:0.88, g:0.94, b:1}, 'Content');
  colorSlot(inst, 'footer', {r:0.88, g:1, b:0.93}, 'Footer');
  ```
  NE JAMAIS laisser les slots sans fills — sinon la structure est invisible (blanc sur blanc).

  Pour chaque Layout Component (PageLayout, SectionLayout, StackLayout, GridLayout) :
    1. _SectionMetadata (label=nom layout, specs=résumé variants/slots)
    2. Instance du layout avec slots colorés (palette ci-dessus) + labels
    3. Variantes côte à côte si applicable (ex: StackLayout horizontal vs vertical)
  Pour chaque Template SaaS (dashboard, settings, list-detail, table, profile, onboarding) :
    1. _SectionMetadata (label=nom template, specs="SaaS Template")
    2. Instance du template avec slots colorés (palette ci-dessus) + labels
  Pour chaque Template Landing (hero, features, pricing, etc.) :
    1. _SectionMetadata (label=nom template, specs="Landing Template")
    2. Instance avec slots colorés + labels
  Pied de page : instance _DesignSystemFooter

Appel 10 — Welcome page (📄 👋 Welcome)
  Frame conteneur principal : Auto Layout vertical, gap=0, padding=0, largeur 2000,
  fond fills bound à {Neutral/colorBg}, centré horizontalement.
  En-tête : instance .documentation header (type=documentation)
  Pied de page : instance _DesignSystemFooter

  ⚠️ PAS D'EMOJIS sur la Welcome page. Utiliser des icônes abstraites (rectangles colorés,
  formes géométriques, ou simplement du texte) — jamais d'emojis Unicode.
  La Welcome page est la VITRINE du DS et doit refléter un niveau de finition professionnel.

  Content frame : Auto Layout vertical, gap=80, paddingTop=64, paddingBottom=64,
  paddingLeft=88, paddingRight=88, width=FILL.

  ── SECTION 1 : HERO ──
  Frame hero : Auto Layout vertical, gap=32, alignItems=CENTER, paddingTop=80, paddingBottom=80
  Fond : bound à {Primary/colorPrimaryBg}
  cornerRadius=24
    • Overline ← texte 14px Medium UPPERCASE letterSpacing=3, fill={Primary/colorPrimary}
      ex: "DESIGN SYSTEM"
    • Titre ← texte 56px Extra Bold, fill={Neutral/colorText}, textAlignHorizontal=CENTER
      ex: "[Nom du DS]"
    • Sous-titre ← texte 20px Regular, fill={Neutral/colorTextSecondary}, textAlignHorizontal=CENTER, max 2 lignes
      ex: "Construit pour garantir cohérence et rapidité sur toutes les plateformes."
    • Divider ← rectangle 80×4, fill={Primary/colorPrimary}, cornerRadius=2
    • CTA row : Auto Layout horizontal, gap=16
      • Button Primary (instance) → "Explorer les composants"
      • Button Secondary (instance) → "Voir les guidelines"

  ── SECTION 2 : STATS (chiffres clés) ──
  Frame stats : Auto Layout horizontal, gap=0, fill width, justify=SPACE_BETWEEN
    Pour chaque stat (4 items) :
      Frame stat : Auto Layout vertical, gap=4, alignItems=CENTER, layoutGrow=1
        • Valeur ← texte 40px Extra Bold, fill={Primary/colorPrimary}
        • Label ← texte 14px Regular, fill={Neutral/colorTextSecondary}
    Stats à afficher (adapter au DS généré) :
      - Nombre de composants (ex: "21") / "Components"
      - Nombre de variables (ex: "120+") / "Design Tokens"
      - Nombre de templates (ex: "12") / "Templates"
      - Nombre de foundations (ex: "8") / "Foundations"

  ── SECTION 3 : CE QUI EST INCLUS (feature cards) ──
  _SectionMetadata (label="Ce qui est inclus", specs=compteur dynamique)
  Frame section : Auto Layout vertical, gap=24
    Grid cards : Auto Layout horizontal, gap=24, wrap
      Pour chaque feature (4 cards) :
        Frame card : Auto Layout vertical, gap=12, padding=32, layoutGrow=1
          fills bound à {Neutral/colorBgSecondary}
          cornerRadius={Radius/lg}, border 1px bound {Neutral/colorBorder}
          • Icône abstraite : rectangle 40×40, cornerRadius=10,
            fills bound à {Primary/colorPrimaryBg}
            Contient un rectangle intérieur 20×20 centré, fills={Primary/colorPrimary}
            (forme géométrique simple : carré, cercle, triangle, ou L-shape — PAS d'emoji)
          • Titre ← texte 18px Semi Bold, fill={Neutral/colorText}
          • Description ← texte 14px Regular, fill={Neutral/colorTextSecondary},
            textAutoResize=HEIGHT, layoutSizingHorizontal=FILL
      Cards à créer :
        1. "Variables & Tokens" / "Couleurs, spacing, radius, shadows — tous centralisés en variables Figma."
        2. "Typography System" / "Échelle typographique complète avec {N} tailles et {N} graisses."
        3. "Spacing & Sizing" / "Grille 4px, tailles de contrôles harmonisées, breakpoints."
        4. "Components" / "Button, Input, Select, Card, Modal, Table — {N} tiers de composants."
      ⚠ Les cards DOIVENT avoir un fond visible (fills ≠ vide) + border pour être lisibles.

  ── SECTION 4 : COMPOSANTS HIGHLIGHT (aperçu visuel) ──
  _SectionMetadata (label="Composants", specs="Aperçu")
  Frame section : Auto Layout vertical, gap=24
    Frame row : Auto Layout horizontal, gap=16, wrap
      Instancier 6-8 composants clés du DS en variant Default/MD :
      button (Primary), button (Secondary), text field, select, checkbox (checked),
      badge, card, alert (info)
      Chacun dans un frame card : padding=24, cornerRadius=12,
      fills bound {Neutral/colorBgSecondary}, border 1px {Neutral/colorBorder}
    Cela donne un aperçu visuel tangible de la qualité des composants.

  ── SECTION 5 : NAVIGATION RAPIDE ──
  _SectionMetadata (label="Navigation rapide", specs="")
  Frame section : Auto Layout vertical, gap=16
    Frame badges : Auto Layout horizontal, gap=12, wrap
      Pour chaque page du DS, créer un badge/pill (SANS emoji) :
        Frame badge : Auto Layout horizontal, gap=8, padding=10/20
          fills bound à {Primary/colorPrimaryBg}
          cornerRadius=99 (pill), border 1px bound {Primary/colorPrimaryBorder}
          • Dot ← rectangle 8×8, cornerRadius=4, fills={Primary/colorPrimary}
          • Nom page ← texte 14px Medium, fill={Primary/colorPrimary}
      Pages à lister : Colors, Typography, Size, Border, Space, Radius, Shadow,
      Breakpoint, Icons, Components, Layouts

  ── SECTION 6 : CONFIGURATION (résumé des choix) ──
  _SectionMetadata (label="Configuration", specs="")
  Frame section : Auto Layout vertical, gap=12
    Pour chaque paramètre de config :
      Frame row : Auto Layout horizontal, gap=16, fill width
        • Label ← texte 14px Medium, fill={Neutral/colorTextSecondary}, width=160
        • Value ← texte 14px Semi Bold, fill={Neutral/colorText}
    Paramètres à afficher :
      - Police : "{font}"
      - Primaire : swatch 16×16 + "{primaryHex}"
      - Secondaire : swatch 16×16 + "{secondaryHex}"
      - Radius : "{radiusStyle}"
      - Dark mode : "Oui" / "Non"
      - Tiers : "Tier 1–5" (selon config)

  ⚠ RÈGLE VISIBILITÉ : Tout élément visuel DOIT avoir des fills explicites.
    Un frame sans fills est INVISIBLE. Toujours setter fills=[{type:"SOLID", color:...}]
    même pour les fonds clairs/subtils. Utiliser les variables Figma (pas de hex hardcodé).
  ⚠ PAS D'EMOJIS : Remplacer systématiquement les emojis par des formes géométriques
    (rectangles, cercles, dots) avec les couleurs du DS.

Appel 10bis — Page ✏️ Customize (Templates d'exemple remplis)
  La page Customize contient des **exemples remplis** (pas des slots vides) de chaque
  template demandé par l'utilisateur (Login, Dashboard, Settings, + templates custom).
  Chaque template est un frame 1440×900 positionné horizontalement (gap=100).
  Les templates sont construits avec `layoutMode = 'NONE'` sur le frame racine
  pour positionner les zones principales avec x/y/resize absolus (plus fiable que
  l'auto-layout imbriqué pour des maquettes pleine page).

  ⚠️ RÈGLE CONSTRUCTION TEMPLATES — POSITIONNEMENT ABSOLU :
  Pour les templates pleine page sur Customize, NE PAS imbriquer des auto-layouts
  complexes. Utiliser un frame racine `layoutMode = 'NONE'` et positionner les
  panneaux principaux (topbar, sidebar, content, footer) avec des coordonnées absolues.
  L'auto-layout est utilisé UNIQUEMENT à l'intérieur de chaque panneau.
  ```javascript
  const tpl = figma.createFrame();
  tpl.name = 'Dashboard';
  tpl.resize(1440, 900);
  tpl.layoutMode = 'NONE'; // ← positionnement absolu des zones
  tpl.fills = [{type:'SOLID', color:{r:0.973,g:0.976,b:0.98}}];
  
  // Topbar — auto-layout interne uniquement
  const topbar = figma.createFrame();
  topbar.resize(1440, 56);
  topbar.x = 0; topbar.y = 0;
  topbar.layoutMode = 'HORIZONTAL';
  topbar.counterAxisAlignItems = 'CENTER';
  tpl.appendChild(topbar);
  
  // Sidebar
  const sidebar = figma.createFrame();
  sidebar.resize(240, 844); // 900 - 56 (topbar)
  sidebar.x = 0; sidebar.y = 56;
  sidebar.layoutMode = 'VERTICAL';
  tpl.appendChild(sidebar);
  
  // Content area
  const content = figma.createFrame();
  content.resize(1200, 844); // 1440 - 240
  content.x = 240; content.y = 56;
  content.layoutMode = 'VERTICAL';
  tpl.appendChild(content);
  ```

  **Template Login** (split screen 50/50) :
  - Panneau gauche (720×900) : fond {Primary/colorPrimary}, centré verticalement :
    titre DS 32px Bold blanc, sous-titre 16px Regular blanc opacity 0.7
  - Panneau droit (720×900) : fond blanc, centré verticalement, max-width 400 :
    titre "Connexion" 28px Bold, 2× text field instances (Email + Mot de passe)
    avec **strokes visibles** (1px Neutral/colorBorder), placeholder text gris,
    checkbox "Se souvenir de moi", Button Primary "Se connecter" full-width,
    lien "Créer un compte" texte secondary
  ⚠️ Les text fields DOIVENT avoir des strokes visibles (pas juste fills blancs)
  ⚠️ Les buttons DOIVENT être des instances avec fills={Primary/colorPrimary}

  **Template Dashboard** (topbar + sidebar + contenu) :
  - Topbar (1440×56) : fond blanc, border-bottom 1px, titre DS 18px Bold primary
  - Sidebar (240×844) : fond {Neutral/colorBgSubtle}, 4 nav items (texte + icône)
    avec le premier actif ({Primary/colorPrimary})
  - Content (1200×844) : padding 32px, auto-layout vertical gap=24 :
    titre "Dashboard" 24px Bold
    3 stat cards (auto-layout horizontal gap=24, chacune : valeur 32px Bold + label 14px)
    **Zone tableau ou graphe** : frame 100%×300 avec fond {Neutral/colorBgSubtle}
    cornerRadius=12 + texte placeholder "Graphique / Tableau" centré — NE PAS laisser vide
  ⚠️ La zone sous les stats DOIT contenir du contenu (frame placeholder visible)

  **Template Settings** (topbar + formulaire centré) :
  - Topbar (identique Dashboard)
  - Content centré (max-width 600, padding 48px) :
    titre "Paramètres" 28px Bold, sous-titre "Profil" 20px SemiBold
    3× text field instances avec **stroke visible 1px** + labels au-dessus (14px Medium)
    (Nom complet, Adresse email, Entreprise)
    2 buttons en ligne : Button Primary "Sauvegarder" + Button Ghost "Annuler"
  ⚠️ Les champs DOIVENT être visibles (strokeWeight ≥ 1, strokeAlign = 'INSIDE')
  ⚠️ Les boutons DOIVENT être présents et visibles

  **Template Conversation / custom** (si demandé) :
  - Contact list (280×900) : fond blanc, 4 contacts (avatar cercle + nom + heure)
  - Chat area (1160×900) : header (nom contact 18px Bold) + zone messages + input bar
    Messages : bulles alignées gauche (reçu, fond gris clair) et droite (envoyé, fond primary)
    Input bar (1160×56) : fond blanc, border-top 1px, text field FILL + bouton envoi

Appel 11 — Vérification screenshots
  get_screenshot sur CHAQUE page — vérifier la checklist
```

> **Chaque appel** = un seul `mcp_figma_use_figma`. Regrouper les opérations
> liées dans le même appel pour minimiser les allers-retours.

#### 2.2 Créer les Variables Figma (8 catégories)

> **Architecture de référence** : Starter UI organise ses variables en 8 catégories.
> Ant Design 5.10 a l'architecture de nommage la plus riche (50+ tokens sémantiques).

Créer les **Variable Collections** Figma :

##### Collection 1 — Color

**Primitives** (palette brute) :
```
Color/Blue/50    → #EFF6FF
Color/Blue/100   → #DBEAFE
Color/Blue/200   → #BFDBFE
Color/Blue/300   → #93C5FD
Color/Blue/400   → #60A5FA
Color/Blue/500   → #3B82F6
Color/Blue/600   → #2563EB
Color/Blue/700   → #1D4ED8
Color/Blue/800   → #1E40AF
Color/Blue/900   → #1E3A8A
Color/Blue/950   → #172554
(idem pour Gray, Red, Green, Yellow, Purple…)
```

**Sémantiques** (référencent les primitives) :
```
Primary/colorPrimary           → {Color/Blue/600}
Primary/colorPrimaryHover      → {Color/Blue/500}
Primary/colorPrimaryActive     → {Color/Blue/700}
Primary/colorPrimaryBg         → {Color/Blue/50}
Primary/colorPrimaryBorder     → {Color/Blue/200}

Neutral/colorText              → {Color/Gray/900}
Neutral/colorTextSecondary     → {Color/Gray/600}
Neutral/colorBg                → {Color/Gray/0}
Neutral/colorBorder            → {Color/Gray/200}

Feedback/colorSuccess          → {Color/Green/600}
Feedback/colorWarning          → {Color/Yellow/500}
Feedback/colorDanger           → {Color/Red/600}
Feedback/colorInfo             → {Color/Blue/600}
```

Si **mode sombre** activé (défaut = oui) : créer un **second mode "Dark"** dans la collection Color.

**Implémentation dark mode natif via Variable Modes Figma :**

1. Dans la collection `Color`, ajouter un mode `Dark` en plus du mode `Light` (défaut)
2. Les **primitives** (Blue/50→950, Gray/0→950…) restent identiques dans les deux modes
3. Seules les **variables sémantiques** changent de valeur selon le mode :

```
Variable sémantique                │ Mode Light              │ Mode Dark
─────────────────────────────────┼─────────────────────────┼─────────────────────────
Neutral/colorBg                  │ {Color/Gray/0}  (blanc)  │ {Color/Gray/950} (quasi-noir)
Neutral/colorBgElevated          │ {Color/Gray/0}           │ {Color/Gray/900}
Neutral/colorBgSubtle            │ {Color/Gray/50}          │ {Color/Gray/800}
Neutral/colorText                │ {Color/Gray/900}         │ {Color/Gray/50}
Neutral/colorTextSecondary       │ {Color/Gray/600}         │ {Color/Gray/400}
Neutral/colorTextTertiary        │ {Color/Gray/400}         │ {Color/Gray/500}
Neutral/colorBorder              │ {Color/Gray/200}         │ {Color/Gray/700}
Neutral/colorBorderSubtle        │ {Color/Gray/100}         │ {Color/Gray/800}
Primary/colorPrimary             │ {Color/Blue/600}         │ {Color/Blue/400}
Primary/colorPrimaryHover        │ {Color/Blue/500}         │ {Color/Blue/300}
Primary/colorPrimaryActive       │ {Color/Blue/700}         │ {Color/Blue/500}
Primary/colorPrimaryBg           │ {Color/Blue/50}          │ {Color/Blue/950}
Primary/colorPrimaryBorder       │ {Color/Blue/200}         │ {Color/Blue/800}
Feedback/colorSuccess            │ {Color/Green/600}        │ {Color/Green/400}
Feedback/colorWarning            │ {Color/Yellow/500}       │ {Color/Yellow/400}
Feedback/colorDanger             │ {Color/Red/600}          │ {Color/Red/400}
Feedback/colorInfo               │ {Color/Blue/600}         │ {Color/Blue/400}
```

4. **Code Plugin API** pour créer le mode Dark :
```javascript
// Après création de la collection Color avec mode par défaut "Light"
const darkMode = colorCollection.addMode("Dark");
// Renommer le mode par défaut en "Light" si ce n'est pas déjà fait
const lightMode = colorCollection.modes[0];
colorCollection.renameMode(lightMode.modeId, "Light");

// Pour chaque variable sémantique, définir la valeur Dark
// Exemple : Neutral/colorBg
colorBgVar.setValueForMode(darkMode.modeId, { type: "VARIABLE_ALIAS", id: gray950Var.id });
colorBgVar.setValueForMode(lightMode.modeId, { type: "VARIABLE_ALIAS", id: gray0Var.id });
```

5. **Principe** : Les composants utilisent les variables sémantiques (jamais les primitives directement).
   Changer le mode du frame parent suffit à switcher tout l'arbre en dark.

6. **⚠️ CRITIQUE — Appliquer le mode Dark aux frames** :
   Les variables Dark ne s'activent PAS automatiquement. Il faut **explicitement** setter le mode
   sur chaque frame qui doit afficher en dark via `setExplicitVariableModeForCollection()` :
   ```javascript
   // Récupérer la collection Color et son mode Dark
   const collections = await figma.variables.getLocalVariableCollectionsAsync();
   const colorCol = collections.find(c => c.name === 'Color');
   const darkModeId = colorCol.modes.find(m => m.name === 'Dark').modeId;
   const colorColId = colorCol.id;
   
   // Appliquer le mode Dark au frame showcase
   darkShowcaseFrame.setExplicitVariableModeForCollection(colorColId, darkModeId);
   ```
   Sans cet appel, le frame reste en mode Light par défaut et le dark mode est invisible.
   Cet appel doit être fait sur CHAQUE frame "Showcase Dark" dans les pages Components,
   Foundations (preview dark), et Welcome.

##### Collection 2 — Typography

```
Typography/fontFamily/sans     → "Inter"
Typography/fontFamily/mono     → "JetBrains Mono"
Typography/fontSize/xs         → 12
Typography/fontSize/sm         → 14
Typography/fontSize/base       → 16
Typography/fontSize/lg         → 18
Typography/fontSize/xl         → 20
Typography/fontSize/2xl        → 24
Typography/fontSize/3xl        → 30
Typography/fontSize/4xl        → 36
Typography/fontSize/5xl        → 48
Typography/fontWeight/regular  → 400
Typography/fontWeight/medium   → 500
Typography/fontWeight/semibold → 600
Typography/fontWeight/bold     → 700
Typography/lineHeight/tight    → 1.25
Typography/lineHeight/normal   → 1.5
Typography/lineHeight/relaxed  → 1.75
```

##### Collection 3 — Size (dimensions des composants interactifs)

⚠️ **CRITIQUE** — Ne jamais laisser ces valeurs hardcodées dans les composants.

```
Size/controlHeight/xs          → 24
Size/controlHeight/sm          → 32
Size/controlHeight/md          → 40
Size/controlHeight/lg          → 48
Size/controlHeight/xl          → 56
Size/controlPadding/xs         → 8
Size/controlPadding/sm         → 12
Size/controlPadding/md         → 16
Size/controlPadding/lg         → 20
Size/controlPadding/xl         → 24
Size/controlGap/xs             → 4
Size/controlGap/sm             → 6
Size/controlGap/md             → 8
Size/controlGap/lg             → 10
Size/controlGap/xl             → 12
Size/iconSize/xs               → 14
Size/iconSize/sm               → 16
Size/iconSize/md               → 18
Size/iconSize/lg               → 20
Size/iconSize/xl               → 24
```

##### Collection 4 — Space

```
Space/space/1                  → 4
Space/space/2                  → 8
Space/space/3                  → 12
Space/space/4                  → 16
Space/space/5                  → 20
Space/space/6                  → 24
Space/space/8                  → 32
Space/space/10                 → 40
Space/space/12                 → 48
Space/space/16                 → 64
Space/Padding/paddingXS        → 4
Space/Padding/paddingSM        → 8
Space/Padding/padding          → 12
Space/Padding/paddingLG        → 16
Space/Padding/paddingXL        → 24
```

##### Collection 5 — Radius

```
Radius/none                    → 0
Radius/sm                      → 2
Radius/md                      → 6
Radius/lg                      → 8
Radius/xl                      → 12
Radius/2xl                     → 16
Radius/full                    → 9999
```

##### Collection 6 — Border

```
Border/width/none              → 0
Border/width/thin              → 1
Border/width/medium            → 2
Border/width/thick             → 3
```

##### Collection 7 — Shadow

```
Shadow/sm                      → 0 1px 2px 0 rgba(0,0,0,0.05)
Shadow/md                      → 0 4px 6px -1px rgba(0,0,0,0.1)
Shadow/lg                      → 0 10px 15px -3px rgba(0,0,0,0.1)
Shadow/xl                      → 0 20px 25px -5px rgba(0,0,0,0.1)
```

##### Collection 8 — Breakpoint

```
Breakpoint/sm                  → 640
Breakpoint/md                  → 768
Breakpoint/lg                  → 1024
Breakpoint/xl                  → 1280
Breakpoint/2xl                 → 1536
```

#### 2.3 Structurer les pages

Créer les pages selon le choix de l'utilisateur (1.8).

> **Convention** : Utiliser des emoji-préfixes pour les pages : 👋, ✏️, 🎨, 🔤, 📏, 🔲, ↔, 📐, ☁️, 🖥️, ⚙️
> Utiliser des pages `---` (nom = `---`, vide) comme **séparateurs visuels** dans le panneau latéral.

**Si tous sur une page (défaut)** :
```
📄 👋 Welcome
📄 ✏️ Customize             ← Guide de personnalisation (optionnel)
📄 ---                       ← Séparateur
📄 🧱 Foundations            ← Colors, Typography, Size, Border, Space, Radius, Shadow, Breakpoint — UNE page
📄 ⚙️ Icons
📄 ---                       ← Séparateur
📄 🏗️ Layouts               ← Présentation des Layout Components + Templates
📄 Components               ← Toutes les présentations composants sur cette unique page
📄 ---                       ← Séparateur
📄 🔒 _Components            ← TOUS les Main Components (Component Sets)
```

> **Note Foundations** : La page `🧱 Foundations` regroupe TOUTES les foundations. Chaque foundation a son propre **layer (Section frame)**
> contenant : `.documentation header` type=documentation + contenu visuel (swatches, type scale, etc.).
> Visuellement, c'est comme N pages empilées verticalement — chaque foundation est clairement séparée par son header.
>
> **Note Components** : La page `Components` regroupe TOUS les composants. Chaque composant a son propre **layer (Section frame)**
> contenant : `.documentation header` type=component + showcase light + showcase dark + elements.
> Visuellement, c'est comme N pages empilées verticalement — chaque composant est clairement séparé par son header.
>
> **Note Layouts** : La page `🏗️ Layouts` présente les Layout Components et Templates avec des instances et des exemples visuels.
> La page `_Components` est préfixée 🔒 car elle est "privée" et contient les Main Components.

**Si une page par composant** :
```
📄 👋 Welcome
📄 ---
📄 🧱 Foundations
📄 ⚙️ Icons
📄 ---
📄 🏗️ Layouts
📄 buttons                   ← Minuscule, sans emoji (convention Starter UI)
📄 inputs
📄 dropdown
📄 checkbox
📄 radio
📄 ---
📄 🔒 _Components
```

**Si hybride (par catégorie)** :
```
📄 🧱 Foundations
📄 ⚙️ Icons
📄 ---
📄 🏗️ Layouts
📄 Foundation pages…
📄 ---
📄 Actions (Button, Link, IconButton)
📄 Inputs (TextField, Select, Checkbox, Radio, Switch)
📄 Navigation / Data Display / Feedback / Overlay
📄 ---
📄 🔒 _Components
```

#### 2.4 Créer les pages Foundations visuelles — CONTENU OBLIGATOIRE

> **RÈGLE CRITIQUE** : Les pages Foundations ne doivent JAMAIS être vides.
> Chaque page doit contenir du contenu visuel concret créé via `mcp_figma_use_figma`.
> Chaque page Foundation a son propre `.documentation header` instance (`type=documentation`).

> **RÈGLE SIZING — HUG CONTENT OBLIGATOIRE** : Dès qu'un frame a un Auto Layout (`layoutMode`),
> il DOIT être en **HUG content** sur son axe principal (`primaryAxisSizingMode = "AUTO"`).
> Ne JAMAIS laisser de dimensions fixes arbitraires (ex : `resize(w, h)` sans corriger le sizing après).
> Ce n'est PAS grave si les layers/frames de composants n'ont pas tous la même taille —
> le contenu doit dicter la taille, pas l'inverse. Une hauteur fixe provoque du clipping ou des espaces vides.
>
> **Pattern obligatoire** pour tout frame Auto Layout :
> ```javascript
> frame.layoutMode = "VERTICAL"; // ou "HORIZONTAL"
> frame.primaryAxisSizingMode = "AUTO"; // ← HUG sur l'axe principal = OBLIGATOIRE
> frame.counterAxisSizingMode = "FIXED"; // largeur fixe si besoin (ex: page width)
> frame.clipsContent = false;
> // Si resize() est nécessaire (pour fixer la largeur) :
> frame.resize(2000, 10); // hauteur placeholder
> frame.primaryAxisSizingMode = "AUTO"; // TOUJOURS remettre après resize()
> ```
>
> **Interdit** :
> - `resize(w, h)` sans `primaryAxisSizingMode = "AUTO"` après → hauteur figée, contenu clippé
> - Forcer une hauteur identique sur tous les layers → certains composants ont plus de variants que d'autres, c'est normal
> - `primaryAxisSizingMode = "FIXED"` sur un conteneur de contenu (sauf frames d'écran type 1440×900)

Créer la page `🧱 Foundations` avec un frame principal (Auto Layout vertical, gap=120, padding=24, largeur 2000).
Chaque foundation ci-dessous est un **Section frame (layer)** avec son propre `.documentation header` type=documentation.
Chaque section est introduite par une instance `_SectionMetadata` avec le nom de la section et les specs pertinentes.
En bas de page, instancier `_DesignSystemFooter`.

Pour chaque foundation, créer via `mcp_figma_use_figma` :

##### 🎨 Colors (section)
Créer **3 frames** sur cette page (chacun avec `.documentation header` type=documentation) :

1. **"Primitive colors"** — Toutes les palettes de couleurs brutes
   - Pour chaque teinte (Blue, Gray, Red, Green, Yellow, Purple…), créer une rangée horizontale
   - Chaque swatch : rectangle 48×48 avec coin arrondi 8px + texte nom + valeur hex en dessous
   - Lier le fill à la variable Figma correspondante
   - Inclure les palettes Alpha (opacité) si applicable

2. **"Semantic color tokens"** — Couleurs par rôle, en tableau
   - Colonnes : Background | Border | Icon | Shadow | Text
   - Lignes : chaque token sémantique (primary, secondary, danger, etc.)
   - Chaque cellule = swatch carré + nom du token + flèche vers primitive
   - Inspiré Starter UI : tableau structuré lisible

3. **"Component color tokens"** (optionnel) — Tokens spécifiques composants
   - Même format tableau, par composant (button-bg, input-border, etc.)

```javascript
// Pattern pour un swatch de couleur :
const swatch = figma.createFrame();
swatch.resize(64, 80);
swatch.layoutMode = "VERTICAL";
swatch.itemSpacing = 4;
swatch.fills = [];

const colorRect = figma.createRectangle();
colorRect.resize(48, 48);
colorRect.cornerRadius = 8;
colorRect.fills = [{ type: "SOLID", color: hexToRgb(hex) }];
swatch.appendChild(colorRect);

const label = figma.createText();
label.characters = "Blue/600";
label.fontSize = 10;
swatch.appendChild(label);
```

##### 🔤 Typography (section)
Créer **2 frames** :

1. **"Typography styles"** — Tableau visuel de chaque style
   - Colonnes : Preview | Alias | Font family | Weight | Size | Line height | Letter spacing
   - Lignes : title-3xl, title-xxl, title-xl… body-lg, body-md, body-sm…
   - La colonne Preview affiche le texte rendu dans la vraie police/taille/poids

2. **"Typography tokens"** — Tokens bruts Desktop vs Mobile
   - Deux colonnes "Desktop" et "Mobile" avec les valeurs de chaque token
   - font-family, weight, size (tous les paliers), line-height

##### 📏 Size (section)
- **"Size primitive tokens"** — Tableau : Name | Token | Value
- Chaque ligne = nom sémantique (controlHeight-sm…) + icône variable + valeur px

##### 🔲 Border (section)
- **"Border tokens"** — Tableau : Name | Token | Value
- Lignes de différentes épaisseurs visibles

##### ↔ Space (section)
- **"Space tokens"** — Inspiré Starter UI :
  - Colonnes : Name | Token | Value (barre visuelle proportionnelle + valeur px)
  - La barre colorée (rose/magenta) grandit proportionnellement à la valeur
  - Icône ↔ devant le nom pour indiquer le spacing
  - Du plus petit (space-none = 0px) au plus grand (space-8xl = 256px)

##### 📐 Radius (section)
Créer **2 frames** :

1. **"Semantic radius tokens"** — Échelle visuelle
   - Chaque ligne : name + token + carré avec le radius appliqué + valeur px
   - Les carrés grandissent visuellement avec le radius (de radius-none 0px à radius-full 9999px)

2. **"Component radius tokens"** — Mapping composant → radius
   - Tableau : Name | Token | ← radius-sm
   - Ex: badge-radius → radius-style ← radius-sm

##### ☁️ Shadow (section)
- **"Shadow styles"** — 4 cards empilées horizontalement (200×120)
  - Fond blanc, coin arrondi, ombre appliquée via `.effects`
  - Label sous chaque card + aperçu réaliste sur fond clair

##### 🖥️ Breakpoint (section)
Créer **2 frames** :

1. **"Breakpoint tokens"** — Tableau structuré avec colonnes par breakpoint
   - Lignes : min-width, max-width, columns, gutter, margin

2. **"Grid"** — Visualisation responsive
   - 4 device frames côte à côte (mobile 375px, tablet 768px, desktop 1440px, large 1920px)
   - Chaque frame à sa taille réelle avec grid overlay
   - Label au centre avec la plage ("0-767", "768-1023", etc.)

##### ⚙️ Icons (page dédiée — séparée des Foundations)
- `.documentation header` type=documentation

**⚠️ Vérification du dossier icônes (AVANT import)** :
Avant de commencer l'import, **vérifier que le dossier d'icônes existe** :
```bash
ls "<chemin_dossier_icones>" | head -5
```
- **Si le dossier existe** → continuer l'import normalement
- **Si le dossier n'existe pas** → demander à l'utilisateur via `vscode_askQuestions` (ou chat) :

```tool_call
vscode_askQuestions({
  questions: [
    {
      header: "Bibliothèque d'icônes introuvable",
      question: "Le dossier d'icônes configuré n'a pas été trouvé.\nQuelle bibliothèque d'icônes veux-tu utiliser ?",
      options: [
        { label: "Flaticon UIcons Regular Rounded", description: "fi-rr-* — 2040 icônes SVG — traits arrondis", recommended: true },
        { label: "Lucide Icons", description: "Open source, 1500+ icônes SVG, style épuré" },
        { label: "Heroicons", description: "Par Tailwind Labs — outline/solid, 300+ icônes" },
        { label: "Autre bibliothèque", description: "Je fournis le chemin manuellement" }
      ]
    },
    {
      header: "Chemin du dossier SVG",
      question: "Chemin absolu vers le dossier contenant les fichiers SVG :\nEx: /Users/nom/Downloads/icons/svg/"
    }
  ]
})
```

**Si chat uniquement :**
> Le dossier d'icônes configuré est introuvable.
> 1. **Quelle bibliothèque ?** Flaticon UIcons _(recommandé)_ · Lucide · Heroicons · Autre
> 2. **Chemin absolu** vers le dossier SVG ? Ex: `/Users/nom/Downloads/icons/svg/`

Une fois le chemin fourni, re-vérifier avec `ls` et continuer.

- **Bibliothèque par défaut : Flaticon UIcons Regular Rounded** (`fi-rr-*`)
  - Source locale : `/Users/arnaudmorvan/Dropbox/uicons-regular-rounded/svg/`
  - ~2040 icônes SVG uniques, viewBox 24×24, single `<path>` element
  - Style : Regular Rounded (traits arrondis, cohérent avec radius soft)

**Icônes essentielles à importer** (30 icônes SaaS minimum) :

```
Navigation :    fi-rr-angle-small-down, fi-rr-angle-small-up, fi-rr-angle-small-left, fi-rr-angle-small-right
                fi-rr-arrow-left, fi-rr-arrow-right, fi-rr-bars-staggered (menu hamburger)
Actions :       fi-rr-plus, fi-rr-cross, fi-rr-check, fi-rr-pencil (edit), fi-rr-trash
                fi-rr-copy, fi-rr-share, fi-rr-download, fi-rr-upload
Recherche :     fi-rr-search, fi-rr-filter, fi-rr-bars-sort
Utilisateur :   fi-rr-user, fi-rr-users, fi-rr-lock, fi-rr-unlock, fi-rr-key
Communication : fi-rr-envelope (mail), fi-rr-bell (notif), fi-rr-comment-alt (chat)
Contenu :       fi-rr-picture (image), fi-rr-file, fi-rr-folder, fi-rr-link
Interface :     fi-rr-eye (voir), fi-rr-eye-crossed (cacher), fi-rr-settings, fi-rr-home
                fi-rr-info, fi-rr-exclamation (warning), fi-rr-ban (error)
Data :          fi-rr-chart-line-up, fi-rr-calendar, fi-rr-clock
Toggles :       fi-rr-sun (light mode), fi-rr-moon (dark mode), fi-rr-globe
Commerce :      fi-rr-shopping-cart, fi-rr-credit-card, fi-rr-star
```

**Import SVG → Figma Component via Plugin API** :
```javascript
// Pattern pour importer un SVG en composant Figma
const fs = require('fs'); // N/A dans Plugin API — utiliser fetch ou pré-encoder

// Les SVGs doivent être importés via figma.createNodeFromSvg()
const svgString = '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="..."/></svg>';
const iconNode = figma.createNodeFromSvg(svgString);
iconNode.name = "fi-rr-plus";
iconNode.resize(24, 24);

// Convertir en Component pour être réutilisable
const iconComponent = figma.createComponent();
iconComponent.name = "icon / plus";
iconComponent.resize(24, 24);
iconComponent.layoutMode = "VERTICAL";
iconComponent.primaryAxisAlignItems = "CENTER";
iconComponent.counterAxisAlignItems = "CENTER";
// Déplacer les enfants du SVG node dans le component
for (const child of [...iconNode.children]) {
  iconComponent.appendChild(child);
}
iconNode.remove();
```

> **Note** : Le code Plugin API n'a pas accès au filesystem.
> L'assistant doit **lire les fichiers SVG locaux** avec ses outils,
> **extraire le `<path d="...">`**, puis injecter le SVG string dans `figma.createNodeFromSvg()`.
> Pattern : `cat /Users/arnaudmorvan/Dropbox/uicons-regular-rounded/svg/fi-rr-{name}.svg`

**Grille d'icônes sur la page ⚙️ Icons** :
- `.documentation header` type=documentation en en-tête
- Titre "Icon Library — UIcons Regular Rounded"
- Grille Auto Layout wrap, gap=24
- Organiser par catégorie (Navigation, Actions, Communication, etc.) — chaque catégorie a son propre titre (16px Bold)
- Chaque icône : **frame 80×100** (Auto Layout vertical, gap=8, centerH)
  - Le SVG node est redimensionné à **32×32** (pas 24×24 par défaut) pour être lisible
  - Label nom en dessous (**12px** Regular, gris 600, textAlign=CENTER)
- Icônes remplies en `Neutral/colorText` (s'adapte automatiquement au dark mode)

> **RÈGLE AFFICHAGE ICÔNES** : Les icônes doivent être **clairement visibles et lisibles** dans la grille.
> Le SVG doit être resize à 32×32 minimum. Le frame conteneur doit être assez grand pour le SVG + le label.
> Pattern :
> ```javascript
> const iconFrame = figma.createFrame();
> iconFrame.resize(80, 100);
> iconFrame.layoutMode = "VERTICAL";
> iconFrame.itemSpacing = 8;
> iconFrame.counterAxisAlignItems = "CENTER";
> iconFrame.primaryAxisAlignItems = "CENTER";
> iconFrame.fills = [];
> // SVG node resize à 32×32
> iconNode.resize(32, 32);
> iconFrame.appendChild(iconNode);
> // Label
> const label = figma.createText();
> label.characters = "icon-name";
> label.fontSize = 12;
> label.textAlignHorizontal = "CENTER";
> iconFrame.appendChild(label);
> ```

### Phase 3 — Création des composants Figma

#### 3.1 `.documentation header` — Component Set sur la page `🔒 _Components`

> **Inspiration SlothUI** : Format professionnel avec header large, gradient décoratif,
> badges de catégorie, breadcrumbs avec logo, et coins arrondis.

Créer **en premier** sur la page `🔒 _Components`. C'est un **Component Set** avec **3 variants** :

```
🔶 .documentation header (Component Set)
│
├── type=component
│   ├── Auto Layout : vertical, gap=80, padding=40
│   ├── cornerRadius = 40
│   ├── border = 1px {Neutral/colorBorder}
│   ├── fills = [{type:"SOLID", color: Neutral/colorBg (blanc)}]
│   ├── overflow = clip
│   │
│   ├── 🎨 Gradient Ellipse (décoration positionnelle)
│   │   ├── Ellipse ~1150×1150, position absolute top=-990, right=-450
│   │   ├── fills = gradient radial {Primary/colorPrimary} → transparent
│   │   ├── opacity = 0.08 (subtil, adapté au mode light/dark)
│   │   └── (positionné hors-cadre pour un wash léger sur le header)
│   │
│   ├── Top Row (Auto Layout horizontal, justify=space-between, fill width)
│   │   ├── Title Block (Auto Layout vertical, gap=24, max-width=960)
│   │   │   ├── [title] ← Text property, 60px ExtraBold, {Neutral/colorText}
│   │   │   └── [description] ← Text property, 20px Regular, {Neutral/colorTextSecondary}
│   │   └── Category Badge (Auto Layout horizontal, gap=10, padding=12/20)
│   │       ├── fills = [{type:"SOLID", color: {Primary/colorPrimary}}]
│   │       ├── cornerRadius = 999 (pill)
│   │       ├── [category icon] ← 20×20, fill=white
│   │       └── [category label] ← Text "Components" ou "Foundations", 16px Bold, white
│   │
│   └── Breadcrumb Bar (Auto Layout horizontal, justify=space-between, fill width)
│       ├── Left (Auto Layout horizontal, gap=12, alignItems=CENTER)
│       │   ├── Logo DS (frame 40×40, cornerRadius=15, fills={Primary/colorPrimary})
│       │   │   └── [Initiales ou icône DS] ← texte blanc centré
│       │   ├── [1st level] ← Text "Foundations" ou "Components", 18px Bold, {Neutral/colorTextSecondary}
│       │   ├── → (icône flèche 24×24)
│       │   └── [2nd level] ← Text "Button" (nom page), 18px Bold, {Neutral/colorTextSecondary}
│       └── Right
│           └── [link] ← Text lien optionnel, 18px SemiBold, {Primary/colorPrimary}, underline
│
├── type=documentation
│   ├── (même structure que type=component, badge label="Foundations")
│   └── (pas de divider — les sections sont séparées par _SectionMetadata)
│
└── type=elements
    ├── Auto Layout : vertical, gap=4, padding=24/0
    ├── [title] ← Text "Elements", 32px Bold
    └── [description] ← Text "Elements that are used to build components. They are not published to the library.", 16px Regular
```

> **⚠️ CRITIQUE** : Quand on instancie le `.documentation header` :
> - **Sur les pages composants** : utiliser `type=component`, badge "Components", alimenter breadcrumbs + title + description
> - **Sur les pages Foundations** : utiliser `type=documentation`, badge "Foundations", alimenter breadcrumbs + title + description
> - **Pour les sections Elements** : utiliser `type=elements`, le titre est fixe
> Ne JAMAIS laisser les valeurs par défaut.

**Code pour créer et alimenter l'instance** :
```javascript
// Trouver le variant souhaité
const docHeaderSet = figma.currentPage.findOne(n => n.type === "COMPONENT_SET" && n.name === ".documentation header");
const variant = docHeaderSet.findOne(n => n.name === "type=component");
const inst = variant.createInstance();

// Alimenter les breadcrumbs
const level1 = inst.findOne(n => n.name === "1st level");
if (level1 && level1.type === "TEXT") level1.characters = "Components";
const level2 = inst.findOne(n => n.name === "2nd level");
if (level2 && level2.type === "TEXT") level2.characters = "Button";
// Alimenter le badge catégorie
const badgeLabel = inst.findOne(n => n.name === "category label");
if (badgeLabel && badgeLabel.type === "TEXT") badgeLabel.characters = "Components";
// Alimenter titre et description
const titleNode = inst.findOne(n => n.name === "title" && n.type === "TEXT");
if (titleNode) titleNode.characters = "Button";
const descNode = inst.findOne(n => n.name === "description" && n.type === "TEXT");
if (descNode) descNode.characters = "The Button is a clickable element that performs actions when pressed.";
```

#### 3.1b `_DesignSystemFooter` — Component sur la page `🔒 _Components`

> **Inspiration SlothUI** : Footer de page avec logo, tagline et lien.
> Même style visuel que le header (gradient, coins arrondis).

```
🔶 _DesignSystemFooter (Main Component)
├── Auto Layout : horizontal, justify=space-between, padding=40, alignItems=CENTER
├── cornerRadius = 40
├── border = 1px {Neutral/colorBorder}
├── fills = [{type:"SOLID", color: Neutral/colorBg}]
├── width = 1952 (parent page 2000, indent 24px)
│
├── 🎨 Gradient Ellipse (même décoration que le header, miroir)
│   └── Ellipse, gradient radial {Primary/colorPrimary} → transparent, opacity=0.06
│
├── Left (Auto Layout horizontal, gap=12, alignItems=CENTER)
│   ├── Logo DS (frame 40×40, cornerRadius=15, fills={Primary/colorPrimary})
│   │   └── [Initiales ou icône DS] ← texte blanc centré
│   └── [DS Name] ← Text 18px Bold, {Neutral/colorTextSecondary}
│
└── Right (Auto Layout vertical, gap=4, alignItems=END)
    ├── [tagline] ← Text property, 14px Regular, {Neutral/colorTextSecondary}
    │   (ex: "Généré par ds-init skill")
    └── [link] ← Text property, 14px SemiBold, {Primary/colorPrimary}, underline
        (ex: nom du DS ou lien personnalisé)
```

> **Usage** : Instancier en bas de chaque page, après le contenu.
> Le footer est optionnel sur les pages internes mais recommandé sur Welcome + Foundations.

#### 3.1c `_SectionMetadata` — Component sur la page `🔒 _Components`

> **Inspiration SlothUI** : Barre fine de métadonnées introduisant chaque section.
> Label à gauche, informations techniques à droite, séparés par une ligne pointillée.

```
🔶 _SectionMetadata (Main Component)
├── Auto Layout : horizontal, justify=space-between, alignItems=CENTER
├── width = FILL (s'adapte au parent, typiquement 1776px)
├── height = 38
├── fills = []
│
├── [label] ← Text property, 14px SemiBold, {Neutral/colorText}
│   (ex: "Color Primitives", "Display lg", "Spacing Scale")
│
├── Dotted Line (rectangle, height=1, fill={Neutral/colorBorder})
│   ├── layoutSizingHorizontal = "FILL" (prend tout l'espace restant)
│   └── dashPattern = [4, 4] (ou strokes pointillés)
│
└── [specs] ← Text property, 13px Regular, {Neutral/colorTextSecondary}
    (ex: "Font size: 60px · Line Height: 68px · Tracking: -1.08px")
```

> **Usage** : Placer avant chaque bloc de contenu dans les pages Foundations et Components.
> Sert de "section divider" informatif. Remplace les simples titres de section.
> Le texte `specs` est optionnel — laisser vide si pas de métadonnées pertinentes.

#### 3.2 Layout Components — 100% Vrais Slots Figma (`createSlot()`), sur `_Components`

> **⚠️ RÈGLE SLOT FIGMA** : Utiliser les **vrais SlotNode natifs** de l'API Plugin Figma.
> NE PAS créer de composant `.slot` séparé. Les slots se créent DANS un Component via :
> ```javascript
> const comp = figma.createComponent();
> comp.name = "PageLayout";
> // Créer un vrai slot natif
> const slot = comp.createSlot();
> slot.name = "header";
> slot.layoutSizingHorizontal = "FILL";
> slot.resize(100, 64);
> ```
> `createSlot()` crée un `SlotNode` (type='SLOT') ET ajoute automatiquement
> une propriété de type `'SLOT'` dans `componentPropertyDefinitions`.
> L'utilisateur peut ensuite remplacer librement le contenu du slot dans les instances.

Créer sur la page `_Components` (pas dans une page "Layouts") :

##### Layouts de base (composables)

> **Notation** : `⬦ name` = SlotNode créé via `component.createSlot()`, renommé en `name`.
> Chaque slot est un vrai `SlotNode` Figma natif (type='SLOT'), PAS un simple frame.

```
🔶 PageLayout (Main Component)
├── Auto Layout : vertical, fill container
├── ⬦ header (SlotNode via createSlot) — hauteur fixe, fill width
├── Auto Layout : horizontal, fill
│   ├── ⬦ sidebar (SlotNode) — largeur fixe 240px, fill height
│   └── ⬦ content (SlotNode) — fill, fill
└── ⬦ footer (SlotNode) — hauteur fixe, fill width

🔶 SectionLayout (Main Component)
├── Auto Layout : vertical, gap=16, padding=24
├── ⬦ header (SlotNode)
├── ⬦ content (SlotNode) — fill
└── ⬦ footer (SlotNode)

🔶 StackLayout (Main Component)
├── Variant : direction = horizontal | vertical
├── Variant : gap = sm(8) | md(16) | lg(24)
└── ⬦ children (SlotNode) — fill

🔶 GridLayout (Main Component)
├── Variant : columns = 1 | 2 | 3 | 4
├── Variant : gap = sm(8) | md(16) | lg(24)
├── Auto Layout : wrap
└── ⬦ children (SlotNode)
```

Ces layouts de base se **composent** entre eux — le fondement des templates ci-dessous.

##### 3.2b Icon Component Set — Architecture sur `_Components`

> **Finding audit** : slothUI a un vrai Component Set icône avec `name × size × color × weight` (360+ variants).
> La plupart des DS (8/10) se contentent de SVGs bruts — ce qui empêche la réutilisation thématisée.
> Un Icon Component Set permet de lier les couleurs aux tokens sémantiques et d'adapter au dark mode.

En plus de l'import des SVGs sur la page ⚙️ Icons, créer un **Icon Component Set** sur `_Components` :

```
🔶 icon (Component Set)
├── Properties :
│   ├── Variant : name = plus | minus | search | settings | user | bell | … (30+ icônes)
│   ├── Variant : size = xs(16) | sm(20) | md(24) | lg(32) | xl(48)
│   └── Variant : color = neutral | brand | danger | warning | success | inherit
│
├── Auto Layout : vertical, CENTER × CENTER
├── resize selon size variant
│
└── [SVG Vector] — le path SVG de l'icône
    ├── fills liés au token couleur selon color variant :
    │   ├── neutral  → {Neutral/colorTextSecondary}
    │   ├── brand    → {Primary/colorPrimary}
    │   ├── danger   → {Feedback/colorDanger}
    │   ├── warning  → {Feedback/colorWarning}
    │   ├── success  → {Feedback/colorSuccess}
    │   └── inherit  → fills=[] (hérite du parent)
    └── strokes = [] (pas de contour)
```

**API pattern pour créer les variants :**
```javascript
// Créer le Component Set avec toutes les combinaisons
const iconComponents = [];
for (const iconName of iconNames) {
  for (const size of ['xs','sm','md','lg','xl']) {
    for (const color of ['neutral','brand','danger','warning','success','inherit']) {
      const comp = figma.createComponent();
      comp.name = `name=${iconName}, size=${size}, color=${color}`;
      const sizeMap = { xs:16, sm:20, md:24, lg:32, xl:48 };
      const s = sizeMap[size];
      comp.resize(s, s);
      comp.layoutMode = "VERTICAL";
      comp.primaryAxisAlignItems = "CENTER";
      comp.counterAxisAlignItems = "CENTER";
      comp.primaryAxisSizingMode = "AUTO";
      comp.fills = [];
      // Importer le SVG
      const svg = figma.createNodeFromSvg(svgStrings[iconName]);
      const vector = svg.children[0];
      comp.appendChild(vector);
      svg.remove();
      vector.resize(s * 0.75, s * 0.75); // icône = 75% du conteneur
      // Appliquer la couleur via variable binding
      if (color !== 'inherit') {
        const colorVar = colorVarMap[color]; // pre-built map
        vector.setBoundVariable('fills', 0, 'color', colorVar.id);
      }
      iconComponents.push(comp);
    }
  }
}
// Combiner en Component Set
const iconSet = figma.combineAsVariants(iconComponents, _componentsPage);
iconSet.name = "icon";
```

**Usage dans les composants** (leading/trailing slots) :
- Button : `icon/name=search/size=md/color=inherit` dans le slot leading
- Alert : `icon/name=info/size=lg/color=warning` comme icône d'alerte
- Navbar : `icon/name=home/size=sm/color=neutral` pour les items de navigation

> **Note** : Pour limiter la taille du Component Set, ne créer que les 10-15 icônes les plus
> utilisées dans le Component Set. Les icônes rares restent comme SVGs individuels sur la page Icons.
> Icônes prioritaires : `plus`, `minus`, `search`, `check`, `cross`, `chevron-down`, `chevron-right`,
> `user`, `settings`, `bell`, `eye`, `eye-crossed`, `info`, `warning`, `trash`.

##### 3.2c Composition de Layouts — Patterns d'imbrication

> **Finding audit** : Les layouts sont des conteneurs composables, pas des templates figés.
> Les DS professionnels combinent PageLayout + SectionLayout + GridLayout pour créer
> des structures complexes. Ce guide montre comment imbriquer les layouts.

**Pattern 1 — Dashboard classique** :
```
PageLayout (instance détachée)
├── header → Navbar (instance)
├── sidebar → SidebarNav (instance avec nav-items remplis)
├── content → SectionLayout (instance détachée)
│     ├── header → Page Title + FilterBar
│     ├── content → GridLayout columns=3 (instance détachée)
│     │     ├── children[0] → StatCard (instance)
│     │     ├── children[1] → StatCard (instance)
│     │     └── children[2] → StatCard (instance)
│     └── footer → Pagination (instance)
└── footer → Footer (instance)
```

**Pattern 2 — Settings avec navigation latérale** :
```
PageLayout (instance détachée)
├── header → Navbar
├── sidebar → (hidden: visible=false)
├── content → StackLayout direction=horizontal, gap=lg (instance détachée)
│     ├── children[0] → StackLayout direction=vertical (menu nav)
│     │     └── Liens : General, Security, Notifications, Billing
│     └── children[1] → SectionLayout (formulaire)
│           ├── header → Section title
│           ├── content → Form fields (inputs empilés)
│           └── footer → Save/Cancel buttons
└── footer → Footer
```

**Pattern 3 — Landing page composée** :
```
StackLayout direction=vertical, gap=0 (conteneur pleine page)
├── children[0] → .templates/landing/hero (instance)
├── children[1] → .templates/landing/features (instance)
├── children[2] → .templates/landing/pricing (instance)
├── children[3] → .templates/landing/testimonials (instance)
├── children[4] → .templates/landing/cta-banner (instance)
└── children[5] → .templates/landing/footer (instance)
```

**Quand imbriquer vs. aplatir** :
| Situation | Approche | Raison |
|-----------|----------|--------|
| Page SaaS avec sidebar | **PageLayout → SectionLayout** (2 niveaux) | Sidebar + content zone distinctes |
| Landing page sections | **StackLayout enfants directs** (1 niveau) | Sections empilées sans sidebar |
| Dashboard avec grille KPI | **PageLayout → SectionLayout → GridLayout** (3 niveaux max) | Structure complexe |
| Formulaire simple | **SectionLayout seul** (1 niveau) | Pas besoin de page wrapper |

> **Règle** : Ne jamais dépasser **3 niveaux d'imbrication** de layouts.
> Au-delà, la structure devient difficile à maintenir et à comprendre.

##### Page Templates SaaS — pré-construits avec vrais SlotNode

> **Principe** : Chaque template est un Main Component prêt à l'emploi.
> L'utilisateur instancie le template et remplit les slots avec son contenu.
> Tous les templates font **1440px de large** et utilisent les variables DS.
> Tous les `⬦ name` ci-dessous = `SlotNode` créés via `component.createSlot()`.

```
🔶 .templates / saas / dashboard (Main Component)
├── Auto Layout : horizontal, fill, min-height 900
├── Sidebar (Auto Layout vertical, width=240, fill height)
│   ├── ⬦ logo (Slot) — 40×40
│   ├── ⬦ nav-items (Slot) — fill, Auto Layout vertical gap=4
│   └── ⬦ sidebar-footer (Slot) — user avatar + settings
├── Main (Auto Layout vertical, fill)
│   ├── Topbar (Auto Layout horizontal, height=64, fill width, justify=space-between)
│   │   ├── ⬦ breadcrumb (Slot)
│   │   ├── ⬦ search (Slot)
│   │   └── ⬦ actions (Slot) — notifications, avatar
│   ├── Content (Auto Layout vertical, fill, padding=32, gap=24)
│   │   ├── ⬦ page-header (Slot) — titre + description + CTA
│   │   ├── ⬦ stats-row (Slot) — 4 cards KPI côte à côte
│   │   ├── ⬦ main-content (Slot) — graphiques, tableaux
│   │   └── ⬦ secondary-content (Slot) — widgets secondaires
│   └── ⬦ footer (Slot) — optionnel

🔶 .templates / saas / settings (Main Component)
├── Auto Layout : horizontal, fill
├── Sidebar (même pattern, width=240)
├── Main (Auto Layout vertical, fill)
│   ├── Topbar (même pattern)
│   └── Content (Auto Layout horizontal, fill, padding=32, gap=32)
│       ├── Settings Nav (Auto Layout vertical, width=220)
│       │   └── ⬦ settings-menu (Slot) — liens de navigation verticaux
│       └── Settings Content (Auto Layout vertical, fill, gap=24)
│           ├── ⬦ section-header (Slot) — titre section
│           ├── ⬦ form-fields (Slot) — champs de formulaire
│           └── ⬦ form-actions (Slot) — boutons save/cancel

🔶 .templates / saas / list-detail (Main Component)
├── Auto Layout : horizontal, fill
├── Sidebar
├── Main (Auto Layout vertical, fill)
│   ├── Topbar
│   └── Content (Auto Layout horizontal, fill, padding=32, gap=0)
│       ├── List Panel (Auto Layout vertical, width=380, border-right)
│       │   ├── ⬦ list-header (Slot) — search + filters
│       │   └── ⬦ list-items (Slot) — scroll area d'items
│       └── Detail Panel (Auto Layout vertical, fill, padding-left=32)
│           ├── ⬦ detail-header (Slot) — titre + actions
│           └── ⬦ detail-content (Slot) — contenu détaillé

🔶 .templates / saas / table (Main Component)
├── Auto Layout : horizontal, fill
├── Sidebar
├── Main (Auto Layout vertical, fill)
│   ├── Topbar
│   └── Content (Auto Layout vertical, fill, padding=32, gap=24)
│       ├── ⬦ table-header (Slot) — titre + boutons + filtres
│       ├── ⬦ table-content (Slot) — tableau de données
│       └── ⬦ table-pagination (Slot) — pagination

🔶 .templates / saas / profile (Main Component)
├── Auto Layout : horizontal, fill
├── Sidebar
├── Main (Auto Layout vertical, fill)
│   ├── Topbar
│   └── Content (Auto Layout vertical, fill, padding=32, gap=32)
│       ├── ⬦ profile-header (Slot) — avatar + banner + infos
│       ├── ⬦ profile-tabs (Slot) — onglets de navigation
│       └── ⬦ profile-content (Slot) — contenu de l'onglet actif

🔶 .templates / saas / onboarding (Main Component)
├── Auto Layout : vertical, fill, center horizontally
├── Header (Auto Layout horizontal, height=64, fill width, padding=0/32)
│   ├── ⬦ logo (Slot)
│   └── ⬦ progress (Slot) — stepper / progress bar
├── Content (Auto Layout vertical, max-width=640, center, fill, gap=32, padding=48)
│   ├── ⬦ step-header (Slot) — titre + description de l'étape
│   ├── ⬦ step-content (Slot) — formulaire / choix / upload
│   └── ⬦ step-actions (Slot) — boutons précédent / suivant
└── ⬦ footer (Slot) — skip, help link
```
vrais SlotNode

> **Principe** : Sections empilées verticalement, full-width.
> Chaque template = une section de landing page à instancier et remplir.
> Tous les `⬦ name` ci-dessous = `SlotNode` créés via `component.createSlot()`
> Chaque template = une section de landing page à instancier et remplir.

```
🔶 .templates / landing / hero (Main Component)
├── Auto Layout : vertical, fill width, padding=80/64, center
├── Variant : layout = centered | split (texte + image côte à côte)
├── (centered) :
│   ├── ⬦ badge (Slot) — badge "Nouveau" / "v2.0" (optionnel, visible=false)
│   ├── ⬦ headline (Slot) — titre principal, 56px Bold, max-width 800
│   ├── ⬦ subheadline (Slot) — sous-titre 20px, gris 600, max-width 600
│   ├── ⬦ cta-row (Slot) — Auto Layout horizontal, 2 boutons (primaire + secondaire)
│   └── ⬦ hero-visual (Slot) — image/illustration/screenshot
├── (split) :
│   ├── Left (Auto Layout vertical, fill, center-vertical)
│   │   ├── ⬦ badge + headline + subheadline + cta-row
│   └── Right (fill)
│       └── ⬦ hero-visual

🔶 .templates / landing / features (Main Component)
├── Auto Layout : vertical, fill width, padding=80/64, gap=48
├── Variant : columns = 3 | 4
├── ⬦ section-header (Slot) — titre + description centrés
└── Features Grid (Auto Layout wrap, gap=32)
    ├── ⬦ feature-1 (Slot) — icône + titre + description
    ├── ⬦ feature-2 (Slot)
    ├── ⬦ feature-3 (Slot)
    └── ⬦ feature-4 … (Slot, visible selon variant columns)

🔶 .templates / landing / pricing (Main Component)
├── Auto Layout : vertical, fill width, padding=80/64, gap=48, center
├── ⬦ section-header (Slot) — titre + toggle mensuel/annuel
└── Cards Row (Auto Layout horizontal, gap=24, center)
    ├── ⬦ plan-1 (Slot) — card: nom, prix, features, CTA
    ├── ⬦ plan-2 (Slot) — card mise en avant (bordure primaire)
    └── ⬦ plan-3 (Slot)

🔶 .templates / landing / testimonials (Main Component)
├── Auto Layout : vertical, fill width, padding=80/64, gap=48
├── Variant : layout = grid | single-featured
├── ⬦ section-header (Slot)
└── (grid) : Testimonials Grid (Auto Layout wrap, gap=24)
    ├── ⬦ testimonial-1…6 (Slots) — avatar + quote + nom + rôle

🔶 .templates / landing / cta-banner (Main Component)
├── Auto Layout : vertical, fill width, padding=64/48, center
├── Background : colorPrimary ou gradient
├── ⬦ headline (Slot) — titre blanc, 36px Bold
├── ⬦ description (Slot) — texte blanc/80%
└── ⬦ cta-row (Slot) — bouton(s)

🔶 .templates / landing / footer (Main Component)
├── Auto Layout : vertical, fill width, padding=48/64, gap=48
├── Top (Auto Layout horizontal, fill, gap=48)
│   ├── ⬦ brand-col (Slot) — logo + description + socials
│   ├── ⬦ links-col-1 (Slot) — titre "Product" + liens
│   ├── ⬦ links-col-2 (Slot) — titre "Company" + liens
│   └── ⬦ links-col-3 (Slot) — titre "Resources" + liens
└── Bottom (Auto Layout horizontal, fill, justify=space-between)
    ├── ⬦ copyright (Slot)
    └── ⬦ legal-links (Slot)
```

> **Nommage** : Tous les templates sont préfixés `.templates /` pour les distinguer
> des composants publics. Ils vivent sur `🔒 _Components` comme tous les Main Components.

> **Appel 6bis** : Créer les templates après les Layout Components (Appel 6).
> Le code est un appel `mcp_figma_use_figma` séparé car il est volumineux.

#### 3.2d Sizing — JAMAIS de pixels hardcodés dans les composants

> **Finding audit** : L'audit de DS Arnaud révèle un anti-pattern critique :
> les tailles de boutons (22/36/43/50/68px) et paddings (5/12/14/19/24px) sont hardcodés.
> Seuls les font-sizes et radius utilisent les tokens. Résultat : changer une taille
> oblige à modifier 60+ composants manuellement.

**🚫 ANTI-PATTERN (INTERDIT) :**
```javascript
// ❌ NE JAMAIS FAIRE
button.resize(200, 50);       // hauteur hardcodée en pixels
button.paddingLeft = 19;       // padding hardcodé
button.paddingTop = 15;        // padding hardcodé
button.itemSpacing = 8;        // gap hardcodé
```

**✅ PATTERN CORRECT (OBLIGATOIRE) :**
```javascript
// ✅ Toutes les dimensions liées aux variables Size
const heightVar = sizeVars.find(v => v.name === 'Size/controlHeight/md');
const paddingVar = sizeVars.find(v => v.name === 'Size/controlPadding/md');
const gapVar = sizeVars.find(v => v.name === 'Size/controlGap/md');
const iconVar = sizeVars.find(v => v.name === 'Size/iconSize/md');

button.setBoundVariable('minHeight', heightVar.id);
button.setBoundVariable('paddingLeft', paddingVar.id);
button.setBoundVariable('paddingRight', paddingVar.id);
button.setBoundVariable('paddingTop', paddingVar.id);
button.setBoundVariable('paddingBottom', paddingVar.id);
button.setBoundVariable('itemSpacing', gapVar.id);
// Résultat : changer le token Size/controlHeight/md de 40→44 met à jour TOUS les composants MD
```

**Composants concernés** (TOUS les composants interactifs) :
| Dimension | Variable | Composants |
|-----------|----------|------------|
| Hauteur conteneur | `Size/controlHeight/{size}` | button, text field, select, checkbox box, radio circle |
| Padding horizontal | `Size/controlPadding/{size}` | button, text field, select, dropdown option |
| Gap icône/label | `Size/controlGap/{size}` | button, text field, select, checkbox, radio |
| Taille icône | `Size/iconSize/{size}` | icon dans slots, chevron select, check/dot |

> **Exception** : Les frames d'écran (1440×900) et les showcases à largeur fixe
> utilisent des dimensions pixel — c'est normal. Seuls les **composants réutilisables** doivent utiliser des tokens.

#### 3.3 Composants interactifs — Architecture Figma

##### 3.3a Décision d'architecture — Pattern A vs A+

> **Finding audit** : Sur 10 DS audités, 5 utilisent le Pattern A (flat variants), 1 utilise le Pattern A+
> (flat + wrapper séparé), et **AUCUN** n'utilise le Pattern B (héritage cascade).
> Le Pattern B est recommandé en théorie (DRY) mais rejeté par le marché car complexe à maintenir.

| Pattern | Quand l'utiliser | Nb variants | Exemple |
|---------|-----------------|-------------|----------|
| **A — Flat** | < 60 variants par Component Set | `type × size × state` = jusqu'à 60 | Button (4×3×5=60) |
| **A+ — Flat + Wrapper** | > 60 variants, axe dimensionnel orthogonal | Deux Component Sets séparés | DS Arnaud : "Buttons variant" (Type×State=32) + "Button Main" (Size only) |
| **B — Cascade** | ❌ JAMAIS | N/A | ButtonBase→ButtonPrimary→ButtonPrimary/SM — non observé en production |

**Règle de décision** :
1. Calculer le nombre total de variants : `types × sizes × states × (loading? ×2 : ×1)`
2. Si ≤ 60 → **Pattern A** (un seul Component Set avec toutes les combinaisons)
3. Si > 60 → **Pattern A+** (séparer l'axe le moins couplé visuellement dans un wrapper)
   - Wrapper typique : Component "Button Wrapper" avec property `size` uniquement
   - Contient une instance de "Button Variant" (type × state) redimensionnée
4. ❌ Ne JAMAIS tenter Pattern B (héritage) — la synchronisation des changements est un cauchemar

> **Pour ce DS** : On utilise le Pattern A (flat) par défaut.
> Seul split recommandé si l'utilisateur veut `loading` en boolean (4×3×5×2 = 120 variants)
> → dans ce cas séparer size dans un wrapper.

Pour chaque composant sélectionné (Tier 1+), créer un **Component Set** avec :

##### Structure type — Button (exemple)

```
🔶 Button (Component Set)
├── Properties :
│   ├── Variant : type = Primary | Secondary | Ghost | Danger
│   ├── Variant : size = SM | MD | LG
│   ├── Variant : state = Default | Hover | Active | Focus | Disabled
│   └── Boolean : loading = false
│
├── Auto Layout : horizontal
│   ├── gap → lié à variable {Size/controlGap/{size}}
│   ├── padding horizontal → lié à variable {Size/controlPadding/{size}}
│   ├── height → liée à variable {Size/controlHeight/{size}}
│   └── border-radius → lié à variable {Radius/md}
│
├── ⬦ leading (SlotNode via createSlot) — pour icône, avatar, badge…
├── [Label] ← Text property
├── ⬦ trailing (SlotNode via createSlot) — pour icône, chevron, loader…
│
├── Couleurs par type (variant) :
│   ├── Primary : fill={Primary/colorPrimary}, text=white
│   ├── Secondary : fill=transparent, border={Neutral/colorBorder}, text={Neutral/colorText}
│   ├── Ghost : fill=transparent, text={Primary/colorPrimary}
│   └── Danger : fill={Feedback/colorDanger}, text=white
│
└── Couleurs par state :
    ├── Hover : fill légèrement plus clair
    ├── Active : fill légèrement plus foncé
    ├── Focus : outline 2px offset
    └── Disabled : opacity 0.5
```

##### Structure type — Text Field

```
🔶 Text Field (Component Set)
├── Properties :
│   ├── Variant : type = Text | Password | Search | Email | Number
│   ├── Variant : size = SM | MD | LG
│   ├── Variant : state = Default | Hover | Focus | Disabled
│   ├── Variant : status = None | Error | Warning | Success
│   └── Boolean : filled = false
│
├── Auto Layout : vertical, gap=4
│
├── Input Row (Auto Layout horizontal)
│   ├── ⚠️ BORDER TOUJOURS VISIBLE — strokes=[{color: Neutral/colorBorder}], strokeWeight=1, strokeAlign="INSIDE"
│   │   Un text field SANS border visible est INVISIBLE sur fond blanc.
│   │   JAMAIS fills=[] sans strokes — toujours au moins un stroke 1px.
│   ├── border → lié à variable {Neutral/colorBorder}
│   ├── border-radius → lié à variable {Radius/md}
│   ├── height → liée à variable {Size/controlHeight/{size}}
│   ├── padding horizontal → lié à variable {Size/controlPadding/{size}}
│   │
│   ├── ⬦ leading (SlotNode via createSlot) — icône search, lock, user…
│   ├── [Value] ← Text property (+ placeholder gris 400)
│   └── ⬦ trailing (SlotNode via createSlot) — icône clear, eye toggle, check…
│
├── Helper Text (Auto Layout horizontal, gap=4) — visible si status ≠ None
│   ├── [Status Icon] ← 16×16 (⚠ warning, ✕ error, ✓ success)
│   └── [Message] ← Text property, 12px, couleur liée au status
│
├── Couleurs border par status :
│   ├── None : {Neutral/colorBorder}
│   ├── Error : {Feedback/colorDanger}
│   ├── Warning : {Feedback/colorWarning}
│   └── Success : {Feedback/colorSuccess}
│
└── Couleurs Helper Text par status :
    ├── Error : text={Feedback/colorDanger}, icon=✕
    ├── Warning : text={Feedback/colorWarning}, icon=⚠
    └── Success : text={Feedback/colorSuccess}, icon=✓
```

> **⚠️ CRITIQUE** : Toutes les dimensions (height, padding, gap, icon size) DOIVENT être
> liées aux variables Figma de la collection Size. **JAMAIS de valeurs px hardcodées.**

##### Nommage des sous-composants : convention `.elements /`

> **Convention inspirée de Starter UI** : les sous-composants internes (non publiés dans la library)
> sont préfixés `.elements /` pour les distinguer des composants publics.

| Type | Nommage | Exemple |
|------|---------|----------|
| Composant public | minuscule, sans préfixe | `button`, `checkbox`, `dropdown` |
| Sous-composant interne | `.elements / {parent} / {name}` | `.elements / select / option` |
| Documentatif | `.documentation header` (type=component) | `.documentation header` |
| Présentation footer | `_DesignSystemFooter` | `_DesignSystemFooter` |
| Séparateur section | `_SectionMetadata` | `_SectionMetadata` |
| Slot | `⬦ {name}` (via `createSlot()`) | `⬦ leading`, `⬦ header`, `⬦ content` |

##### Matrice des composants à créer par famille

| Famille | Properties Figma | Slots | Sous-composants `.elements /` | Variables liées |
|---------|-----------------|-------|-------------------------------|------------------|
| **button** | type, size, state, loading | leading, trailing | `.elements / button / shadow`, `.elements / button / focus` | controlHeight, controlPadding, controlGap, iconSize, colorPrimary, radius |
| **text field** | type *(text/password/search/email/number)*, size, state, **status** *(none/error/warning/success)* | leading, trailing | — | controlHeight, controlPadding, controlGap, colorBorder, colorFeedback, radius |
| **dropdown** | size, state, **status** *(none/error/warning/success)*, open | leading | `.elements / menu item` | controlHeight, controlPadding, colorBorder, colorFeedback, radius |
| **checkbox** | size, state, checked | — | — | controlHeight (xs/sm), colorPrimary, radius/sm |
| **radio** | size, state, selected | — | — | controlHeight (xs/sm), colorPrimary |

##### Pattern Status — Architecture de validation transversale

> **Finding audit** : Le pattern `status = None | Error | Warning | Success` est
> observé dans Ant Design 5.10 et UUI pour TOUS les composants de formulaire.
> ds-init l'implémente déjà sur TextField et Select mais il doit être documenté
> comme **pattern réutilisable** pour tout composant de saisie.

**Composants qui DOIVENT implémenter status** :
- ✅ `text field` — implémenté (3.3 supra)
- ✅ `select` — implémenté (3.3 supra)
- ⬜ `checkbox group` — optionnel (validation de groupe, pas d'un seul checkbox)
- ⬜ `radio group` — optionnel (validation de groupe)
- ⬜ Tout futur composant de formulaire (date picker, textarea, file upload…)

**Architecture status réutilisable** :
```
Tout composant formulaire avec status :
├── Variant : status = None | Error | Warning | Success
├── Couleurs border par status :
│   ├── None    → {Neutral/colorBorder}      (gris par défaut)
│   ├── Error   → {Feedback/colorDanger}     (rouge)
│   ├── Warning → {Feedback/colorWarning}    (orange)
│   └── Success → {Feedback/colorSuccess}    (vert)
├── Helper Text (Auto Layout horizontal, gap=4)
│   ├── visible = (status ≠ None)
│   ├── [Status Icon] ← 16×16 vector (✕ error, ⚠ warning, ✓ success)
│   └── [Message] ← Text 12px, couleur = même que border status
└── Couleurs Helper Text :
    ├── Error   → text/icon = {Feedback/colorDanger}
    ├── Warning → text/icon = {Feedback/colorWarning}
    └── Success → text/icon = {Feedback/colorSuccess}
```

**Helper Text réutilisable** — Créer un sous-composant `.elements / helper-text` :
```javascript
const helperComp = figma.createComponent();
helperComp.name = ".elements / helper-text";
helperComp.layoutMode = "HORIZONTAL";
helperComp.itemSpacing = 4;
helperComp.counterAxisAlignItems = "CENTER";
helperComp.primaryAxisSizingMode = "AUTO";
// Icon (vector 16×16)
const icon = figma.createVector();
icon.resize(16, 16);
helperComp.appendChild(icon);
// Message text
const msg = figma.createText();
msg.fontSize = 12;
msg.characters = "Helper message";
helperComp.appendChild(msg);
msg.layoutSizingHorizontal = "FILL";
```

> **Avantage** : Un seul sous-composant helper-text instancié dans TextField, Select,
> et tout futur composant de formulaire. Modifier le style une fois = propagation partout.

##### Architecture Select — État ouvert obligatoire

Le Select DOIT avoir une property `open = true | false` et un `status` :
- **open=false** : Trigger seul (comme un input bordé avec chevron)
- **open=true** : Trigger + dropdown panel en dessous contenant des options listées

```
🔶 Select (Component Set)
├── Properties :
│   ├── Variant : size = SM | MD | LG
│   ├── Variant : state = Default | Hover | Focus | Disabled
│   ├── Variant : status = None | Error | Warning | Success
│   └── Variant : open = false | true
│
├── Auto Layout : vertical, gap=4
│
├── Trigger (Auto Layout horizontal)
│   ├── border → lié à variable {Neutral/colorBorder} (ou status color si status ≠ None)
│   ├── border-radius → lié à variable {Radius/md}
│   ├── height → liée à variable {Size/controlHeight/{size}}
│   ├── padding horizontal → lié à variable {Size/controlPadding/{size}}
│   │
│   ├── ⬦ leading (SlotNode via createSlot) — icône catégorie, flag, avatar…
│   ├── [Value] ← Text property
│   └── Chevron ↓ (vecteur SVG, PAS un frame vide)
│
├── Dropdown (visible seulement si open=true)
│   ├── Auto Layout : vertical, gap=0
│   ├── Shadow/md, Border/thin, Radius/md
│   ├── SelectOption × 4-5 (instances)
│   │   ├── Auto Layout horizontal, padding 8/12
│   │   ├── [Label] ← Text
│   │   └── État : default, hover, selected (✓ check)
│   └── Padding vertical 4px
│
├── Helper Text (Auto Layout horizontal, gap=4) — visible si status ≠ None
│   ├── [Status Icon] ← 16×16 (⚠ warning, ✕ error, ✓ success)
│   └── [Message] ← Text property, 12px, couleur liée au status
│
├── Couleurs border par status :
│   ├── None : {Neutral/colorBorder}
│   ├── Error : {Feedback/colorDanger}
│   ├── Warning : {Feedback/colorWarning}
│   └── Success : {Feedback/colorSuccess}
│
└── Couleurs Helper Text par status :
    ├── Error : text={Feedback/colorDanger}, icon=✕
    ├── Warning : text={Feedback/colorWarning}, icon=⚠
    └── Success : text={Feedback/colorSuccess}, icon=✓

🔶 SelectOption (Component auxiliaire)
├── Properties : state = default | hover | selected
├── Auto Layout horizontal, padding 8/12
├── [Label] ← Text
└── ✓ Check icon (visible seulement si selected)
```

##### Architecture Checkbox — Composant complet

Le checkbox est un Component Set avec l'architecture suivante :

```
🔶 checkbox (Component Set)
├── Properties :
│   ├── Variant : size = SM | MD
│   ├── Variant : state = Default | Hover | Focus | Disabled
│   └── Variant : checked = false | true
│
├── Auto Layout : HORIZONTAL, gap=8, counterAxisAlignItems=CENTER
│
├── Box (frame) ← taille selon size (SM=16×16, MD=20×20)
│   ├── cornerRadius = variable {Radius/sm} (~4px)
│   ├── Si checked=false :
│   │   ├── fills = [] (vide, transparent)
│   │   ├── strokes = [{color: Neutral/colorBorder}]
│   │   └── strokeWeight = 1.5, strokeAlign = "INSIDE"
│   ├── Si checked=true :
│   │   ├── fills = [{color: Primary/colorPrimary}] (fond bleu)
│   │   ├── strokes = [] (pas de border)
│   │   └── Check mark = VECTEUR SVG centré (voir ci-dessous)
│   └── layoutMode = "VERTICAL", primaryAxisAlignItems = "CENTER",
│       counterAxisAlignItems = "CENTER" (← centrage du check)
│
└── [Label] ← Text, 14px (MD) ou 12px (SM), Regular, fill={Neutral/colorText}
```

**⚠️ RÈGLE CRITIQUE — Check Mark** :
Le check mark DOIT être un **vecteur SVG** (polyline/path), **PAS** un caractère texte (“✓” ou “✔”).
Un caractère texte dépend de la font, a un line-height imprévisible, et déborde de la box.

**Construction du check mark :**
```javascript
// Créer le vecteur check DANS la box
const check = figma.createVector();
// Tailles proportionnelles à la box :
// MD (box 20×20) : check 12×9 — SM (box 16×16) : check 10×7
const w = size === 'MD' ? 12 : 10;
const h = size === 'MD' ? 9 : 7;
check.resize(w, h);
// Path d'un checkmark classique (polyline épaisse) :
check.vectorPaths = [{
  windingRule: "NONZERO",
  // Pour un check 12×9 : départ gauche-milieu, descente centre-bas, montée droite-haut
  data: "M 1 4.5 L 4.5 8 L 11 1"
}];
check.strokeWeight = 2;
check.strokes = [{type: 'SOLID', color: {r:1, g:1, b:1}}]; // blanc
check.fills = [];        // PAS de remplissage
check.strokeCap = "ROUND";
check.strokeJoin = "ROUND";
box.appendChild(check);  // Le box avec Auto Layout centré positionne le check
```

**Interdit :**
- ❌ Utiliser `figma.createText()` avec "✓" ou "✔" comme check mark
- ❌ Créer un check plus grand que 60% de la box (déborde visuellement)
- ❌ Utiliser x/y absolus pour positionner le check — toujours Auto Layout centré
- ❌ Oublier `fills = []` sur le vecteur (sinon remplissage noir par défaut)

##### Architecture Radio — Composant complet

```
🔶 radio (Component Set)
├── Properties :
│   ├── Variant : size = SM | MD
│   ├── Variant : state = Default | Hover | Focus | Disabled
│   └── Variant : selected = false | true
│
├── Auto Layout : HORIZONTAL, gap=8, counterAxisAlignItems=CENTER
│
├── Outer circle (frame) ← SM=16×16, MD=20×20
│   ├── cornerRadius = 9999 (cercle parfait)
│   ├── layoutMode = "VERTICAL", primaryAxisAlignItems = "CENTER",
│   │   counterAxisAlignItems = "CENTER"
│   ├── Si selected=false :
│   │   ├── fills = [] (transparent)
│   │   └── strokes = [{color: Neutral/colorBorder}], strokeWeight=1.5
│   ├── Si selected=true :
│   │   ├── fills = [{color: Primary/colorPrimary}] (fond bleu)
│   │   └── Inner dot (ellipse) = SM=6×6, MD=8×8, fills=[{blanc}]
│   └── strokes = []
│
└── [Label] ← Text, 14px (MD) ou 12px (SM), Regular, fill={Neutral/colorText}
```

**Règles Radio :**
- Le dot intérieur DOIT être centré via Auto Layout du parent, PAS via x/y absolus
- Utiliser `figma.createEllipse()` pour le dot, jamais un frame carré arrondi
| **Card** | variant *(default/elevated/outline)* | header, media, content, footer, actions | space, radius, shadow |
| **Alert** | type *(info/success/warning/error)* | icon, title, description, action | padding, colorFeedback, radius |
| **Badge** | variant, size | content | fontSize, colorPrimary, radius/full |
| **Modal** | size *(sm/md/lg/full)* | header, content, footer | padding, shadow/xl, radius |

#### 3.4 Appliquer le ComponentPageHeader

Pour chaque composant dans la page de présentation, placer une **instance** de ComponentPageHeader avec :
- `title` = nom du composant
- `description` = quand et pourquoi l'utiliser
- `status` = Draft (au lancement)

#### 3.5 Showcase — PRÉSENTATION DES COMPOSANTS

> **RÈGLE CRITIQUE** : La page de présentation ne contient QUE des instances.
> Les Component Sets (Main Components) sont sur la page `🔒 _Components`.

##### Mode "Tous sur une page" (défaut)

La page `Components` contient UN SEUL frame conteneur principal :

```
📄 Components (page)
├── Frame conteneur principal (Auto Layout vertical, gap=80, padding=24, largeur 2000)
│   ├── .documentation header (type=component, badge "Components", title="Components")
│   ├── _SectionMetadata (label="Button", specs="4 types × 3 sizes × 5 states")
│   ├── Showcase Light + Showcase Dark
│   ├── Séparateur ─
│   ├── _SectionMetadata (label="Text Field", specs="5 types × 3 sizes × 4 states × 4 status")
│   ├── Showcase Light + Showcase Dark
│   ├── ...
│   ├── _DesignSystemFooter
│   └── etc.
```

> **IMPORTANT** : Le frame conteneur principal DOIT être en Auto Layout vertical.
> Il ne doit JAMAIS y avoir de positionnement absolu entre les sections.

##### Mode "Une page par composant" — inspiré Starter UI

Chaque page composant suit ce pattern :

```
📄 buttons (page)
│
├── Frame conteneur principal (Auto Layout vertical, gap=80, padding=24, largeur 2000)
│
├── .documentation header instance (type=component)
│   badge: "Components"
│   breadcrumbs: "[DS Name] / Components / Buttons"
│   title: "Button"
│   description: "The Button is a clickable element that performs actions when pressed."
│
├── Content Frame (Auto Layout vertical, gap=80, paddingLeft=88, width=FILL)
│   │   → contenu effectif à 112px du bord (24+88), largeur ~1776px
│   │
│   ├── _SectionMetadata (label="Types", specs="Primary, Secondary, Ghost, Danger")
│   ├── ═══ SHOWCASE LIGHT ═══ (fond clair #F8F9FA / bg fond page)
│   │   ├── Section "Types" (Auto Layout vertical, gap=16)
│   │   │   └── Row : Primary, Secondary, Ghost, Danger
│   │   ├── Section "Sizes"
│   │   │   └── Row : SM, MD, LG
│   │   ├── Section "With Slots" (Auto Layout vertical, gap=16)
│   │   │   └── Row : [🔍 Search] [Submit →] [+ Add Item →] [⚙] (icon only)
│   │   │   (instances avec leading/trailing slots remplis par des icônes)
│   │   └── Section "States"
│   │       └── Row : Default, Hover, Active, Focus, Disabled
│   │
│   ├── ═══ SHOWCASE DARK ═══ (fond sombre via Neutral/colorBg en mode Dark)
│   │   ├── ⚠️ frame.setExplicitVariableModeForCollection(colorColId, darkModeId)
│   │   ├── fills liés à {Neutral/colorBg} (sera automatiquement sombre via le mode)
│   │   ├── Même sections que le light mais sur fond sombre
│   │   └── Toutes les instances identiques — le mode Dark applique les couleurs
│   │
│   ├── .documentation header instance (type=elements)
│   │   title: "Elements"
│   │   description: "Elements that are used to build components."
│   │
│   └── Section Elements (sous-composants)
│       └── Instances des `.elements /` liés à ce composant
│
└── _DesignSystemFooter instance
```

```
📄 inputs (page)
│
├── Frame conteneur principal (Auto Layout vertical, gap=80, padding=24, largeur 2000)
│
├── .documentation header instance (type=component)
│   badge: "Components"
│   breadcrumbs: "[DS Name] / Components / Text Field"
│   title: "Text Field"
│   description: "Text input field for forms. Supports leading/trailing slots and validation states."
│
├── Content Frame (Auto Layout vertical, gap=80, paddingLeft=88, width=FILL)
│   ├── _SectionMetadata (label="Types", specs="Text, Password, Search, Email, Number")
│   ├── ═══ SHOWCASE LIGHT ═══
│   │   ├── Section "Types" (Auto Layout vertical, gap=16)
│   │   │   └── Row : Text, Password, Search, Email, Number
│   │   ├── _SectionMetadata (label="With Slots", specs="Leading, Trailing, Both")
│   │   ├── Section "With Slots" (Auto Layout vertical, gap=16)
│   │   │   └── Row : [🔍|placeholder] [placeholder|👁] [👤|value|✓]
│   │   │   (instances avec leading/trailing slots remplis par des icônes)
│   │   ├── _SectionMetadata (label="Status", specs="None, Error, Warning, Success")
│   │   ├── Section "Status" (Auto Layout vertical, gap=24) ⚠ IMPORTANT
│   │   │   └── 4 colonnes côte à côte :
│   │   │       ├── None : input normal, pas de helper text
│   │   │       ├── Error : border rouge, helper "This field is required" en rouge
│   │   │       ├── Warning : border orange, helper "Password is weak" en orange
│   │   │       └── Success : border vert, helper "Username is available" en vert
│   │   │   (chaque instance a status=Error/Warning/Success + helper text visible)
│   │   └── Section "States"
│   │       └── Row : Default, Hover, Focus, Disabled
│   │
│   └── ═══ SHOWCASE DARK ═══
│       ├── frame.setExplicitVariableModeForCollection(colorColId, darkModeId)
│       ├── fills = [{type:"SOLID", boundTo: Neutral/colorBg}]
│       └── (même contenu — le mode Dark change automatiquement les couleurs)
│
└── _DesignSystemFooter instance
```
│
├── Frame conteneur principal (Auto Layout vertical, gap=80, padding=24, largeur 2000)
│
├── .documentation header instance (type=component)
│   badge: "Components"
│   breadcrumbs: "[DS Name] / Components / Select"
│   title: "Select"
│   description: "Dropdown select for choosing from a list. Supports leading slot and validation states."
│
├── Content Frame (Auto Layout vertical, gap=80, paddingLeft=88, width=FILL)
│   ├── ═══ SHOWCASE LIGHT ═══
│   │   ├── _SectionMetadata (label="Default", specs="Closed state")
│   │   ├── Section "Default" : Select closed (MD)
│   │   ├── _SectionMetadata (label="Open", specs="With dropdown panel")
│   │   ├── Section "Open" : Select open=true avec dropdown visible
│   │   ├── _SectionMetadata (label="With Leading Slot", specs="Icon in leading position")
│   │   ├── Section "With Leading Slot" : Select avec icône dans le leading slot
│   │   ├── _SectionMetadata (label="Status", specs="None, Error, Warning, Success")
│   │   ├── Section "Status" (Auto Layout vertical, gap=24) ⚠ IMPORTANT
│   │   │   └── 4 colonnes côte à côte :
│   │   │       ├── None : select normal, pas de helper text
│   │   │       ├── Error : border rouge, helper "Please select an option" en rouge
│   │   │       ├── Warning : border orange, helper "This selection may change" en orange
│   │   │       └── Success : border vert, helper "Selection confirmed" en vert
│   │   └── Section "States"
│   │       └── Row : Default, Hover, Focus, Disabled
│   │
│   └── ═══ SHOWCASE DARK ═══
│       ├── frame.setExplicitVariableModeForCollection(colorColId, darkModeId)
│       ├── fills = [{type:"SOLID", boundTo: Neutral/colorBg}]
│       └── (même contenu — le mode Dark change automatiquement les couleurs)
│
└── _DesignSystemFooter instance
```

> **⚠️ IMPORTANT** :
> - Le **showcase dark** n'est PAS juste un frame avec un fond sombre hardcodé.
>   C'est un frame avec `setExplicitVariableModeForCollection()` qui active le mode Dark.
>   Le fond DOIT utiliser la variable `Neutral/colorBg` (qui devient automatiquement sombre).
> - La section **Elements** montre les sous-composants internes (`.elements / button / shadow`, etc.)
>   qui ne sont pas publiés dans la library.

#### 3.6 Showcase avancés — Patterns optionnels

> **Finding audit** : Aucun des 10 DS audités ne propose d'anatomy diagram, de Do/Don't,
> de specs overlay ni de notes a11y. Ce sont pourtant des standards de documentation
> DS professionnelle (Material Design, Atlassian). Les ajouter distingue un DS mature.

Ces patterns sont **optionnels** — à ajouter en second pass ou sur demande de l'utilisateur.

##### Anatomy Diagram — Annotations des parties
Un composant agrandi avec des callouts sur chaque zone + dimensions :
```
┌──────────────────────────────────┐
│ [⬦leading] [Label] [⬦trailing]  │  ← Auto Layout horizontal
└──────────────────────────────────┘
│←─ controlGap ─→│
↑                    ↑
controlPadding    controlPadding
↑─── controlHeight ───↑
```
Pattern : Frame 800×400, composant centré à 2×, lignes (strokes 1px dashed), labels 10px.
Bénéfice : Les développeurs comprennent visuellement les règles de spacing.

##### Do / Don't — Exemples d'usage correct/incorrect
Layout 2 colonnes :
```
┌─────────── DO ───────────┐  ┌────────── DON'T ────────────┐
│  ✅ [illustration]       │  │  ❌ [illustration]           │
│  "Small button in dense  │  │  "Large button in dense      │
│   form — 32px height"    │  │   form — 56px wastes space"  │
└──────────────────────────┘  └──────────────────────────────┘
```
Pattern : 2 frames côte à côte, bordure gauche verte (Do) / rouge (Don't), 14px body.
Bénéfice : Guide les non-designers vers de bonnes décisions UX.

##### Specs Overlay — Dimensions annotées sur screenshot
Capture du composant avec des flèches et mesures :
- Height: 40px
- Padding H: 16px, V: 8px
- Gap: 8px (icon↔label)
- Border radius: 6px
- Font: 16px/SemiBold

Pattern : Screenshot en fond + lignes de cote (strokes magenta 1px) + labels 10px.
Bénéfice : Handoff design→dev — toutes les mesures en un seul endroit.

##### Accessibility Notes — Conformité a11y
Section documentaire :
```
✅ Touch target minimum : 44×44px (WCAG 2.5.5)
✅ Contraste couleur : 4.5:1 (WCAG AA) — texte sur fond
✅ Focus visible : outline 2px, offset 2px ({Primary/colorPrimary})
⚠️ Bouton icon-only : DOIT avoir aria-label="action"
⚠️ SVG dans bouton : DOIT avoir aria-hidden="true"
```
Bénéfice : Les développeurs implémentent l'accessibilité sans deviner.

> **Implémentation** : Créer ces sections APRÈS les Showcase Light/Dark de chaque composant,
> dans un frame optionnel "Advanced Documentation". Ne les générer que si l'utilisateur
> a sélectionné l'option "Documentation avancée" dans le Round 2.

### Phase 4 — Vérification et récapitulatif

#### 4.1 Vérification visuelle

Utiliser `mcp_figma_get_screenshot` sur les pages principales pour vérifier :
- Les couleurs sont cohérentes
- Les tailles de composants sont harmonieuses
- Les variables sont bien liées (pas de px hardcodés)
- Le ComponentPageHeader est instancié partout

#### 4.2 Récapitulatif

Produire un résumé de tout ce qui a été créé :

```markdown
## 🎨 DS Figma initialisé : [Nom du projet]

### Configuration
- Police : [font]
- Couleur primaire : [color]
- Radius : [style]
- Spacing base : [X]px
- Mode sombre : [oui/non] (natif via Variable Modes Figma)
- Icônes : [Flaticon UIcons / autre]

### Fichier Figma
- **fileKey** : [clé du fichier]
- **Pages** : [nombre] pages créées
- **Variable Collections** : 8 collections (Color, Typography, Size, Border, Space, Radius, Shadow, Breakpoint)
- **Nombre de variables** : [X] tokens

### Composants créés
- [ ] ComponentPageHeader (racine)
- [ ] PageLayout, SectionLayout, StackLayout, GridLayout (racine)
- [ ] Templates SaaS : dashboard, settings, list-detail, table, profile, onboarding
- [ ] Templates Landing : hero, features, pricing, testimonials, cta-banner, footer
- [ ] Button (Component Set : type × size × state)
- [ ] Input (Component Set : type × size × state)
- [ ] ...
- [ ] Icônes importées : [X] icônes Flaticon UIcons

### Prochaines étapes
1. Valider les couleurs et la typo avec les stakeholders
2. Créer les composants Tier 2
3. Tester les composants dans un prototype
4. Personnaliser les templates avec le contenu réel
```

## Règles

### Règles générales
- Toujours demander AVANT de créer — ne jamais supposer les préférences design
- Proposer des défauts basés sur la [base de connaissances](copilot-skill:/ds-audit/references/ds-knowledge-base.md) (Inter, radius 6px, etc.)
- Valider la cohérence entre les choix (ex: radius full + style corporate = incohérent)
- Si mode sombre demandé (défaut = oui), créer un second mode `Dark` dans la collection Color dès le départ (voir table d'inversion 2.2). Les composants n'ont PAS besoin de variantes dark séparées — le mode Figma s'applique automatiquement via les variables sémantiques.

### Règles de format visuel — Inspiration SlothUI
> Le DS utilise un style de présentation inspiré de SlothUI (Community DS).
> Chaque page est structurée comme une "page web" professionnelle avec header, contenu et footer.

- **Pages 2000px** de large — TOUTES les pages du DS utilisent cette largeur
- **Header** (`.documentation header`) : cornerRadius=40, border 1px, gradient décoratif subtil (ellipse avec couleur primaire, opacity~8%), titre 60px ExtraBold, badge catégorie (pill rempli primary), breadcrumbs avec logo DS
- **Footer** (`_DesignSystemFooter`) : même style arrondi, logo + tagline. Instancié en bas de chaque page principale
- **Barres de section** (`_SectionMetadata`) : 38px de haut, label gauche + specs droite + ligne pointillée. Introduisent chaque bloc de contenu
- **Layout par page** : Header en haut (padding 24px du bord), contenu centré (indent 112px → largeur 1776px), Footer en bas
- **Spacing généreux** : gap=80 entre les sections principales, gap=24-32 entre les éléments internes
- **Coins arrondis** : 40px sur header/footer, bordures subtiles (#E2E8F0 ou Neutral/colorBorder)

### Règles de nommage & tokens
- **Nommage sémantique obligatoire** → `Primary/colorPrimary`, `Size/controlHeight/md` — jamais `CP/9`
- **Toutes les dimensions de composants interactifs → liées aux variables Size** — JAMAIS de px hardcodés
- **8 Variable Collections minimum** — Color, Typography, Size, Border, Space, Radius, Shadow, Breakpoint
- **Nommage composants publics** : minuscule sans emoji (`button`, `checkbox`, `dropdown`)
- **Nommage sous-composants** : préfixe `.elements /` (`..elements / select / option`, `.elements / button / shadow`)
- **Nommage documentation** : `.documentation header` (Component Set avec 3 variants)
- **Emoji-préfixes** sur les pages Foundations (🎨, 🔤, 📏…) — PAS sur les pages composants

### Règles de structure & séparation)
- **Pages composants** (buttons, inputs…) : contiennent UNIQUEMENT des instances pour la documentation
- **Pages Foundations** : page unique `🧱 Foundations` avec un layer par foundation — JAMAIS vides
- **Page `🏗️ Layouts`** : présentation des Layout Components et Templates avec instanc
- **Pages Foundations** : contiennent du contenu visuel concret (swatches, type scale, etc.) — JAMAIS vides
- **Séparateurs `---`** : pages vides entre les sections dans le panneau latéral
- **Showcase light + dark** : chaque page composant a les deux versions sur fond clair et sombre

### Règle Dark Mode — CRITIQUE
> **Le mode Dark ne s'applique PAS automatiquement.** Créer les variables avec des valeurs Light/Dark
> ne suffit pas — il faut **explicitement activer** le mode sur chaque frame dark.

**Pattern obligatoire pour tout frame devant afficher en dark :**
```javascript
// 1. Récupérer la collection Color et ses modes
const collections = await figma.variables.getLocalVariableCollectionsAsync();
const colorCol = collections.find(c => c.name === 'Color');
const darkModeId = colorCol.modes.find(m => m.name === 'Dark').modeId;

// 2. Créer le frame dark showcase
const darkFrame = figma.createFrame();
darkFrame.name = "Showcase Dark";
darkFrame.layoutMode = "VERTICAL";

// 3. ⚠️ APPLIQUER LE MODE DARK explicitement
darkFrame.setExplicitVariableModeForCollection(colorCol.id, darkModeId);

// 4. Lier le fond à la variable sémantique (sera automatiquement sombre)
const neutralBgVar = allVars.find(v => v.name === 'Neutral/colorBg');
darkFrame.setBoundVariable('fills', 0, 'color', neutralBgVar.id);
// Alternative si setBoundVariable n'est pas dispo pour fills :
// Utiliser la valeur résolue du mode dark
```

**Sans `setExplicitVariableModeForCollection()`, le frame reste en Light et le dark mode est invisible.**
Cet appel propage le mode à TOUS les enfants du frame (instances, textes, rectangles).

**Erreurs courantes à éviter :**
- ❌ Mettre un fill hex hardcodé (#1A1A2E) → ne s'adapte pas aux variables
- ❌ Oublier `setExplicitVariableModeForCollection()` → frame identique au Light
- ❌ Utiliser `setBoundVariable` pour le mode → ce n'est pas la bonne API
- ✅ `setExplicitVariableModeForCollection(collectionId, modeId)` est la seule bonne méthode
- **Un Component Set par composant** avec des properties pour hierarchy, size, state, tone

### Règles d'Auto Layout — CRITIQUES
- **Auto Layout PARTOUT** — jamais de positionnement absolu dans les composants ni entre les sections
- **Component Sets** : Le frame parent du Component Set doit avoir `layoutMode = "VERTICAL"` ou `"HORIZONTAL"` avec `itemSpacing` suffisant pour que les variantes ne se chevauchent pas
- **Pages de présentation** : Un frame conteneur principal en Auto Layout vertical englobe tout le contenu de la page
- **Principe** : Si des éléments se superposent, c'est qu'il manque un Auto Layout parent

### Règles de contenu — NE JAMAIS LAISSER VIDE
- **ComponentPageHeader** : les instacréés via `component.createSlot()` (vrais SlotNode natifs Figma). Les slots vides doivent être en `visible = false` par défaut, ou contenir un placeholder coloré (petit carré gris 50%) — JAMAIS un frame transparent/blanc invisible. NE PAS créer de composant `.slot` séparé.
- **Slots ⬦ Leading / ⬦ Trailing** : les frames vides doivent être en `visible = false` par défaut, ou contenir un placeholder coloré (petit carré gris 50%) — JAMAIS un frame transparent/blanc invisible
- **Icônes** : Utiliser la bibliothèque **Flaticon UIcons Regular Rounded** (`fi-rr-*`) comme source par défaut. Lire les SVGs depuis `/Users/arnaudmorvan/Dropbox/uicons-regular-rounded/svg/` et les importer via `figma.createNodeFromSvg()`. Si un SVG spécifique n'est pas trouvé, créer des shapes vectorielles simples (chevron, check, X) — NE JAMAIS laisser un frame vide blanc
- **Chevron Select** : Doit être un vecteur visible (stroke gris foncé ≥ 1.5px, fills=[] pour éviter le remplissage blanc)
- **⚠️ Text field / Select — BORDER OBLIGATOIRE** : Le Component Set text field et select DOIVENT avoir des `strokes` visibles (1px, couleur {Neutral/colorBorder}, strokeAlign='INSIDE') sur TOUS les variants (y compris state=Default, status=None). Sans strokes, ces composants sont des rectangles blancs invisibles sur fond blanc. C'est la cause #1 de régressions visuelles dans les templates et le showcase.
- **⚠️ Button Secondary / Ghost — CONTRASTE OBLIGATOIRE** : Le button Secondary DOIT avoir un stroke visible ({Neutral/colorBorder}). Le button Ghost DOIT avoir un texte coloré visible ({Primary/colorPrimary} ou {Neutral/colorText}). Sans ça, ces boutons sont invisibles sur fond blanc dans les templates.
- **⚠️ Templates Customize — COMPLÉTUDE** : Chaque template sur la page Customize DOIT contenir tout le contenu prévu dans sa spec (champs de formulaire, boutons d'action, zones de contenu). NE JAMAIS livrer un template avec seulement le topbar/sidebar et un content vide. Si une zone n'a pas de contenu final, utiliser un frame placeholder visible (fond {Neutral/colorBgSubtle}, cornerRadius=12, texte descriptif centré).

### Règles de vérification
- **Vérifier visuellement** avec `mcp_figma_get_screenshot` sur CHAQUE page après création
- **Checklist par page** :
  - [ ] Aucun élément superposé / chevauchement
  - [ ] Auto Layout vertical sur le conteneur principal
  - [ ] ComponentPageHeader alimenté (pas "Component Name")
  - [ ] Tous les swatches/samples visibles et lisibles
  - [ ] Les slots vides sont masqués ou contiennent un placeholder visible

### Règle Utilisation des Templates et Slots — CRITIQUE
> **Les pages d'exemple DOIVENT utiliser les templates (PageLayout, SectionLayout, .templates/saas/dashboard, etc.) et le système de slots.**
> Ne JAMAIS construire une page à la main quand un template existe.

**Le problème** : Les slots dans les templates sont des **instances** du composant `.slot`. L'API Figma **interdit** `appendChild()` sur un nœud qui est une instance ou à l'intérieur d'une instance :
```
Error: Cannot move node. New parent is an instance or is inside of an instance.
```

**Pattern obligatoire — Instancier → Détacher → Remplir les slots :**
```javascript
// 1. Instancier le template
const templateComponent = await figma.getNodeByIdAsync('COMPONENT_ID');
const inst = templateComponent.createInstance();
inst.name = "MaPage";
inst.x = currentX; inst.y = 0;

// 2. Détacher l'instance (transforme en frame éditable)
const frame = inst.detachInstance();
frame.resize(1440, 900);
frame.layoutSizingHorizontal = "FIXED";
frame.layoutSizingVertical = "FIXED";

// 3. Trouver un slot par nom (recherche récursive)
function find(node, name) {
  if (node.name === name) return node;
  if (node.children) for (const c of node.children) {
    const r = find(c, name);
    if (r) return r;
  }
  return null;
}

// 4. Détacher le slot instance & supprimer le placeholder
const slot = find(frame, 'nom-du-slot');
const detachedSlot = slot.detachInstance(); // OBLIGATOIRE avant appendChild
const placeholder = detachedSlot.children?.find(c => c.name === 'Slot' && c.type === 'TEXT');
if (placeholder) placeholder.remove();

// 5. Configurer le slot et ajouter du contenu
detachedSlot.layoutMode = "VERTICAL";
detachedSlot.itemSpacing = 12;
const content = figma.createText();
// ... créer le contenu
detachedSlot.appendChild(content);
content.layoutSizingHorizontal = "FILL"; // APRÈS appendChild
```

**Templates disponibles et leurs slots :**
- **PageLayout** (`12:2`) : `header`, `sidebar` (dans body), `content` (dans body), `footer`
- **SectionLayout** (`12:12`) : `header`, `content`, `footer`
- **.templates/saas/dashboard** (`13:27`) : `logo`, `nav-items`, `sidebar-footer`, `breadcrumb`, `search`, `actions`, `page-header`, `stats-row`, `main-content`, `secondary-content`
- **.templates/saas/settings** (`13:52`) : mêmes slots sidebar + `settings-menu`, `section-header`, `form-fields`, `form-actions`
- **.templates/saas/profile** (`13:79`) : mêmes slots sidebar + slots profil

**Erreurs courantes :**
- ❌ `appendChild()` sur un slot sans le détacher d'abord → erreur "Cannot move node"
- ❌ Construire une page à la main au lieu d'utiliser un template existant
- ❌ Oublier de supprimer le texte placeholder "Slot" après détachement
- ✅ Toujours `detachInstance()` sur le template ET sur chaque slot avant d'y ajouter du contenu
- ✅ Pour masquer un slot inutilisé : `slot.visible = false` (ex: sidebar sur Login)

### Règle Positionnement des Frames sur une Page — CRITIQUE
> **Figma ne positionne PAS automatiquement les frames ajoutés à une page.**
> Tous les frames créés via `figma.createFrame()` démarrent à `x=0, y=0` et **se superposent**.

**Pattern obligatoire quand on crée plusieurs frames sur une même page :**
```javascript
// Espacer les frames horizontalement avec un gap de 100px
let currentX = 0;
const GAP = 100;

const frame1 = figma.createFrame();
frame1.name = "Page A";
frame1.resize(1440, 900);
frame1.x = currentX;
frame1.y = 0;
currentX += frame1.width + GAP;

const frame2 = figma.createFrame();
frame2.name = "Page B";
frame2.resize(1440, 900);
frame2.x = currentX;
frame2.y = 0;
currentX += frame2.width + GAP;
// etc.
```

**Alternatives de disposition :**
- **Horizontal** : `frame.x = index * (frameWidth + gap)` — pour des frames de même nature (écrans, exemples)
- **Grille** : Combiner x/y pour des grilles — `x = (i % cols) * (w + gap)`, `y = Math.floor(i / cols) * (h + gap)`
- **Vertical** : `frame.y = index * (frameHeight + gap)` — pour des frames empilés

**Taille standard des écrans d'exemple :**
- Toujours utiliser des dimensions standard : **1440×900** (desktop), **375×812** (mobile), **768×1024** (tablet)
- Les frames d'écran doivent être en `layoutSizingVertical = "FIXED"` — jamais HUG (sinon la hauteur collapse)
- Pattern : `frame.resize(1440, 900); frame.layoutSizingVertical = "FIXED";`

**Erreurs courantes :**
- ❌ Créer plusieurs frames sans définir `.x` / `.y` → ils se superposent tous à (0,0)
- ❌ Oublier de cumuler la largeur des frames précédents
- ❌ Laisser un écran en HUG vertical → hauteur non standard (ex: 1440×517 au lieu de 1440×900)
- ✅ Toujours tracker `currentX` ou `currentY` et incrémenter après chaque frame
- ✅ Toujours forcer `layoutSizingVertical = "FIXED"` sur les frames d'écran

### Règles Figma Plugin API
- Utiliser `await figma.setCurrentPageAsync(page)` (pas `figma.currentPage = page`)
- Appliquer `layoutSizingHorizontal = "FILL"` uniquement APRÈS `appendChild()` dans un parent Auto Layout
- Charger les fonts AVANT de créer du texte : `await figma.loadFontAsync({ family, style })`
- Pour les vecteurs (chevron, check) : toujours mettre `fills = []` pour éviter le remplissage noir par défaut
- **⚠️⚠️ RÈGLE CRITIQUE — HUG CONTENT** : Tout frame avec Auto Layout (`layoutMode != "NONE"`) DOIT avoir `primaryAxisSizingMode = "AUTO"` (= HUG content). C'est la règle la plus importante. Les seules exceptions sont les frames d'écran (ex: 1440×900) qui simulent un viewport. Ne JAMAIS mettre de tailles fixes sur l'axe principal d'un conteneur de contenu — laisser le contenu dicter la taille. Ce n'est PAS un problème si les layers n'ont pas tous la même hauteur.
- **⚠️ PIÈGE `resize()` + Auto Layout** : `resize(w, h)` force `primaryAxisSizingMode = "FIXED"`. Pour les frames verticaux auto-expandables, TOUJOURS remettre `primaryAxisSizingMode = "AUTO"` APRÈS `resize()`. Pattern sûr :
  ```js
  frame.resize(2000, 10); // width de la page
  frame.primaryAxisSizingMode = "AUTO"; // ré-activer l'expansion verticale — OBLIGATOIRE
  ```
- **⚠️ PIÈGE hauteur collapse** : Si un frame auto-layout a `height = 10` au lieu d'englober ses enfants, c'est que `primaryAxisSizingMode` est resté à `"FIXED"`. Fix bottom-up (enfants avant parents).
- **⚠️ PIÈGE `strokesAlign`** : La propriété Figma Plugin API est `strokeAlign` (singulier), PAS `strokesAlign`. Utiliser `strokesAlign` provoque `TypeError: object is not extensible`. Pattern correct :
  ```js
  frame.strokes = [paint]; frame.strokeWeight = 1; frame.strokeAlign = "INSIDE";
  ```
- **⚠️ PIÈGE `FILL` avant `appendChild`** : `layoutSizingHorizontal/Vertical = "FILL"` ne peut être défini que sur un enfant déjà rattaché à un parent auto-layout. Toujours faire `parent.appendChild(child)` AVANT `child.layoutSizingHorizontal = "FILL"`.
- **⚠️ Texte et tailles fixes** : Pour les textes larges (> 24px), préférer `textAutoResize = "HEIGHT"` (largeur fixe, hauteur auto) plutôt que `"NONE"` pour éviter le clipping.
- **Structure recommandée** : Un seul frame principal par page (contient doc header + sections + footer). Pattern :
  ```js
  const main = figma.createFrame();
  main.name = "Page Title";
  main.layoutMode = "VERTICAL";
  main.counterAxisSizingMode = "FIXED";
  main.resize(2000, 10);
  main.primaryAxisSizingMode = "AUTO"; // crucial
  main.itemSpacing = 80;
  main.paddingTop = 24; main.paddingBottom = 24;
  main.paddingLeft = 24; main.paddingRight = 24;
  main.fills = [{type:"SOLID",color:{r:1,g:1,b:1}}];
  // → Header et Footer à width=1952 (FILL dans parent)
  // → Content frames à paddingLeft=88 en plus (24+88=112 depuis le bord page)
  //   ou wrap dans un content frame width=1776 centré
  ```
- **Vérification des pages** : Après génération de contenu, vérifier avec `page.children.length` que les nodes sont bien persistés. Si 0, recharger la référence page avec `figma.getNodeByIdAsync()`.
- **⚠️ PIÈGE fills `a` (alpha) INTERDIT** : La propriété `fills` de Figma Plugin API N'ACCEPTE PAS la clé `a` pour l'opacité. Utiliser `opacity` sur le fill ou sur le node à la place :
  ```js
  // ❌ INTERDIT — provoque "Unrecognized key(s) in object: 'a'"
  frame.fills = [{type: 'SOLID', color: {r: 1, g: 1, b: 1, a: 0.7}}];
  
  // ✅ CORRECT — opacity sur le fill
  frame.fills = [{type: 'SOLID', color: {r: 1, g: 1, b: 1}, opacity: 0.7}];
  
  // ✅ CORRECT — opacity sur le node entier
  frame.fills = [{type: 'SOLID', color: {r: 1, g: 1, b: 1}}];
  frame.opacity = 0.7;
  ```
- **⚠️ PIÈGE composants blancs sur fond blanc** : Les instances de text field, select, button Secondary/Ghost, et card sont INVISIBLES sur fond blanc pur si elles n'ont pas de stroke visible. Causes fréquentes :
  - Text field sans strokes → rectangle blanc sur fond blanc = invisible
  - Button Secondary avec border transparent → invisible
  - Card default sans shadow ni border → invisible
  **Règle** : Tout composant de formulaire (text field, select) DOIT avoir `strokes = [{type: 'SOLID', boundVariable: Neutral/colorBorder}]` avec `strokeWeight ≥ 1` et `strokeAlign = 'INSIDE'` dans sa définition de Component. Ceci s'applique à TOUS les variants (y compris status=None, state=Default).
  **Règle templates** : Sur la page Customize, quand on crée des text fields/selects individuels (pas des instances du Component Set), toujours explicitement ajouter des strokes :
  ```js
  const inputRow = figma.createFrame();
  inputRow.fills = [{type: 'SOLID', color: {r:1, g:1, b:1}}];
  // ⚠️ OBLIGATOIRE — sans ça le champ est invisible
  const borderVar = allVars.find(v => v.name === 'Neutral/colorBorder');
  inputRow.strokes = [{type: 'SOLID', color: {r:0.89, g:0.91, b:0.94}}]; // fallback
  if (borderVar) inputRow.setBoundVariable('strokes', 0, 'color', borderVar.id);
  inputRow.strokeWeight = 1;
  inputRow.strokeAlign = 'INSIDE';
  ```
