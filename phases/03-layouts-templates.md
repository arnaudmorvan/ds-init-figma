# Phase 3 — Layouts et Templates

> **Appels MCP** : 3× `mcp_figma_use_figma` (1 layout components, 1 templates, 1 page Layouts)
> **Prérequis** : Phase 1 (variables) + Phase 2 (doc components)
> **Produit** : Layout/Template Components sur 🔒 _Components + Page 🏗️ Layouts
> **Vérification** : Screenshot de _Components + Screenshot page Layouts

## ⛔ RÈGLE — PAS de createSlot()

`createSlot()` **N'EXISTE PAS** dans le Plugin API Figma. L'appeler silencieusement échoue
et produit des composants **vides**.

Les "slots" sont des **frames colorées nommées** à l'intérieur des composants.
L'utilisateur remplace ces frames par son propre contenu après instanciation.

### Helper createSlotFrame

```javascript
// Couleurs de slots (voir style-guide.md)
const SLOT_COLORS = {
  header:    {r:1, g:0.878, b:0.929},         // #FFE0ED rose
  topbar:    {r:1, g:0.878, b:0.929},         // #FFE0ED rose
  sidebar:   {r:0.922, g:0.878, b:1},         // #EBE0FF violet
  nav:       {r:0.922, g:0.878, b:1},         // #EBE0FF violet
  content:   {r:0.878, g:0.941, b:1},         // #E0F0FF bleu
  main:      {r:0.878, g:0.941, b:1},         // #E0F0FF bleu
  footer:    {r:0.878, g:1, b:0.929},         // #E0FFED vert
  default:   {r:1, g:0.973, b:0.878},         // #FFF8E0 jaune
};

function createSlotFrame(name, opts = {}) {
  const slot = figma.createFrame();
  slot.name = name;
  slot.layoutMode = opts.direction || "VERTICAL";
  slot.primaryAxisSizingMode = "AUTO";
  slot.counterAxisSizingMode = "FIXED";
  slot.clipsContent = false;
  slot.fills = [{
    type: 'SOLID',
    color: SLOT_COLORS[name] || SLOT_COLORS.default,
    opacity: 0.5
  }];
  slot.cornerRadius = 4;
  slot.paddingTop = 8; slot.paddingBottom = 8;
  slot.paddingLeft = 8; slot.paddingRight = 8;
  // Dimensions par défaut (à override après appendChild)
  if (opts.width) slot.resize(opts.width, opts.height || 40);
  else slot.resize(200, opts.height || 40);

  // Label dans le slot
  const label = figma.createText();
  label.characters = name;
  label.fontSize = 11;
  label.fontName = { family: 'Inter', style: 'Medium' };
  label.fills = [{type:'SOLID', color:{r:0.4,g:0.4,b:0.4}}];
  label.textAutoResize = "WIDTH_AND_HEIGHT";
  slot.appendChild(label);

  return slot;
}
```

**Usage** : après `parent.appendChild(slot)`, appliquer `layoutSizingHorizontal = "FILL"` etc.

## Appel 3a — Layout Components (sur _Components)

> 4 Main Components, chacun enfant direct de la page _Components.

### Charger les fonts d'abord ⚠️

```javascript
await figma.loadFontAsync({ family: 'Inter', style: 'Medium' });
await figma.loadFontAsync({ family: 'Inter', style: 'Regular' });
```

### PageLayout (Component)

```
Component "PageLayout", 1440×900, layoutMode=VERTICAL
├── slot "header" — h=64, FILL width, rose
├── Frame "body" — HORIZONTAL, FILL both
│   ├── slot "sidebar" — w=240, FILL height, violet
│   └── slot "content" — FILL both, bleu
└── slot "footer" — h=48, FILL width, vert
```

```javascript
const pageLayout = figma.createComponent();
pageLayout.name = "PageLayout";
pageLayout.resize(1440, 900);
pageLayout.layoutMode = "VERTICAL";
pageLayout.primaryAxisSizingMode = "FIXED";
pageLayout.counterAxisSizingMode = "FIXED";
pageLayout.fills = [{type:'SOLID', color:{r:1,g:1,b:1}}];
pageLayout.clipsContent = true;

// Header slot
const headerSlot = createSlotFrame("header", {height: 64});
pageLayout.appendChild(headerSlot);
headerSlot.layoutSizingHorizontal = "FILL";

// Body
const body = figma.createFrame();
body.name = "body";
body.layoutMode = "HORIZONTAL";
body.primaryAxisSizingMode = "AUTO";
body.counterAxisSizingMode = "FIXED";
body.fills = [];
body.clipsContent = false;
pageLayout.appendChild(body);
body.layoutSizingHorizontal = "FILL";
body.layoutSizingVertical = "FILL";

// Sidebar slot
const sidebarSlot = createSlotFrame("sidebar", {width: 240, height: 100});
body.appendChild(sidebarSlot);
sidebarSlot.layoutSizingVertical = "FILL";

// Content slot
const contentSlot = createSlotFrame("content", {height: 100});
body.appendChild(contentSlot);
contentSlot.layoutSizingHorizontal = "FILL";
contentSlot.layoutSizingVertical = "FILL";

// Footer slot
const footerSlot = createSlotFrame("footer", {height: 48});
pageLayout.appendChild(footerSlot);
footerSlot.layoutSizingHorizontal = "FILL";

compPage.appendChild(pageLayout);
```

### SectionLayout (Component)

```
Component "SectionLayout", gap=16, padding=24, HUG vertical
├── slot "header" — FILL width, rose
├── slot "content" — FILL width, bleu
└── slot "footer" — FILL width, vert
```

```javascript
const sectionLayout = figma.createComponent();
sectionLayout.name = "SectionLayout";
sectionLayout.resize(800, 10);
sectionLayout.layoutMode = "VERTICAL";
sectionLayout.primaryAxisSizingMode = "AUTO";
sectionLayout.counterAxisSizingMode = "FIXED";
sectionLayout.itemSpacing = 16;
sectionLayout.paddingTop = 24; sectionLayout.paddingBottom = 24;
sectionLayout.paddingLeft = 24; sectionLayout.paddingRight = 24;
sectionLayout.fills = [{type:'SOLID', color:{r:1,g:1,b:1}}];
sectionLayout.clipsContent = false;

for (const slotName of ['header', 'content', 'footer']) {
  const s = createSlotFrame(slotName, {height: 40});
  sectionLayout.appendChild(s);
  s.layoutSizingHorizontal = "FILL";
}
compPage.appendChild(sectionLayout);
```

### StackLayout (Component Set)

```
Variants: direction=horizontal|vertical × gap=sm|md|lg (6 variants)
Chaque variant = auto layout (direction) avec slot "children" FILL
```

```javascript
const stackVariants = [];
for (const dir of ['horizontal', 'vertical']) {
  for (const [gapName, gapVal] of [['sm',8],['md',16],['lg',24]]) {
    const comp = figma.createComponent();
    comp.name = `direction=${dir}, gap=${gapName}`;
    comp.resize(400, 10);
    comp.layoutMode = dir === 'horizontal' ? "HORIZONTAL" : "VERTICAL";
    comp.primaryAxisSizingMode = "AUTO";
    comp.counterAxisSizingMode = "FIXED";
    comp.itemSpacing = gapVal;
    comp.fills = [];
    comp.clipsContent = false;
    const child = createSlotFrame("children", {height: 60});
    comp.appendChild(child);
    child.layoutSizingHorizontal = "FILL";
    stackVariants.push(comp);
  }
}
const stackSet = figma.combineAsVariants(stackVariants, compPage);
stackSet.name = "StackLayout";
```

### GridLayout (Component Set)

```
Variants: columns=1|2|3|4 × gap=sm|md|lg (12 variants)
Auto Layout wrap, slots enfants avec largeur calculée
```

```javascript
const gridVariants = [];
for (const cols of [1, 2, 3, 4]) {
  for (const [gapName, gapVal] of [['sm',8],['md',16],['lg',24]]) {
    const comp = figma.createComponent();
    comp.name = `columns=${cols}, gap=${gapName}`;
    comp.resize(800, 10);
    comp.layoutMode = "HORIZONTAL";
    comp.layoutWrap = "WRAP";
    comp.primaryAxisSizingMode = "AUTO";
    comp.counterAxisSizingMode = "FIXED";
    comp.itemSpacing = gapVal;
    comp.counterAxisSpacing = gapVal;
    comp.fills = [];
    comp.clipsContent = false;
    // Créer `cols` slots pour montrer la grille
    for (let i = 0; i < cols; i++) {
      const cell = createSlotFrame(`cell-${i+1}`, {height: 80});
      comp.appendChild(cell);
      // Largeur proportionnelle approximative
      const cellWidth = Math.floor((800 - (cols - 1) * gapVal) / cols);
      cell.resize(cellWidth, 80);
    }
    gridVariants.push(comp);
  }
}
const gridSet = figma.combineAsVariants(gridVariants, compPage);
gridSet.name = "GridLayout";
```

→ **Screenshot** de _Components après cet appel.

## Appel 3b — Templates (sur _Components, selon config utilisateur)

> Chaque template = 1 Component, enfant direct de `_Components`.
> Les templates utilisent les mêmes `createSlotFrame()`.

### Templates SaaS

Chaque template fait **1440×900** et utilise les variables DS + slots colorés.

**dashboard** :
```
Component "template/dashboard", 1440×900, HORIZONTAL
├── slot "sidebar" — w=240, FILL height, violet
│   intérieur VERTICAL, gap=16, padding=16
│   ├── slot "logo" (h=40, jaune)
│   ├── slot "nav-items" (FILL, violet)
│   └── slot "sidebar-footer" (h=40, vert)
├── Frame "main" — VERTICAL, FILL both
│   ├── slot "topbar" — h=64, FILL width, rose
│   │   intérieur HORIZONTAL, centerV, padding-h=16, gap=16
│   │   ├── slot "breadcrumb" (FILL, jaune)
│   │   ├── slot "search" (w=240, jaune)
│   │   └── slot "actions" (w=120, jaune)
│   └── Frame "content-area" — VERTICAL, FILL, padding=32, gap=24
│       ├── slot "page-header" (h=48, rose)
│       ├── slot "stats-row" (h=120, bleu)
│       ├── slot "main-content" (FILL, bleu)
│       └── slot "secondary-content" (h=200, jaune)
```

**settings** : Même sidebar + topbar. Content = HORIZONTAL :
- slot "settings-menu" (w=220, violet)
- slot "form" (FILL, bleu)

**list-detail** : Même sidebar + topbar. Content = HORIZONTAL :
- slot "list-panel" (w=380, violet)
- slot "detail-panel" (FILL, bleu)

**table** : Même sidebar + topbar. Content = VERTICAL :
- slot "table-header" (h=48, rose)
- slot "table-content" (FILL, bleu)
- slot "table-pagination" (h=48, vert)

**profile** : Même sidebar + topbar. Content = VERTICAL :
- slot "profile-header" (h=200, rose)
- slot "profile-tabs" (h=48, jaune)
- slot "profile-content" (FILL, bleu)

**onboarding** : VERTICAL centré, pas de sidebar :
- slot "logo-bar" (h=64, FILL width, rose)
- Frame "step-container" (w=640, centered, FILL height, gap=32)
  - slot "step-header" (h=48, rose)
  - slot "step-content" (FILL, bleu)
  - slot "step-actions" (h=56, vert)

### Templates Landing

Sections empilées verticalement, **1440px** de large, HUG vertical.

**hero** (Component Set, layout=centered|split) :
- `layout=centered` : VERTICAL centré, gap=24, padding=80, slots badge/headline/subheadline/cta-row/hero-visual empilés
- `layout=split` : HORIZONTAL, gap=48, padding=80, left-col (VERTICAL, FILL) avec slots texte, right-col slot hero-visual (FILL)

**features** (Component Set, columns=3|4) :
- slot "section-header" (FILL, rose)
- Grid HORIZONTAL wrap avec slots "feature-1" à "feature-N" (taille calculée)

**pricing** (Component) :
- slot "section-header" (rose)
- Row HORIZONTAL gap=24 : slot "plan-1" / "plan-2" / "plan-3" (FILL, bleu)

**testimonials** (Component Set, layout=grid|single-featured) :
- slot "section-header" (rose)
- `grid` : 3 slots testimonial en row wrap
- `single-featured` : 1 slot featured (large) + 2 slots small en colonne

**cta-banner** (Component) :
- Background Primary/colorPrimary, padding=64, centré
- slot "headline" (jaune), slot "description" (jaune), slot "cta-row" (jaune)

**landing-footer** (Component) :
- VERTICAL, gap=32
- Row top HORIZONTAL : slot "brand-col" + "links-col-1" / "links-col-2" / "links-col-3"
- Row bottom HORIZONTAL : slot "copyright" + "legal-links"

→ **Screenshot** de _Components et vérification.

## Appel 3c — Page 🏗️ Layouts

> **Page de présentation**, PAS des Main Components.
> Instancie ou affiche visuellement les layouts/templates avec leurs slots colorés.

### Structure

```
Frame principal (2000px, Auto Layout vertical, gap=80, padding=24)
├── .documentation header (type=documentation, badge "Structure", title "Layouts & Templates")
│
├── Section "Layouts" (VERTICAL, gap=48)
│   ├── Titre "Layouts" (32px Bold)
│   │
│   ├── Sous-section "PageLayout"
│   │   ├── Nom 16px SemiBold
│   │   ├── Description 14px Regular : "Structure page complète avec header/sidebar/content/footer"
│   │   └── Instance de PageLayout (ou frame démonstrative 1:2 scale)
│   │
│   ├── Sous-section "SectionLayout"
│   │   ├── Nom + description
│   │   └── Instance ou copie visuelle
│   │
│   ├── Sous-section "StackLayout"
│   │   ├── Nom + description
│   │   └── Row d'exemples (horizontal + vertical, gap=sm/md)
│   │
│   └── Sous-section "GridLayout"
│       ├── Nom + description
│       └── Exemples columns=2 et columns=3
│
├── Section "Templates SaaS" (VERTICAL, gap=48)
│   ├── Titre "Templates SaaS" (32px Bold)
│   └── Pour chaque template config :
│       ├── Nom 16px SemiBold
│       ├── Description 14px Regular
│       └── Instance à échelle réduite (scale 0.5 ou dans un frame 720×450)
│
├── Section "Templates Landing" (VERTICAL, gap=48)
│   ├── Titre "Templates Landing" (32px Bold)
│   └── Pour chaque template landing config :
│       ├── Nom 16px SemiBold
│       └── Instance ou frame démonstrative
│
└── _DesignSystemFooter
```

### Pattern d'instanciation pour la page

```javascript
// Trouver le composant par nom
const allComponents = compPage.children.filter(c =>
  c.type === 'COMPONENT' || c.type === 'COMPONENT_SET'
);

// Pour un Component simple
const pageLayoutComp = allComponents.find(c => c.name === 'PageLayout');
if (pageLayoutComp) {
  const inst = pageLayoutComp.createInstance();
  // Scale down pour la page de présentation (optionnel)
  inst.resize(720, 450);
  section.appendChild(inst);
  inst.layoutSizingHorizontal = "FILL";
}

// Pour un Component Set — instancier un variant spécifique
const stackSet = allComponents.find(c => c.name === 'StackLayout');
if (stackSet && stackSet.type === 'COMPONENT_SET') {
  const variant = stackSet.children.find(c => c.name === 'direction=horizontal, gap=md');
  if (variant) {
    const inst = variant.createInstance();
    section.appendChild(inst);
    inst.layoutSizingHorizontal = "FILL";
  }
}
```

→ **Screenshot** page 🏗️ Layouts pour vérification.

## Utilisation des templates (pour d'autres phases)

Les templates DOIVENT être détachés avant de remplir les slots :

```javascript
const inst = templateComponent.createInstance();
const frame = inst.detachInstance(); // → frame éditable

// Trouver un slot par nom (recherche récursive)
function findSlot(node, name) {
  if (node.name === name) return node;
  if (children in node) for (const c of node.children) {
    const r = findSlot(c, name);
    if (r) return r;
  }
  return null;
}

const slot = findSlot(frame, 'content');
// Remplacer le contenu du slot
slot.children.forEach(c => c.remove()); // supprimer le label placeholder
// Maintenant appendChild fonctionne
myContent.forEach(child => slot.appendChild(child));
```

## Checklist

- [ ] Fonts chargées (Inter Medium, Regular)
- [ ] createSlotFrame() helper défini (frames colorées avec label)
- [ ] 4 Layout Components créés sur _Components (PageLayout, SectionLayout, StackLayout CS, GridLayout CS)
- [ ] Chaque Component = enfant direct de la page _Components (règle #16)
- [ ] Templates SaaS créés sur _Components (selon config)
- [ ] Templates Landing créés sur _Components (selon config)
- [ ] Page 🏗️ Layouts avec instances/previews de chaque layout + template
- [ ] Slots visibles avec couleurs (rose/violet/bleu/vert/jaune)
- [ ] Screenshots vérifiés (_Components + page Layouts)
