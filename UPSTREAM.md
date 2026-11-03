# Upstream

Most skills here are a port of [pstack](https://github.com/cursor/plugins/tree/main/pstack/skills) by Lauren Tan (MIT, see `THIRD_PARTY_NOTICES.md`). This file records what the port changed, so an upstream update can be merged without losing local edits.

## Base

| Field | Value |
| --- | --- |
| Upstream repo | `https://github.com/cursor/plugins`, path `pstack/` |
| Last synced commit | `ae5650f` (2026-10-31) |
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
| 24 `principle-*/SKILL.md` skills | One `principles/` skill, one `<name>.md` per principle | Fewer entries in the skill list. Upstream edits to `principle-<name>/SKILL.md` go into `principles/<name>.md` by hand. |
| `automate-me` | Dropped | Not wanted. |
| `make-bot-ui` | Dropped | Not wanted. |
| `poteto-help` | Dropped | Help for pstack's Cursor setup. |
| `poteto-mode` | Dropped | Lauren's personal working style, built on delegating to subagents. `~/.claude/CLAUDE.md` covers the style. `let-him-cook` replaces its playbooks and triggers. |
| `setup-pstack` | Dropped | Writes Cursor's per-role model rule. `HARNESSES.md` replaces it. |
| `pstack/agents/comment-sicko.md` | `no-comments/references/comment-sicko.md` | Read by `no-comments` instead of loaded as a Cursor agent. |
| `pstack/agents/poteto-agent.md` | Not ported | Cursor agent for `poteto-mode`. |
| (none) | `<skill>/agents/openai.yaml` | Codex skill metadata, added locally. |
| (none) | `HARNESSES.md`, `UPSTREAM.md` | Harness map and this file. |
| `LICENSE` (Lauren Tan) | `LICENSE` (Kyros Abas), with Lauren's notice in `THIRD_PARTY_NOTICES.md` | The root license covers this repo's own work. If a merge changes `LICENSE`, keep `main`'s version and copy any change to Lauren's notice into `THIRD_PARTY_NOTICES.md`. |
| (none) | `let-him-cook/` | Local skill, not tracked from pstack. Routes a task to the skills it needs, inline. Written from the ideas in `poteto-mode`'s playbooks and Non-negotiables, with every delegation step removed. |
| `disable-model-invocation: true` on every skill | Removed from `benchmark-checklist`, `blast-radius`, `tdd`, `technical-writing`, `typescript-best-practices`, `unslop`, and `principles` | These load on their own when a task matches. Keep the flag off when merging upstream edits to them. |

Folders that don't come from pstack: `pr` (combined from Matt Pocock's `pr` and Theo's `file-pr`) and `let-him-cook`. Emil Kowalski's skills are installed locally from [emilkowalski/skills](https://github.com/emilkowalski/skills) but not tracked, per `.gitignore`. This file tracks pstack only.

## Local tweaks

One line per skill changed beyond the rules above, newest first.

- `teach`: runs `how` and `why` one after the other instead of in parallel.
- `arena`: writes candidates one at a time and skips the cross-judge when inline.
- `why`: searches each source one at a time instead of spawning one investigator per source.

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
cd ~/.claude/skills
git worktree add $W/up upstream
git -C $W/up rm -rq . && cp -r $W/plugins/pstack/skills/. $W/plugins/pstack/LICENSE $W/up/
git -C $W/up add -A && git -C $W/up commit -m "chore(upstream): import pstack <sha>"
git worktree remove $W/up
git diff upstream~1 upstream --stat   # what upstream changed since the last sync
git merge upstream
```

When resolving conflicts:

- Git tracks 21 of the 24 principles as renames of `principle-<name>/SKILL.md` to `principles/<name>.md`, so it usually carries upstream edits across on its own. Check the result anyway. The other three (`encode-lessons-in-structure`, `exhaust-the-design-space`, `guard-the-context-window`) changed too much for git to pair them, so an upstream edit to one shows up as a modify/delete conflict. Port the edit into `principles/<name>.md` by hand, then `git rm` the upstream path.
- A modify/delete conflict on a dropped skill means upstream edited it. Read the change, then `git rm` it again unless you want it back.
- A new upstream skill arrives as a new folder. Apply the rules above before keeping it, and add it to the tables here.

Finish by updating "Last synced commit" in this file and committing.

## Second copy

`~/.claude/skills` is the source. `~/.agents/skills` is a copy for other harnesses. After a merge or tweak, mirror it (this deletes files in the copy that no longer exist here):

```powershell
robocopy "$env:USERPROFILE\.claude\skills" "$env:USERPROFILE\.agents\skills" /MIR /XD .git synced .trash
```
