# Phase 2 — Documentation Components

> **Appels MCP** : 1× `mcp_figma_use_figma`
> **Prérequis** : Phase 1 terminée (fichier + variables créés)
> **Produit** : 4 composants de documentation sur la page 🔒 _Components
> **Vérification** : Screenshot de la page _Components

## Composants à créer

Tous sur la page `🔒 _Components`. Ce sont les "briques de mise en page" utilisées par toutes les autres phases.

### 2.1 `.documentation header` — Component Set (3 variants)

Un header de page professionnel, inspiré SlothUI.

**Variants :**
- `type=component` — pour les pages composants (badge "Components")
- `type=documentation` — pour les pages Foundations (badge "Foundations")
- `type=elements` — pour les sections Elements (titre fixe + description)

**Structure de chaque variant (component et documentation) :**
```
Auto Layout vertical, gap=80, padding=40
├── cornerRadius=40, border 1px {Neutral/colorBorder}, fills={Neutral/colorBg}
├── Gradient Ellipse (décoration) — ellipse ~1150×1150, absolute top=-990 right=-450
│   gradient radial {Primary/colorPrimary}→transparent, opacity=0.08
├── Top Row (horizontal, justify=space-between, fill width)
│   ├── Title Block (vertical, gap=24, max-width=960)
│   │   ├── [title] ← Text property, 60px ExtraBold
│   │   └── [description] ← Text property, 20px Regular
│   └── Category Badge (horizontal, gap=10, padding=12/20)
│       ├── fills={Primary/colorPrimary}, cornerRadius=999
│       ├── [icon] ← 20×20 blanc
│       └── [label] ← "Components"/"Foundations", 16px Bold, blanc
└── Breadcrumb Bar (horizontal, justify=space-between, fill width)
    ├── Logo DS (40×40, cornerRadius=15) + [1st level] → [2nd level]
    └── [link] optionnel
```

**Variant `type=elements` :**
```
Auto Layout vertical, gap=4, padding=24/0
├── [title] ← "Elements", 32px Bold
└── [description] ← 16px Regular
```

### 2.2 `_DesignSystemFooter` — Main Component

**⚠️ Règle anti-clipping** : Tous les textes du footer DOIVENT avoir `textAutoResize = "WIDTH_AND_HEIGHT"` ou être dans un Auto Layout avec `layoutSizingHorizontal = "FILL"` + `textAutoResize = "HEIGHT"`.
Ne JAMAIS `resize()` un texte sans mettre `textAutoResize` après.

```
Auto Layout horizontal, justify=space-between, padding=40, alignItems=CENTER
├── cornerRadius=40, border 1px {Neutral/colorBorder}, fills={Neutral/colorBg}
├── Gradient Ellipse (même décoration, miroir, opacity=0.06)
├── Left: Logo DS (40×40) + [DS Name] 18px Bold
└── Right: [tagline] 14px Regular + [link] 14px SemiBold primary
```

```javascript
// Pattern texte safe pour le footer
const text = figma.createText();
text.fontName = {family:'Inter', style:'Regular'};
text.fontSize = 14;
text.characters = "Design System v1.0";
text.textAutoResize = "WIDTH_AND_HEIGHT"; // ← OBLIGATOIRE
// OU dans un auto layout :
parent.appendChild(text);
text.layoutSizingHorizontal = "FILL"; // le texte prend la largeur dispo
text.textAutoResize = "HEIGHT"; // et grandit en hauteur
```

### 2.3 `_SectionMetadata` — Main Component

```
Auto Layout horizontal, justify=space-between, alignItems=CENTER, height=38
├── [label] ← Text property, 14px SemiBold
├── Dotted Line ← rectangle height=1, FILL, dashPattern=[4,4]
└── [specs] ← Text property, 13px Regular, gris secondaire
```

### 2.4 `divider` — Main Component

Simple ligne horizontale :
```
Rectangle height=1, FILL width, fills={Neutral/colorBorder}, opacity=0.3
```

## Code pour alimenter les instances

```javascript
// Trouver un variant
const docHeaderSet = page.findOne(n => n.type === "COMPONENT_SET" && n.name === ".documentation header");
const variant = docHeaderSet.findOne(n => n.name === "type=component");
const inst = variant.createInstance();

// Alimenter les textes
function setText(instance, nodeName, text) {
  const node = instance.findOne(n => n.name === nodeName && n.type === "TEXT");
  if (node) node.characters = text;
}
setText(inst, "title", "Button");
setText(inst, "description", "Clickable element that performs actions.");
setText(inst, "1st level", "Components");
setText(inst, "2nd level", "Button");
setText(inst, "category label", "Components");
```

## Checklist

- [ ] `.documentation header` Component Set avec 3 variants fonctionnels
- [ ] `_DesignSystemFooter` Main Component visible et alimentable
- [ ] `_SectionMetadata` Main Component avec ligne pointillée
- [ ] `divider` Main Component
- [ ] Screenshot vérifié — les 4 composants visibles sur _Components
