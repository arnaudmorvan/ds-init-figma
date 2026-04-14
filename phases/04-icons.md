# Phase 4 — Icons

> **MCP Calls**: 2× `mcp_figma_use_figma` (1 for Icon Component Set, 1 for Icons page)
> **Prerequisites**: Phase 1 (Color/Size variables exist)
> **Output**: Icon Component Set on \_Components + ⚙️ Icons Page
> **Verification**: Screenshot of Icons page
> **⚠️ MUST complete before Phase 5** — Components use INSTANCE references to the icon CS

## Icon Source — `iconBase/` folder

Icons are stored in `ds-init/iconBase/` as plain SVG files (viewBox `0 0 24 24`).
This is a minimal curated set — fast to init, easy to extend later.

**Default icons** (4 files):

| File                   | Variant name            | Usage                          |
| ---------------------- | ----------------------- | ------------------------------ |
| `user.svg`             | `name=user`             | Default icon for DS components |
| `exclamation.svg`      | `name=exclamation`      | Alerts, helper-text warnings   |
| `check-circle.svg`     | `name=check-circle`     | Success states, checkbox       |
| `triangle-warning.svg` | `name=triangle-warning` | Danger/warning states          |

### Pre-check

```bash
ls ds-init/iconBase/*.svg
```

If the folder is missing or empty → ask the user for SVG files to place there.

## ⛔ ABSOLUTE RULE — Use REAL SVGs, never inline paths

**FORBIDDEN** to draw icons by hand with simplified paths (circles, basic lines).
Icons MUST come from `iconBase/` SVG files.

**Required workflow**:

1. **Read** ALL SVG files with `read_file` or `cat` **BEFORE** the MCP call
2. **Inject** the **exact** SVG strings into `figma.createNodeFromSvg()` in the MCP code

**❌ FORBIDDEN**:

```javascript
// NEVER invent paths
const svgStrings = {
  user: '<svg viewBox="0 0 24 24"><circle cx="12" cy="8" r="4"/></svg>',
};
```

**✅ REQUIRED**:

```javascript
// Use the EXACT content read from the file
const svgStrings = {
  user: '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M12,12.5A..." /></svg>',
};
```

## Call 4a — Icon Component Set (on \_Components)

### Architecture

```
🔶 icon (Component Set)  — 1 variant per SVG, single axis
├── name=user
├── name=exclamation
├── name=check-circle
└── name=triangle-warning
```

No size/color variants — keep the CS minimal. Size = 24×24. Color = bound to `Neutral/colorText`.

### Creation Pattern

```javascript
// 1. Read SVGs BEFORE MCP call (via read_file / cat)
// 2. Inject exact SVG strings

const svgStrings = {
  user: `...exact SVG content...`,
  exclamation: `...exact SVG content...`,
  "check-circle": `...exact SVG content...`,
  "triangle-warning": `...exact SVG content...`,
};

// 3. Resolve Neutral/colorText variable
const collections = await figma.variables.getLocalVariableCollectionsAsync();
const semanticCol = collections.find((c) => c.name === "Semantic colors");
const allVars = await Promise.all(
  semanticCol.variableIds.map((id) => figma.variables.getVariableByIdAsync(id)),
);
const colorTextVar = allVars.find((v) => v.name === "Neutral/colorText");
const modeId = semanticCol.modes[0].modeId;

// Resolve literal RGB (handle VARIABLE_ALIAS)
let resolvedColor = { r: 0.07, g: 0.09, b: 0.15 }; // fallback dark text
const rawVal = colorTextVar.valuesByMode[modeId];
if (rawVal && rawVal.r !== undefined) {
  resolvedColor = { r: rawVal.r, g: rawVal.g, b: rawVal.b };
}

// 4. Create icon components
const _componentsPage = figma.root.children.find(
  (p) => p.name === "_Components",
);
const iconComponents = [];

for (const [iconName, svgContent] of Object.entries(svgStrings)) {
  const comp = figma.createComponent();
  comp.name = `name=${iconName}`;
  comp.resize(24, 24);
  comp.layoutMode = "VERTICAL";
  comp.primaryAxisAlignItems = "CENTER";
  comp.counterAxisAlignItems = "CENTER";
  comp.primaryAxisSizingMode = "FIXED";
  comp.counterAxisSizingMode = "FIXED";
  comp.fills = [];

  // SVG import via createNodeFromSvg
  const svgFrame = figma.createNodeFromSvg(svgContent);

  // Flatten all vectors, apply color + variable binding
  const vectors = svgFrame.findAll((n) => n.type === "VECTOR");
  for (const v of vectors) {
    comp.appendChild(v);
    v.resize(20, 20);
    v.constraints = { horizontal: "CENTER", vertical: "CENTER" };
    // Set literal fill THEN bind variable
    v.fills = [{ type: "SOLID", color: resolvedColor }];
    v.setBoundVariable("fills", 0, "color", colorTextVar.id);
  }
  svgFrame.remove();

  iconComponents.push(comp);
}

// 5. Combine as variants
const iconSet = figma.combineAsVariants(iconComponents, _componentsPage);
iconSet.name = "icon";
iconSet.layoutMode = "HORIZONTAL";
iconSet.itemSpacing = 20;
iconSet.primaryAxisSizingMode = "AUTO";
iconSet.counterAxisSizingMode = "AUTO";
iconSet.fills = [];
```

### ⚠️ Critical: VARIABLE_ALIAS handling

`Neutral/colorText` in Light mode may resolve to a `VARIABLE_ALIAS` (reference to another variable), not a literal `{r,g,b}`.
**Always** check `rawVal.type === 'VARIABLE_ALIAS'` and use the fallback color `{r:0.07, g:0.09, b:0.15}`.
The `setBoundVariable` call will still work — the alias is only a problem for setting literal fills.

→ **Screenshot** of \_Components — verify the icon Component Set is visible with 4 clean variants.

## Call 4b — ⚙️ Icons Page

### Structure

```
Page "⚙️ Icons" (Auto Layout vertical, gap=40, padding=60)
├── Title frame (Auto Layout vertical, gap=8)
│   ├── "Icons" — 32px Bold, Neutral/colorText
│   └── Subtitle — 16px Regular, Neutral/colorTextSecondary
├── Icon grid (Auto Layout HORIZONTAL, wrap, gap=24)
│   └── For each icon:
│       └── Card 120×120 (Auto Layout vertical, gap=12, center)
│           ├── Icon instance 24×24, fills Neutral/colorText
│           └── Label 12px Regular, Neutral/colorTextSecondary
```

### Reading SVGs for the page

```bash
# Read all iconBase SVGs before MCP call
for f in ds-init/iconBase/*.svg; do
  echo "=== $(basename "$f" .svg) ==="
  cat "$f"
done
```

## Adding Icons Later

To extend the icon set:

1. Add new `.svg` files (viewBox `0 0 24 24`) to `ds-init/iconBase/`
2. Re-run Phase 4a or add variants to the existing icon CS
3. Components using `type: INSTANCE, component: "icon"` in CSpec will automatically have access

## Checklist

- [ ] `iconBase/` folder verified with SVG files
- [ ] SVGs read with exact content (never invented)
- [ ] Icon Component Set created on \_Components (1 variant per SVG, 24×24)
- [ ] All icon fills bound to `Neutral/colorText` variable
- [ ] ⚙️ Icons page with grid of icon cards
- [ ] Screenshots verified — icons render cleanly at 24×24
