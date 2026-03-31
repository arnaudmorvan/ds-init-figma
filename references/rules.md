# Critical Rules — Figma Plugin API for ds-init

> These rules cause **silent errors or visual regressions** if ignored.
> Each rule was discovered after a real bug and documented.

## 1. Auto Layout — HUG Content MANDATORY

Every frame with `layoutMode` MUST have `primaryAxisSizingMode = "AUTO"` (HUG content).

**Exceptions**: screen frames simulating a viewport (1440×900).

```javascript
// ✅ Safe pattern
frame.layoutMode = "VERTICAL";
frame.counterAxisSizingMode = "FIXED";
frame.resize(2000, 10); // width de page
frame.primaryAxisSizingMode = "AUTO"; // ALWAYS after resize()
frame.clipsContent = false;
```

**Pitfall**: `resize(w, h)` forces `primaryAxisSizingMode = "FIXED"` → always reset to `"AUTO"` afterwards.

## 2. FILL after appendChild — Mandatory Order

`layoutSizingHorizontal/Vertical = "FILL"` only works on an already attached child.

```javascript
// ❌ CRASH
child.layoutSizingHorizontal = "FILL";
parent.appendChild(child);

// ✅ CORRECT
parent.appendChild(child);
child.layoutSizingHorizontal = "FILL";
```

## 3. Fills — No `a` property (alpha)

```javascript
// ❌ "Unrecognized key(s) in object: 'a'"
frame.fills = [{type: 'SOLID', color: {r:1, g:1, b:1, a: 0.7}}];

// ✅ Use opacity on the fill
frame.fills = [{type: 'SOLID', color: {r:1, g:1, b:1}, opacity: 0.7}];
```

## 4. strokeAlign singular

```javascript
// ❌ TypeError: object is not extensible
frame.strokesAlign = "INSIDE";

// ✅ CORRECT
frame.strokeAlign = "INSIDE";
```

## 5. textAlignHorizontal (not textAlign)

```javascript
// ❌ read-only, causes error
text.textAlign = "CENTER";

// ✅ CORRECT
text.textAlignHorizontal = "CENTER";
```

## 6. Dark Mode — Explicit Activation Mandatory

Dark variables do **NOT** activate automatically. You must call `setExplicitVariableModeForCollection()` on **every** frame that needs to display in dark.

```javascript
const collections = await figma.variables.getLocalVariableCollectionsAsync();
const colorCol = collections.find(c => c.name === 'Color');
const darkModeId = colorCol.modes.find(m => m.name === 'Dark').modeId;

// ⚠️ THIS LINE IS CRITICAL — without it, frame = Light
darkFrame.setExplicitVariableModeForCollection(colorCol.id, darkModeId);

// Bind background to semantic variable
const neutralBgVar = allVars.find(v => v.name === 'Neutral/colorBg');
darkFrame.setBoundVariable('fills', 0, 'color', neutralBgVar.id);
```

**Without this call**: white background, black text, dark mode invisible.

## 7. Vectors — fills = [] mandatory

Vectors created via `figma.createVector()` have a black fill by default.

```javascript
const check = figma.createVector();
check.fills = [];  // ← MANDATORY for stroke-only shapes
check.strokes = [{type: 'SOLID', color: {r:1,g:1,b:1}}];
```

## 8. setCurrentPageAsync (no direct assignment)

```javascript
// ❌ FORBIDDEN
figma.currentPage = myPage;

// ✅ CORRECT
await figma.setCurrentPageAsync(myPage);
```

## 9. Invisible Components on White Background

**#1 cause of visual regressions.** Form components MUST have visible strokes:

| Component | Problem without strokes | Fix |
|---|---|---|
| Text Field | White rectangle invisible | `strokes=[{Neutral/colorBorder}], strokeWeight=1, strokeAlign="INSIDE"` |
| Select | Same | Same |
| Button Secondary | No outline | Add stroke `{Neutral/colorBorder}` |
| Button Ghost | Invisible text if colorText=black | `text.fills = [{Primary/colorPrimary}]` |
| Card default | White rectangle invisible | Add shadow OR stroke |

Apply on **ALL variants** (including state=Default, status=None).

## 10. Frame Positioning on a Page

Figma starts all frames at `x=0, y=0`. Without explicit positioning, they overlap.

```javascript
let currentX = 0;
const GAP = 100;

const frame1 = figma.createFrame();
frame1.x = currentX;
frame1.y = 0;
currentX += frame1.width + GAP;

const frame2 = figma.createFrame();
frame2.x = currentX;
frame2.y = 0;
```

## 11. Fonts — Load Before Any Text Operation

```javascript
await figma.loadFontAsync({ family: 'Inter', style: 'Regular' });
await figma.loadFontAsync({ family: 'Inter', style: 'Medium' });
await figma.loadFontAsync({ family: 'Inter', style: 'Semi Bold' });
await figma.loadFontAsync({ family: 'Inter', style: 'Bold' });
await figma.loadFontAsync({ family: 'Inter', style: 'Extra Bold' });
```

## 12. Slots — createSlot() DOES NOT EXIST

**⛔ `createSlot()` is NOT available** in the Figma Plugin API.
Calling it doesn't throw a visible error but creates nothing → empty components.

Slots must be simulated with **colored named frames**:

```javascript
// ✅ CORRECT — Colored frame as slot
function createSlotFrame(name, opts = {}) {
  const slot = figma.createFrame();
  slot.name = name;
  slot.layoutMode = opts.direction || "VERTICAL";
  slot.primaryAxisSizingMode = "AUTO";
  slot.counterAxisSizingMode = "FIXED";
  slot.clipsContent = false;
  slot.fills = [{type:'SOLID', color: SLOT_COLORS[name] || {r:1,g:0.973,b:0.878}, opacity: 0.5}];
  slot.cornerRadius = 4;
  slot.paddingTop = 8; slot.paddingBottom = 8;
  slot.paddingLeft = 8; slot.paddingRight = 8;
  if (opts.width) slot.resize(opts.width, opts.height || 40);
  else slot.resize(200, opts.height || 40);
  // Label texte
  const label = figma.createText();
  label.characters = name;
  label.fontSize = 11;
  label.fontName = { family: 'Inter', style: 'Medium' };
  label.fills = [{type:'SOLID', color:{r:0.4,g:0.4,b:0.4}}];
  label.textAutoResize = "WIDTH_AND_HEIGHT";
  slot.appendChild(label);
  return slot;
}

// ❌ FORBIDDEN — does not exist
const slot = comp.createSlot(); // silently fails
```

See slot colors in style-guide.md.

## 13. Detach Before appendChild in Instances

You CANNOT add children to an instance. Always detach first.

```javascript
const inst = templateComponent.createInstance();
const frame = inst.detachInstance(); // → editable frame
// Now we can appendChild into the slots
```

## 14. Large Text — textAutoResize = "HEIGHT"

For text > 24px, use `textAutoResize = "HEIGHT"` (fixed width, auto height) to avoid clipping.

## 15. Verification After Each Call

**Golden rule**: Screenshot + verification **immediately** after each `mcp_figma_use_figma`.

```
AFTER EACH mcp_figma_use_figma call:
1. get_screenshot of the modified page
2. Verify: elements visible? overlaps? collapsed heights?
3. If problem → fix BEFORE moving to the next call
```

## 16. One LAYER per Component Set on _Components

Each Component Set created with `combineAsVariants()` MUST be a **direct child** of the `_Components` page. No shared parent frame.

In Figma's Layers panel, each Component Set should appear as a separate layer: `button`, `text field`, `select`, `checkbox`, `radio`.

```javascript
// ✅ CORRECT
const buttonSet = figma.combineAsVariants(variants, compPage); // directly on the page

// ❌ FORBIDDEN
wrapper.appendChild(buttonSet); // inside a parent frame
```

## 17. Color Swatches — Resolve the Actual Value

Bound variables don't change the visual base fill in MCP code.
For color swatches to be **visible in screenshots**, you must:

1. Resolve the actual variable value (via `valuesByMode`)
2. Apply it as the base fill (native Figma color)
3. THEN bind the variable on top

```javascript
// ❌ Invisible in screenshot (gray fill + binding)
swatch.fills = [{type:'SOLID', color:{r:0.9,g:0.9,b:0.9}}];
bindFill(swatch, v);

// ✅ Visible (resolved fill + binding)
const resolved = resolveColor(v, allVars, collections);
swatch.fills = [{type:'SOLID', color: resolved}];
bindFill(swatch, v);
```

## 18. Text in Components — Anti-clipping

All text in a component (footer, header, card) MUST have:
- `textAutoResize = "WIDTH_AND_HEIGHT"` (free text)
- OR `textAutoResize = "HEIGHT"` + fixed/FILL width (text in a layout)

**NEVER** leave `textAutoResize = "NONE"` (default) which clips the text.

```javascript
// ✅ Text in an auto layout
parent.appendChild(text);
text.layoutSizingHorizontal = "FILL";
text.textAutoResize = "HEIGHT";

// ✅ Free text
text.textAutoResize = "WIDTH_AND_HEIGHT";
```
