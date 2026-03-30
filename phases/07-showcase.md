# Phase 7 — Showcase (Pages Components)

> **Appels MCP** : 1 appel par composant (5 appels pour Tier 1)
> **Prérequis** : Phase 2 (doc components) + Phase 4 (Icon CS) + Phase 5 (Component Sets)
> **Produit** : Page Components avec un layer séparé par composant (Light uniquement)
> **Vérification** : Screenshot après CHAQUE composant

## Structure de la page

### Un layer séparé par composant

**⚠️ CRITIQUE** : Chaque composant = 1 frame indépendant (layer) sur la page, PAS un frame enfant d'un frame principal partagé.

```
📄 Components (page)
├── [Layer Button] — Frame 2000px, Auto Layout vertical
│   ├── .documentation header (type=component, titre="Button")
│   └── Showcase (Light uniquement)
├── [Layer Text Field] — Frame 2000px
│   ├── .documentation header (titre="Text Field")
│   └── Showcase
├── [Layer Select]
├── [Layer Checkbox]
├── [Layer Radio]
├── [Layer Card]
├── [Layer Alert]
├── [Layer Badge]
├── [Layer Avatar]
├── [Layer Tag]
├── [Layer Tabs]
└── [Layer Modal]
```

**Pas de frame principal englobant** — chaque layer est positionné avec un gap Y de 200px.
**Pas de Showcase Dark** — Light uniquement.
**Pas de _DesignSystemFooter** sur cette page.

### Créer un layer composant

```javascript
function createCompLayer(name, yPos) {
  const frame = figma.createFrame();
  frame.name = name;
  frame.layoutMode = "VERTICAL";
  frame.counterAxisSizingMode = "FIXED";
  frame.resize(2000, 100);
  frame.primaryAxisSizingMode = "AUTO";
  frame.itemSpacing = 40;
  frame.paddingBottom = 40;
  frame.fills = [{type:'SOLID', color:{r:1,g:1,b:1}}];
  frame.clipsContent = false;
  frame.x = 0;
  frame.y = yPos;
  showPage.appendChild(frame);

  // Doc header — ATTENTION aux noms des champs texte
  // title → nom du composant
  // category label → "Component"
  // 2nd level → nom du composant
  // description → vide
  if (docVariant) {
    const hi = docVariant.createInstance();
    frame.appendChild(hi);
    hi.layoutSizingHorizontal = "FILL";
    findText(hi, 'title').characters = name;
    findText(hi, 'category label').characters = 'Component';
    findText(hi, '2nd level').characters = name;
    findText(hi, 'description').characters = '';
  }
  return frame;
}
```

**⚠️ Champs texte du `.documentation header`** :
- `title` (pas "Page Title")
- `description`
- `category label` (pas "Badge")
- `1st level` → "ds linkedin"
- `2nd level` → nom du composant

## ~~Règle Dark Mode~~ — SUPPRIMÉE

**PAS de Showcase Dark.** Light uniquement pour chaque composant.

## Règle Labels de section

Chaque groupe DOIT être précédé d'un label visuel :

```
Frame section (vertical, gap=12)
├── Label 14px SemiBold, UPPERCASE, letterSpacing=2
├── Divider 1px
└── Row (horizontal, gap=16, wrap) → instances
```

## Règle Icônes dans showcase

Les sections "Sizes" et "With Slots" DOIVENT montrer des instances du Icon Component Set.
**JAMAIS** de frames vides ou de texte emoji.

## Contenu par composant

### Button
| Section | Contenu |
|---|---|
| TYPES | Primary+leading(plus)+"Create", Secondary+leading(download)+"Export", Ghost+leading(settings)+"Settings", Danger+leading(trash)+"Delete" |
| SIZES | SM(plus+"Add"), MD(plus+"Add Item"), LG(plus+"Add New Item") — icône scale avec size |
| WITH SLOTS | leading only, trailing only, both, icon-only |
| LOADING | Primary loading=true, Secondary loading=true |
| STATES | Default, Hover, Active, Focus, Disabled (Primary MD) |
| COMPOSITION | 2 boutons (Primary "Save" + Ghost "Cancel") en footer formulaire |

### Text Field
| Section | Contenu |
|---|---|
| TYPES | Text, Password(trailing eye), Search(leading search), Email(leading envelope), Number |
| SIZES | SM, MD, LG (chacun avec leading search) |
| WITH SLOTS | Leading, Trailing, Both |
| STATUS | None, Error("This field is required"), Warning("Password is weak"), Success("Username is available") |
| STATES | Default, Hover, Focus, Disabled |
| COMPOSITION | 3 inputs empilés (Name, Email, Password) en formulaire vertical |

### Select
| Section | Contenu |
|---|---|
| DEFAULT | Select closed (MD) |
| SIZES | SM(globe+"Language"), MD, LG |
| OPEN STATE | Select open=true avec dropdown |
| WITH LEADING SLOT | globe+"Language", user+"Assignee", calendar+"Date range" |
| STATUS | None, Error, Warning, Success |
| STATES | Default, Hover, Focus, Disabled |
| COMPOSITION | Select "Country" + Select "City" côte à côte |

### Checkbox
| Section | Contenu |
|---|---|
| DEFAULT | unchecked + checked (MD) |
| SIZES | SM + MD (unchecked + checked) |
| GROUP | 4 checkboxes verticales (formulaire notifications) |
| STATES | Default, Hover, Focus, Disabled |

### Radio
| Section | Contenu |
|---|---|
| DEFAULT | unselected + selected (MD) |
| SIZES | SM + MD |
| GROUP | 4 radios verticales (choix de plan) |
| STATES | Default, Hover, Focus, Disabled |

### Alert

**⚠️ SIZING CRITIQUE** : Les instances d'alert DOIVENT être en `layoutSizingHorizontal = "FILL"`, PAS FIXED à 480px. Les mettre dans un col vertical en FILL aussi.

| Section | Contenu |
|---|---|
| TYPES | Info, Success, Warning, Error — chaque instance en FILL width, empilées verticalement avec gap=12 |

```javascript
// ⚠️ OBLIGATOIRE pour les Alert dans le showcase
// Le col contenant les alertes doit être en FILL
col.layoutSizingHorizontal = "FILL";
// Chaque instance doit aussi être en FILL
for (const inst of col.children) {
  if (inst.type === 'INSTANCE') inst.layoutSizingHorizontal = "FILL";
}
```

### Card
| Section | Contenu |
|---|---|
| TYPES | default, elevated, outline |

### Badge
| Section | Contenu |
|---|---|
| SIZES | SM, MD par variant |

### Avatar
| Section | Contenu |
|---|---|
| SIZES | XS, SM, MD, LG, XL |

### Tag
| Section | Contenu |
|---|---|
| VARIANTS | Default, avec removable=true |

### Tabs
| Section | Contenu |
|---|---|
| DEFAULT | Instance par défaut |

### Modal
| Section | Contenu |
|---|---|
| SIZES | SM, MD, LG |

## Page 🏗️ Layouts (appel séparé)

Présentation des Layout Components + Templates avec instances.

Pour chaque layout/template :
1. _SectionMetadata
2. Instance avec **slots colorés** (palette rose/violet/bleu/vert/jaune) + labels texte

```javascript
function colorSlot(instance, slotName, color, label) {
  const slot = instance.findOne(n => n.name === slotName);
  if (!slot) return;
  slot.fills = [{type: 'SOLID', color: color}];
  // Ajouter label texte centré dans le slot
}
```

**Ne JAMAIS laisser les slots sans fills** — sinon blanc sur blanc = invisible.

## Checklist

- [ ] Chaque composant = 1 layer séparé (frame indépendant sur la page)
- [ ] .documentation header avec `title` = nom du composant (pas "Page Title")
- [ ] Showcase Light uniquement (PAS de Dark)
- [ ] Labels de section (UPPERCASE) avant chaque groupe
- [ ] Section COMPOSITION pour chaque composant
- [ ] Gap Y de 200px entre chaque layer
- [ ] Screenshot vérifié après chaque composant
