# Upstream

Most skills here are a port of [pstack](https://github.com/cursor/plugins/tree/main/pstack/skills) by [Lauren Tan](https://github.com/poteto) (MIT, see `THIRD_PARTY_NOTICES.md`). This file records what the port changed, so an upstream update can be merged without losing local edits.

## Base

| Field | Value |
| --- | --- |
| Upstream repo | `https://github.com/cursor/plugins`, path `pstack/` |
| Last synced commit | `2f87c85` (2026-11-16) |
| Git branch holding upstream as published | `upstream` |
| Git branch holding this folder | `main` |

The `upstream` branch stores `pstack/skills/*` and `pstack/LICENSE` at the repo root, so its paths match `main`. Never edit `upstream` by hand. It only receives new pstack snapshots.

## Rules every local edit follows

Rewrite any upstream change that breaks one of these. Don't copy it in as is.

1. **Delegation is never the default.** Skills run inline. A skill spawns subagents only when the user asks in the current session, per the Delegation section of `HARNESSES.md`. Subagent steps sit under a "**Dispatch** (only when the user asks)" note. `swarm` is the one skill that always dispatches, and it runs only when invoked.
2. **No harness-specific config.** No Cursor model slugs, `pstack-models.mdc`, `generalPurpose`, `cloud_base_branch` or `~/.cursor` paths. Skills name a model role (judgment, fast, another family) and a tool term, and `HARNESSES.md` maps both per harness.
3. **Links between skills are relative paths**, such as `[how](../how/SKILL.md)` or `[laziness-protocol](../principles/laziness-protocol.md)`, not bold names.

## Structural changes

| Upstream | Here | Why |
| --- | --- | --- |
| 24 `principle-*/SKILL.md` skills | A `principles/` reference folder, one `<name>.md` per principle, with no index and no skill entry | Skills link the file they need at the step that needs it, so nothing loads on its own. Upstream edits to `principle-<name>/SKILL.md` go into `principles/<name>.md` by hand. |
| `principle-build-the-lever`, `principle-exhaust-the-design-space`, `principle-experience-first`, `principle-explain-the-number`, `principle-guard-the-context-window` | Dropped | Each repeats another skill or the default behavior, or no step links to it. An upstream edit to one shows up as a modify/delete conflict. `git rm` it again. |
| `automate-me` | Dropped | Not wanted. |
| `make-bot-ui` | Dropped | Not wanted. |
| `poteto-help` | Dropped | Help for pstack's Cursor setup. |
| `poteto-mode` | Dropped | [Lauren](https://github.com/poteto)'s personal working style, built on delegating to subagents. `~/.claude/CLAUDE.md` covers the style. `let-him-cook` replaces its playbooks and triggers. |
| `setup-pstack` | Dropped | Writes Cursor's per-role model rule. `HARNESSES.md` replaces it. |
| `statechart` | Dropped | Not wanted. An upstream edit to it shows up as a modify/delete conflict. `git rm` it again. |
| `pstack/agents/comment-sicko.md` | `no-comments/references/comment-sicko.md` | Read by `no-comments` instead of loaded as a Cursor agent. |
| `pstack/agents/poteto-agent.md` | Not ported | Cursor agent for `poteto-mode`. |
| (none) | `<skill>/agents/openai.yaml` | Codex skill metadata, added locally. |
| (none) | `HARNESSES.md`, `UPSTREAM.md` | Harness map and this file. |
| `LICENSE` ([Lauren Tan](https://github.com/poteto)) | `LICENSE` ([Kyros Abas](https://github.com/KyrosHendrix)), with [Lauren](https://github.com/poteto)'s notice in `THIRD_PARTY_NOTICES.md` | The root license covers this repo's own work. If a merge changes `LICENSE`, keep `main`'s version and copy any change to Lauren's notice into `THIRD_PARTY_NOTICES.md`. |
| (none) | `let-him-cook/` | Local skill, not tracked from pstack. Routes a task to the skills it needs, inline. Written from the ideas in `poteto-mode`'s playbooks and Non-negotiables, with every delegation step removed. |
| (none) | `sdd/` | Local skill, not tracked from pstack. Writes one spec file with testable requirements to `docs/specs/`, then stops. |
| `disable-model-invocation: true` on every skill | Removed from `benchmark-checklist`, `blast-radius`, `how`, `interrogate`, `tdd`, `teach`, `technical-writing`, `typescript-best-practices`, `unslop`, and `why` | These load on their own when a task matches. Keep the flag off when merging upstream edits to them. |

Folders that don't come from pstack: `pr` (combined from [Matt Pocock](https://github.com/mattpocock)'s `pr` and [Theo](https://github.com/t3dotgg)'s `file-pr`), `let-him-cook`, and `sdd`. This file tracks pstack only.

## Local tweaks

One line per skill changed beyond the rules above, newest first.

- `recall`: searches each source of the shared record one at a time instead of spawning one investigator per source.
- `teach`: runs `how` and `why` one after the other instead of in parallel.
- `arena`: writes candidates one at a time and skips the cross-judge when inline.

## Checking for an upstream update

Run these from Git Bash. `$W` is a scratch folder outside the skills folder.

```bash
W=/tmp/pstack-sync && rm -rf $W && mkdir -p $W
git clone -q --depth 1 --filter=blob:none --sparse https://github.com/cursor/plugins $W/plugins
git -C $W/plugins sparse-checkout set pstack
git -C $W/plugins log -1 --format='%h %cd'
```

If the commit is newer than "Last synced commit", put the snapshot on `upstream` from a separate worktree. This keeps the live skills folder untouched until the merge.

```bash
cd ~/.claude/skills   # or ~/.agents/skills
git worktree add $W/up upstream
git -C $W/up rm -rq . && cp -r $W/plugins/pstack/skills/. $W/plugins/pstack/LICENSE $W/up/
git -C $W/up add -A && git -C $W/up commit -m "chore(upstream): import pstack <sha>"
git worktree remove $W/up
git diff upstream~1 upstream --stat   # what upstream changed since the last sync
git merge upstream
```

When resolving conflicts:

- Git tracks 18 of the 19 principles as renames of `principle-<name>/SKILL.md` to `principles/<name>.md`, so it usually carries upstream edits across on its own. Check the result anyway. The other one (`encode-lessons-in-structure`) changed too much for git to pair it, so an upstream edit to it shows up as a modify/delete conflict. Port the edit into `principles/<name>.md` by hand, then `git rm` the upstream path.
- A modify/delete conflict on a dropped skill means upstream edited it. Read the change, then `git rm` it again unless you want it back.
- A new upstream skill arrives as a new folder. Apply the rules above before keeping it, and add it to the tables here.

Finish by updating "Last synced commit" in this file, committing, and pushing both branches with `git push origin main upstream`. Then update the other clone (see below).

## Two clones

`~/.claude/skills` (Claude Code) and `~/.agents/skills` (other harnesses) are separate clones of the same GitHub repo. Work in one, push both branches, then update the other before you change anything there:

```bash
cd ~/.claude/skills   # the clone you didn't work in
git pull --ff-only
git fetch origin upstream:upstream
```
