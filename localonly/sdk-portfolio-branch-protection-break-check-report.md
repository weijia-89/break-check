# SDK report: portfolio-branch-protection-break-check

**Queue id:** `portfolio-branch-protection-break-check`  
**Repo:** `weijia-89/break-check`  
**Cwd:** `/Users/wjia/Projects/break-check.skill`  
**Branch:** `chore/branch-protection-sdk`  
**Agent:** worker (trainer v0.10.1)  
**Date:** 2026-05-25

## Manifest row

| Field | Value |
| ----- | ----- |
| slug | `break-check` |
| gh_repo | `weijia-89/break-check` |
| default_branch | `main` |
| visibility | PUBLIC |
| action | `apply` |
| apply_live | `true` |
| branch | `chore/branch-protection-sdk` |
| protected (audit) | `no` |

## Remote preflight

```bash
gh repo view weijia-89/break-check --json name,defaultBranchRef,isPrivate,viewerPermission
```

```json
{
  "defaultBranchRef": { "name": "main" },
  "isPrivate": false,
  "name": "break-check",
  "viewerPermission": "ADMIN"
}
```

Current protection (GET before apply):

```bash
gh api repos/weijia-89/break-check/branches/main/protection
```

**Result:** HTTP 404 — `Branch not protected` (expected; manifest audit agreed).

## Deliverables

| Path | Status |
| ---- | ------ |
| `docs/BRANCH_PROTECTION.md` | refreshed (policy table, solo-maintainer tradeoff, UI + `gh api` steps) |
| `scripts/apply_branch_protection.sh` | present, executable, `DRY_RUN=1` default, `GH_REPO=weijia-89/break-check` |

## Dry run (executed)

```bash
cd "/Users/wjia/Projects/break-check.skill"
DRY_RUN=1 GH_REPO=weijia-89/break-check ./scripts/apply_branch_protection.sh
```

**Exit:** 0 — payload printed (no PUT).

## Live apply

**Skipped:** `APPLY` not set to `1` in operator environment (`APPLY=0`). Manifest allows live apply when operator runs:

```bash
cd "/Users/wjia/Projects/break-check.skill"
APPLY=1 DRY_RUN=0 GH_REPO=weijia-89/break-check ./scripts/apply_branch_protection.sh
```

Post-apply verify:

```bash
gh api repos/weijia-89/break-check/branches/main/protection
```

**Solo maintainer note:** default payload sets `required_approving_review_count: 1`. See `docs/BRANCH_PROTECTION.md` § Solo maintainer tradeoff if merges block without a second approver.

## Verification (mechanical gate)

```bash
test -f "/Users/wjia/Projects/break-check.skill/docs/BRANCH_PROTECTION.md"
test -x "/Users/wjia/Projects/break-check.skill/scripts/apply_branch_protection.sh"
```

**Result:** PASS

## Git / PR

Feature branch `chore/branch-protection-sdk` carries product commits (`docs/`, `scripts/`). Playground may open PR when `OPEN_PR=1` via `_sdk_verify_and_pr.sh`.

## Open TODOs

- Operator: run live apply with `APPLY=1` when ready (after PR merge or on demand).
- Optional: add required status checks to script + docs when CI is wired.
