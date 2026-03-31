# Phase 9 — Pages d'exemples

> **Appels MCP** : 1 appel par page d'exemple
> **Prérequis** : Phase 2 (doc components) + Phase 5 (Component Sets)
> **Produit** : Page 📐 Examples avec 4 layers (Login, Dashboard, Settings, User List)
> **Vérification** : Screenshot de chaque layer après création

## Objectif

Montrer des **compositions réalistes** utilisant les composants du DS.
Contrairement au Showcase (Phase 7) qui montre les composants isolés,
les Examples assemblent plusieurs composants en pages fonctionnelles.

## Page Figma

Créer une page `📐 Examples` entre `Components` et le séparateur `---`.

## Structure de la page

```
📐 Examples (page)
├── [Layer Login] — 1440×900, écran de connexion
├── [Layer Dashboard] — 1440×1200, page dashboard SaaS
├── [Layer Settings] — 1440×1200, page paramètres
└── [Layer User List] — 1440×1200, page liste utilisateurs
```

**Chaque layer** = frame indépendant positionné avec un gap Y de 200px.
**Largeur fixe** : 1440px (taille desktop réaliste, pas 2000px).
**Pas de doc header** — ce sont des maquettes pure composition.

## Règles communes

- Fond bound à `{Neutral/colorBg}`
- Tous les composants instanciés depuis la page `🔒 _Components`
- Personnaliser les textes des instances (pas de "Label" ou "Placeholder" génériques)
- Couleurs des éléments customisés bound aux variables sémantiques
- **Pas d'emoji** — formes géométriques uniquement

## Appel 9a — Login Page

```
Frame "Login" 1440×900
├── fills={Neutral/colorBg}
├── Layout : VERTICAL, center both
└── Card (elevated) — 420px wide, vertical, gap=32, p=40, cornerRadius=16
    ├── Logo zone — rectangle 48×48 colorPrimary + cornerRadius=12
    ├── Frame titles (vertical, gap=8, center)
    │   ├── "Sign in" — 28px Bold, colorText
    │   └── "Welcome back! Enter your credentials." — 14px Regular, colorTextSecondary
    ├── Frame form (vertical, gap=16, FILL width)
    │   ├── TextField instance (size=md, status=none) → placeholder "Email address"
    │   ├── TextField instance (size=md, status=none) → placeholder "Password"
    │   └── Frame remember-row (horizontal, SPACE_BETWEEN)
    │       ├── Checkbox instance (checked=false) → "Remember me"
    │       └── Text "Forgot password?" 14px Medium, colorPrimary
    ├── Button Primary instance (size=lg, FILL width) → "Sign in"
    ├── Divider + "or" text
    └── Button Secondary instance (size=lg, FILL width) → "Continue with Google"
```

## Appel 9b — Dashboard Page

```
Frame "Dashboard" 1440×1200
├── fills={Neutral/colorBg}
├── Navbar instance — FILL width, en haut
├── Frame body (horizontal, FILL)
│   ├── Sidebar instance — FILL height
│   └── Frame main (vertical, gap=32, padding=32, FILL both)
│       ├── Frame header-row (horizontal, SPACE_BETWEEN)
│       │   ├── Frame titles (vertical)
│       │   │   ├── "Dashboard" — 24px Bold
│       │   │   └── "Welcome back, here's what's happening." — 14px Regular
│       │   └── Button Primary "New Report" (leading=plus)
│       ├── Frame stats-row (horizontal, gap=24, wrap)
│       │   ├── 4× Stat card (Card elevated, vertical, gap=8, p=24, cornerRadius=12)
│       │   │   ├── Label — 14px Regular, colorTextSecondary
│       │   │   ├── Value — 28px Bold, colorText
│       │   │   └── Badge instance (variant appropriée)
│       │   Values: "Total Users" 2,847 | "Revenue" $48,290 | "Active Now" 342 | "Conversion" 12.5%
│       ├── Frame recent-section (vertical, gap=16)
│       │   ├── "Recent Activity" — 18px SemiBold
│       │   └── Table instance
│       └── Frame actions-row (horizontal, gap=16)
│           ├── Alert instance (type=info) "3 reports pending review"
│           └── (filler)
```

## Appel 9c — Settings Page

```
Frame "Settings" 1440×1200
├── fills={Neutral/colorBg}
├── Navbar instance
├── Frame body (horizontal, FILL)
│   ├── Sidebar instance
│   └── Frame main (vertical, gap=32, padding=32)
│       ├── "Account Settings" — 24px Bold
│       ├── Frame tabs-row → 4× TabItem instances (Profile*, Security, Notifications, Billing)
│       ├── Card outline (vertical, gap=24, p=32) — "Profile Information"
│       │   ├── Frame avatar-row (horizontal, gap=16)
│       │   │   ├── Avatar instance (size=xl)
│       │   │   └── Frame (vertical) name + role text
│       │   ├── Frame fields-2col (horizontal, gap=24, wrap)
│       │   │   ├── TextField "First name" → "Arnaud"
│       │   │   ├── TextField "Last name" → "Morvan"
│       │   │   ├── TextField "Email" → "arnaud@company.com"
│       │   │   └── Select "Role" → "Designer"
│       │   └── Frame actions (horizontal, gap=12, align=END)
│       │       ├── Button Ghost "Cancel"
│       │       └── Button Primary "Save changes"
│       ├── Card outline — "Notifications"
│       │   ├── Checkbox "Email notifications"
│       │   ├── Checkbox "Push notifications" (checked)
│       │   ├── Checkbox "Weekly digest"
│       │   └── Checkbox "Marketing emails"
│       └── Alert instance (type=success) "Changes saved successfully"
```

## Appel 9d — User List Page

```
Frame "User List" 1440×1200
├── fills={Neutral/colorBg}
├── Navbar instance
├── Frame body (horizontal, FILL)
│   ├── Sidebar instance
│   └── Frame main (vertical, gap=24, padding=32)
│       ├── Frame header-row (horizontal, SPACE_BETWEEN)
│       │   ├── Frame (vertical)
│       │   │   ├── Breadcrumb instance
│       │   │   ├── "Users" — 24px Bold
│       │   │   └── "Manage team members and their permissions." — 14px Regular
│       │   └── Button Primary "Add user" (leading=plus)
│       ├── Frame filters-row (horizontal, gap=12)
│       │   ├── TextField search (leading=search) → "Search users..."
│       │   ├── Select "Role" (leading=filter)
│       │   ├── Select "Status"
│       │   └── Tag "Active" (removable=true)
│       ├── Frame table-section (vertical, gap=0)
│       │   ├── Table instance (FILL width)
│       │   └── Frame pagination-row (horizontal, align=END, p=16)
│       │       └── Pagination instance (size=md)
│       └── Alert (type=warning) "2 users have pending invitations"
```

## Positionnement des layers

```javascript
let yPos = 0;
// Login: 1440×900
loginFrame.y = yPos; yPos += 900 + 200;
// Dashboard: 1440×1200
dashFrame.y = yPos; yPos += 1200 + 200;
// Settings: 1440×1200
settingsFrame.y = yPos; yPos += 1200 + 200;
// User List: 1440×1200
userListFrame.y = yPos;
```
