# Phase 3 — Layouts and Templates

> **MCP Calls**: 3× `mcp_figma_use_figma` (1 layout components, 1 templates, 1 Layouts page)
> **Prerequisites**: Phase 1 (variables) + Phase 2 (doc components)
> **Output**: Layout/Template Components on 🔒 _Components + 🏗️ Layouts Page
> **Verification**: Screenshot of _Components + Screenshot of Layouts page

## ⛔ RULE — NO createSlot()

`createSlot()` **DOES NOT EXIST** in the Figma Plugin API. Calling it silently fails
and produces **empty** components.

"Slots" are **colored named frames** inside components.
The user replaces these frames with their own content after instantiation.

### createSlotFrame Helper

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
  // Default dimensions (override after appendChild)
  if (opts.width) slot.resize(opts.width, opts.height || 40);
  else slot.resize(200, opts.height || 40);

  // Label inside the slot
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

**Usage**: after `parent.appendChild(slot)`, apply `layoutSizingHorizontal = "FILL"` etc.

## Call 3a — Layout Components (on _Components)

> 4 Main Components, each a direct child of the _Components page.

### Load fonts first ⚠️

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
Each variant = auto layout (direction) with "children" slot FILL
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
Auto Layout wrap, child slots with calculated width
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
      // Create `cols` slots to show the grid
    for (let i = 0; i < cols; i++) {
      const cell = createSlotFrame(`cell-${i+1}`, {height: 80});
      comp.appendChild(cell);
      // Approximate proportional width
      const cellWidth = Math.floor((800 - (cols - 1) * gapVal) / cols);
      cell.resize(cellWidth, 80);
    }
    gridVariants.push(comp);
  }
}
const gridSet = figma.combineAsVariants(gridVariants, compPage);
gridSet.name = "GridLayout";
```

→ **Screenshot** of _Components after this call.

## Call 3b — Templates (on _Components, based on user config)

> Each template = 1 Component, direct child of `_Components`.
> Templates use the same `createSlotFrame()`.

### SaaS Templates

Each template is **1440×900** and uses DS variables + colored slots.

**dashboard** :
```
Component "template/dashboard", 1440×900, HORIZONTAL
├── slot "sidebar" — w=240, FILL height, violet
│   inner VERTICAL, gap=16, padding=16
│   ├── slot "logo" (h=40, jaune)
│   ├── slot "nav-items" (FILL, violet)
│   └── slot "sidebar-footer" (h=40, vert)
├── Frame "main" — VERTICAL, FILL both
│   ├── slot "topbar" — h=64, FILL width, rose
│   │   inner HORIZONTAL, centerV, padding-h=16, gap=16
│   │   ├── slot "breadcrumb" (FILL, jaune)
│   │   ├── slot "search" (w=240, jaune)
│   │   └── slot "actions" (w=120, jaune)
│   └── Frame "content-area" — VERTICAL, FILL, padding=32, gap=24
│       ├── slot "page-header" (h=48, rose)
│       ├── slot "stats-row" (h=120, bleu)
│       ├── slot "main-content" (FILL, bleu)
│       └── slot "secondary-content" (h=200, jaune)
```

**settings**: Same sidebar + topbar. Content = HORIZONTAL:
- slot "settings-menu" (w=220, violet)
- slot "form" (FILL, bleu)

**list-detail**: Same sidebar + topbar. Content = HORIZONTAL:
- slot "list-panel" (w=380, violet)
- slot "detail-panel" (FILL, bleu)

**table**: Same sidebar + topbar. Content = VERTICAL:
- slot "table-header" (h=48, rose)
- slot "table-content" (FILL, bleu)
- slot "table-pagination" (h=48, vert)

**profile**: Same sidebar + topbar. Content = VERTICAL:
- slot "profile-header" (h=200, rose)
- slot "profile-tabs" (h=48, jaune)
- slot "profile-content" (FILL, bleu)

**onboarding**: VERTICAL centered, no sidebar:
- slot "logo-bar" (h=64, FILL width, rose)
- Frame "step-container" (w=640, centered, FILL height, gap=32)
  - slot "step-header" (h=48, rose)
  - slot "step-content" (FILL, bleu)
  - slot "step-actions" (h=56, vert)

### Landing Templates

Sections stacked vertically, **1440px** wide, HUG vertical.

**hero** (Component Set, layout=centered|split) :
- `layout=centered` : VERTICAL centré, gap=24, padding=80, slots badge/headline/subheadline/cta-row/hero-visual empilés
- `layout=split` : HORIZONTAL, gap=48, padding=80, left-col (VERTICAL, FILL) with text slots, right-col slot hero-visual (FILL)

**features** (Component Set, columns=3|4) :
- slot "section-header" (FILL, pink)
- Grid HORIZONTAL wrap with "feature-1" to "feature-N" slots (calculated size)

**pricing** (Component) :
- slot "section-header" (pink)
- Row HORIZONTAL gap=24 : slot "plan-1" / "plan-2" / "plan-3" (FILL, blue)

**testimonials** (Component Set, layout=grid|single-featured) :
- slot "section-header" (pink)
- `grid` : 3 testimonial slots in row wrap
- `single-featured` : 1 large featured slot + 2 small slots in column

**cta-banner** (Component) :
- Background Primary/colorPrimary, padding=64, centered
- slot "headline" (yellow), slot "description" (yellow), slot "cta-row" (yellow)

**landing-footer** (Component) :
- VERTICAL, gap=32
- Row top HORIZONTAL : slot "brand-col" + "links-col-1" / "links-col-2" / "links-col-3"
- Row bottom HORIZONTAL : slot "copyright" + "legal-links"

→ **Screenshot** of _Components and verification.

## Call 3c — 🏗️ Layouts Page

> **Presentation page**, NOT Main Components.
> Instantiates or visually displays layouts/templates with their colored slots.

### Structure

```
Frame principal (2000px, Auto Layout vertical, gap=80, padding=24)
├── .documentation header (type=documentation, badge "Structure", title "Layouts & Templates")
│
├── Section "Layouts" (VERTICAL, gap=48)
│   ├── Title "Layouts" (32px Bold)
│   │
│   ├── Subsection "PageLayout"
│   │   ├── Name 16px SemiBold
│   │   ├── Description 14px Regular : "Full page structure with header/sidebar/content/footer"
│   │   └── PageLayout instance (or demo frame at 1:2 scale)
│   │
│   ├── Subsection "SectionLayout"
│   │   ├── Name + description
│   │   └── Instance or visual copy
│   │
│   ├── Subsection "StackLayout"
│   │   ├── Name + description
│   │   └── Row of examples (horizontal + vertical, gap=sm/md)
│   │
│   └── Subsection "GridLayout"
│       ├── Name + description
│       └── Examples columns=2 and columns=3
│
├── Section "SaaS Templates" (VERTICAL, gap=48)
│   ├── Title "SaaS Templates" (32px Bold)
│   └── For each template config:
│       ├── Name 16px SemiBold
│       ├── Description 14px Regular
│       └── Scaled-down instance (scale 0.5 or in a 720×450 frame)
│
├── Section "Landing Templates" (VERTICAL, gap=48)
│   ├── Title "Landing Templates" (32px Bold)
│   └── For each landing template config:
│       ├── Name 16px SemiBold
│       └── Instance or demo frame
│
└── _DesignSystemFooter
```

### Instantiation Pattern for the Page

```javascript
// Find the component by name
const allComponents = compPage.children.filter(c =>
  c.type === 'COMPONENT' || c.type === 'COMPONENT_SET'
);

// For a simple Component
const pageLayoutComp = allComponents.find(c => c.name === 'PageLayout');
if (pageLayoutComp) {
  const inst = pageLayoutComp.createInstance();
  // Scale down for the presentation page (optional)
  inst.resize(720, 450);
  section.appendChild(inst);
  inst.layoutSizingHorizontal = "FILL";
}

// For a Component Set — instantiate a specific variant
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

→ **Screenshot** of 🏗️ Layouts page for verification.

## Using Templates (for other phases)

Templates MUST be detached before filling the slots:

```javascript
const inst = templateComponent.createInstance();
const frame = inst.detachInstance(); // → editable frame

// Find a slot by name (recursive search)
function findSlot(node, name) {
  if (node.name === name) return node;
  if (children in node) for (const c of node.children) {
    const r = findSlot(c, name);
    if (r) return r;
  }
  return null;
}

const slot = findSlot(frame, 'content');
// Replace slot content
slot.children.forEach(c => c.remove()); // remove placeholder label
// Now appendChild works
myContent.forEach(child => slot.appendChild(child));
```

## Checklist

- [ ] Fonts loaded (Inter Medium, Regular)
- [ ] createSlotFrame() helper defined (colored frames with label)
- [ ] 4 Layout Components created on _Components (PageLayout, SectionLayout, StackLayout CS, GridLayout CS)
- [ ] Each Component = direct child of the _Components page (rule #16)
- [ ] SaaS Templates created on _Components (based on config)
- [ ] Landing Templates created on _Components (based on config)
- [ ] 🏗️ Layouts page with instances/previews of each layout + template
- [ ] Slots visible with colors (pink/purple/blue/green/yellow)
- [ ] Screenshots verified (_Components + Layouts page)
