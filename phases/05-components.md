# Phase 5 — Component Sets

> **Appels MCP** : 1 appel par composant (5 appels pour Tier 1)
> **Prérequis** : Phase 1 (variables Size, Color, Radius) + Phase 4 (Icon CS)
> **Produit** : Component Sets sur 🔒 _Components
> **Vérification** : Screenshot après CHAQUE composant

## Règle n°1 : Un composant par appel MCP

**Ne PAS créer tous les composants en un seul appel.** Faire :
- Appel 5a → Button
- Appel 5b → Text Field
- Appel 5c → Select
- Appel 5d → Checkbox
- Appel 5e → Radio
- Appel 5f → .elements / helper-text (sous-composant partagé)

Screenshot de vérification entre chaque appel.

## Règle n°1bis : Un LAYER par Component Set sur _Components

**⛔ CRITIQUE** : Chaque Component Set DOIT être un enfant DIRECT de la page `_Components`, PAS imbriqué dans un frame commun. Figma considère les enfants directs d'une page comme des "layers" dans le panel Layers.

```javascript
// ✅ CORRECT — chaque CS est un layer séparé sur la page
const compPage = figma.root.children.find(p => p.name === '🔒 _Components');
const buttonSet = figma.combineAsVariants(buttonVariants, compPage);
buttonSet.name = "button";
buttonSet.y = lastY + 200; // Espacement vertical entre layers

// ❌ INTERDIT — ne PAS mettre les CS dans un frame parent commun
const wrapper = figma.createFrame();
wrapper.name = "All Components";
compPage.appendChild(wrapper);
wrapper.appendChild(buttonSet); // ← NON, le CS doit être enfant direct de la page
```

**Positionnement** : Calculer `startY` en parcourant `compPage.children` pour trouver le point le plus bas, puis ajouter 200px de gap.

## Règle n°2 : Toutes les dimensions liées aux variables Size

```javascript
// ✅ OBLIGATOIRE — jamais de px hardcodés
button.setBoundVariable('minHeight', heightVar.id);
button.setBoundVariable('paddingLeft', paddingVar.id);
button.setBoundVariable('paddingRight', paddingVar.id);
button.setBoundVariable('itemSpacing', gapVar.id);
```

## Règle n°2bis : TOUTES les couleurs liées aux variables sémantiques (Dark mode)

**⛔ CRITIQUE** : AUCUNE couleur hardcodée dans les composants. Chaque fill et stroke doit être **bound** à une variable sémantique pour que le switch Light/Dark fonctionne.

### Mapping couleurs → variables

| Couleur | Variable sémantique |
|---|---|
| Fond blanc (input, card, modal) | `Neutral/colorBgElevated` |
| Fond page | `Neutral/colorBg` |
| Fond gris clair (hover, subtle) | `Neutral/colorBgSubtle` |
| Texte principal | `Neutral/colorText` |
| Texte secondaire | `Neutral/colorTextSecondary` |
| Texte tertiaire / disabled | `Neutral/colorTextTertiary` |
| Bordures | `Neutral/colorBorder` |
| Bordures subtiles | `Neutral/colorBorderSubtle` |
| Bleu primaire | `Primary/colorPrimary` |
| Bleu hover | `Primary/colorPrimaryHover` |
| Bleu active | `Primary/colorPrimaryActive` |
| Bleu fond léger | `Primary/colorPrimaryBg` |
| Violet secondaire | `Secondary/colorSecondary` |
| Vert success | `Feedback/colorSuccess` |
| Jaune warning | `Feedback/colorWarning` |
| Rouge danger | `Feedback/colorDanger` |
| Bleu info | `Feedback/colorInfo` |
| Fond success | `Feedback/colorSuccessBg` |
| Fond warning | `Feedback/colorWarningBg` |
| Fond danger | `Feedback/colorDangerBg` |
| Fond info | `Feedback/colorInfoBg` |
| Icônes (noir) | `Neutral/colorText` |
| Check/dot blanc | `Neutral/colorBg` |

### Pattern de binding obligatoire

```javascript
// Pour chaque fill coloré :
const colorVar = getColor('Primary/colorPrimary');
const resolved = resolveColor(colorVar); // voir 06-foundations.md
node.fills = [{type:'SOLID', color: resolved}]; // couleur de base visible
const fills = JSON.parse(JSON.stringify(node.fills));
fills[0] = figma.variables.setBoundVariableForPaint(fills[0], 'color', colorVar);
node.fills = fills;

// Pour chaque stroke :
const strokeVar = getColor('Neutral/colorBorder');
const resolvedStroke = resolveColor(strokeVar);
node.strokes = [{type:'SOLID', color: resolvedStroke}];
const strokes = JSON.parse(JSON.stringify(node.strokes));
strokes[0] = figma.variables.setBoundVariableForPaint(strokes[0], 'color', strokeVar);
node.strokes = strokes;

// ⚠️ INCLURE LES FONDS BLANCS — ils doivent devenir sombres en Dark mode
// Input backgrounds, card bg, modal bg → Neutral/colorBgElevated
// Checkbox/radio unchecked box → Neutral/colorBgElevated
inputRow.fills = [{type:'SOLID', color: {r:1,g:1,b:1}}];
const bgFills = JSON.parse(JSON.stringify(inputRow.fills));
bgFills[0] = figma.variables.setBoundVariableForPaint(bgFills[0], 'color', getColor('Neutral/colorBgElevated'));
inputRow.fills = bgFills;
```

**⚠️ ERREUR FRÉQUENTE** : oublier de binder les fonds blancs. Un `fills=[{type:'SOLID', color:{r:1,g:1,b:1}}]` sans variable reste blanc en Dark mode → composant cassé visuellement.

## Règle n°3 : Pattern A (flat variants) par défaut

Un seul Component Set par composant. Si > 60 variants → Pattern A+ (wrapper séparé).

---

## 5a — Button

```
🔶 button (Component Set)
├── Variants : type(Primary|Secondary|Ghost|Danger) × size(SM|MD|LG) × state(Default|Hover|Active|Focus|Disabled) + boolean loading
├── Auto Layout horizontal
│   ├── gap → {Size/controlGap/{size}}
│   ├── padding H → {Size/controlPadding/{size}}
│   ├── minHeight → {Size/controlHeight/{size}}
│   └── cornerRadius → {Radius/md}
├── ⬦ leading (SlotNode)
├── [Label] ← Text property
└── ⬦ trailing (SlotNode)
```

**Couleurs par type :**
| Type | Fill | Text | Border |
|---|---|---|---|
| Primary | {Primary/colorPrimary} | blanc | aucun |
| Secondary | transparent | {Neutral/colorText} | {Neutral/colorBorder} |
| Ghost | transparent | {Primary/colorPrimary} | aucun |
| Danger | {Feedback/colorDanger} | blanc | aucun |

**Couleurs par state :**
| State | Modification |
|---|---|
| Default | couleurs de base |
| Hover | fill légèrement plus clair (ex: PrimaryHover) |
| Active | fill plus foncé (ex: PrimaryActive) |
| Focus | outline 2px offset {Primary/colorPrimary} |
| Disabled | opacity=0.5 |

---

## 5b — Text Field

```
🔶 text field (Component Set)
├── Variants : type(Text|Password|Search|Email|Number) × size(SM|MD|LG) × state(Default|Hover|Focus|Disabled) × status(None|Error|Warning|Success)
├── Auto Layout vertical, gap=4
├── Input Row (horizontal)
│   ├── border → {Neutral/colorBorder} (ou status color)
│   ├── cornerRadius → {Radius/md}
│   ├── minHeight → {Size/controlHeight/{size}}
│   ├── padding H → {Size/controlPadding/{size}}
│   ├── ⬦ leading (SlotNode)
│   ├── [Value] ← Text property
│   └── ⬦ trailing (SlotNode)
├── Helper Text (horizontal, gap=4) — visible si status ≠ None
│   ├── [Status Icon] 16×16
│   └── [Message] 12px
```

**⚠️ CRITIQUE** : `strokes` visibles sur TOUS les variants (y compris status=None, state=Default).

---

## 5c — Select

```
🔶 select (Component Set)
├── Variants : size(SM|MD|LG) × state(Default|Hover|Focus|Disabled) × status(None|Error|Warning|Success) × open(false|true)
├── Auto Layout vertical, gap=4
├── Trigger (horizontal)
│   ├── même structure que text field
│   ├── ⬦ leading (SlotNode)
│   ├── [Value] ← Text property
│   └── Chevron ↓ (vecteur SVG, PAS frame vide)
├── Dropdown (visible si open=true)
│   ├── Shadow/md, Border/thin, Radius/md
│   └── SelectOption × 4-5 instances
├── Helper Text (même pattern que text field)
```

---

## 5d — Checkbox

```
🔶 checkbox (Component Set)
├── Variants : size(SM|MD) × state(Default|Hover|Focus|Disabled) × checked(false|true)
├── Auto Layout horizontal, gap=8, counterAxisAlignItems=CENTER
├── Box (SM=16×16, MD=20×20)
│   ├── cornerRadius={Radius/sm}
│   ├── unchecked : fills=transparent, strokes={Neutral/colorBorder}
│   └── checked : fills={Primary/colorPrimary} (COULEUR PLEINE, PAS transparent), check mark VECTEUR SVG blanc
└── [Label] ← Text
```

**⚠️ APPARENCE CHECKED — CRITIQUE** :
Quand `checked=true`, la box DOIT être **visuellement pleine** avec la couleur primaire.
Le check mark blanc par-dessus crée le contraste.

```javascript
// ✅ CHECKED = couleur pleine + check mark
if (checked === 'true') {
  box.fills = [{type:'SOLID', color:{r:0.15, g:0.39, b:0.92}}]; // couleur de base visible
  bindFill(box, getColor('Primary/colorPrimary')); // + variable binding
  box.strokes = []; // PAS de stroke quand checked
  // Check mark SVG
  const check = figma.createVector();
  check.vectorPaths = [{windingRule:'NONZERO', data:'M 3 8 L 7 12 L 13 4'}]; // adapté à la taille
  check.strokeWeight = 2;
  check.strokes = [{type:'SOLID', color:{r:1,g:1,b:1}}];
  check.fills = [];
  check.strokeCap = 'ROUND';
  check.strokeJoin = 'ROUND';
  check.resize(boxSize * 0.7, boxSize * 0.7);
  box.appendChild(check);
}

// ❌ INTERDIT : box.fills = [] quand checked (invisible !)
// ❌ INTERDIT : même apparence checked et unchecked
```
│   └── checked : fills={Primary/colorPrimary}, check mark VECTEUR SVG
└── [Label] ← Text
```

**⚠️ Le check mark DOIT être un vecteur SVG, PAS un caractère texte "✓".**

```javascript
const check = figma.createVector();
check.vectorPaths = [{ windingRule: "NONZERO", data: "M 1 4.5 L 4.5 8 L 11 1" }];
check.strokeWeight = 2;
check.strokes = [{type: 'SOLID', color: {r:1,g:1,b:1}}];
check.fills = []; // ← OBLIGATOIRE
check.strokeCap = "ROUND";
check.strokeJoin = "ROUND";
```

---

## 5e — Radio

```
🔶 radio (Component Set)
├── Variants : size(SM|MD) × state(Default|Hover|Focus|Disabled) × selected(false|true)
├── Auto Layout horizontal, gap=8, counterAxisAlignItems=CENTER
├── Outer circle (SM=16×16, MD=20×20)
│   ├── cornerRadius=9999
│   ├── unselected : fills=transparent, strokes={Neutral/colorBorder}
│   └── selected : fills={Primary/colorPrimary} (COULEUR PLEINE), inner dot (ellipse SM=6×6, MD=8×8, blanc)
└── [Label] ← Text
```

**⚠️ APPARENCE SELECTED — CRITIQUE** :
Quand `selected=true`, le cercle extérieur DOIT être **rempli couleur primaire**.
Le dot intérieur blanc crée le contraste.

```javascript
// ✅ SELECTED = cercle plein + dot blanc
if (selected === 'true') {
  outer.fills = [{type:'SOLID', color:{r:0.15, g:0.39, b:0.92}}]; // couleur de base
  bindFill(outer, getColor('Primary/colorPrimary')); // + variable
  outer.strokes = []; // PAS de stroke
  // Dot blanc
  const dot = figma.createEllipse();
  dot.resize(dotSize, dotSize);
  dot.fills = [{type:'SOLID', color:{r:1,g:1,b:1}}];
  outer.layoutMode = 'VERTICAL';
  outer.primaryAxisAlignItems = 'CENTER';
  outer.counterAxisAlignItems = 'CENTER';
  outer.appendChild(dot);
}

// ❌ INTERDIT : même apparence selected et unselected
// Le user DOIT voir une différence nette (plein vs vide)
```

---

## 5f — .elements / helper-text

Sous-composant partagé par text field et select :

```
🔶 .elements / helper-text (Main Component)
├── Auto Layout horizontal, gap=4, counterAxisAlignItems=CENTER
├── [Status Icon] ← vector 16×16
└── [Message] ← Text 12px, FILL
```

---

## Composants Tier 2+ (si sélectionnés)

Même pattern : **un appel MCP par composant**, screenshot après chaque.

| Composant | Variants principales | Slots |
|---|---|---|
| Card | variant(default\|elevated\|outline) | header, media, content, footer, actions |
| Modal | size(sm\|md\|lg) | header, content, footer |
| Tabs | — | tab items |
| Alert | type(info\|success\|warning\|error) | icon, title, description, action |

### Alert — Spécification détaillée

```
🔶 alert (Component Set)
├── Variants : type(info|success|warning|error)
├── Auto Layout HORIZONTAL, gap=12
│   ├── counterAxisSizingMode = "AUTO" ← ⚠️ CRITIQUE (pas FIXED sinon hauteur 10px !)
│   ├── primaryAxisSizingMode = "FIXED"
│   ├── padding: 12/16/12/16
│   └── cornerRadius = {Radius/md} (ex: 6)
├── stripe (Frame 3×FILL)
│   ├── layoutSizingVertical = "FILL" ← stretch pleine hauteur
│   ├── cornerRadius = 2
│   └── fills = couleur du type (info=bleu, success=vert, warning=jaune, error=rouge)
├── Icon Frame (20×20)
│   └── Vector SVG (icône du type)
└── text-content (Frame vertical, gap=2)
    ├── layoutMode = "VERTICAL"
    ├── primaryAxisSizingMode = "AUTO" ← ⚠️ CRITIQUE (hug vertical)
    ├── counterAxisSizingMode = "AUTO"
    ├── [Title] ← Text 14px Semi Bold ("Info", "Success", "Warning", "Error")
    └── [Description] ← Text 13px Regular
```

**Couleurs par type :**
| Type | Fill (opacity 8%) | Stripe | Icône |
|---|---|---|---|
| info | bleu primaire | bleu primaire | ℹ️ vecteur SVG |
| success | vert | vert | ✅ vecteur SVG |
| warning | jaune | jaune | ⚠️ vecteur SVG |
| error | rouge | rouge | ❗ vecteur SVG |

**⚠️ BUGS CONNUS À ÉVITER :**
1. `counterAxisSizingMode = "FIXED"` → hauteur 10px, alertes écrasées/chevauchées. **TOUJOURS "AUTO".**
2. `text-content` avec `primaryAxisSizingMode = "FIXED"` → texte coupé. **TOUJOURS "AUTO".**
3. `stripe` sans `layoutSizingVertical = "FILL"` → strip minuscule (1px). **TOUJOURS "FILL".**
4. Après création des variants, **repositionner** tous les enfants du CS avec un gap de 20px et **resize le CS** pour englober tous les variants.

```javascript
// ⚠️ OBLIGATOIRE après création du Component Set alert
// Les variants sont empilées à y=0 par défaut → les repositionner
let currentY = 0;
for (const v of alertCS.children) {
  v.x = 0;
  v.y = currentY;
  currentY += v.height + 20;
}
alertCS.resize(480, currentY - 20);
```
| Badge | variant, size(sm\|md) | content |
| Navbar | — | logo, nav-links, search, user |
| Sidebar | collapsed(false\|true) | logo, nav-items, footer |
| Breadcrumb | — | items |
| Pagination | size(sm\|md) | — |
| Table | striped(false\|true) | header, rows |
| Avatar | size(xs\|sm\|md\|lg\|xl) | — |
| Tag | variant, removable(false\|true) | — |

## Checklist

- [ ] Chaque composant créé dans un appel MCP séparé
- [ ] Screenshot vérifié après chaque composant
- [ ] Toutes les dimensions liées aux variables Size (aucun px hardcodé)
- [ ] **TOUTES les couleurs liées aux variables sémantiques (aucun hex hardcodé)**
- [ ] **Fonds blancs (input-row, trigger, card, modal) → Neutral/colorBgElevated**
- [ ] **Icônes noires → Neutral/colorText**
- [ ] **Check marks / dots blancs → Neutral/colorBg**
- [ ] Strokes visibles sur text field et select (tous variants)
- [ ] Check mark = vecteur SVG (pas caractère texte)
- [ ] SlotNode natifs (createSlot) pour leading/trailing
- [ ] Alert : counterAxisSizingMode="AUTO" + text-content primaryAxisSizingMode="AUTO" + stripe FILL
- [ ] Alert : variants repositionnées avec gap 20px + CS resize après création
- [ ] Showcase Alert : instances en layoutSizingHorizontal="FILL" (pas FIXED à 480px)
- [ ] **Test Dark mode : créer un frame avec setExplicitVariableModeForCollection → vérifier visuellement**
