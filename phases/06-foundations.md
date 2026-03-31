# Phase 6 — Foundations Page

> **MCP Calls**: 1 call per foundation (8 calls)
> **Prerequisites**: Phase 1 (variables) + Phase 2 (doc components)
> **Output**: 🧱 Foundations Page with visual content
> **Verification**: Screenshot after EACH foundation

## Principle: one call = one foundation

Each foundation is a **Section frame (layer)** in the main frame of the 🧱 Foundations page.

**First call**: create the main frame + doc header + first foundation.
**Subsequent calls**: add sections to the existing main frame.
**Last call**: add the footer.

### Main frame (created on the first call)

```javascript
const main = figma.createFrame();
main.name = "🧱 Foundations";
main.layoutMode = "VERTICAL";
main.counterAxisSizingMode = "FIXED";
main.resize(2000, 10);
main.primaryAxisSizingMode = "AUTO"; // HUG vertical
main.itemSpacing = 120;
main.paddingTop = 24;
main.paddingBottom = 24;
main.paddingLeft = 24;
main.paddingRight = 24;
main.fills = [{ type: "SOLID", color: { r: 1, g: 1, b: 1 } }];
main.clipsContent = false;
```

## Call 6a — Colors

`_SectionMetadata` (label="Colors", specs="Primitives + Semantics") + 3 frames:

### Primitive colors

**⚠️ CRITICAL**: Swatches MUST display actual colors, not gray.
Variable binding alone may not be visible in the screenshot if the variable resolves to a color identical to the initial fill.

**Required pattern**: resolve the actual variable value and use it as the base fill, THEN bind the variable on top.

#### Category → Variable Prefix Mapping

| Category  | Variable Prefix | Shades                                       |
| --------- | --------------- | -------------------------------------------- |
| Primary   | Blue            | 50,100,200,300,400,500,600,700,800,900,950   |
| Secondary | Purple          | 50,100,200,300,400,500,600,700,800,900,950   |
| Neutral   | Gray            | 0,50,100,200,300,400,500,600,700,800,900,950 |
| Success   | Green           | 50,100,200,300,400,500,600,700,800,900,950   |
| Warning   | Yellow          | 50,100,200,300,400,500,600,700,800,900,950   |
| Danger    | Red             | 50,100,200,300,400,500,600,700,800,900,950   |

#### Complete Structure Per Row

```javascript
// Retrieve variables AND collections to resolve values
const allVars = await figma.variables.getLocalVariablesAsync("COLOR");
const collections = await figma.variables.getLocalVariableCollectionsAsync();

function resolveColor(varObj) {
  if (!varObj) return { r: 0.9, g: 0.9, b: 0.9 };
  const col = collections.find((c) => c.id === varObj.variableCollectionId);
  const modeId = col.modes[0].modeId; // Light mode
  const val = varObj.valuesByMode[modeId];
  if (val && val.r !== undefined) return { r: val.r, g: val.g, b: val.b };
  if (val && val.type === "VARIABLE_ALIAS") {
    const target = allVars.find((v) => v.id === val.id);
    if (target) return resolveColor(target);
  }
  return { r: 0.9, g: 0.9, b: 0.9 };
}

function rgbToHex(c) {
  return (
    "#" +
    [c.r, c.g, c.b]
      .map((x) =>
        Math.round(x * 255)
          .toString(16)
          .padStart(2, "0"),
      )
      .join("")
      .toUpperCase()
  );
}

// For each category → create a HORIZONTAL row
for (const cat of categories) {
  const row = figma.createFrame();
  row.name = cat.label;
  row.layoutMode = "HORIZONTAL";
  row.counterAxisSizingMode = "AUTO";
  row.primaryAxisSizingMode = "AUTO";
  row.itemSpacing = 12;
  row.fills = [];

  // Label wrapper (vertical, paddingTop=16 to center vs swatches)
  const labelWrap = figma.createFrame();
  labelWrap.layoutMode = "VERTICAL";
  labelWrap.fills = [];
  labelWrap.counterAxisSizingMode = "AUTO";
  labelWrap.primaryAxisSizingMode = "AUTO";
  labelWrap.paddingTop = 16;
  const catLabel = figma.createText();
  catLabel.fontName = { family: "Inter", style: "Semi Bold" };
  catLabel.characters = cat.label;
  catLabel.fontSize = 14;
  catLabel.resize(80, catLabel.height);
  labelWrap.appendChild(catLabel);
  row.appendChild(labelWrap);

  // For each shade → swatchContainer vertical (swatch + shade + hex)
  for (const shade of cat.shades) {
    const varName = `${cat.prefix}/${shade}`;
    const v = allVars.find((vv) => vv.name === varName);
    const resolved = resolveColor(v);

    const container = figma.createFrame();
    container.layoutMode = "VERTICAL";
    container.counterAxisSizingMode = "AUTO";
    container.primaryAxisSizingMode = "AUTO";
    container.itemSpacing = 4;
    container.fills = [];

    // Swatch 48×48, cornerRadius=8, resolved fill + variable bound
    const swatch = figma.createFrame();
    swatch.resize(48, 48);
    swatch.cornerRadius = 8;
    swatch.fills = [{ type: "SOLID", color: resolved }];
    if (v) {
      const fills = JSON.parse(JSON.stringify(swatch.fills));
      fills[0] = figma.variables.setBoundVariableForPaint(fills[0], "color", v);
      swatch.fills = fills;
    }
    container.appendChild(swatch);

    // Shade label (e.g. "500") + hex label (e.g. "#2563EB")
    const sl = figma.createText();
    sl.fontName = { family: "Inter", style: "Regular" };
    sl.characters = shade;
    sl.fontSize = 10;
    sl.textAlignHorizontal = "CENTER";
    container.appendChild(sl);

    const hl = figma.createText();
    hl.fontName = { family: "Inter", style: "Regular" };
    hl.characters = rgbToHex(resolved);
    hl.fontSize = 9;
    hl.fills = [{ type: "SOLID", color: { r: 0.55, g: 0.55, b: 0.55 } }];
    hl.textAlignHorizontal = "CENTER";
    container.appendChild(hl);

    row.appendChild(container);
  }
  colorsSection.insertChild(insertIdx++, row);
}
```

#### Primitive Colors Checklist

- [ ] Each ROW = HORIZONTAL frame (AUTO × AUTO, itemSpacing=12, fills=[])
- [ ] Label wrapper = VERTICAL frame with paddingTop=16
- [ ] Each swatch = 48×48, cornerRadius=8, RESOLVED fill + variable bound
- [ ] Shade label (fontSize=10) + hex label (fontSize=9) under each swatch
- [ ] Use `resolveColor()` — NEVER gray or empty fills
- [ ] `insertChild` after the "Primitive Colors" title in colorsSection

### Semantic color tokens

Structure: `semantic-grid` frame (VERTICAL, itemSpacing=16) with:

- For each category: text label + HORIZONTAL row of token swatches

**Categories and tokens to display:**

| Category | Tokens                                              |
| -------- | --------------------------------------------------- |
| Primary  | colorPrimary, colorPrimaryHover, colorPrimaryActive |
| Neutral  | colorText, colorTextSecondary, colorBorder          |
| Feedback | colorSuccess, colorWarning, colorDanger, colorInfo  |

**Structure per token swatch:**

```
tokenContainer (HORIZONTAL, itemSpacing=8, fills=[])
  → circle (FRAME 32×32, cornerRadius=16, fill résolu + variable bound)
  → tokenName (TEXT fontSize=11)
```

**Pattern**: same `resolveColor()` as for Primitives — resolve the value, apply as fill, then bind.

→ **Screenshot** of the Foundations page.

## Call 6b — Typography

`_SectionMetadata` (label="Typography") + 2 frames:

### Typography styles

Visual table: Preview | Alias | Font | Weight | Size | Line height
Each row = text rendered in the actual font/size.

### Typography tokens

Two columns Desktop / Mobile with raw values.

→ **Screenshot**.

## Call 6c — Size

`_SectionMetadata` (label="Size") + 1 frame:

Table: Name | Token | Value
→ controlHeight (xs→xl), controlPadding, controlGap, iconSize

## Call 6d — Border

`_SectionMetadata` (label="Border") + 1 frame:

Visible thickness lines (none=0, thin=1, medium=2, thick=3).

## Call 6e — Space

`_SectionMetadata` (label="Spacing") + 1 frame:

Columns: Name | Token | Value (proportional visual bar + px value)
The colored bar grows proportionally.

## Call 6f — Radius

`_SectionMetadata` (label="Radius") + 2 frames:

### Semantic radius tokens

Each row: name + square with the applied radius + px value

### Component radius tokens

Table: Name | Token | mapping

## Call 6g — Shadow

`_SectionMetadata` (label="Shadow") + 1 frame:

4 cards (200×120) side by side: white background, cornerRadius, shadow applied via `.effects`.

## Call 6h — Breakpoint + Footer

`_SectionMetadata` (label="Breakpoint") + 2 frames:

### Breakpoint tokens

Table: breakpoint | min-width | max-width | columns | gutter | margin

### Grid visualization

4 device frames side by side (mobile 375, tablet 768, desktop 1440, large 1920).

- `_DesignSystemFooter` at the end of the main frame.

→ **Final screenshot** of the complete Foundations page.

## Checklist

- [ ] Main frame created with vertical Auto Layout, gap=120
- [ ] .documentation header (type=documentation) at the top
- [ ] 8 foundation sections with concrete visual content
- [ ] Each section introduced by \_SectionMetadata
- [ ] \_DesignSystemFooter at the bottom
- [ ] No empty sections
- [ ] Screenshot verified for each foundation
