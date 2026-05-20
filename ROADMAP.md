# break-check ROADMAP

**Current version:** v1.0.0 (env-agnostic fork from safe-terminal v1.1.2,
2026-05-20)
**Status:** initial public release. The 19 Tier-1 / Tier-2 clauses are
stable, inherited from a year of incident-driven refinement in the
sibling `safe-terminal` repo.

## Near-term

- **Calibration-data feedback.** After the skill has been installed
  through `npx skills add weijia-89/break-check` for a few weeks, the
  GitHub issues queue is the channel for operators to surface clause
  shapes that fire on legitimate workflows in their environment. A
  promotion-log entry for any pattern that bites two operators
  independently is a candidate Tier-1 clause for the next MINOR release.
- **Cross-agent installation verification.** The skill is built around
  Cascade's `run_command` shape. The same primitive exists under different
  parameter names in Claude Code, Codex, Cursor, Gemini CLI, and Aider.
  v1.0.x patches will catalog the per-agent parameter renamings (`Cwd`
  vs `cwd`, `Blocking` vs `is_blocking`, the
  `WaitMsBeforeAsync`-equivalent across agents) so the Tier-1 clauses
  read transparently to any operator. Until those patches land, the
  canonical body uses Cascade's naming with parenthetical notes.
- **Portable reference implementation.** The runtime wrapper
  (`safe-cmd.sh`) lives in the sibling `safe-terminal` repo with paths
  baked in for one operator's environment. A v1.1.0 of `break-check`
  may ship its own `scripts/break-check.sh` that is path-portable
  (config via env vars, no hardcoded `~/Projects` references). Gated on
  enough installation traction to justify the maintenance burden of two
  wrapper implementations.

## Mid-term

- **Network-egress allowlist.** No layer gates outbound HTTP / DNS / TCP
  today, which is named explicitly as residual risk class #1 in the
  canonical body. Penligent's `ALLOWED_HOSTS = {api.github.com,
  registry.npmjs.org, pypi.org}` pattern is the reference design; Docker
  Sandboxes' network-deny-by-default with proxied HTTPS egress is the
  stricter alternative. Either is a separate scope, gated on a
  network-exfiltration incident materializing as motivation.
- **Tier-1 / Tier-2 metrics dashboard.** A simple aggregator that
  consumes the wrapper's history log and emits a weekly breakdown of
  which clauses fire most often. Useful for operators who want to know
  which Tier-1 clauses they hit by reflex vs which Tier-2 audits saved
  them. Gated on at least one operator other than the author asking for
  it.

## Out of scope

- **Sandbox containment.** Penligent's Option-5 architectural fix
  (containerize the agent's shell) requires IDE-side changes the skill
  cannot reach from inside any of the agents skills.sh supports.
  Surfaced in the canonical body's residual-risk class #3.
- **Agent-writes-then-human-commits-to-CI chain.** A Layer 5-style
  sentinel catches agent-mediated execution chains. A modified
  `.github/workflows/*.yml` that the human later commits and pushes
  outside the agent's session is gated only by the operator's PR
  review, not by anything this skill enforces. Surfaced as residual-
  risk class #2.
- **Auto-curating the allow/deny lists.** The seeder scripts in the
  sibling `safe-terminal` repo propose patterns and report false-
  positive counts; the operator curates by hand. Auto-curation would
  re-introduce the "permission prompts are not a durable security
  model" failure mode that the framing principle exists to prevent.

## Open questions

- Should the chain-proposal flow learn from approval frequency? A
  pattern an operator approves more than five times in a week looks
  like an allowlist candidate, but the promotion should not be
  automatic. "Frequently approved" and "should be auto-approved" are
  different properties, and conflating them is how a permission prompt
  becomes a rubber stamp. The current answer is no; operators curate.
  An empirical answer would require a longitudinal study across multiple
  operators that this skill is not built to collect.
- The Layer 0 workspace-trust advisory is text-only today. The chat
  transcript is the gate the skill relies on; a runtime alternative
  that intercepts `write_to_file` lives in IDE architecture territory
  the skill cannot reach. Whether the advisory holds up across diverse
  operator workflows is what installation traction over the next few
  months should reveal.
