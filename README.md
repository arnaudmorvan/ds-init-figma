# initDsWithClaude

Skill Copilot pour créer un **Design System complet dans Figma** via l'API Plugin et les outils MCP.

## Principe

Ce skill guide un agent IA (Copilot / Claude) à travers une série de phases pour générer un Design System Figma structuré : variables, tokens, composants, pages de documentation, layouts et showcase.

## Structure

```
SKILL.md              # Point d'entrée du skill
phases/
  01-setup.md         # Création du fichier Figma, variables, tokens
  02-doc-components.md # Composants de documentation (Header, Section, etc.)
  03-layouts-templates.md # Templates de pages et layouts
  04-icons.md         # Système d'icônes
  05-components.md    # Composants UI (Button, Input, Alert, etc.)
  06-foundations.md   # Pages fondations (couleurs, typographie, spacing)
  07-showcase.md      # Pages showcase par composant
  08-welcome.md       # Page d'accueil du DS
references/
  plugin-api-patterns.md # Patterns Figma Plugin API testés
  rules.md            # Règles et contraintes
  style-guide.md      # Guide de style visuel
```

## Utilisation

1. Ajouter ce dossier dans `~/.copilot/skills/ds-init/`
2. Dans VS Code avec Copilot, invoquer le skill avec le nom du projet
3. Le skill pose les questions de configuration (polices, couleurs, arrondis, ombres, spacing, breakpoints) puis crée le fichier Figma phase par phase

## Prérequis

- VS Code avec GitHub Copilot
- Serveur MCP Figma configuré (accès API Plugin)
