# Phase 8 — Welcome Page

> **MCP Calls**: 1-2× `mcp_figma_use_figma`
> **Prerequisites**: Phase 2 (doc components) + Phase 5 (Component Sets, for highlight instances)
> **Output**: Page 📄 👋 Welcome
> **Verification**: Screenshot of the complete page

## Structure

```
Main frame (Auto Layout vertical, gap=0, padding=0, width 2000)
├── fills bound to {Neutral/colorBg}
├── .documentation header (type=documentation)
├── Content Frame (vertical, gap=80, paddingTop=64, paddingBottom=64, paddingLeft=88, paddingRight=88, width=FILL)
│   ├── SECTION 1: HERO
│   ├── SECTION 2: STATS
│   ├── SECTION 3: WHAT'S INCLUDED
│   ├── SECTION 4: COMPONENTS HIGHLIGHT
│   ├── SECTION 5: QUICK NAVIGATION
│   └── SECTION 6: CONFIGURATION
└── _DesignSystemFooter
```

## ⚠️ NO EMOJIS on the Welcome page

Use geometric shapes (rectangles, circles, dots) with DS colors.

## Sections

### Section 1 — Hero
```
Frame hero (vertical, gap=32, alignItems=CENTER, padding=80)
├── fills={Primary/colorPrimaryBg}, cornerRadius=24
├── Overline — 14px Medium UPPERCASE, letterSpacing=3, {Primary/colorPrimary}
├── Title — 56px Extra Bold, {Neutral/colorText}, CENTER
├── Subtitle — 20px Regular, {Neutral/colorTextSecondary}, CENTER
├── Divider — rectangle 80×4, {Primary/colorPrimary}, cornerRadius=2
└── CTA row (horizontal, gap=16)
    ├── Button Primary instance → "Explore components"
    └── Button Secondary instance → "View guidelines"
```

### Section 2 — Stats (key figures)
```
Frame stats (horizontal, gap=0, fill width, justify=SPACE_BETWEEN)
├── 4 items, each:
│   ├── Value — 40px Extra Bold, {Primary/colorPrimary}
│   └── Label — 14px Regular, {Neutral/colorTextSecondary}
```
Stats: "21" Components, "120+" Design Tokens, "12" Templates, "8" Foundations (adapt to actual DS).

### Section 3 — What’s Included (feature cards)
```
_SectionMetadata + Grid (horizontal wrap, gap=24)
├── 4 cards: padding=32, fills={Neutral/colorBgSecondary}, cornerRadius={Radius/lg}, border 1px
│   ├── Abstract icon — rectangle 40×40, cornerRadius=10, fills={Primary/colorPrimaryBg}
│   │   └── Inner rectangle 20×20 centered, {Primary/colorPrimary} (NO emoji)
│   ├── Title — 18px Semi Bold
│   └── Description — 14px Regular, {Neutral/colorTextSecondary}
```
Cards: "Variables & Tokens", "Typography System", "Spacing & Sizing", "Components"

### Section 4 — Components Highlight
```
_SectionMetadata + Row (horizontal wrap, gap=16)
├── 6-8 Default/MD instances in cards (padding=24, cornerRadius=12, border 1px)
│   button Primary, button Secondary, text field, select, checkbox, badge...
```

### Section 5 — Quick Navigation
```
_SectionMetadata + Frame badges (horizontal wrap, gap=12)
├── Badge/pill per page (WITHOUT emoji)
│   ├── fills={Primary/colorPrimaryBg}, cornerRadius=99, border 1px
│   ├── Dot — rectangle 8×8, cornerRadius=4, {Primary/colorPrimary}
│   └── Page name — 14px Medium, {Primary/colorPrimary}
```
Pages: Colors, Typography, Size, Border, Space, Radius, Shadow, Breakpoint, Icons, Components, Layouts

### Section 6 — Configuration
```
_SectionMetadata + Frame rows (vertical, gap=12)
├── Row per parameter:
│   ├── Label — 14px Medium, {Neutral/colorTextSecondary}, width=160
│   └── Value — 14px Semi Bold, {Neutral/colorText}
```
Parameters: Font, Primary (swatch+hex), Secondary, Radius, Dark mode, Tiers.

## Visibility Rule

Every visual element MUST have explicit fills. A frame without fills is INVISIBLE.
Cards MUST have background + border to be readable.

## Checklist

- [ ] Main frame 2000px
- [ ] .documentation header at the top
- [ ] 6 sections with complete content
- [ ] NO emojis (geometric shapes only)
- [ ] Cards with visible background + border
- [ ] Real component instances in the Highlight
- [ ] _DesignSystemFooter at the bottom
- [ ] Screenshot verified
