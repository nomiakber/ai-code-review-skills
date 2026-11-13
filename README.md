# AI Code Review Skills (Prime Review Skills)

Read-only **AI code-review** for [Cursor](https://cursor.com) and [Claude Code](https://claude.com/claude-code).

One command, nine stacks. Pass a **stack** and an optional **path** (one file, a folder, or the whole repo). Verified findings land in an **auto-named TODO** file — the skill never edits your source.

**Repo:** [github.com/nomiakber/ai-code-review-skills](https://github.com/nomiakber/ai-code-review-skills)

---

## Command

```text
/prime-review <stack> [path]
```

| Part | Required | Meaning |
|------|----------|---------|
| `stack` | yes | `php` · `js` · `react` · `angular` · `vue` · `laravel` · `nestjs` · `python` · `dotnet` |
| `path` | no | One **file**, one **directory**, or omit / `.` for the **repository root** |

| Client | Invoke |
|--------|--------|
| Cursor | `/prime-review <stack> [path]` |
| Claude Code | `/prime-skills:prime-review <stack> [path]` |

**TODO file (always automatic, never an argument)**

```text
TODO-prime-review-<stack>-YYYYMMDD-HHMM.md
```

At the repo root. If that name exists, the skill uses seconds (`YYYYMMDD-HHMMSS`).

**Analysis only:** the only file the review may create or change is that TODO file.

---

## Stacks

| Stack | Focus | Extra when detected |
|-------|--------|---------------------|
| `php` | WordPress PHP plugins/themes | Uses `wp-core-reference` if installed |
| `js` | WordPress JS/TS (`@wordpress/*`) | — |
| `react` | React UI | Next.js App Router |
| `angular` | Angular | Standalone and NgModules |
| `vue` | Vue Options + Composition | Nuxt |
| `laravel` | Laravel PHP / Blade | — |
| `nestjs` | NestJS | Microservices and GraphQL |
| `python` | General Python | FastAPI |
| `dotnet` | C# /.NET | ASP.NET Core |

Also available: **WP Core Reference** — `/wp-core-reference` (separate plugin; stubs lookup before writing WordPress code).

---

## Examples

### Whole project (path omitted)

Cursor:

```text
/prime-review php
/prime-review js
/prime-review react
/prime-review angular
/prime-review vue
/prime-review laravel
/prime-review nestjs
/prime-review python
/prime-review dotnet
```

Claude Code:

```text
/prime-skills:prime-review php
/prime-skills:prime-review js
/prime-skills:prime-review react
/prime-skills:prime-review angular
/prime-skills:prime-review vue
/prime-skills:prime-review laravel
/prime-skills:prime-review nestjs
/prime-skills:prime-review python
/prime-skills:prime-review dotnet
```

### One folder

```text
/prime-review php includes
/prime-review php app/public
/prime-review js src
/prime-review react src/components
/prime-review react app
/prime-review angular src/app/features/orders
/prime-review vue components
/prime-review vue server
/prime-review laravel app/Http/Controllers
/prime-review laravel app/Models
/prime-review nestjs src/orders
/prime-review nestjs src/modules
/prime-review python app/api
/prime-review python src
/prime-review dotnet src/MyApp.Api
/prime-review dotnet src/MyApp.Domain
```

### One file

```text
/prime-review php includes/class-plugin.php
/prime-review php src/Rest/OrdersController.php
/prime-review js src/blocks/edit.js
/prime-review react src/components/UserTable.tsx
/prime-review react app/api/orders/route.ts
/prime-review angular src/app/orders/orders.service.ts
/prime-review vue components/FilterBar.vue
/prime-review laravel app/Http/Controllers/OrderController.php
/prime-review nestjs src/orders/orders.controller.ts
/prime-review python app/main.py
/prime-review python app/api/users.py
/prime-review dotnet src/MyApp.Api/OrdersController.cs
```

### Repo root explicitly

```text
/prime-review php .
/prime-review js .
/prime-review react .
/prime-review angular .
/prime-review vue .
/prime-review laravel .
/prime-review nestjs .
/prime-review python .
/prime-review dotnet .
```

### What you get

For `/prime-review react src/components` started at 19:30 on 13 Nov 2026:

1. Reviews React (and Next rules if Next is detected) under `src/components`
2. Writes Pending tasks to `TODO-prime-review-react-20261113-1930.md` at the repo root
3. Does not modify application source, tests, or lockfiles

---

## How it works

1. Load [ponytail](https://github.com/DietrichGebert/ponytail) (smallest-fix ladder)
2. Load stack pack: `plugins/prime-skills/skills/prime-review/stacks/<stack>/`
3. Map the path, run project lint/type-check when available
4. Verify findings against that stack’s criteria + finding contract
5. Write the auto-named TODO file

Shared process: `plugins/prime-skills/shared/audit-procedure.md`  
Per stack: `profile.md`, `review-criteria.md`, `finding-contract.md`

---

## Prerequisite: ponytail

Claude Code:

```text
/plugin marketplace add DietrichGebert/ponytail
/plugin install ponytail@ponytail
```

Cursor:

```bash
git clone https://github.com/DietrichGebert/ponytail
node ponytail/scripts/cursor-hooks.js install
```

---

## Install in Claude Code

```bash
claude plugin marketplace add nomiakber/ai-code-review-skills
claude plugin install prime-skills@prime-skills
```

One project only:

```bash
claude plugin install prime-skills@prime-skills --scope project
```

Optional WordPress stubs helper:

```bash
claude plugin install wp-core-reference@prime-skills
```

---

## Install in Cursor

**Team:** Dashboard → Plugins & MCPs → Add Marketplace → Import from Repo →  
`https://github.com/nomiakber/ai-code-review-skills`  
Then **Customize** → **prime-skills** → Install.

**Local link:**

```bash
git clone https://github.com/nomiakber/ai-code-review-skills.git
cd ai-code-review-skills
mkdir -p ~/.cursor/plugins/local
ln -s "$PWD/plugins/prime-skills" ~/.cursor/plugins/local/prime-skills
ln -s "$PWD/plugins/wp-core-reference" ~/.cursor/plugins/local/wp-core-reference
```

Reload Cursor. Link whole plugin folders (skills need `shared/audit-procedure.md`).

---

## Optional: wp-core-reference

Stack `php` can use it for core functions, hooks, and deprecations:

```bash
composer require --dev php-stubs/wordpress-stubs php-stubs/wordpress-globals php-stubs/wp-cli-stubs wp-hooks/wordpress-core
```

Claude Code rule (plugins can’t ship rules there) — add to `CLAUDE.md` or `.claude/rules/wp-core-reference.md`:

```markdown
# WordPress core reference

For any WordPress plugin or theme code, always use the wp-core-reference skill first, before writing code.
```

---

## Develop

```bash
claude plugin validate .
claude plugin marketplace add ./
```

---

## License

MIT — see `LICENSE`. Issues welcome on [GitHub](https://github.com/nomiakber/ai-code-review-skills).
