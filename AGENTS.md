# AGENTS.md

## Project Overview

Single-page tool that lists the menu for a scout summer camp ("vacanze di branco") and computes ingredient quantities. Dishes are grouped by day and meal. Each ingredient dose is a pre-populated, editable input; a shopping list at the bottom of the page sums identical ingredients across all dishes and recomputes live as doses change.

**Tech stack:** single static `index.html`, vanilla JavaScript, no dependencies, no build step. Hosted on GitHub Pages.

## Group and Portion Model

- Group: **24 children (8-12 years) + 8 adults = 32 people**.
- Doses are computed on **30 equivalent portions**: a child counts as **0.9** of an adult portion (`24 × 0.9 + 8 = 29.6 ≈ 30`). Earlier the child ratio was 0.75 (26 equivalent portions); it was raised to 0.9 on user request. Changing the ratio scales all mass/volume/count doses proportionally.
- Pre-set doses are kept only when coherent with the group and scale with the equivalent-portion base for consistency.

## Data Model (in `index.html`)

The whole menu lives in the `MENU` array inside the inline `<script>`:

```js
{day, note?, meals:[ {meal, dishes:[ {name, note?, ings:[ {n, k, v, u, note?, ready?} ]} ]} ]}
```

- `n` — display name of the ingredient.
- `k` — merge key. Ingredients sharing a `k` are summed together in the shopping list (e.g. all `wurstel`, `formaggio`, `carote`, `pasta`, `patate`, `zucchine`, `olio`, `sale`).
- `v` — numeric dose (the input's pre-populated value).
- `u` — unit: `gr`, `kg`, `ml`, `lt`, `pz`. Normalized to a base (mass=gr, vol=ml, count=pz) via the `UNIT` map for summing; displayed back in kg/lt when large.
- `ready:true` — "Già pronto, non servono dosi": rendered as text, no input, excluded from the shopping list.
- `note` (on ingredient or dish) — assumption or clarification shown under the dish.

Shopping-list label per key comes from the first ingredient seen with that `k`, stripped of any parenthetical.

## Editorial Decisions Baked Into the Data

- `Insalata di riso`: doses are sized for the full group; a single/family portion (riso 250gr) would be far too little.
- `Frittata di zucchine`: eggs are included (~21) with a note, since a frittata needs them.
- Olio EVO and Sale are added to every savory dish that needs them (values are indicative, q.b.).
- Units for `Carote` are all `gr` so the shopping list sums cleanly.
- There is **no Venerdì** (days jump Giovedì → Sabato); flagged with a `note` on the Sabato day.
- `Panini`, `Verdure grigliate`, and `Frutta` have no explicit ingredient breakdown; they carry estimated quantities with a "non specificato" note.

## Editing Doses

- Change a `v` in the `MENU` array to change a pre-populated dose.
- To make two ingredients sum together in the shopping list, give them the same `k`.
- To rescale the whole menu for a different child ratio, recompute the equivalent-portion base (`24 × ratio + 8`), multiply every mass/volume/count `v` by `new_base / old_base`, round sensibly, and update the header note (`<header>` paragraph) with the new base and ratio.
- After editing, validate the array parses:

```bash
node -e "const fs=require('fs');const h=fs.readFileSync('index.html','utf8');const m=h.match(/const MENU = (\[[\s\S]*?\]);/);eval('var MENU='+m[1]);let c=0;MENU.forEach(d=>d.meals.forEach(me=>me.dishes.forEach(di=>di.ings.forEach(i=>c++))));console.log('parse ok, ings',c)"
```

## Print

`@media print` hides the header, the menu, and the `.controls`, printing **only the shopping list**. Keep this behavior when editing CSS.

## Deploy

Served by GitHub Pages from the `main` branch, root path. `index.html` at the repo root is the published page: https://lussoluca.github.io/menu_vdb_2026/

```bash
git add index.html
git commit -m "<conventional commit>" --trailer "Assisted-by: <agent>/<model-id>"
git push origin main   # Pages rebuilds automatically (~1 min)
```

A working copy of the same file is kept at `~/Downloads/menu_vacanze_branco.html`; keep it in sync after edits.

## Git Workflow

### Commits

Conventional Commits: `<type>(<scope>): <description>`, lowercase imperative, no trailing period. Every commit carries an `Assisted-by: <agent>/<model-id>` trailer.

### Branching

Prefix + kebab-case (`feat/`, `fix/`, `chore/`, `docs/`). This is a personal single-file project with no issue tracker; commits go straight to `main` and Pages deploys from `main`.

## Command Safety

### Safe (run autonomously)

- `git status`, `git log`, `git diff`, `ls`
- `node -e "..."` parse check above
- `open index.html`

### Dangerous (ask user first)

- `git push origin main` (publishes to the live Pages site)
- `gh api ... /pages` (changes Pages configuration)

### Destructive (never run)

- `git push --force`, `git reset --hard`, `rm -rf`
- `gh repo delete`

## Important Rules

- No dependencies, no build step: keep it a single self-contained `index.html` with vanilla JS.
- Keep the current group and portion model in sync between the `MENU` doses and the `<header>` note.
- Ingredients that should sum in the shopping list must share the same `k`.
- Print output must stay limited to the shopping list.
- After editing doses, run the parse check and keep the `~/Downloads` copy in sync.
- Follow conventional commits with the `Assisted-by` trailer.
