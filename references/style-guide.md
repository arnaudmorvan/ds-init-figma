# Style Guide — Convention visuelle ds-init

> Conventions esthétiques inspirées de SlothUI. Ce ne sont PAS des règles critiques
> mais des recommandations pour un rendu professionnel cohérent.

## Dimensions de page

- **Largeur page** : 2000px
- **Largeur contenu** : 1952px (page - padding 24×2)
- **Largeur contenu indenté** : 1776px (pour les sections sous le header, padding supplémentaire 88px)
- **Gap entre sections** : 80px
- **Gap entre éléments internes** : 24-32px

## Header (.documentation header)

- cornerRadius = 40
- border = 1px `{Neutral/colorBorder}`
- Gradient décoratif : ellipse ~1150×1150, position hors-cadre, gradient radial `{Primary/colorPrimary}` → transparent, opacity=0.08
- Titre : 60px ExtraBold
- Badge catégorie : pill rempli `{Primary/colorPrimary}`, texte blanc, cornerRadius=999
- Breadcrumbs : logo DS 40×40 + niveaux 18px Bold

## Footer (_DesignSystemFooter)

- Même style arrondi que le header (cornerRadius=40)
- Gradient décoratif miroir
- Logo + tagline + lien
- Largeur = FILL dans le parent

## Barres de section (_SectionMetadata)

- Hauteur 38px
- Label gauche : 14px SemiBold
- Specs droite : 13px Regular, gris secondaire
- Ligne pointillée au milieu : FILL, dashPattern=[4,4]

## Labels de showcase

Chaque groupe de variants DOIT être précédé d'un titre visuel :

```
Frame section (Auto Layout vertical, gap=12)
├── Label ← 14px SemiBold, UPPERCASE, letterSpacing=2, colorTextSecondary
├── Divider ← 1px, colorBorder, opacity=0.5
└── Row content (Auto Layout horizontal, gap=16, wrap)
```

## Emoji-préfixes des pages

```
📄 👋 Welcome
📄 ---                       ← Séparateur
📄 🧱 Foundations
📄 ⚙️ Icons
📄 ---
📄 🏗️ Layouts
📄 Components
📄 ---
📄 🔒 _Components
```

- Emoji-préfixes sur les pages système (Foundations, Icons, Layouts, Welcome)
- PAS d'emoji sur les pages composants (buttons, inputs…) — minuscule

## Nommage

| Type | Convention | Exemple |
|------|-----------|---------|
| Composant public | minuscule, sans préfixe | `button`, `checkbox` |
| Sous-composant | `.elements / parent / name` | `.elements / select / option` |
| Documentation | `.documentation header` | Component Set, 3 variants |
| Footer | `_DesignSystemFooter` | Main Component |
| Section divider | `_SectionMetadata` | Main Component |
| Slot natif | nom simple | `header`, `leading`, `content` |
| Collection variable | PascalCase | `Color`, `Typography`, `Size` |
| Variable | `Category/tokenName` | `Primary/colorPrimary`, `Size/controlHeight/md` |

## Composition showcase

Après les sections de variants, ajouter une section "COMPOSITION" montrant l'usage réel :

```
Frame "Composition" (Auto Layout vertical, gap=16)
├── Label "COMPOSITION" (14px SemiBold UPPERCASE)
├── Divider (1px)
└── Frame contexte (padding=24, cornerRadius=12,
    fills=colorBgSecondary, border 1px colorBorder)
    └── Agencement réaliste de composants combinés
```

## Welcome page

- PAS D'EMOJIS — formes géométriques + couleurs DS uniquement
- Sections : Hero, Stats, Features, Components Highlight, Navigation rapide, Config
- Cards avec fond visible (`colorBgSecondary` + border) — jamais transparentes

## Couleurs slots (page Layouts)

Les SlotNode sont invisibles par défaut. Sur la page Layouts, les colorier :

| Slot | Couleur | Hex |
|------|---------|-----|
| header / topbar | rose clair | #FFE0ED |
| sidebar / nav | violet clair | #EBE0FF |
| content / main | bleu clair | #E0F0FF |
| footer | vert clair | #E0FFED |
| secondaires | jaune clair | #FFF8E0 |
