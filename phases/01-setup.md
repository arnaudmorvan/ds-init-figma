# Phase 1 — Setup: Figma File, Pages and Variables

> **MCP Calls**: `mcp_figma_create_new_file` + 4× `mcp_figma_use_figma`
> **Prerequisites**: User config collected (Phase 0)
> **Output**: Figma file with structured pages + 8 variable collections
> **Verification**: Screenshot of the page list

## Step 1.1 — Create the Figma file

```tool_call
mcp_figma_create_new_file({ name: "[DS Name]" })
```

Retrieve the `fileKey` for all subsequent calls.

## Step 1.2 — Create pages (1 mcp_figma_use_figma call)

Default page structure ("all on one page" mode):

```
📄 👋 Welcome
📄 ---
📄 🧱 Foundations
📄 ⚙️ Icons
📄 ---
📄 🏗️ Layouts
📄 Components
📄 ---
📄 🔒 _Components
```

**"One page per component" mode**: replace `Components` with individual lowercase pages (buttons, inputs, dropdown, checkbox, radio…)

**"Hybrid" mode**: by category (Actions, Inputs, Navigation…)

Convention:
- Emoji prefixes on system pages
- `---` pages as visual separators
- _Components prefixed with 🔒 (private)

→ **Screenshot**: verify the page list in the side panel.

## Step 1.3 — Color Variables (1 call)

### Color Collection — with Light/Dark modes

**1. Create the collection and modes:**
```javascript
const colorCollection = figma.variables.createVariableCollection('Color');
const lightModeId = colorCollection.modes[0].modeId;
colorCollection.renameMode(lightModeId, 'Light');
const darkMode = colorCollection.addMode('Dark');
const darkModeId = darkMode.modeId;
```

**2. Primitives (identical in Light/Dark):**

```
Color/Blue/50→950    (11 shades)
Color/Gray/0→950     (12 shades, 0=pure white)
Color/Red/50→950
Color/Green/50→950
Color/Yellow/50→950
Color/Purple/50→950
```

Default Blue palette:
```
50=#EFF6FF  100=#DBEAFE  200=#BFDBFE  300=#93C5FD  400=#60A5FA
500=#3B82F6  600=#2563EB  700=#1D4ED8  800=#1E40AF  900=#1E3A8A  950=#172554
```

Gray palette:
```
0=#FFFFFF  50=#F9FAFB  100=#F3F4F6  200=#E5E7EB  300=#D1D5DB  400=#9CA3AF
500=#6B7280  600=#4B5563  700=#374151  800=#1F2937  900=#111827  950=#030712
```

Adapt Primary/Secondary palettes based on the user's chosen hex values.

**3. Semantics (aliases, inverted in Dark) — 25 variables:**

| Variable | Light | Dark |
|---|---|---|
| `Neutral/colorBg` | Gray/0 | Gray/950 |
| `Neutral/colorBgElevated` | Gray/0 | Gray/900 |
| `Neutral/colorBgSubtle` | Gray/50 | Gray/800 |
| `Neutral/colorBgSecondary` | Gray/50 | Gray/900 |
| `Neutral/colorText` | Gray/900 | Gray/50 |
| `Neutral/colorTextSecondary` | Gray/600 | Gray/400 |
| `Neutral/colorTextTertiary` | Gray/400 | Gray/500 |
| `Neutral/colorBorder` | Gray/200 | Gray/700 |
| `Neutral/colorBorderSubtle` | Gray/100 | Gray/800 |
| `Primary/colorPrimary` | Blue/600 | Blue/400 |
| `Primary/colorPrimaryHover` | Blue/500 | Blue/300 |
| `Primary/colorPrimaryActive` | Blue/700 | Blue/500 |
| `Primary/colorPrimaryBg` | Blue/50 | Blue/950 |
| `Primary/colorPrimaryBorder` | Blue/200 | Blue/800 |
| `Secondary/colorSecondary` | Purple/600 | Purple/400 |
| `Secondary/colorSecondaryHover` | Purple/500 | Purple/300 |
| `Secondary/colorSecondaryBg` | Purple/50 | Purple/950 |
| `Feedback/colorSuccess` | Green/600 | Green/400 |
| `Feedback/colorSuccessBg` | Green/50 | Green/950 |
| `Feedback/colorWarning` | Yellow/500 | Yellow/400 |
| `Feedback/colorWarningBg` | Yellow/50 | Yellow/950 |
| `Feedback/colorDanger` | Red/600 | Red/400 |
| `Feedback/colorDangerBg` | Red/50 | Red/950 |
| `Feedback/colorInfo` | Blue/600 | Blue/400 |
| `Feedback/colorInfoBg` | Blue/50 | Blue/950 |

> **Note**: Blue = Primary color, Purple = Secondary color. Adapt prefixes if colors change.
> The `*Bg` are light variants (Light=50, Dark=950).

→ **Screenshot**: not visual here, but verify the collection in Figma.

## Step 1.4 — Non-Color Variables (1 call)

6 additional collections, all FLOAT (no modes):

### Typography (16 FLOAT vars)

> **⚠️** STRING variables (fontFamily) are NOT supported in FLOAT mode.
> The font name is documented in the doc header and Foundations page only.

```
Typography/fontSize/xs          → 12
Typography/fontSize/sm          → 14
Typography/fontSize/base        → 16
Typography/fontSize/lg          → 18
Typography/fontSize/xl          → 20
Typography/fontSize/2xl         → 24
Typography/fontSize/3xl         → 30
Typography/fontSize/4xl         → 36
Typography/fontSize/5xl         → 48
Typography/fontWeight/regular   → 400
Typography/fontWeight/medium    → 500
Typography/fontWeight/semibold  → 600
Typography/fontWeight/bold      → 700
Typography/lineHeight/tight     → 1.25
Typography/lineHeight/normal    → 1.5
Typography/lineHeight/relaxed   → 1.75
```

### Size (interactive component dimensions)
```
Size/controlHeight/xs→xl       → 24, 32, 40, 48, 56
Size/controlPadding/xs→xl      → 8, 12, 16, 20, 24
Size/controlGap/xs→xl          → 4, 6, 8, 10, 12
Size/iconSize/xs→xl            → 14, 16, 18, 20, 24
```

### Space
```
Space/space/1→16               → 4, 8, 12, 16, 20, 24, 32, 40, 48, 64
Space/Padding/paddingXS→paddingXL → 4, 8, 12, 16, 24
```

### Radius (8 vars)

Adapt values to the chosen style (Sharp/Soft/Round/Full):

```
Radius/none   → 0
Radius/xs     → 2
Radius/sm     → 4      ← Soft: 4 | Sharp: 1 | Round: 8
Radius/md     → 6      ← Soft: 6 | Sharp: 2 | Round: 12
Radius/lg     → 8      ← Soft: 8 | Sharp: 4 | Round: 16
Radius/xl     → 12     ← Soft: 12 | Sharp: 6 | Round: 20
Radius/2xl    → 16     ← Soft: 16 | Sharp: 8 | Round: 24
Radius/full   → 9999
```

### Border
```
Border/width/none→thick        → 0, 1, 2, 3
```

### Breakpoint
```
Breakpoint/sm→2xl              → 640, 768, 1024, 1280, 1536
```

→ **Screenshot**: verify that collections appear in the Variables panel.

## Step 1.5 — Shadow Effect Styles (1 call)

Shadows are not Variables but Figma **Effect Styles**.

```javascript
const shadows = [
  { name: 'Shadow/sm', effects: [{type: 'DROP_SHADOW', color: {r:0,g:0,b:0,a:0.05}, offset:{x:0,y:1}, radius:2, spread:0, visible:true}] },
  { name: 'Shadow/md', effects: [{type: 'DROP_SHADOW', color: {r:0,g:0,b:0,a:0.1}, offset:{x:0,y:4}, radius:6, spread:-1, visible:true}] },
  { name: 'Shadow/lg', effects: [{type: 'DROP_SHADOW', color: {r:0,g:0,b:0,a:0.1}, offset:{x:0,y:10}, radius:15, spread:-3, visible:true}] },
  { name: 'Shadow/xl', effects: [{type: 'DROP_SHADOW', color: {r:0,g:0,b:0,a:0.1}, offset:{x:0,y:20}, radius:25, spread:-5, visible:true}] },
];
for (const s of shadows) {
  const style = figma.createEffectStyle();
  style.name = s.name;
  style.effects = s.effects;
}
```

> **Note**: The `a` property is allowed in `effects[].color` (unlike `fills`).

## Verification Checklist

- [ ] Figma file created with the correct name
- [ ] Pages created in the correct order with separators
- [ ] Color collection with Light + Dark modes
- [ ] Primitive variables (Blue, Gray, Red, Green, Yellow, Purple)
- [ ] Semantic variables with Light/Dark aliases (25: Neutral 9 + Primary 5 + Secondary 3 + Feedback 8)
- [ ] 6 additional collections (Typography 16, Size 20, Space 15, Radius 8, Border 4, Breakpoint 5)
- [ ] 4 Shadow Effect Styles (sm, md, lg, xl)
- [ ] Expected total: ~92 COLOR + ~68 FLOAT + 4 Effect Styles
