# Per-skill finding contract — Design

Date: 2026-11-13  
Status: Approved; implement in working tree. Human owns commit/push.  
Supersedes: runtime use of `plugins/prime-skills/shared/finding-contract.md` from the multi-framework and Wave 2 designs.

## Problem

WordPress PHP/JS embed a full Finding contract in `review-criteria.md`. Other skills point at a shared kernel. The human wants **maximum isolation**: every skill owns a full private contract copy, with no runtime shared finding-contract.

## Decisions (locked)

| Topic | Choice |
|-------|--------|
| Isolation | Full private finding contract per skill (no shared runtime kernel) |
| Layout | `finding-contract.md` inside each skill folder; criteria = review areas (+ Category list where already present) |
| Migration | Approach 1: copy-then-retarget for non-WP; verbatim extract for WP PHP/JS |
| Shared file | Delete `shared/finding-contract.md` after copies exist |
| Procedure | Retarget wording to the calling skill’s Finding contract file |
| Git | Human only |

## Architecture

```
plugins/prime-skills/
  shared/
    audit-procedure.md          # still shared; mentions skill-local finding-contract
  skills/prime-review-<name>/
    SKILL.md                    # Finding contract: finding-contract.md in this folder
    review-criteria.md          # categories + review areas (WP: contract sections removed)
    finding-contract.md         # full private What-not-to-flag + Finding contract
```

Skills: php, js, react, angular, laravel, vue, nestjs, python, dotnet.

## Content rules

- **Non-WP**: copy current shared kernel verbatim into each skill’s `finding-contract.md`; retarget profile + criteria pointers.
- **WP PHP/JS**: move existing inline `## What not to flag` and `## Finding contract` blocks verbatim into `finding-contract.md`; add Finding contract line to SKILL profiles; leave review areas in criteria.
- Do not unify WP Category bullets with non-WP Category list sections in this change.

## Delivery

1. Update `audit-procedure.md` finding-contract references.
2. Add local contracts + retarget for non-WP skills.
3. Extract WP PHP then JS contracts; retarget profiles.
4. Delete `shared/finding-contract.md`.
5. Update README (and brief follow-up notes on prior specs).
6. Verify; stop for human git.

## Non-goals

- Changing non-WP severity/confidence text beyond relocation.
- Unifying WP vs non-WP category list shapes.
- New stacks or agent commit/push.

## Success criteria

- Every review skill has a local `finding-contract.md` and points at it.
- No remaining references to `shared/finding-contract.md`.
- WP PHP/JS contract wording preserved (key phrases intact).
- `shared/audit-procedure.md` still shared for process only.
- No agent commit or push.
