---
name: prime-review
description: Use when the user asks for a read-only Prime Review audit of a path (file, folder, or project) for a supported stack (php, js, react, angular, vue, laravel, nestjs, python, dotnet), with findings recorded as Pending tasks in an auto-named TODO markdown file.
argument-hint: "<stack> [path]"
disable-model-invocation: true
---

# Prime Review

Audit a path with one stack pack and record verified, actionable findings in a TODO file. Analysis only.

Read `../../shared/audit-procedure.md` (relative to this skill's folder) and follow it with the profile for the chosen stack.

## Arguments

`$ARGUMENTS` is: `<stack> [path]`

1. **stack** (required): one of `php`, `js`, `react`, `angular`, `vue`, `laravel`, `nestjs`, `python`, `dotnet`.
2. **path** (optional): a file, a directory, or omit / `.` for the repository root.

Do **not** take a TODO path from the user. The audit procedure sets `TODO_FILE` automatically.

## Load stack pack

After parsing `stack`, load these files from `stacks/<stack>/` (relative to this skill's folder):

- `profile.md` — Source, Exclusions, Project files, Tools, Map, Extra scope, Cross-slice, Lenses, Example, Extra prep
- `review-criteria.md` — Criteria
- `finding-contract.md` — Finding contract

If `stack` is missing or not in the list above, ask once with the valid values; do not start the audit.

## Profile shell

- **Arguments**: `$ARGUMENTS` (see above; procedure derives `FOLDER` / `PATH` and `TODO_FILE`)
- **Criteria**: `stacks/<stack>/review-criteria.md`
- **Finding contract**: `stacks/<stack>/finding-contract.md`
- All other profile fields: from `stacks/<stack>/profile.md`
