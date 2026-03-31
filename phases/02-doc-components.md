# Phase 2 — Documentation Components

> **MCP Calls**: 1× `mcp_figma_use_figma`
> **Prerequisites**: Phase 1 completed (file + variables created)
> **Output**: 4 documentation components on the 🔒 _Components page
> **Verification**: Screenshot of the _Components page

## Components to Create

All on the `🔒 _Components` page. These are the "layout building blocks" used by all other phases.

### 2.1 `.documentation header` — Component Set (3 variants)

A professional page header, SlothUI-inspired.

**Variants:**
- `type=component` — for component pages (badge "Components")
- `type=documentation` — for Foundations pages (badge "Foundations")
- `type=elements` — for Elements sections (fixed title + description)

**Structure of each variant (component and documentation):**
```
Auto Layout vertical, gap=80, padding=40
├── cornerRadius=40, border 1px {Neutral/colorBorder}, fills={Neutral/colorBg}
├── Gradient Ellipse (decoration) — ellipse ~1150×1150, absolute top=-990 right=-450
│   radial gradient {Primary/colorPrimary}→transparent, opacity=0.08
├── Top Row (horizontal, justify=space-between, fill width)
│   ├── Title Block (vertical, gap=24, max-width=960)
│   │   ├── [title] ← Text property, 60px ExtraBold
│   │   └── [description] ← Text property, 20px Regular
│   └── Category Badge (horizontal, gap=10, padding=12/20)
│       ├── fills={Primary/colorPrimary}, cornerRadius=999
│       ├── [icon] ← 20×20 white
│       └── [label] ← "Components"/"Foundations", 16px Bold, white
└── Breadcrumb Bar (horizontal, justify=space-between, fill width)
    ├── DS Logo (40×40, cornerRadius=15) + [1st level] → [2nd level]
    └── [link] optional
```

**Variant `type=elements`:**
```
Auto Layout vertical, gap=4, padding=24/0
├── [title] ← "Elements", 32px Bold
└── [description] ← 16px Regular
```

### 2.2 `_DesignSystemFooter` — Main Component

**⚠️ Anti-clipping rule**: All footer text MUST have `textAutoResize = "WIDTH_AND_HEIGHT"` or be in an Auto Layout with `layoutSizingHorizontal = "FILL"` + `textAutoResize = "HEIGHT"`.
NEVER `resize()` text without setting `textAutoResize` afterwards.

```
Auto Layout horizontal, justify=space-between, padding=40, alignItems=CENTER
├── cornerRadius=40, border 1px {Neutral/colorBorder}, fills={Neutral/colorBg}
├── Gradient Ellipse (same decoration, mirrored, opacity=0.06)
├── Left: DS Logo (40×40) + [DS Name] 18px Bold
└── Right: [tagline] 14px Regular + [link] 14px SemiBold primary
```

```javascript
// Safe text pattern for footer
const text = figma.createText();
text.fontName = {family:'Inter', style:'Regular'};
text.fontSize = 14;
text.characters = "Design System v1.0";
text.textAutoResize = "WIDTH_AND_HEIGHT"; // ← REQUIRED
// OR in an auto layout:
parent.appendChild(text);
text.layoutSizingHorizontal = "FILL"; // text takes available width
text.textAutoResize = "HEIGHT"; // and grows in height
```

### 2.3 `_SectionMetadata` — Main Component

```
Auto Layout horizontal, justify=space-between, alignItems=CENTER, height=38
├── [label] ← Text property, 14px SemiBold
├── Dotted Line ← rectangle height=1, FILL, dashPattern=[4,4]
└── [specs] ← Text property, 13px Regular, secondary gray
```

### 2.4 `divider` — Main Component

Simple horizontal line:
```
Rectangle height=1, FILL width, fills={Neutral/colorBorder}, opacity=0.3
```

## Code to Populate Instances

```javascript
// Find a variant
const docHeaderSet = page.findOne(n => n.type === "COMPONENT_SET" && n.name === ".documentation header");
const variant = docHeaderSet.findOne(n => n.name === "type=component");
const inst = variant.createInstance();

// Populate text fields
function setText(instance, nodeName, text) {
  const node = instance.findOne(n => n.name === nodeName && n.type === "TEXT");
  if (node) node.characters = text;
}
setText(inst, "title", "Button");
setText(inst, "description", "Clickable element that performs actions.");
setText(inst, "1st level", "Components");
setText(inst, "2nd level", "Button");
setText(inst, "category label", "Components");
```

## Checklist

- [ ] `.documentation header` Component Set with 3 working variants
- [ ] `_DesignSystemFooter` Main Component visible and populatable
- [ ] `_SectionMetadata` Main Component with dotted line
- [ ] `divider` Main Component
- [ ] Screenshot verified — all 4 components visible on _Components
