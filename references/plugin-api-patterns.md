# Figma Plugin API — Reusable Patterns

> Tested code snippets for common operations.
> Copy-paste directly into `mcp_figma_use_figma`.

## Basic Helpers

### hexToRgb — Hex to Figma color conversion

```javascript
function hexToRgb(hex) {
  hex = hex.replace('#', '');
  return {
    r: parseInt(hex.substring(0, 2), 16) / 255,
    g: parseInt(hex.substring(2, 4), 16) / 255,
    b: parseInt(hex.substring(4, 6), 16) / 255,
  };
}
```

### Main page frame creation

```javascript
async function createPageFrame(pageName, width = 2000) {
  const main = figma.createFrame();
  main.name = pageName;
  main.layoutMode = "VERTICAL";
  main.counterAxisSizingMode = "FIXED";
  main.resize(width, 10);
  main.primaryAxisSizingMode = "AUTO"; // HUG vertical
  main.itemSpacing = 80;
  main.paddingTop = 24;
  main.paddingBottom = 24;
  main.paddingLeft = 24;
  main.paddingRight = 24;
  main.clipsContent = false;
  main.fills = [{type: "SOLID", color: {r: 1, g: 1, b: 1}}];
  return main;
}
```

### Section (layer) creation

```javascript
function createSection(name, width = 1952) {
  const section = figma.createFrame();
  section.name = name;
  section.layoutMode = "VERTICAL";
  section.counterAxisSizingMode = "FIXED";
  section.resize(width, 10);
  section.primaryAxisSizingMode = "AUTO";
  section.itemSpacing = 24;
  section.fills = [];
  section.clipsContent = false;
  return section;
}
```

### Text creation

```javascript
async function createText(chars, fontSize, fontStyle = 'Regular', colorVar = null) {
  await figma.loadFontAsync({ family: 'Inter', style: fontStyle });
  const t = figma.createText();
  t.fontName = { family: 'Inter', style: fontStyle };
  t.fontSize = fontSize;
  t.characters = chars;
  if (fontSize > 24) t.textAutoResize = "HEIGHT";
  if (colorVar) t.setBoundVariable('fills', 0, 'color', colorVar.id);
  return t;
}
```

## Slots — Colored Named Frames

> `createSlot()` **DOES NOT EXIST**. Simulate slots with colored frames.

```javascript
const SLOT_COLORS = {
  header:  {r:1, g:0.878, b:0.929},     // #FFE0ED rose
  topbar:  {r:1, g:0.878, b:0.929},
  sidebar: {r:0.922, g:0.878, b:1},      // #EBE0FF violet
  nav:     {r:0.922, g:0.878, b:1},
  content: {r:0.878, g:0.941, b:1},      // #E0F0FF bleu
  main:    {r:0.878, g:0.941, b:1},
  footer:  {r:0.878, g:1, b:0.929},      // #E0FFED vert
  default: {r:1, g:0.973, b:0.878},      // #FFF8E0 jaune
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
  if (opts.width) slot.resize(opts.width, opts.height || 40);
  else slot.resize(200, opts.height || 40);
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

After `parent.appendChild(slot)` → apply `layoutSizingHorizontal = "FILL"` etc.

## Variables — Binding Fills/Strokes

### bindFill / bindStroke — Bind a color to a variable

```javascript
function bindFill(node, varObj) {
  if (!varObj || !node.fills || !node.fills.length) return;
  const fills = JSON.parse(JSON.stringify(node.fills));
  fills[0] = figma.variables.setBoundVariableForPaint(fills[0], 'color', varObj);
  node.fills = fills;
}

function bindStroke(node, varObj) {
  if (!varObj || !node.strokes || !node.strokes.length) return;
  const strokes = JSON.parse(JSON.stringify(node.strokes));
  strokes[0] = figma.variables.setBoundVariableForPaint(strokes[0], 'color', varObj);
  node.strokes = strokes;
}
```

### resolveColor — Resolve the actual value of a COLOR variable

**⚠️ ESSENTIAL for Foundations swatches** — variable binding alone doesn't guarantee
that the base fill is visible. Resolve the value and apply as fill, THEN bind.

```javascript
function resolveColor(varObj, allVars, collections) {
  if (!varObj) return {r: 0.9, g: 0.9, b: 0.9};
  const col = collections.find(c => c.id === varObj.variableCollectionId);
  if (!col) return {r: 0.9, g: 0.9, b: 0.9};
  const modeId = col.modes[0].modeId; // Light mode
  const val = varObj.valuesByMode[modeId];
  if (val && val.r !== undefined) return val;
  // If alias, resolve
  if (val && val.type === 'VARIABLE_ALIAS') {
    const target = allVars.find(v => v.id === val.id);
    if (target) return resolveColor(target, allVars, collections);
  }
  return {r: 0.9, g: 0.9, b: 0.9};
}

// Usage for swatch:
const resolved = resolveColor(v, allVars, collections);
swatch.fills = [{type: 'SOLID', color: resolved}];
if (v) bindFill(swatch, v);
```

## Variables — Creation

### Create a COLOR variable

```javascript
async function createColorVar(collection, name, hexLight, hexDark, lightModeId, darkModeId) {
  const v = figma.variables.createVariable(name, collection, 'COLOR');
  v.setValueForMode(lightModeId, hexToRgb(hexLight));
  if (hexDark) v.setValueForMode(darkModeId, hexToRgb(hexDark));
  return v;
}
```

### Create a semantic variable (alias)

```javascript
function createSemanticVar(collection, name, lightPrimitive, darkPrimitive, lightModeId, darkModeId) {
  const v = figma.variables.createVariable(name, collection, 'COLOR');
  v.setValueForMode(lightModeId, { type: 'VARIABLE_ALIAS', id: lightPrimitive.id });
  if (darkPrimitive) v.setValueForMode(darkModeId, { type: 'VARIABLE_ALIAS', id: darkPrimitive.id });
  return v;
}
```

### Create a FLOAT variable

```javascript
function createFloatVar(collection, name, value) {
  const v = figma.variables.createVariable(name, collection, 'FLOAT');
  v.setValueForMode(collection.modes[0].modeId, value);
  return v;
}
```

## Components — Patterns

### Bind component dimensions to Size variables

```javascript
function bindSizeVars(node, sizeKey, sizeVarMap) {
  const height = sizeVarMap[`Size/controlHeight/${sizeKey}`];
  const padding = sizeVarMap[`Size/controlPadding/${sizeKey}`];
  const gap = sizeVarMap[`Size/controlGap/${sizeKey}`];
  
  if (height) node.setBoundVariable('minHeight', height.id);
  if (padding) {
    node.setBoundVariable('paddingLeft', padding.id);
    node.setBoundVariable('paddingRight', padding.id);
  }
  if (gap) node.setBoundVariable('itemSpacing', gap.id);
}
```

### Create a vector check mark (checkbox)

```javascript
function createCheckMark(size) {
  const check = figma.createVector();
  const w = size === 'MD' ? 12 : 10;
  const h = size === 'MD' ? 9 : 7;
  check.resize(w, h);
  check.vectorPaths = [{
    windingRule: "NONZERO",
    data: "M 1 4.5 L 4.5 8 L 11 1"
  }];
  check.strokeWeight = 2;
  check.strokes = [{type: 'SOLID', color: {r: 1, g: 1, b: 1}}];
  check.fills = [];
  check.strokeCap = "ROUND";
  check.strokeJoin = "ROUND";
  return check;
}
```

### Create a chevron SVG

```javascript
function createChevron(direction = 'down', size = 16) {
  const paths = {
    down: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="${size}" height="${size}"><path d="M6 9l6 6 6-6" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`,
    right: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="${size}" height="${size}"><path d="M9 6l6 6-6 6" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>`,
  };
  const node = figma.createNodeFromSvg(paths[direction]);
  node.resize(size, size);
  // Fix child path color
  const path = node.findOne(n => n.type === 'VECTOR');
  if (path) {
    path.fills = [];
    path.strokes = [{type: 'SOLID', color: {r: 0.4, g: 0.4, b: 0.4}}];
    path.strokeWeight = 1.5;
  }
  return node;
}
```

### Import a Flaticon SVG as component

```javascript
function importSvgAsComponent(svgString, name, size = 24) {
  const iconNode = figma.createNodeFromSvg(svgString);
  const comp = figma.createComponent();
  comp.name = name;
  comp.resize(size, size);
  comp.layoutMode = "VERTICAL";
  comp.primaryAxisAlignItems = "CENTER";
  comp.counterAxisAlignItems = "CENTER";
  comp.primaryAxisSizingMode = "AUTO";
  comp.fills = [];
  for (const child of [...iconNode.children]) {
    comp.appendChild(child);
  }
  iconNode.remove();
  return comp;
}
```

## Showcase — Patterns

### Showcase section label

```javascript
async function createShowcaseLabel(text) {
  await figma.loadFontAsync({ family: 'Inter', style: 'Semi Bold' });
  const frame = figma.createFrame();
  frame.layoutMode = "VERTICAL";
  frame.primaryAxisSizingMode = "AUTO";
  frame.counterAxisSizingMode = "FIXED";
  frame.itemSpacing = 8;
  frame.fills = [];
  
  const label = figma.createText();
  label.fontName = { family: 'Inter', style: 'Semi Bold' };
  label.fontSize = 14;
  label.characters = text.toUpperCase();
  label.letterSpacing = { value: 2, unit: 'PIXELS' };
  frame.appendChild(label);
  
  const divider = figma.createRectangle();
  divider.resize(200, 1);
  divider.fills = [{type: 'SOLID', color: {r: 0.9, g: 0.9, b: 0.9}}];
  frame.appendChild(divider);
  divider.layoutSizingHorizontal = "FILL";
  
  return frame;
}
```

### Dark Showcase frame with explicit mode

```javascript
async function createDarkShowcase(width = 1776) {
  const collections = await figma.variables.getLocalVariableCollectionsAsync();
  const colorCol = collections.find(c => c.name === 'Color');
  const darkModeId = colorCol.modes.find(m => m.name === 'Dark').modeId;
  const allVars = await figma.variables.getLocalVariablesAsync('COLOR');
  const bgVar = allVars.find(v => v.name === 'Neutral/colorBg');
  
  const frame = figma.createFrame();
  frame.name = "Showcase Dark";
  frame.layoutMode = "VERTICAL";
  frame.counterAxisSizingMode = "FIXED";
  frame.resize(width, 10);
  frame.primaryAxisSizingMode = "AUTO";
  frame.itemSpacing = 24;
  frame.paddingTop = 32;
  frame.paddingBottom = 32;
  frame.paddingLeft = 32;
  frame.paddingRight = 32;
  frame.cornerRadius = 16;
  frame.clipsContent = false;
  
  // ⚠️ CRITICAL
  frame.setExplicitVariableModeForCollection(colorCol.id, darkModeId);
  frame.fills = [{type: 'SOLID', color: {r: 0.1, g: 0.1, b: 0.1}}];
  if (bgVar) frame.setBoundVariable('fills', 0, 'color', bgVar.id);
  
  return frame;
}
```

### Color slots in templates (Layouts page)

```javascript
async function colorSlot(instance, slotName, color, label) {
  const slot = instance.findOne(n => n.name === slotName);
  if (!slot) return;
  slot.fills = [{type: 'SOLID', color: color}];
  await figma.loadFontAsync({ family: 'Inter', style: 'Medium' });
  const txt = figma.createText();
  txt.characters = label || slotName;
  txt.fontSize = 12;
  txt.fontName = {family: 'Inter', style: 'Medium'};
  txt.fills = [{type: 'SOLID', color: {r: 0.6, g: 0.6, b: 0.6}}];
  txt.textAlignHorizontal = 'CENTER';
  slot.appendChild(txt);
}

// Slot color palette:
const SLOT_COLORS = {
  header:  {r: 1.0, g: 0.88, b: 0.93}, // pink
  sidebar: {r: 0.92, g: 0.88, b: 1.0},  // purple
  content: {r: 0.88, g: 0.94, b: 1.0},  // blue
  footer:  {r: 0.88, g: 1.0, b: 0.93},  // green
  other:   {r: 1.0, g: 0.97, b: 0.88},  // yellow
};
```
