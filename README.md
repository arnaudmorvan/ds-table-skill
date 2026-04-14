# ds-table-skill

> Copilot Skill pour générer des **data tables Figma** via MCP — architecture column-first avec native slots (v3).

## Qu'est-ce que c'est ?

Un skill Copilot (GitHub Copilot Agent) qui permet de construire des tableaux de données complets dans Figma. Il utilise le MCP Figma pour créer des tables avec :

- **6 types de colonnes** : checkbox, texte, avatar+texte, badge, boutons d'action, icônes
- **Native slots Figma** : les cellules td restent des instances connectées au DS (zero `detachInstance()`)
- **Zebra striping**, **sort indicators**, **empty states**
- **Densités** : MD (défaut) et SM (compact)

## Installation

### Option 1 — Comme skill Copilot

Copier le dossier dans `~/.copilot/skills/ds-table/` :

```bash
git clone https://github.com/<user>/ds-table-skill.git ~/.copilot/skills/ds-table
```

Puis ajouter dans `.github/copilot-instructions.md` ou le fichier d'instructions de votre workspace :

```yaml
<skill>
  <name>ds-table</name>
  <description>Génère des data tables Figma via MCP (column-first, native slots v3)</description>
  <file>~/.copilot/skills/ds-table/SKILL.md</file>
</skill>
```

### Option 2 — Comme référence Knowledge Base

Copier `knowledge-base/` dans le dossier de votre projet et référencer les YAML depuis votre workflow.

## Prérequis

- **Figma MCP** configuré et fonctionnel
- Un **Design System Figma** avec les Component Sets : `th`, `td`, `badge`, `button`, `checkbox`, `avatar-profile-photo`
- Les Node IDs dans `table.yaml` sont à adapter à votre fichier Figma

## Structure

```
ds-table-skill/
  SKILL.md                              ← Point d'entrée du skill
  README.md                             ← Ce fichier
  knowledge-base/
    cspec/
      builders/
        table.yaml                      ← Spec principale (architecture, recipes, pitfalls)
      components/
        th.yaml                         ← Table Header Cell (variants, layout)
        td.yaml                         ← Table Data Cell (native slot pattern)
      pages/
        template-table.yaml             ← Layout template SaaS (sidebar+table)
        user-list.yaml                  ← Exemple complet de page avec table
      property-map.yaml                 ← Mapping Figma ↔ CSS ↔ React
```

## Utilisation

Demander à Copilot :

> "Crée une table Orders dans Figma avec 8 lignes, colonnes checkbox, order, customer, date, status, amount, payment et actions."

Le skill va :
1. Lire les specs YAML pour comprendre l'architecture
2. Générer 2 appels MCP (structure + contenu complexe)
3. Construire la table avec zebra striping et sort indicators

## Provenance

Extrait du projet [DS-SKILLS](https://github.com/arnaudmorvan/DS-SKILLS) — skill `ds-init` (Knowledge Base → CSpec → builders/table).

## Licence

MIT
