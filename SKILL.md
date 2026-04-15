---
name: ds-table
description: "Génère des data tables Figma via MCP. USE WHEN: l'utilisateur veut créer un tableau/table dans Figma, un data table avec colonnes typées (texte, badges, checkboxes, actions), une page type user-list/orders-list, ou tout écran CRUD avec tableau. Deux architectures: row-first (défaut) ou column-first."
argument-hint: "Description de la table souhaitée (ex: 'table Orders 8 lignes avec checkbox, customer, status, actions')"
---

# Skill — Figma Table Builder (Native Slots)

> Génère des tables de données complètes dans Figma via MCP, en utilisant des composants natifs (th/td) et des **slots natifs Figma** (zero `detachInstance()`). Supporte deux architectures : **row-first** (défaut) et **column-first**.

## Quand utiliser ce skill

- L'utilisateur demande de **créer une table/tableau** dans Figma
- L'utilisateur veut un **data table** avec colonnes typées (texte, badges, checkboxes, actions…)
- L'utilisateur veut un **template de page table** (sidebar + topbar + table)
- L'utilisateur demande une **user list**, **order list**, ou tout écran CRUD avec tableau

## Routing — Choix de l'architecture

| Demande utilisateur | Builder | Fichier |
|--------------------|---------|---------|
| "table", "tableau", "data table" (sans précision) | **Row-first** (défaut) | `table-row.yaml` |
| "table en lignes", "row-based", "row layout" | **Row-first** | `table-row.yaml` |
| "table en colonnes", "column-first", "column layout" | **Column-first** | `table-column.yaml` |

**Par défaut → row-first** car :
- Zebra striping trivial (fills sur le Row frame)
- Sélection de ligne triviale (même principe)
- Modèle mental naturel (comme HTML `<tr>`)
- Élimine les pitfalls P9, P10, P11 du column-first

## Prérequis

Ce skill **consomme des composants** d'un Design System Figma existant. Les Component Sets suivants doivent exister dans le fichier Figma :

| Composant | Rôle | Node ID (exemple) |
|-----------|------|--------------------|
| `th` | Header cell | `150:1915` |
| `td` | Data cell (Text + Slot natif) | `153:1859` |
| `badge` | Status indicators | `56:34` |
| `button` | Action buttons (Secondary) | `44:302` |
| `checkbox` | Selection column | `53:1244` |
| `avatar-profile-photo` | Customer name + photo | `147:1747` |

> **Les Node IDs sont à adapter** au fichier Figma cible. Utiliser `get_metadata` pour retrouver les bons IDs.

## Architecture

### Row-first (défaut) — `table-row.yaml`

```
Table (VERTICAL, gap=0, cornerRadius=8, DROP_SHADOW)
  ├── Header Row (HORIZONTAL, gap=0)
  │   ├── th "Checkbox"   (FIXED 60px)
  │   ├── th "Name"       (FIXED 230px)
  │   ├── th "Status"     (FIXED 160px)
  │   └── th "Actions"    (FIXED 120px)
  ├── Data Row 1 (HORIZONTAL, gap=0, fills=transparent)
  │   ├── td Slot          (checkbox)
  │   ├── td Slot          (avatar+name)
  │   ├── td Slot          (badge)
  │   └── td Slot          (icons)
  ├── Data Row 2 (HORIZONTAL, gap=0, fills=Subtle) ← zebra
  │   └── ...
  └── Data Row N
```

**Principe** : Chaque ligne est un frame HORIZONTAL contenant N td. Le zebra = fills sur le Row frame. Toutes les td utilisent `Fill=Default`.

### Column-first (legacy) — `table-column.yaml`

```
Table (HORIZONTAL, gap=0, cornerRadius=8, DROP_SHADOW)
  ├── Column "Checkbox"    (VERTICAL, width=60 FIXED)
  │   ├── th instance      (FILL width)
  │   ├── td INSTANCE      (Content=Slot → checkbox)
  │   └── ...
  ├── Column "Order"       (VERTICAL, width=170 FIXED)
  │   ├── th instance      (override "Column" text)
  │   ├── td INSTANCE      (Content=Text)
  │   └── ...
  └── Column "Actions"     (VERTICAL, width=230 FIXED)
      ├── th instance
      ├── td INSTANCE      (Content=Slot → icons)
      └── ...
```

**Principe** : Chaque colonne est un frame VERTICAL contenant 1 th + N td. Le zebra nécessite alternance Default/Subtle par cellule au build-time.

## Types de colonnes

| Type | Largeur | Cell | Contenu |
|------|---------|------|---------|
| `checkbox` | 60px | td Slot | Checkbox instance (labels cachés) |
| `data-text` | 160-170px | td Text | Texte simple (override "Value") |
| `data-avatar-text` | 230px | td Slot | Avatar 24×24 + texte nom |
| `data-badge` | 160px | td Slot | Badge instance (texte overridé avant injection) |
| `actions` | 230px | td Slot | 3 Secondary buttons (text color fix obligatoire) |
| `icon-actions` | 120px | td Slot | 3 frames 28×28 avec icônes vectoriels |

## Stratégie de build MCP

Diviser en **2 appels MCP maximum** :

### Call 1 — Structure
- Créer le frame table (HORIZONTAL, AUTO sizing, DROP_SHADOW)
- Créer toutes les colonnes (VERTICAL, FIXED width)
- Injecter les th (headers) avec texte overridé
- Remplir les colonnes **texte simples** (td Content=Text)

### Call 2 — Contenu complexe
- Colonnes **checkbox** : td Slot → checkbox instance
- Colonnes **avatar+nom** : td Slot → avatar + createText
- Colonnes **badge** : td Slot → badge instance
- Colonnes **actions** : td Slot → buttons ou icon frames

## Knowledge Base

```
knowledge-base/
  cspec/
    builders/
      table-row.yaml      ← Row-first builder (défaut, recommandé)
      table-column.yaml   ← Column-first builder (legacy)
    components/
      th.yaml             ← Table Header Cell (variants, layout)
      td.yaml             ← Table Data Cell (variants, native slot, recipes)
```

### Lecture des specs

1. **Choisir le builder** selon le routing (row-first par défaut)
2. Lire `builders/table-row.yaml` OU `builders/table-column.yaml` — contient l'architecture, les helpers, et les pitfalls
3. Lire `components/th.yaml` et `components/td.yaml` pour les détails de variantes et le pattern slot natif

## Règles critiques

### Native Slot Pattern (td Content=Slot)
```js
// 1. Créer l'enfant (badge, checkbox, etc.)
var bi = badgeComp.createInstance();
// 2. Configurer AVANT injection (inaccessible après !)
bi.findOne(n => n.type==="TEXT" && n.name==="content").characters = "Pending";
// 3. Trouver le slot natif
var slot = tdInst.findOne(n => n.type === "SLOT");
// 4. Supprimer le placeholder
if (slot.children[0]?.type === "TEXT") slot.children[0].remove();
// 5. Injecter
slot.appendChild(bi);
// tdInst.type === "INSTANCE" ✅ (connexion DS préservée)
```

### Noms internes réels (PAS les noms du YAML spec)
| Composant | Nom TEXT réel | PAS |
|-----------|---------------|-----|
| th | `"Column"` | ~~"title"~~ |
| td | `"Value"` | ~~"label"~~ |
| badge | `"content"` | — |
| button | `"Button"` | — |

### Zebra striping
- **NE JAMAIS** utiliser `swapComponent()` sur des td Content=Slot (détruit le contenu du slot)
- Choisir la variante Fill (Default/Subtle) **à la création** de chaque td
- `swapComponent()` est safe uniquement sur td Content=Text et th

### Hauteur uniforme
- **Toutes les data cells** : height=48px FIXED (`tdInst.resize(w, 48)`)
- Sinon décalage vertical entre colonnes (architecture column-first)

### DROP_SHADOW
- `blendMode: "NORMAL"` **OBLIGATOIRE** — sinon l'ombre est silencieusement ignorée

### Secondary button text fix
- Le texte Secondary est **BLANC par défaut** → override fills avant injection :
```js
btnTxt.fills = [{ type: "SOLID", color: { r: 0.067, g: 0.094, b: 0.153 } }];
```

## Helpers réutilisables

Copier au début de chaque appel MCP :

```js
// Variables
var allV = await figma.variables.getLocalVariablesAsync("COLOR");
var colls = await figma.variables.getLocalVariableCollectionsAsync();
function gv(n) { return allV.find(function(v) { return v.name === n; }); }
function rv(o) {
  if (!o) return { r: 0.9, g: 0.9, b: 0.9 };
  var c = colls.find(function(x) { return x.id === o.variableCollectionId; });
  if (!c) return { r: 0.9, g: 0.9, b: 0.9 };
  var m = c.modes[0].modeId, val = o.valuesByMode[m];
  if (val && val.r !== undefined) return val;
  if (val && val.type === "VARIABLE_ALIAS") {
    var t = allV.find(function(v) { return v.id === val.id; });
    if (t) return rv(t);
  }
  return { r: 0.9, g: 0.9, b: 0.9 };
}
function bf(node, v) {
  if (!v || !node.fills || !node.fills.length) return;
  var f = JSON.parse(JSON.stringify(node.fills));
  f[0] = figma.variables.setBoundVariableForPaint(f[0], "color", v);
  node.fills = f;
}

// Component Set helpers
function findV(cs, variantName) {
  return cs.children.find(function(c) { return c.name === variantName; });
}

// Slot cell factory (td reste INSTANCE, zero detach)
function mkSlotCell(colFrame, isSubtle, tdSlot, tdSubSlot) {
  var comp = isSubtle ? tdSubSlot : tdSlot;
  var inst = comp.createInstance();
  colFrame.appendChild(inst);
  inst.layoutSizingHorizontal = "FILL";
  inst.resize(inst.width, 48);
  inst.primaryAxisSizingMode = "FIXED";
  var slot = inst.findOne(function(n) { return n.type === "SLOT"; });
  if (slot.children[0] && slot.children[0].type === "TEXT") slot.children[0].remove();
  return { inst: inst, slot: slot };
}
```

## Pitfalls documentés

Voir `builders/table.yaml` section `pitfalls` pour les 11 pitfalls identifiés et corrigés :
- P1: Noms internes ≠ specs YAML
- P2: Children inaccessibles après injection slot
- P3: Secondary button texte invisible (blanc)
- P4: 3 Secondary buttons débordent (colonne ≥ 230px)
- P5: Placeholder "slot" visible
- P6: Colonnes effondrées (counterAxisSizingMode AUTO)
- P7: Checkbox labels débordent
- P8: Crash MCP rollback → séparer calls
- P9: Décalage vertical → height FIXED 48px
- P10: swapComponent() détruit slots
- P11: Zebra = planification build-time

## Exemples validés

| Nom | Features | Densité |
|-----|----------|---------|
| Orders Table v3 | Zebra + sort + icon-actions | MD |
| Orders Table SM | Compact, 4 colonnes | SM |
| Orders Table Empty State | Header + empty message + CTA | MD |
| User List page | Navbar + sidebar + filters + table + pagination | MD |
