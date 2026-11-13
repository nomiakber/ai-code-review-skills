# Unified Prime Review skill — Design

Date: 2026-11-13  
Status: Implemented in working tree. Human owns commit/push.

## Decisions

| Topic | Choice |
|-------|--------|
| Command | `/prime-review <stack> [path]` (Claude: `/prime-skills:prime-review …`) |
| Layout | One skill `prime-review` + `stacks/<stack>/{profile,review-criteria,finding-contract}.md` |
| Path | File, directory, or omit/`.` = repo root |
| TODO | Auto `TODO-prime-review-<stack>-YYYYMMDD-HHMM.md` at repo root; never a user arg |
| Old skills | Removed `prime-review-*` per-stack skill folders |

## Stacks

`php`, `js`, `react`, `angular`, `vue`, `laravel`, `nestjs`, `python`, `dotnet`
