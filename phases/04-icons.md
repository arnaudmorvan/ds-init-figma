# Phase 4 — Icônes

> **Appels MCP** : 2× `mcp_figma_use_figma` (1 pour Icon Component Set, 1 pour page Icons)
> **Prérequis** : Phase 1 (variables Color/Size)
> **Produit** : Icon Component Set sur _Components + Page ⚙️ Icons
> **Vérification** : Screenshot page Icons

## Vérification préalable du dossier icônes

**Avant de commencer**, vérifier que le dossier d'icônes existe :

```bash
ls "/Users/arnaudmorvan/Dropbox/uicons-regular-rounded/svg/" | head -5
```

- **Si existe** → continuer avec ces SVGs
- **Si absent** → demander à l'utilisateur (via `vscode_askQuestions` ou chat) :
  - Quelle bibliothèque ? (Flaticon UIcons, Lucide, Heroicons, autre)
  - Chemin absolu vers le dossier SVG

## ⛔ RÈGLE ABSOLUE — Utiliser les VRAIS SVGs, jamais des paths inline

**INTERDIT** de dessiner des icônes à la main avec des paths simplifiés (cercles, lignes basiques).
Les icônes DOIVENT venir du dossier SVG configuré par l'utilisateur.

**Workflow obligatoire** :

1. **Mapper** les noms d'icônes aux fichiers SVG réels :
   ```javascript
   // Noms Flaticon UIcons Regular Rounded → préfixe "fi-rr-"
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
     // Recherche
     'search': 'fi-rr-search.svg',
     'filter': 'fi-rr-filter.svg',
     'bars-sort': 'fi-rr-bars-sort.svg',
     // Utilisateur
     'user': 'fi-rr-user.svg',
     'users': 'fi-rr-users.svg',
     'lock': 'fi-rr-lock.svg',
     'unlock': 'fi-rr-unlock.svg',
     'key': 'fi-rr-key.svg',
     // Communication
     'envelope': 'fi-rr-envelope.svg',
     'bell': 'fi-rr-bell.svg',
     'comment-alt': 'fi-rr-comment-alt.svg',
     // Contenu
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

2. **Lire** TOUS les fichiers SVG avec `cat` **AVANT** l'appel MCP (voir commande bash complète dans la section "Import SVG — Pattern" plus bas)

3. **Injecter** les strings SVG complètes dans le code MCP :
   ```javascript
   const svgStrings = {
     'plus': '<svg xmlns="http://www.w3.org/2000/svg" ...>...</svg>', // contenu RÉEL du fichier
     // ...
   };
   ```

**❌ INTERDIT** :
```javascript
// Ne JAMAIS inventer des paths
const svgStrings = {
  'plus': '<svg viewBox="0 0 24 24"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>'
};
```

**✅ OBLIGATOIRE** :
```javascript
// Utiliser le contenu EXACT lu depuis le fichier
const svgStrings = {
  'plus': '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M23,..." /></svg>'
};
```

## Appel 4a — Icon Component Set (sur _Components)

### Architecture

```
🔶 icon (Component Set)
├── Variants :
│   ├── name = plus | search | check | cross | angle-small-down | angle-small-right | user | settings | bell | eye | eye-crossed | info | exclamation | trash | (15 icônes minimum)
│   ├── size = xs(16) | sm(20) | md(24) | lg(32) | xl(48)
│   └── color = neutral | brand | danger | warning | success | inherit
```

### Icônes prioritaires (15 minimum pour le Component Set)

```
plus, minus, search, check, cross, angle-small-down, angle-small-right,
user, settings, bell, eye, eye-crossed, info, exclamation, trash
```

### Pattern de création

```javascript
// 1. Lire les SVGs (via terminal, pas le Plugin API)
// L'assistant lit les fichiers SVG avec ses outils et injecte les strings

// 2. Créer les variants
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
      
      // Couleur via variable binding
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

### Map couleur → variable

| color | Variable |
|---|---|
| neutral | `Neutral/colorTextSecondary` |
| brand | `Primary/colorPrimary` |
| danger | `Feedback/colorDanger` |
| warning | `Feedback/colorWarning` |
| success | `Feedback/colorSuccess` |
| inherit | fills=[] (hérite du parent) |

→ **Screenshot** de _Components — vérifier que le Component Set icon est visible.

## Appel 4b — Page ⚙️ Icons

### Structure

```
Page principale (2000px, Auto Layout vertical, gap=80, padding=24)
├── .documentation header (type=documentation, badge "Foundations", title "Icons")
├── Pour chaque catégorie :
│   ├── Titre catégorie (16px Bold)
│   └── Grille Auto Layout wrap, gap=24
│       └── Frame 80×100 par icône (vertical, gap=8, centerH)
│           ├── SVG node resize 32×32, fills={Neutral/colorText}
│           └── Label 12px Regular, gris 600, textAlign=CENTER
└── _DesignSystemFooter
```

### Catégories et icônes

```
Navigation :    angle-small-down, angle-small-up, angle-small-left, angle-small-right,
                arrow-left, arrow-right, bars-staggered
Actions :       plus, cross, check, pencil, trash, copy, share, download, upload
Recherche :     search, filter, bars-sort
Utilisateur :   user, users, lock, unlock, key
Communication : envelope, bell, comment-alt
Contenu :       picture, file, folder, link
Interface :     eye, eye-crossed, settings, home, info, exclamation, ban
Data :          chart-line-up, calendar, clock
Toggles :       sun, moon, globe
Commerce :      shopping-cart, credit-card, star
```

### Import SVG — Pattern

Le code Plugin API n'a pas accès au filesystem. L'assistant DOIT :
1. Lire les SVG locaux avec `cat` ou `read_file` **dans un appel terminal SÉPARÉ avant le MCP**
2. Stocker le contenu SVG complet de chaque fichier
3. Injecter les strings SVG **exactes** dans `figma.createNodeFromSvg()` dans le code MCP

**⚠️ Ne JAMAIS inventer ou simplifier les paths SVG** — toujours le contenu exact du fichier.

```bash
# Lire toutes les icônes nécessaires en une seule commande
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

- [ ] Dossier icônes vérifié et accessible
- [ ] SVGs lus et paths extraits (30 icônes minimum)
- [ ] Icon Component Set créé sur _Components (15 icônes × 5 sizes × 6 colors)
- [ ] Page ⚙️ Icons avec grille catégorisée
- [ ] Icônes visibles (32×32 minimum, fills Neutral/colorText)
- [ ] Screenshots vérifiés
