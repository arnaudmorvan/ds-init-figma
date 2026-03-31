# Phase 7 — Showcase (Components Pages)

> **MCP Calls**: 1 call per component (5 calls for Tier 1)
> **Prerequisites**: Phase 2 (doc components) + Phase 4 (Icon CS) + Phase 5 (Component Sets)
> **Output**: Components Page with one separate layer per component (Light only)
> **Verification**: Screenshot after EACH component

## Page Structure

### One separate layer per component

**⚠️ CRITICAL**: Each component = 1 independent frame (layer) on the page, NOT a child frame of a shared parent frame.

```
📄 Components (page)
├── [Layer Button] — Frame 2000px, Auto Layout vertical
│   ├── .documentation header (type=component, titre="Button")
│   └── Showcase (Light uniquement)
├── [Layer Text Field] — Frame 2000px
│   ├── .documentation header (titre="Text Field")
│   └── Showcase
├── [Layer Select]
├── [Layer Checkbox]
├── [Layer Radio]
├── [Layer Card]
├── [Layer Alert]
├── [Layer Badge]
├── [Layer Avatar]
├── [Layer Tag]
├── [Layer Tabs]
└── [Layer Modal]
```

**No wrapping parent frame** — each layer is positioned with a 200px Y gap.
**No Dark Showcase** — Light only.
**No _DesignSystemFooter** on this page.

### Create a component layer

```javascript
function createCompLayer(name, yPos) {
  const frame = figma.createFrame();
  frame.name = name;
  frame.layoutMode = "VERTICAL";
  frame.counterAxisSizingMode = "FIXED";
  frame.resize(2000, 100);
  frame.primaryAxisSizingMode = "AUTO";
  frame.itemSpacing = 40;
  frame.paddingBottom = 40;
  frame.fills = [{type:'SOLID', color:{r:1,g:1,b:1}}];
  frame.clipsContent = false;
  frame.x = 0;
  frame.y = yPos;
  showPage.appendChild(frame);

  // Doc header — PAY ATTENTION to text field names
  // title → component name
  // category label → "Component"
  // 2nd level → component name
  // description → empty
  if (docVariant) {
    const hi = docVariant.createInstance();
    frame.appendChild(hi);
    hi.layoutSizingHorizontal = "FILL";
    findText(hi, 'title').characters = name;
    findText(hi, 'category label').characters = 'Component';
    findText(hi, '2nd level').characters = name;
    findText(hi, 'description').characters = '';
  }
  return frame;
}
```

**⚠️ `.documentation header` text fields**:
- `title` (not "Page Title")
- `description`
- `category label` (not "Badge")
- `1st level` → "ds linkedin"
- `2nd level` → component name

## ~~Dark Mode Rule~~ — REMOVED

**NO Dark Showcase.** Light only for each component.

## Section Labels Rule

Each group MUST be preceded by a visual label:

```
Frame section (vertical, gap=12)
├── Label 14px SemiBold, UPPERCASE, letterSpacing=2
├── Divider 1px
└── Row (horizontal, gap=16, wrap) → instances
```

## Icons in Showcase Rule

The "Sizes" and "With Slots" sections MUST show instances of the Icon Component Set.
**NEVER** use empty frames or emoji text.

## Content per Component

### Button
| Section | Content |
|---|---|
| TYPES | Primary+leading(plus)+"Create", Secondary+leading(download)+"Export", Ghost+leading(settings)+"Settings", Danger+leading(trash)+"Delete" |
| SIZES | SM(plus+"Add"), MD(plus+"Add Item"), LG(plus+"Add New Item") — icon scales with size |
| WITH SLOTS | leading only, trailing only, both, icon-only |
| LOADING | Primary loading=true, Secondary loading=true |
| STATES | Default, Hover, Active, Focus, Disabled (Primary MD) |
| COMPOSITION | 2 buttons (Primary "Save" + Ghost "Cancel") as form footer |

### Text Field
| Section | Content |
|---|---|
| TYPES | Text, Password(trailing eye), Search(leading search), Email(leading envelope), Number |
| SIZES | SM, MD, LG (each with leading search) |
| WITH SLOTS | Leading, Trailing, Both |
| STATUS | None, Error("This field is required"), Warning("Password is weak"), Success("Username is available") |
| STATES | Default, Hover, Focus, Disabled |
| COMPOSITION | 3 stacked inputs (Name, Email, Password) as vertical form |

### Select
| Section | Content |
|---|---|
| DEFAULT | Select closed (MD) |
| SIZES | SM(globe+"Language"), MD, LG |
| OPEN STATE | Select open=true with dropdown |
| WITH LEADING SLOT | globe+"Language", user+"Assignee", calendar+"Date range" |
| STATUS | None, Error, Warning, Success |
| STATES | Default, Hover, Focus, Disabled |
| COMPOSITION | Select "Country" + Select "City" side by side |

### Checkbox
| Section | Content |
|---|---|
| DEFAULT | unchecked + checked (MD) |
| SIZES | SM + MD (unchecked + checked) |
| GROUP | 4 vertical checkboxes (notification form) |
| STATES | Default, Hover, Focus, Disabled |

### Radio
| Section | Content |
|---|---|
| DEFAULT | unselected + selected (MD) |
| SIZES | SM + MD |
| GROUP | 4 vertical radios (plan choice) |
| STATES | Default, Hover, Focus, Disabled |

### Alert

**⚠️ CRITICAL SIZING**: Alert instances MUST use `layoutSizingHorizontal = "FILL"`, NOT FIXED at 480px. Place them in a vertical col with FILL as well.

| Section | Content |
|---|---|
| TYPES | Info, Success, Warning, Error — each instance in FILL width, stacked vertically with gap=12 |

```javascript
// ⚠️ MANDATORY for Alerts in the showcase
// The col containing alerts must be FILL
col.layoutSizingHorizontal = "FILL";
// Each instance must also be FILL
for (const inst of col.children) {
  if (inst.type === 'INSTANCE') inst.layoutSizingHorizontal = "FILL";
}
```

### Card
| Section | Content |
|---|---|
| TYPES | default, elevated, outline |

### Badge
| Section | Content |
|---|---|
| SIZES | SM, MD per variant |

### Avatar
| Section | Content |
|---|---|
| SIZES | XS, SM, MD, LG, XL |

### Tag
| Section | Content |
|---|---|
| VARIANTS | Default, with removable=true |

### Tabs
| Section | Content |
|---|---|
| DEFAULT | Default instance |

### Modal
| Section | Content |
|---|---|
| SIZES | SM, MD, LG |

## 🏗️ Layouts Page (separate call)

Presentation of Layout Components + Templates with instances.

For each layout/template:
1. _SectionMetadata
2. Instance with **colored slots** (pink/purple/blue/green/yellow palette) + text labels

```javascript
function colorSlot(instance, slotName, color, label) {
  const slot = instance.findOne(n => n.name === slotName);
  if (!slot) return;
  slot.fills = [{type: 'SOLID', color: color}];
  // Add centered text label inside the slot
}
```

**NEVER leave slots without fills** — otherwise white on white = invisible.

## Checklist

- [ ] Each component = 1 separate layer (independent frame on the page)
- [ ] .documentation header with `title` = component name (not "Page Title")
- [ ] Light Showcase only (NO Dark)
- [ ] Section labels (UPPERCASE) before each group
- [ ] COMPOSITION section for each component
- [ ] 200px Y gap between each layer
- [ ] Screenshot verified after each component
