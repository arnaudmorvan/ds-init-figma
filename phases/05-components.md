# Phase 5 — Component Sets

> **MCP Calls**: 1 call per component (5 calls for Tier 1)
> **Prerequisites**: Phase 1 (Size, Color, Radius variables) + Phase 4 (Icon CS)
> **Output**: Component Sets on 🔒 \_Components
> **Verification**: Screenshot after EACH component

## Rule #1: One component per MCP call

**DO NOT create all components in a single call.** Do:

- Call 5a → Button
- Call 5b → Text Field
- Call 5c → Select
- Call 5d → Checkbox
- Call 5e → Radio
- Call 5f → .elements / helper-text (shared sub-component)

Screenshot verification between each call.

## Rule #1bis: One LAYER per Component Set on \_Components

**⛔ CRITICAL**: Each Component Set MUST be a DIRECT child of the `_Components` page, NOT nested inside a shared frame. Figma treats direct children of a page as "layers" in the Layers panel.

```javascript
// ✅ CORRECT — each CS is a separate layer on the page
const compPage = figma.root.children.find((p) => p.name === "🔒 _Components");
const buttonSet = figma.combineAsVariants(buttonVariants, compPage);
buttonSet.name = "button";
buttonSet.y = lastY + 200; // Vertical spacing between layers

// ❌ FORBIDDEN — do NOT put CS inside a shared parent frame
const wrapper = figma.createFrame();
wrapper.name = "All Components";
compPage.appendChild(wrapper);
wrapper.appendChild(buttonSet); // ← NO, CS must be a direct child of the page
```

**Positioning**: Calculate `startY` by scanning `compPage.children` to find the lowest point, then add 200px gap.

## Rule #2: All dimensions bound to Size variables

```javascript
// ✅ REQUIRED — never hardcode px values
button.setBoundVariable("minHeight", heightVar.id);
button.setBoundVariable("paddingLeft", paddingVar.id);
button.setBoundVariable("paddingRight", paddingVar.id);
button.setBoundVariable("itemSpacing", gapVar.id);
```

## Rule #2bis: ALL colors bound to semantic variables (Dark mode)

**⛔ CRITICAL**: NO hardcoded colors in components. Every fill and stroke must be **bound** to a semantic variable for the Light/Dark switch to work.

### Color → Variable Mapping

| Color                                 | Semantic Variable            |
| ------------------------------------- | ---------------------------- |
| White background (input, card, modal) | `Neutral/colorBgElevated`    |
| Page background                       | `Neutral/colorBg`            |
| Light gray background (hover, subtle) | `Neutral/colorBgSubtle`      |
| Primary text                          | `Neutral/colorText`          |
| Secondary text                        | `Neutral/colorTextSecondary` |
| Tertiary text / disabled              | `Neutral/colorTextTertiary`  |
| Borders                               | `Neutral/colorBorder`        |
| Subtle borders                        | `Neutral/colorBorderSubtle`  |
| Primary blue                          | `Primary/colorPrimary`       |
| Blue hover                            | `Primary/colorPrimaryHover`  |
| Blue active                           | `Primary/colorPrimaryActive` |
| Blue light background                 | `Primary/colorPrimaryBg`     |
| Secondary purple                      | `Secondary/colorSecondary`   |
| Green success                         | `Feedback/colorSuccess`      |
| Yellow warning                        | `Feedback/colorWarning`      |
| Red danger                            | `Feedback/colorDanger`       |
| Blue info                             | `Feedback/colorInfo`         |
| Success background                    | `Feedback/colorSuccessBg`    |
| Warning background                    | `Feedback/colorWarningBg`    |
| Danger background                     | `Feedback/colorDangerBg`     |
| Info background                       | `Feedback/colorInfoBg`       |
| Icons (black)                         | `Neutral/colorText`          |
| White check/dot                       | `Neutral/colorBg`            |

### Required Binding Pattern

```javascript
// For each colored fill:
const colorVar = getColor("Primary/colorPrimary");
const resolved = resolveColor(colorVar); // see 06-foundations.md
node.fills = [{ type: "SOLID", color: resolved }]; // visible base color
const fills = JSON.parse(JSON.stringify(node.fills));
fills[0] = figma.variables.setBoundVariableForPaint(
  fills[0],
  "color",
  colorVar,
);
node.fills = fills;

// For each stroke:
const strokeVar = getColor("Neutral/colorBorder");
const resolvedStroke = resolveColor(strokeVar);
node.strokes = [{ type: "SOLID", color: resolvedStroke }];
const strokes = JSON.parse(JSON.stringify(node.strokes));
strokes[0] = figma.variables.setBoundVariableForPaint(
  strokes[0],
  "color",
  strokeVar,
);
node.strokes = strokes;

// ⚠️ INCLUDE WHITE BACKGROUNDS — they must become dark in Dark mode
// Input backgrounds, card bg, modal bg → Neutral/colorBgElevated
// Checkbox/radio unchecked box → Neutral/colorBgElevated
inputRow.fills = [{ type: "SOLID", color: { r: 1, g: 1, b: 1 } }];
const bgFills = JSON.parse(JSON.stringify(inputRow.fills));
bgFills[0] = figma.variables.setBoundVariableForPaint(
  bgFills[0],
  "color",
  getColor("Neutral/colorBgElevated"),
);
inputRow.fills = bgFills;
```

**⚠️ COMMON MISTAKE**: forgetting to bind white backgrounds. A `fills=[{type:'SOLID', color:{r:1,g:1,b:1}}]` without a variable stays white in Dark mode → visually broken component.

## Rule #3: Pattern A (flat variants) by default

One Component Set per component. If > 60 variants → Pattern A+ (separate wrapper).

---

## 5a — Button

```
🔶 button (Component Set)
├── Variants : type(Primary|Secondary|Ghost|Danger) × size(SM|MD|LG) × state(Default|Hover|Active|Focus|Disabled) + boolean loading
├── Auto Layout horizontal
│   ├── gap → {Size/controlGap/{size}}
│   ├── padding H → {Size/controlPadding/{size}}
│   ├── minHeight → {Size/controlHeight/{size}}
│   └── cornerRadius → {Radius/md}
├── ⬦ leading (SlotNode)
├── [Label] ← Text property
└── ⬦ trailing (SlotNode)
```

**Colors by type:**
| Type | Fill | Text | Border |
|---|---|---|---|
| Primary | {Primary/colorPrimary} | white | none |
| Secondary | transparent | {Neutral/colorText} | {Neutral/colorBorder} |
| Ghost | transparent | {Primary/colorPrimary} | none |
| Danger | {Feedback/colorDanger} | white | none |

**Colors by state:**
| State | Modification |
|---|---|
| Default | base colors |
| Hover | slightly lighter fill (e.g. PrimaryHover) |
| Active | darker fill (e.g. PrimaryActive) |
| Focus | outline 2px offset {Primary/colorPrimary} |
| Disabled | opacity=0.5 |

---

## 5b — Text Field

```
🔶 text field (Component Set)
├── Variants : type(Text|Password|Search|Email|Number) × size(SM|MD|LG) × state(Default|Hover|Focus|Disabled) × status(None|Error|Warning|Success)
├── Auto Layout vertical, gap=4
├── Input Row (horizontal)
│   ├── border → {Neutral/colorBorder} (ou status color)
│   ├── cornerRadius → {Radius/md}
│   ├── minHeight → {Size/controlHeight/{size}}
│   ├── padding H → {Size/controlPadding/{size}}
│   ├── ⬦ leading (SlotNode)
│   ├── [Value] ← Text property
│   └── ⬦ trailing (SlotNode)
### Helper Text (horizontal, gap=4) — visible if status ≠ None
│   ├── [Status Icon] 16×16
│   └── [Message] 12px
```

**⚠️ CRITICAL**: `strokes` visible on ALL variants (including status=None, state=Default).

---

## 5c — Select

```
🔶 select (Component Set)
├── Variants : size(SM|MD|LG) × state(Default|Hover|Focus|Disabled) × status(None|Error|Warning|Success) × open(false|true)
├── Auto Layout vertical, gap=4
├── Trigger (horizontal)
│   ├── same structure as text field
│   ├── ⬦ leading (SlotNode)
│   ├── [Value] ← Text property
│   └── Chevron ↓ (SVG vector, NOT an empty frame)
├── Dropdown (visible if open=true)
│   ├── Shadow/md, Border/thin, Radius/md
│   └── SelectOption × 4-5 instances
├── Helper Text (same pattern as text field)
```

---

## 5d — Checkbox

```
🔶 checkbox (Component Set)
├── Variants : size(SM|MD) × state(Default|Hover|Focus|Disabled) × checked(false|true)
├── Auto Layout horizontal, gap=8, counterAxisAlignItems=CENTER
├── Box (SM=16×16, MD=20×20)
│   ├── cornerRadius={Radius/sm}
│   ├── unchecked : fills=transparent, strokes={Neutral/colorBorder}
│   └── checked : fills={Primary/colorPrimary} (COULEUR PLEINE, PAS transparent), check mark VECTEUR SVG blanc
└── [Label] ← Text
```

**⚠️ CHECKED APPEARANCE — CRITICAL**:
When `checked=true`, the box MUST be **visually filled** with the primary color.
The white check mark on top creates the contrast.

```javascript
// ✅ CHECKED = solid color + check mark
if (checked === "true") {
  box.fills = [{ type: "SOLID", color: { r: 0.15, g: 0.39, b: 0.92 } }]; // visible base color
  bindFill(box, getColor("Primary/colorPrimary")); // + variable binding
  box.strokes = []; // NO stroke when checked
  // Check mark SVG
  const check = figma.createVector();
  check.vectorPaths = [{ windingRule: "NONZERO", data: "M 3 8 L 7 12 L 13 4" }]; // adapted to size
  check.strokeWeight = 2;
  check.strokes = [{ type: "SOLID", color: { r: 1, g: 1, b: 1 } }];
  check.fills = [];
  check.strokeCap = "ROUND";
  check.strokeJoin = "ROUND";
  check.resize(boxSize * 0.7, boxSize * 0.7);
  box.appendChild(check);
}

// ❌ FORBIDDEN: box.fills = [] when checked (invisible!)
// ❌ FORBIDDEN: same appearance for checked and unchecked
```

│ └── checked : fills={Primary/colorPrimary}, check mark VECTEUR SVG
└── [Label] ← Text

````

**⚠️ The check mark MUST be an SVG vector, NOT a text character "✓".**

```javascript
const check = figma.createVector();
check.vectorPaths = [{ windingRule: "NONZERO", data: "M 1 4.5 L 4.5 8 L 11 1" }];
check.strokeWeight = 2;
check.strokes = [{type: 'SOLID', color: {r:1,g:1,b:1}}];
check.fills = []; // ← REQUIRED
check.strokeCap = "ROUND";
check.strokeJoin = "ROUND";
````

---

## 5e — Radio

```
🔶 radio (Component Set)
├── Variants : size(SM|MD) × state(Default|Hover|Focus|Disabled) × selected(false|true)
├── Auto Layout horizontal, gap=8, counterAxisAlignItems=CENTER
├── Outer circle (SM=16×16, MD=20×20)
│   ├── cornerRadius=9999
│   ├── unselected : fills=transparent, strokes={Neutral/colorBorder}
│   └── selected : fills={Primary/colorPrimary} (COULEUR PLEINE), inner dot (ellipse SM=6×6, MD=8×8, blanc)
└── [Label] ← Text
```

**⚠️ SELECTED APPEARANCE — CRITICAL**:
When `selected=true`, the outer circle MUST be **filled with the primary color**.
The inner white dot creates the contrast.

```javascript
// ✅ SELECTED = solid circle + white dot
if (selected === "true") {
  outer.fills = [{ type: "SOLID", color: { r: 0.15, g: 0.39, b: 0.92 } }]; // base color
  bindFill(outer, getColor("Primary/colorPrimary")); // + variable
  outer.strokes = []; // NO stroke
  // White dot
  const dot = figma.createEllipse();
  dot.resize(dotSize, dotSize);
  dot.fills = [{ type: "SOLID", color: { r: 1, g: 1, b: 1 } }];
  outer.layoutMode = "VERTICAL";
  outer.primaryAxisAlignItems = "CENTER";
  outer.counterAxisAlignItems = "CENTER";
  outer.appendChild(dot);
}

// ❌ FORBIDDEN: same appearance for selected and unselected
// The user MUST see a clear difference (filled vs empty)
```

---

## 5f — .elements / helper-text

Shared sub-component for text field and select:

```
🔶 .elements / helper-text (Main Component)
├── Auto Layout horizontal, gap=4, counterAxisAlignItems=CENTER
├── [Status Icon] ← vector 16×16
└── [Message] ← Text 12px, FILL
```

---

## Tier 2+ Components (if selected)

Same pattern: **one MCP call per component**, screenshot after each.

| Component | Main Variants                       | Slots                                   |
| --------- | ----------------------------------- | --------------------------------------- |
| Card      | variant(default\|elevated\|outline) | header, media, content, footer, actions |
| Modal     | size(sm\|md\|lg)                    | header, content, footer                 |
| Tabs      | —                                   | tab items                               |
| Alert     | type(info\|success\|warning\|error) | icon, title, description, action        |

### Alert — Detailed Specification

```
🔶 alert (Component Set)
├── Variants : type(info|success|warning|error)
├── Auto Layout HORIZONTAL, gap=12
│   ├── counterAxisSizingMode = "AUTO" ← ⚠️ CRITICAL (not FIXED otherwise 10px height!)
│   ├── primaryAxisSizingMode = "FIXED"
│   ├── padding: 12/16/12/16
│   └── cornerRadius = {Radius/md} (ex: 6)
├── stripe (Frame 3×FILL)
│   ├── layoutSizingVertical = "FILL" ← stretch full height
│   ├── cornerRadius = 2
│   └── fills = type color (info=blue, success=green, warning=yellow, error=red)
├── Icon Frame (20×20)
│   └── Vector SVG (type icon)
└── text-content (Frame vertical, gap=2)
    ├── layoutMode = "VERTICAL"
    ├── primaryAxisSizingMode = "AUTO" ← ⚠️ CRITICAL (hug vertical)
    ├── counterAxisSizingMode = "AUTO"
    ├── [Title] ← Text 14px Semi Bold ("Info", "Success", "Warning", "Error")
    └── [Description] ← Text 13px Regular
```

**Colors by type:**
| Type | Fill (opacity 8%) | Stripe | Icon |
|---|---|---|---|
| info | primary blue | primary blue | ℹ️ SVG vector |
| success | green | green | ✅ SVG vector |
| warning | yellow | yellow | ⚠️ SVG vector |
| error | red | red | ❗ SVG vector |

**⚠️ KNOWN BUGS TO AVOID:**

1. `counterAxisSizingMode = "FIXED"` → 10px height, crushed/overlapping alerts. **ALWAYS "AUTO".**
2. `text-content` with `primaryAxisSizingMode = "FIXED"` → clipped text. **ALWAYS "AUTO".**
3. `stripe` without `layoutSizingVertical = "FILL"` → tiny strip (1px). **ALWAYS "FILL".**
4. After variant creation, **reposition** all CS children with 20px gap and **resize the CS** to encompass all variants.

```javascript
// ⚠️ REQUIRED after creating the alert Component Set
// Variants are stacked at y=0 by default → reposition them
let currentY = 0;
for (const v of alertCS.children) {
  v.x = 0;
  v.y = currentY;
  currentY += v.height + 20;
}
alertCS.resize(480, currentY - 20);
```

| Badge | variant, size(sm\|md) | content |
| Navbar | — | logo, nav-links, search, user |
| Sidebar | collapsed(false\|true) | logo, nav-items, footer |
| Breadcrumb | — | items |
| Pagination | size(sm\|md) | — |
| Table | striped(false\|true) | header, rows |
| Avatar | size(xs\|sm\|md\|lg\|xl) | — |
| Tag | variant, removable(false\|true) | — |

## Checklist

- [ ] Each component created in a separate MCP call
- [ ] Screenshot verified after each component
- [ ] All dimensions bound to Size variables (no hardcoded px)
- [ ] **ALL colors bound to semantic variables (no hardcoded hex)**
- [ ] **White backgrounds (input-row, trigger, card, modal) → Neutral/colorBgElevated**
- [ ] **Black icons → Neutral/colorText**
- [ ] **White check marks / dots → Neutral/colorBg**
- [ ] Strokes visible on text field and select (all variants)
- [ ] Check mark = SVG vector (not text character)
- [ ] Native SlotNodes (createSlot) for leading/trailing
- [ ] Alert: counterAxisSizingMode="AUTO" + text-content primaryAxisSizingMode="AUTO" + stripe FILL
- [ ] Alert: variants repositioned with 20px gap + CS resize after creation
- [ ] Showcase Alert: instances with layoutSizingHorizontal="FILL" (not FIXED at 480px)
- [ ] **Dark mode test: create a frame with setExplicitVariableModeForCollection → verify visually**
