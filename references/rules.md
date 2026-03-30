# Règles critiques — Figma Plugin API pour ds-init

> Ces règles provoquent des **erreurs silencieuses ou des régressions visuelles** si ignorées.
> Chaque règle a été découverte après un bug réel et documentée.

## 1. Auto Layout — HUG Content OBLIGATOIRE

Tout frame avec `layoutMode` DOIT avoir `primaryAxisSizingMode = "AUTO"` (HUG content).

**Exceptions** : frames d'écran simulant un viewport (1440×900).

```javascript
// ✅ Pattern sûr
frame.layoutMode = "VERTICAL";
frame.counterAxisSizingMode = "FIXED";
frame.resize(2000, 10); // width de page
frame.primaryAxisSizingMode = "AUTO"; // TOUJOURS après resize()
frame.clipsContent = false;
```

**Piège** : `resize(w, h)` force `primaryAxisSizingMode = "FIXED"` → toujours remettre `"AUTO"` après.

## 2. FILL après appendChild — Ordre obligatoire

`layoutSizingHorizontal/Vertical = "FILL"` ne fonctionne QUE sur un enfant déjà rattaché.

```javascript
// ❌ CRASH
child.layoutSizingHorizontal = "FILL";
parent.appendChild(child);

// ✅ CORRECT
parent.appendChild(child);
child.layoutSizingHorizontal = "FILL";
```

## 3. Fills — Pas de propriété `a` (alpha)

```javascript
// ❌ "Unrecognized key(s) in object: 'a'"
frame.fills = [{type: 'SOLID', color: {r:1, g:1, b:1, a: 0.7}}];

// ✅ Utiliser opacity sur le fill
frame.fills = [{type: 'SOLID', color: {r:1, g:1, b:1}, opacity: 0.7}];
```

## 4. strokeAlign singulier

```javascript
// ❌ TypeError: object is not extensible
frame.strokesAlign = "INSIDE";

// ✅ CORRECT
frame.strokeAlign = "INSIDE";
```

## 5. textAlignHorizontal (pas textAlign)

```javascript
// ❌ read-only, causes error
text.textAlign = "CENTER";

// ✅ CORRECT
text.textAlignHorizontal = "CENTER";
```

## 6. Dark Mode — Activation explicite obligatoire

Les variables Dark ne s'activent **PAS** automatiquement. Il faut appeler `setExplicitVariableModeForCollection()` sur **chaque** frame qui doit afficher en dark.

```javascript
const collections = await figma.variables.getLocalVariableCollectionsAsync();
const colorCol = collections.find(c => c.name === 'Color');
const darkModeId = colorCol.modes.find(m => m.name === 'Dark').modeId;

// ⚠️ CETTE LIGNE EST CRITIQUE — sans elle, frame = Light
darkFrame.setExplicitVariableModeForCollection(colorCol.id, darkModeId);

// Lier le fond à la variable sémantique
const neutralBgVar = allVars.find(v => v.name === 'Neutral/colorBg');
darkFrame.setBoundVariable('fills', 0, 'color', neutralBgVar.id);
```

**Sans cet appel** : fond blanc, texte noir, dark mode invisible.

## 7. Vecteurs — fills = [] obligatoire

Les vecteurs créés via `figma.createVector()` ont un remplissage noir par défaut.

```javascript
const check = figma.createVector();
check.fills = [];  // ← OBLIGATOIRE pour les traits (strokes only)
check.strokes = [{type: 'SOLID', color: {r:1,g:1,b:1}}];
```

## 8. setCurrentPageAsync (pas d'assignation directe)

```javascript
// ❌ INTERDIT
figma.currentPage = myPage;

// ✅ CORRECT
await figma.setCurrentPageAsync(myPage);
```

## 9. Composants invisibles sur fond blanc

**Cause #1 de régressions visuelles.** Les composants formulaire DOIVENT avoir des strokes visibles :

| Composant | Problème sans strokes | Fix |
|---|---|---|
| Text Field | Rectangle blanc invisible | `strokes=[{Neutral/colorBorder}], strokeWeight=1, strokeAlign="INSIDE"` |
| Select | Idem | Idem |
| Button Secondary | Pas de contour | Ajouter stroke `{Neutral/colorBorder}` |
| Button Ghost | Texte invisible si colorText=noir | `text.fills = [{Primary/colorPrimary}]` |
| Card default | Rectangle blanc invisible | Ajouter shadow OU stroke |

Appliquer sur **TOUS les variants** (y compris state=Default, status=None).

## 10. Positionnement des frames sur une page

Figma démarre tous les frames à `x=0, y=0`. Sans positionnement explicite, ils se superposent.

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

## 11. Fonts — Charger avant toute opération texte

```javascript
await figma.loadFontAsync({ family: 'Inter', style: 'Regular' });
await figma.loadFontAsync({ family: 'Inter', style: 'Medium' });
await figma.loadFontAsync({ family: 'Inter', style: 'Semi Bold' });
await figma.loadFontAsync({ family: 'Inter', style: 'Bold' });
await figma.loadFontAsync({ family: 'Inter', style: 'Extra Bold' });
```

## 12. Slots — createSlot() N'EXISTE PAS

**⛔ `createSlot()` n'est PAS disponible** dans le Plugin API Figma.
L'appeler ne lève pas d'erreur visible mais ne crée rien → composants vides.

Les "slots" doivent être simulés par des **frames colorées nommées** :

```javascript
// ✅ CORRECT — Frame colorée comme slot
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

// ❌ INTERDIT — n'existe pas
const slot = comp.createSlot(); // silently fails
```

Voir les couleurs de slots dans style-guide.md.

## 13. Detach avant appendChild dans les instances

On ne peut PAS ajouter d'enfants dans une instance. Toujours détacher d'abord.

```javascript
const inst = templateComponent.createInstance();
const frame = inst.detachInstance(); // → frame éditable
// Maintenant on peut appendChild dans les slots
```

## 14. Texte large — textAutoResize = "HEIGHT"

Pour les textes > 24px, utiliser `textAutoResize = "HEIGHT"` (largeur fixe, hauteur auto) pour éviter le clipping.

## 15. Vérification après chaque appel

**Règle d'or** : Screenshot + vérification **immédiatement** après chaque `mcp_figma_use_figma`.

```
APRÈS CHAQUE appel mcp_figma_use_figma :
1. get_screenshot de la page modifiée
2. Vérifier : éléments visibles ? superpositions ? hauteurs effondrées ?
3. Si problème → corriger AVANT de passer à l'appel suivant
```

## 16. Un LAYER par Component Set sur _Components

Chaque Component Set créé avec `combineAsVariants()` DOIT être un **enfant direct** de la page `_Components`. Pas de frame parent commun.

Dans le panel Layers de Figma, chaque Component Set doit apparaître comme un layer séparé : `button`, `text field`, `select`, `checkbox`, `radio`.

```javascript
// ✅ CORRECT
const buttonSet = figma.combineAsVariants(variants, compPage); // direct sur la page

// ❌ INTERDIT
wrapper.appendChild(buttonSet); // dans un frame parent
```

## 17. Swatches de couleur — Résoudre la valeur réelle

Les variables bindées ne changent pas le fill visuel de base dans le code MCP.
Pour que les swatches de couleur soient **visibles dans les screenshots**, il faut :

1. Résoudre la valeur réelle de la variable (via `valuesByMode`)
2. L'appliquer comme fill de base (couleur Figma native)
3. PUIS binder la variable par-dessus

```javascript
// ❌ Invisible dans screenshot (fill gris + binding)
swatch.fills = [{type:'SOLID', color:{r:0.9,g:0.9,b:0.9}}];
bindFill(swatch, v);

// ✅ Visible (fill résolu + binding)
const resolved = resolveColor(v, allVars, collections);
swatch.fills = [{type:'SOLID', color: resolved}];
bindFill(swatch, v);
```

## 18. Texte dans les composants — Anti-clipping

Tout texte dans un composant (footer, header, card) DOIT avoir :
- `textAutoResize = "WIDTH_AND_HEIGHT"` (texte libre)
- OU `textAutoResize = "HEIGHT"` + largeur fixe/FILL (texte dans un layout)

**JAMAIS** laisser `textAutoResize = "NONE"` (défaut) qui clippe le texte.

```javascript
// ✅ Texte dans un auto layout
parent.appendChild(text);
text.layoutSizingHorizontal = "FILL";
text.textAutoResize = "HEIGHT";

// ✅ Texte libre
text.textAutoResize = "WIDTH_AND_HEIGHT";
```
