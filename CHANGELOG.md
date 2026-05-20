# Changelog, break-check

Format: Keep a Changelog with SemVer.

## SemVer rules for this skill

- MAJOR: load-bearing Tier-1 / Tier-2 enumeration changes (a clause
  removed; numbering shifted; meaning inverted). These changes break
  agent muscle-memory along with any external reference to "clause #N."
- MINOR: new Tier-1 / Tier-2 clauses, new wrapper layers documented in
  the canonical body, new known-incident-pattern rows that codify a
  previously-undocumented hazard.
- PATCH: wording, em-dash normalization, citation updates,
  cross-reference touchups, doc-only changes.

## [1.0.0], 2026-05-20, Env-agnostic fork from safe-terminal

Initial public release. Forked from
[weijia-89/safe-terminal v1.1.2](https://github.com/weijia-89/safe-terminal)
to produce an env-agnostic skill body that installs cleanly across the
agents skills.sh supports (Windsurf, Claude Code, Cursor, Codex, Gemini
CLI, and others). `safe-terminal` remains the operator-specific bundle
that includes the runtime wrapper and the local sync helpers;
`break-check` ships the skill body and metadata only.

### What forked

The Tier-1 / Tier-2 enumeration is identical to `safe-terminal v1.1.2`.
Twelve Tier-1 reject clauses (syntax-shape and shape-of-failure) plus
seven Tier-2 audits, same numbering, same wording. The discipline does
not change across the fork; what changes is the framing and the
scaffolding around it.

### What's different from safe-terminal

- **Name and metaphor.** `safe-terminal` framed the skill as iron-law
  safety. `break-check` reframes it as the pre-paddle-out check a
  surfer runs against the break shape before committing. Same audit
  passes; different metaphor at the entry point.
- **De-Wei-fication.** Operator-specific paths replaced with generic
  references. `/Users/wjia/Projects/scripts/safe-cmd.sh` → "the
  reference implementation at `weijia-89/safe-terminal`."
  `~/Projects/.safe-cmd-allowlist` → "the allowlist file."
  Cascade-specific cross-reference notation (memory IDs, internal
  pointer strings) stripped.
- **Promotion log abstracted.** The original repo's dated promotion log
  (incident dates, near-miss counts from one operator's chat history)
  becomes a generic "Known incident patterns" section listing the
  pattern shapes that promoted from Tier-2 to Tier-1, without the
  operator-specific dates and incident-count attribution.
- **No wrapper bundled.** The runtime wrapper (`safe-cmd.sh`), its test
  suite (`scripts/tests/test_safe_cmd.sh`), the local sync helpers
  (`scripts/bundle.sh`, `scripts/verify_safe_terminal_sync.sh`), and
  the operator-bin seeders (`seed_safe_cmd_allowlist.py`,
  `seed_safe_cmd_denylist.py`) all live in `safe-terminal`. Operators
  wanting the runtime gate point at `safe-terminal` directly; `npx
  skills add weijia-89/break-check` installs only the skill body.

### What's the same

- The 19 Tier-1 / Tier-2 clauses, verbatim
- The Allowed / Banned section
- The Gitignored-file access workaround
- The Workspace-trust file allergens Layer 0 advisory
- The Recovery if the terminal freezes procedure
- The Self-check after a freeze or audit-fail rule
- The framing principle (Anthropic auto-mode disclosure + Penligent
  *Sandboxes for Coding Agents*)
- The three named residual risk classes (network egress, agent-writes-
  CI deferred chain, closed-loop on the wrapper itself)

### Files in this initial release

- `SKILL.md`, the canonical body (~180 lines)
- `README.md`, public-facing intro with the surf-check framing,
  Mermaid architecture diagram, exit-code table, install instructions
- `CHANGELOG.md`, this file
- `ROADMAP.md`, near-term / mid-term / out-of-scope items
- `metadata.json`, skills.sh leaderboard metadata
- `LICENSE`, MIT, copyright Wei Jia 2026

### Why fork rather than rename

The two repos serve different audiences. `safe-terminal` carries the
operator-specific wrapper and the dated promotion-log evidence that
makes a strong case to one specific reader (Wei's future self,
maintainers of Wei's bin directory). `break-check` carries the
generalizable discipline for any operator running a coding agent.
Renaming `safe-terminal` would have collapsed both audiences into one
repo where every reader has to skip the parts not for them. Forking
keeps each repo readable end-to-end for its actual audience.

### Em-dash invariant

Zero em-dashes across all files in this release. The invariant is
maintained by the bundling and verification helpers in the sibling
`safe-terminal` repo; `break-check` inherits the invariant through the
fork but does not ship verification tooling.

### Citation verification

The two references in `metadata.json` and `SKILL.md`'s framing principle
are live as of 2026-05-20:

- Anthropic Claude Code auto-mode disclosure:
  `https://www.anthropic.com/engineering/claude-code-auto-mode` (the
  93%-permission-approval data source)
- Penligent *Sandboxes for Coding Agents*:
  `https://www.penligent.ai/hackinglabs/sandboxes-for-coding-agents/`
  (the framing-principle anchor)
- skills.sh ecosystem landing: `https://skills.sh`
