# Style Guide — ds-init Visual Conventions

> Visual conventions inspired by SlothUI. These are NOT critical rules
> but recommendations for a consistent professional result.

## Page Dimensions

- **Page width**: 2000px
- **Content width**: 1952px (page - 24×2 padding)
- **Indented content width**: 1776px (for sections under the header, additional 88px padding)
- **Gap between sections**: 80px
- **Gap between internal elements**: 24-32px

## Header (.documentation header)

- cornerRadius = 40
- border = 1px `{Neutral/colorBorder}`
- Decorative gradient: ellipse ~1150×1150, off-frame position, radial gradient `{Primary/colorPrimary}` → transparent, opacity=0.08
- Title: 60px ExtraBold
- Category badge: filled pill `{Primary/colorPrimary}`, white text, cornerRadius=999
- Breadcrumbs: DS logo 40×40 + levels 18px Bold

## Footer (_DesignSystemFooter)

- Same rounded style as the header (cornerRadius=40)
- Mirrored decorative gradient
- Logo + tagline + link
- Width = FILL in parent

## Section Bars (_SectionMetadata)

- Height 38px
- Left label: 14px SemiBold
- Right specs: 13px Regular, secondary gray
- Dashed line in the middle: FILL, dashPattern=[4,4]

## Showcase Labels

Each variant group MUST be preceded by a visual title:

```
Frame section (Auto Layout vertical, gap=12)
├── Label ← 14px SemiBold, UPPERCASE, letterSpacing=2, colorTextSecondary
├── Divider ← 1px, colorBorder, opacity=0.5
└── Row content (Auto Layout horizontal, gap=16, wrap)
```

## Page Emoji Prefixes

```
📄 👋 Welcome
📄 ---                       ← Separator
📄 🧱 Foundations
📄 ⚙️ Icons
📄 ---
📄 🏗️ Layouts
📄 Components
📄 ---
📄 🔒 _Components
```

- Emoji prefixes on system pages (Foundations, Icons, Layouts, Welcome)
- NO emoji on component pages (buttons, inputs…) — lowercase

## Naming

| Type | Convention | Example |
|------|-----------|---------|
| Public component | lowercase, no prefix | `button`, `checkbox` |
| Sub-component | `.elements / parent / name` | `.elements / select / option` |
| Documentation | `.documentation header` | Component Set, 3 variants |
| Footer | `_DesignSystemFooter` | Main Component |
| Section divider | `_SectionMetadata` | Main Component |
| Native slot | simple name | `header`, `leading`, `content` |
| Variable collection | PascalCase | `Color`, `Typography`, `Size` |
| Variable | `Category/tokenName` | `Primary/colorPrimary`, `Size/controlHeight/md` |

## Composition Showcase

After variant sections, add a "COMPOSITION" section showing real usage:

```
Frame "Composition" (Auto Layout vertical, gap=16)
├── Label "COMPOSITION" (14px SemiBold UPPERCASE)
├── Divider (1px)
└── Context frame (padding=24, cornerRadius=12,
    fills=colorBgSecondary, border 1px colorBorder)
    └── Realistic arrangement of combined components
```

## Welcome Page

- NO EMOJIS — geometric shapes + DS colors only
- Sections: Hero, Stats, Features, Components Highlight, Quick Navigation, Config
- Cards with visible background (`colorBgSecondary` + border) — never transparent

## Slot Colors (Layouts Page)

SlotNodes are invisible by default. On the Layouts page, color them:

| Slot | Color | Hex |
|------|-------|-----|
| header / topbar | light pink | #FFE0ED |
| sidebar / nav | light purple | #EBE0FF |
| content / main | light blue | #E0F0FF |
| footer | light green | #E0FFED |
| secondary | light yellow | #FFF8E0 |
