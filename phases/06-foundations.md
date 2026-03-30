# Phase 6 — Page Foundations

> **Appels MCP** : 1 appel par foundation (8 appels)
> **Prérequis** : Phase 1 (variables) + Phase 2 (doc components)
> **Produit** : Page 🧱 Foundations avec contenu visuel
> **Vérification** : Screenshot après CHAQUE foundation

## Principe : un appel = une foundation

Chaque foundation est un **Section frame (layer)** dans le frame principal de la page 🧱 Foundations.

**Premier appel** : créer le frame principal + le doc header + la première foundation.
**Appels suivants** : ajouter les sections au frame principal existant.
**Dernier appel** : ajouter le footer.

### Frame principal (créé au premier appel)

```javascript
const main = figma.createFrame();
main.name = "🧱 Foundations";
main.layoutMode = "VERTICAL";
main.counterAxisSizingMode = "FIXED";
main.resize(2000, 10);
main.primaryAxisSizingMode = "AUTO"; // HUG vertical
main.itemSpacing = 120;
main.paddingTop = 24; main.paddingBottom = 24;
main.paddingLeft = 24; main.paddingRight = 24;
main.fills = [{type: "SOLID", color: {r: 1, g: 1, b: 1}}];
main.clipsContent = false;
```

## Appel 6a — Colors

`_SectionMetadata` (label="Colors", specs="Primitives + Semantics") + 3 frames :

### Primitive colors

**⚠️ CRITIQUE** : Les swatches DOIVENT afficher les couleurs réelles, pas du gris.
Le binding variable seul peut ne pas se voir dans le screenshot si la variable résout vers une couleur identique au fill initial.

**Pattern obligatoire** : résoudre la valeur réelle de la variable et l'utiliser comme fill de base, PUIS binder la variable par-dessus.

#### Mapping catégories → préfixes variables

| Catégorie | Préfixe variable | Shades |
|-----------|-----------------|--------|
| Primary | Blue | 50,100,200,300,400,500,600,700,800,900,950 |
| Secondary | Purple | 50,100,200,300,400,500,600,700,800,900,950 |
| Neutral | Gray | 0,50,100,200,300,400,500,600,700,800,900,950 |
| Success | Green | 50,100,200,300,400,500,600,700,800,900,950 |
| Warning | Yellow | 50,100,200,300,400,500,600,700,800,900,950 |
| Danger | Red | 50,100,200,300,400,500,600,700,800,900,950 |

#### Structure complète par rangée

```javascript
// Récupérer les variables ET les collections pour résoudre les valeurs
const allVars = await figma.variables.getLocalVariablesAsync('COLOR');
const collections = await figma.variables.getLocalVariableCollectionsAsync();

function resolveColor(varObj) {
  if (!varObj) return {r:0.9,g:0.9,b:0.9};
  const col = collections.find(c => c.id === varObj.variableCollectionId);
  const modeId = col.modes[0].modeId; // Light mode
  const val = varObj.valuesByMode[modeId];
  if (val && val.r !== undefined) return {r: val.r, g: val.g, b: val.b};
  if (val && val.type === 'VARIABLE_ALIAS') {
    const target = allVars.find(v => v.id === val.id);
    if (target) return resolveColor(target);
  }
  return {r:0.9,g:0.9,b:0.9};
}

function rgbToHex(c) {
  return '#' + [c.r,c.g,c.b].map(x => Math.round(x*255).toString(16).padStart(2,'0')).join('').toUpperCase();
}

// Pour chaque catégorie → créer une ROW horizontale
for (const cat of categories) {
  const row = figma.createFrame();
  row.name = cat.label;
  row.layoutMode = "HORIZONTAL";
  row.counterAxisSizingMode = "AUTO";
  row.primaryAxisSizingMode = "AUTO";
  row.itemSpacing = 12;
  row.fills = [];

  // Label wrapper (vertical, paddingTop=16 pour centrer vs swatches)
  const labelWrap = figma.createFrame();
  labelWrap.layoutMode = "VERTICAL"; labelWrap.fills = [];
  labelWrap.counterAxisSizingMode = "AUTO"; labelWrap.primaryAxisSizingMode = "AUTO";
  labelWrap.paddingTop = 16;
  const catLabel = figma.createText();
  catLabel.fontName = {family: "Inter", style: "Semi Bold"};
  catLabel.characters = cat.label; catLabel.fontSize = 14;
  catLabel.resize(80, catLabel.height);
  labelWrap.appendChild(catLabel);
  row.appendChild(labelWrap);

  // Pour chaque shade → swatchContainer vertical (swatch + shade + hex)
  for (const shade of cat.shades) {
    const varName = `${cat.prefix}/${shade}`;
    const v = allVars.find(vv => vv.name === varName);
    const resolved = resolveColor(v);

    const container = figma.createFrame();
    container.layoutMode = "VERTICAL";
    container.counterAxisSizingMode = "AUTO"; container.primaryAxisSizingMode = "AUTO";
    container.itemSpacing = 4; container.fills = [];

    // Swatch 48×48, cornerRadius=8, fill résolu + variable bound
    const swatch = figma.createFrame();
    swatch.resize(48, 48); swatch.cornerRadius = 8;
    swatch.fills = [{type:'SOLID', color: resolved}];
    if (v) {
      const fills = JSON.parse(JSON.stringify(swatch.fills));
      fills[0] = figma.variables.setBoundVariableForPaint(fills[0], 'color', v);
      swatch.fills = fills;
    }
    container.appendChild(swatch);

    // Shade label (ex: "500") + hex label (ex: "#2563EB")
    const sl = figma.createText();
    sl.fontName = {family:"Inter",style:"Regular"}; sl.characters = shade;
    sl.fontSize = 10; sl.textAlignHorizontal = "CENTER";
    container.appendChild(sl);

    const hl = figma.createText();
    hl.fontName = {family:"Inter",style:"Regular"}; hl.characters = rgbToHex(resolved);
    hl.fontSize = 9; hl.fills = [{type:'SOLID',color:{r:0.55,g:0.55,b:0.55}}];
    hl.textAlignHorizontal = "CENTER";
    container.appendChild(hl);

    row.appendChild(container);
  }
  colorsSection.insertChild(insertIdx++, row);
}
```

#### Checklist Primitive Colors
- [ ] Chaque ROW = frame HORIZONTAL (AUTO × AUTO, itemSpacing=12, fills=[])
- [ ] Label wrapper = frame VERTICAL avec paddingTop=16
- [ ] Chaque swatch = 48×48, cornerRadius=8, fill RÉSOLU + variable bound
- [ ] Shade label (fontSize=10) + hex label (fontSize=9) sous chaque swatch
- [ ] Utiliser `resolveColor()` — JAMAIS de fill gris ou vide
- [ ] `insertChild` après le titre "Primitive Colors" dans colorsSection

### Semantic color tokens

Structure : `semantic-grid` frame (VERTICAL, itemSpacing=16) avec :
- Pour chaque catégorie : label texte + row HORIZONTAL de token swatches

**Catégories et tokens à afficher :**

| Catégorie | Tokens |
|-----------|--------|
| Primary | colorPrimary, colorPrimaryHover, colorPrimaryActive |
| Neutral | colorText, colorTextSecondary, colorBorder |
| Feedback | colorSuccess, colorWarning, colorDanger, colorInfo |

**Structure par token swatch :**
```
tokenContainer (HORIZONTAL, itemSpacing=8, fills=[])
  → circle (FRAME 32×32, cornerRadius=16, fill résolu + variable bound)
  → tokenName (TEXT fontSize=11)
```

**Pattern** : même `resolveColor()` que pour les Primitives — résoudre la valeur, appliquer en fill, puis binder.

→ **Screenshot** de la page Foundations.

## Appel 6b — Typography

`_SectionMetadata` (label="Typography") + 2 frames :

### Typography styles
Tableau visuel : Preview | Alias | Font | Weight | Size | Line height
Chaque ligne = le texte rendu dans la vraie police/taille.

### Typography tokens
Deux colonnes Desktop / Mobile avec les valeurs brutes.

→ **Screenshot**.

## Appel 6c — Size

`_SectionMetadata` (label="Size") + 1 frame :

Tableau : Name | Token | Value
→ controlHeight (xs→xl), controlPadding, controlGap, iconSize

## Appel 6d — Border

`_SectionMetadata` (label="Border") + 1 frame :

Lignes d'épaisseurs visibles (none=0, thin=1, medium=2, thick=3).

## Appel 6e — Space

`_SectionMetadata` (label="Spacing") + 1 frame :

Colonnes : Name | Token | Value (barre visuelle proportionnelle + valeur px)
La barre colorée grandit proportionnellement.

## Appel 6f — Radius

`_SectionMetadata` (label="Radius") + 2 frames :

### Semantic radius tokens
Chaque ligne : name + carré avec le radius appliqué + valeur px

### Component radius tokens
Tableau : Name | Token | mapping

## Appel 6g — Shadow

`_SectionMetadata` (label="Shadow") + 1 frame :

4 cards (200×120) côte à côte : fond blanc, cornerRadius, ombre appliquée via `.effects`.

## Appel 6h — Breakpoint + Footer

`_SectionMetadata` (label="Breakpoint") + 2 frames :

### Breakpoint tokens
Tableau : breakpoint | min-width | max-width | columns | gutter | margin

### Grid visualization
4 device frames côte à côte (mobile 375, tablet 768, desktop 1440, large 1920).

+ `_DesignSystemFooter` à la fin du frame principal.

→ **Screenshot final** de la page Foundations complète.

## Checklist

- [ ] Frame principal créé avec Auto Layout vertical, gap=120
- [ ] .documentation header (type=documentation) en haut
- [ ] 8 sections de foundation avec contenu visuel concret
- [ ] Chaque section introduite par _SectionMetadata
- [ ] _DesignSystemFooter en bas
- [ ] Aucune section vide
- [ ] Screenshot vérifié pour chaque foundation
