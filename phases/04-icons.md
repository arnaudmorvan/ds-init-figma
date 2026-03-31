# Phase 4 — Icons

> **MCP Calls**: 2× `mcp_figma_use_figma` (1 for Icon Component Set, 1 for Icons page)
> **Prerequisites**: Phase 1 (Color/Size variables)
> **Output**: Icon Component Set on _Components + ⚙️ Icons Page
> **Verification**: Screenshot of Icons page

## Pre-check for Icons Folder

**Before starting**, verify the icons folder exists:

```bash
ls "/Users/arnaudmorvan/Dropbox/uicons-regular-rounded/svg/" | head -5
```

- **If exists** → continue with these SVGs
- **If missing** → ask the user (via chat or interactive form):
  - Which library? (Flaticon UIcons, Lucide, Heroicons, other)
  - Absolute path to the SVG folder

## ⛔ ABSOLUTE RULE — Use REAL SVGs, never inline paths

**FORBIDDEN** to draw icons by hand with simplified paths (circles, basic lines).
Icons MUST come from the SVG folder configured by the user.

**Required workflow**:

1. **Mapper** les noms d'icônes aux fichiers SVG réels :
   ```javascript
   // Flaticon UIcons Regular Rounded names → prefix "fi-rr-"
   const iconFileMap = {
     // Navigation
     'angle-small-down': 'fi-rr-angle-small-down.svg',
     'angle-small-up': 'fi-rr-angle-small-up.svg',
     'angle-small-left': 'fi-rr-angle-small-left.svg',
     'angle-small-right': 'fi-rr-angle-small-right.svg',
     'arrow-left': 'fi-rr-arrow-left.svg',
     'arrow-right': 'fi-rr-arrow-right.svg',
     'bars-staggered': 'fi-rr-bars-staggered.svg',
     // Actions
     'plus': 'fi-rr-plus.svg',
     'minus': 'fi-rr-minus.svg',
     'cross': 'fi-rr-cross.svg',
     'check': 'fi-rr-check.svg',
     'pencil': 'fi-rr-pencil.svg',
     'trash': 'fi-rr-trash.svg',
     'copy': 'fi-rr-copy.svg',
     'share': 'fi-rr-share.svg',
     'download': 'fi-rr-download.svg',
     'upload': 'fi-rr-upload.svg',
     // Search
     'search': 'fi-rr-search.svg',
     'filter': 'fi-rr-filter.svg',
     'bars-sort': 'fi-rr-bars-sort.svg',
     // User
     'user': 'fi-rr-user.svg',
     'users': 'fi-rr-users.svg',
     'lock': 'fi-rr-lock.svg',
     'unlock': 'fi-rr-unlock.svg',
     'key': 'fi-rr-key.svg',
     // Communication
     'envelope': 'fi-rr-envelope.svg',
     'bell': 'fi-rr-bell.svg',
     'comment-alt': 'fi-rr-comment-alt.svg',
     // Content
     'picture': 'fi-rr-picture.svg',
     'file': 'fi-rr-file.svg',
     'folder': 'fi-rr-folder.svg',
     'link': 'fi-rr-link.svg',
     // Interface
     'eye': 'fi-rr-eye.svg',
     'eye-crossed': 'fi-rr-eye-crossed.svg',
     'settings': 'fi-rr-settings.svg',
     'home': 'fi-rr-home.svg',
     'info': 'fi-rr-info.svg',
     'exclamation': 'fi-rr-exclamation.svg',
     'ban': 'fi-rr-ban.svg',
     // Data
     'chart-line-up': 'fi-rr-chart-line-up.svg',
     'calendar': 'fi-rr-calendar.svg',
     'clock': 'fi-rr-clock.svg',
     // Toggles
     'sun': 'fi-rr-sun.svg',
     'moon': 'fi-rr-moon.svg',
     'globe': 'fi-rr-globe.svg',
     // Commerce
     'shopping-cart': 'fi-rr-shopping-cart.svg',
     'credit-card': 'fi-rr-credit-card.svg',
     'star': 'fi-rr-star.svg',
   };
   ```

2. **Read** ALL SVG files with `cat` **BEFORE** the MCP call (see full bash command in the "SVG Import — Pattern" section below)

3. **Inject** the complete SVG strings into the MCP code:
   ```javascript
   const svgStrings = {
     'plus': '<svg xmlns="http://www.w3.org/2000/svg" ...>...</svg>', // REAL file content
     // ...
   };
   ```

**❌ FORBIDDEN**:
```javascript
// NEVER invent paths
const svgStrings = {
  'plus': '<svg viewBox="0 0 24 24"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>'
};
```

**✅ REQUIRED**:
```javascript
// Use the EXACT content read from the file
const svgStrings = {
  'plus': '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M23,..." /></svg>'
};
```

## Call 4a — Icon Component Set (on _Components)

### Architecture

```
🔶 icon (Component Set)
├── Variants :
│   ├── name = plus | search | check | cross | angle-small-down | angle-small-right | user | settings | bell | eye | eye-crossed | info | exclamation | trash | (15 icons minimum)
│   ├── size = xs(16) | sm(20) | md(24) | lg(32) | xl(48)
│   └── color = neutral | brand | danger | warning | success | inherit
```

### Priority Icons (15 minimum for the Component Set)

```
plus, minus, search, check, cross, angle-small-down, angle-small-right,
user, settings, bell, eye, eye-crossed, info, exclamation, trash
```

### Creation Pattern

```javascript
// 1. Read SVGs (via terminal, not Plugin API)
// The assistant reads SVG files with its tools and injects the strings

// 2. Create variants
const iconComponents = [];
for (const iconName of iconNames) {
  for (const size of ['xs','sm','md','lg','xl']) {
    for (const color of ['neutral','brand','danger','warning','success','inherit']) {
      const comp = figma.createComponent();
      comp.name = `name=${iconName}, size=${size}, color=${color}`;
      const sizeMap = { xs:16, sm:20, md:24, lg:32, xl:48 };
      const s = sizeMap[size];
      comp.resize(s, s);
      comp.layoutMode = "VERTICAL";
      comp.primaryAxisAlignItems = "CENTER";
      comp.counterAxisAlignItems = "CENTER";
      comp.primaryAxisSizingMode = "AUTO";
      comp.fills = [];
      
      // SVG import
      const svg = figma.createNodeFromSvg(svgStrings[iconName]);
      const vector = svg.children[0];
      comp.appendChild(vector);
      svg.remove();
      vector.resize(s * 0.75, s * 0.75);
      
      // Color via variable binding
      if (color !== 'inherit') {
        const colorVar = colorVarMap[color];
        vector.setBoundVariable('fills', 0, 'color', colorVar.id);
      }
      iconComponents.push(comp);
    }
  }
}
const iconSet = figma.combineAsVariants(iconComponents, _componentsPage);
iconSet.name = "icon";
```

### Color → Variable Map

| color | Variable |
|---|---|
| neutral | `Neutral/colorTextSecondary` |
| brand | `Primary/colorPrimary` |
| danger | `Feedback/colorDanger` |
| warning | `Feedback/colorWarning` |
| success | `Feedback/colorSuccess` |
| inherit | fills=[] (inherits from parent) |

→ **Screenshot** of _Components — verify the icon Component Set is visible.

## Call 4b — ⚙️ Icons Page

### Structure

```
Page principale (2000px, Auto Layout vertical, gap=80, padding=24)
├── .documentation header (type=documentation, badge "Foundations", title "Icons")
├── For each category:
│   ├── Category title (16px Bold)
│   └── Auto Layout wrap grid, gap=24
│       └── Frame 80×100 per icon (vertical, gap=8, centerH)
│           ├── SVG node resize 32×32, fills={Neutral/colorText}
│           └── Label 12px Regular, gray 600, textAlign=CENTER
└── _DesignSystemFooter
```

### Categories and Icons

```
Navigation:     angle-small-down, angle-small-up, angle-small-left, angle-small-right,
                arrow-left, arrow-right, bars-staggered
Actions:        plus, cross, check, pencil, trash, copy, share, download, upload
Search:         search, filter, bars-sort
User:           user, users, lock, unlock, key
Communication:  envelope, bell, comment-alt
Content:        picture, file, folder, link
Interface:      eye, eye-crossed, settings, home, info, exclamation, ban
Data:           chart-line-up, calendar, clock
Toggles:        sun, moon, globe
Commerce:       shopping-cart, credit-card, star
```

### SVG Import — Pattern

The Plugin API code has no filesystem access. The assistant MUST:
1. Read local SVGs with `cat` or `read_file` **in a SEPARATE terminal call before the MCP**
2. Store the complete SVG content of each file
3. Inject the **exact** SVG strings into `figma.createNodeFromSvg()` in the MCP code

**⚠️ NEVER invent or simplify SVG paths** — always use the exact file content.

```bash
# Read all needed icons in a single command
for f in fi-rr-plus fi-rr-minus fi-rr-search fi-rr-check fi-rr-cross \
         fi-rr-angle-small-down fi-rr-angle-small-right fi-rr-user \
         fi-rr-settings fi-rr-bell fi-rr-eye fi-rr-eye-crossed \
         fi-rr-info fi-rr-exclamation fi-rr-trash \
         fi-rr-arrow-left fi-rr-arrow-right fi-rr-bars-staggered \
         fi-rr-pencil fi-rr-copy fi-rr-share fi-rr-download fi-rr-upload \
         fi-rr-filter fi-rr-bars-sort fi-rr-users fi-rr-lock fi-rr-unlock \
         fi-rr-key fi-rr-envelope fi-rr-comment-alt fi-rr-picture fi-rr-file \
         fi-rr-folder fi-rr-link fi-rr-home fi-rr-ban fi-rr-chart-line-up \
         fi-rr-calendar fi-rr-clock fi-rr-sun fi-rr-moon fi-rr-globe \
         fi-rr-shopping-cart fi-rr-credit-card fi-rr-star; do
  echo "=== $f ==="
  cat "/Users/arnaudmorvan/Dropbox/uicons-regular-rounded/svg/$f.svg" 2>/dev/null || echo "MISSING"
done
```

## Checklist

- [ ] Icons folder verified and accessible
- [ ] SVGs read and paths extracted (30 icons minimum)
- [ ] Icon Component Set created on _Components (15 icons × 5 sizes × 6 colors)
- [ ] ⚙️ Icons page with categorized grid
- [ ] Icons visible (32×32 minimum, fills Neutral/colorText)
- [ ] Screenshots verified
