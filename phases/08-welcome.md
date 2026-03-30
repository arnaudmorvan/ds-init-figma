# Phase 8 — Page Welcome

> **Appels MCP** : 1-2× `mcp_figma_use_figma`
> **Prérequis** : Phase 2 (doc components) + Phase 5 (Component Sets, pour les instances highlight)
> **Produit** : Page 📄 👋 Welcome
> **Vérification** : Screenshot de la page complète

## Structure

```
Frame principal (Auto Layout vertical, gap=0, padding=0, largeur 2000)
├── fills bound à {Neutral/colorBg}
├── .documentation header (type=documentation)
├── Content Frame (vertical, gap=80, paddingTop=64, paddingBottom=64, paddingLeft=88, paddingRight=88, width=FILL)
│   ├── SECTION 1 : HERO
│   ├── SECTION 2 : STATS
│   ├── SECTION 3 : CE QUI EST INCLUS
│   ├── SECTION 4 : COMPOSANTS HIGHLIGHT
│   ├── SECTION 5 : NAVIGATION RAPIDE
│   └── SECTION 6 : CONFIGURATION
└── _DesignSystemFooter
```

## ⚠️ PAS D'EMOJIS sur la Welcome page

Utiliser des formes géométriques (rectangles, cercles, dots) avec les couleurs DS.

## Sections

### Section 1 — Hero
```
Frame hero (vertical, gap=32, alignItems=CENTER, padding=80)
├── fills={Primary/colorPrimaryBg}, cornerRadius=24
├── Overline — 14px Medium UPPERCASE, letterSpacing=3, {Primary/colorPrimary}
├── Titre — 56px Extra Bold, {Neutral/colorText}, CENTER
├── Sous-titre — 20px Regular, {Neutral/colorTextSecondary}, CENTER
├── Divider — rectangle 80×4, {Primary/colorPrimary}, cornerRadius=2
└── CTA row (horizontal, gap=16)
    ├── Button Primary instance → "Explorer les composants"
    └── Button Secondary instance → "Voir les guidelines"
```

### Section 2 — Stats (chiffres clés)
```
Frame stats (horizontal, gap=0, fill width, justify=SPACE_BETWEEN)
├── 4 items, chacun :
│   ├── Valeur — 40px Extra Bold, {Primary/colorPrimary}
│   └── Label — 14px Regular, {Neutral/colorTextSecondary}
```
Stats : "21" Components, "120+" Design Tokens, "12" Templates, "8" Foundations (adapter au DS réel).

### Section 3 — Ce qui est inclus (feature cards)
```
_SectionMetadata + Grid (horizontal wrap, gap=24)
├── 4 cards : padding=32, fills={Neutral/colorBgSecondary}, cornerRadius={Radius/lg}, border 1px
│   ├── Icône abstraite — rectangle 40×40, cornerRadius=10, fills={Primary/colorPrimaryBg}
│   │   └── Rectangle intérieur 20×20 centré, {Primary/colorPrimary} (PAS d'emoji)
│   ├── Titre — 18px Semi Bold
│   └── Description — 14px Regular, {Neutral/colorTextSecondary}
```
Cards : "Variables & Tokens", "Typography System", "Spacing & Sizing", "Components"

### Section 4 — Composants Highlight
```
_SectionMetadata + Row (horizontal wrap, gap=16)
├── 6-8 instances Default/MD dans des cards (padding=24, cornerRadius=12, border 1px)
│   button Primary, button Secondary, text field, select, checkbox, badge...
```

### Section 5 — Navigation rapide
```
_SectionMetadata + Frame badges (horizontal wrap, gap=12)
├── Badge/pill par page (SANS emoji)
│   ├── fills={Primary/colorPrimaryBg}, cornerRadius=99, border 1px
│   ├── Dot — rectangle 8×8, cornerRadius=4, {Primary/colorPrimary}
│   └── Nom page — 14px Medium, {Primary/colorPrimary}
```
Pages : Colors, Typography, Size, Border, Space, Radius, Shadow, Breakpoint, Icons, Components, Layouts

### Section 6 — Configuration
```
_SectionMetadata + Frame rows (vertical, gap=12)
├── Ligne par paramètre :
│   ├── Label — 14px Medium, {Neutral/colorTextSecondary}, width=160
│   └── Value — 14px Semi Bold, {Neutral/colorText}
```
Paramètres : Police, Primaire (swatch+hex), Secondaire, Radius, Dark mode, Tiers.

## Règle visibilité

Tout élément visuel DOIT avoir des fills explicites. Un frame sans fills est INVISIBLE.
Les cards DOIVENT avoir fond + border pour être lisibles.

## Checklist

- [ ] Frame principal 2000px
- [ ] .documentation header en haut
- [ ] 6 sections avec contenu complet
- [ ] PAS d'emojis (formes géométriques uniquement)
- [ ] Cards avec fond visible + border
- [ ] Instances de composants réels dans le Highlight
- [ ] _DesignSystemFooter en bas
- [ ] Screenshot vérifié
