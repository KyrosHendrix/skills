# Upstream

Most skills here are a port of [pstack](https://github.com/cursor/plugins/tree/main/pstack/skills) by [Lauren Tan](https://github.com/poteto) (MIT, see `THIRD_PARTY_NOTICES.md`). This file records what the port changed, so an upstream update can be merged without losing local edits.

## Base

| Field | Value |
| --- | --- |
| Upstream repo | `https://github.com/cursor/plugins`, path `pstack/` |
| Last synced commit | `b7b93c3` (2026-11-26) |
| Git branch holding upstream as published | `upstream` |
| Git branch holding this folder | `dev` (work lands here; `main` changes only through a PR from `dev`) |

The `upstream` branch stores `pstack/skills/*` and `pstack/LICENSE` at the repo root, so its paths match `dev` and `main`. Never edit `upstream` by hand. It only receives new pstack snapshots.

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
| `bro` | Dropped | Replaced by Matt Pocock's `wait-what`. An upstream edit to it shows up as a modify/delete conflict. `git rm` it again. |
| `ketchup`, `ketchup-video` | Dropped | Not wanted. An upstream edit to either shows up as a modify/delete conflict. `git rm` it again. |
| `make-bot-ui` | Dropped | Not wanted. |
| `poteto-help` | Dropped | Help for pstack's Cursor setup. |
| `poteto-mode` | Dropped | [Lauren](https://github.com/poteto)'s personal working style, built on delegating to subagents. `~/.claude/CLAUDE.md` covers the style. `let-him-cook` replaces its playbooks and triggers. |
| `setup-pstack` | Dropped | Writes Cursor's per-role model rule. `HARNESSES.md` replaces it. |
| `reframe` | Dropped | Not wanted. An upstream edit to it shows up as a modify/delete conflict. `git rm` it again. |
| `pstack/agents/comment-sicko.md` | `no-comments/references/comment-sicko.md` | Read by `no-comments` instead of loaded as a Cursor agent. |
| `pstack/agents/poteto-agent.md` | Not ported | Cursor agent for `poteto-mode`. |
| (none) | `<skill>/agents/openai.yaml` | Codex skill metadata, added locally. |
| (none) | `HARNESSES.md`, `UPSTREAM.md` | Harness map and this file. |
| `LICENSE` ([Lauren Tan](https://github.com/poteto)) | `LICENSE` ([Kyros Abas](https://github.com/KyrosHendrix)), with [Lauren](https://github.com/poteto)'s notice in `THIRD_PARTY_NOTICES.md` | The root license covers this repo's own work. If a merge changes `LICENSE`, keep `dev`'s version and copy any change to Lauren's notice into `THIRD_PARTY_NOTICES.md`. |
| (none) | `let-him-cook/` | Local skill, not tracked from pstack. Routes a task to the skills it needs, inline. Written from the ideas in `poteto-mode`'s playbooks and Non-negotiables, with every delegation step removed. |
| (none) | `sdd/` | Local skill, not tracked from pstack. Writes one spec file with testable requirements to `docs/specs/`, then stops. |
| (none) | `grill-me/` | Local skill, not tracked from pstack. Matt Pocock's `grill-me` with his `grilling` body merged in and fact-finding run inline. |
| (none) | `domain-modeling/` | Local skill, not tracked from pstack. Matt Pocock's `domain-modeling`, as published. It writes `GLOSSARY.md` and ADRs, and the skills in Local tweaks read the glossary. |
| (none) | `wait-what/` | Local skill, not tracked from pstack. Matt Pocock's `wait-what`, as published. Replaces `bro`. |
| `disable-model-invocation: true` on every skill | Removed from `benchmark-checklist`, `blast-radius`, `how`, `interrogate`, `tdd`, `teach`, `technical-writing`, `typescript-best-practices`, `unslop`, and `why` | These load on their own when a task matches. Keep the flag off when merging upstream edits to them. |

Folders that don't come from pstack: `pr` (combined from [Matt Pocock](https://github.com/mattpocock)'s `pr` and [Theo](https://github.com/t3dotgg)'s `file-pr`), `let-him-cook`, `sdd`, `grill-me`, `domain-modeling`, and `wait-what`. The pstack sections here don't cover them. [Matt Pocock's skills](#matt-pococks-skills) tracks `pr`, `grill-me`, `domain-modeling`, and `wait-what`.

## Local tweaks

One line per skill changed beyond the rules above, newest first.

- `architect`: names types and modules with `GLOSSARY.md` terms when the file exists, and adds new names through `domain-modeling`.
- `how`: reads `GLOSSARY.md` first when it exists and uses its terms.
- `interrogate`: uses `GLOSSARY.md` terms in the intent and findings when the file exists.
- `sharpen`: uses `GLOSSARY.md` terms in the rewritten prompt when the file exists.
- `tdd`: names the regression test with `GLOSSARY.md` terms when the file exists.
- `why`: uses `GLOSSARY.md` terms when the file exists.
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

If the commit is newer than "Last synced commit", put the snapshot on `upstream` from a separate worktree. This keeps the live skills folder untouched until the merge. The worktree shares `.git/info/exclude`, which ignores every top-level folder, so `git add` needs `-f` there.

```bash
cd ~/.claude/skills   # or ~/.agents/skills
git worktree add $W/up upstream
git -C $W/up rm -rq . && cp -r $W/plugins/pstack/skills/. $W/plugins/pstack/LICENSE $W/up/
git -C $W/up add -fA . && git -C $W/up commit -m "chore(upstream): import pstack <sha>"
git worktree remove $W/up
git diff upstream~1 upstream --stat   # what upstream changed since the last sync
git merge upstream
```

When resolving conflicts:

- Git tracks 18 of the 19 principles as renames of `principle-<name>/SKILL.md` to `principles/<name>.md`, so it usually carries upstream edits across on its own. Check the result anyway. The other one (`encode-lessons-in-structure`) changed too much for git to pair it, so an upstream edit to it shows up as a modify/delete conflict. Port the edit into `principles/<name>.md` by hand, then `git rm` the upstream path.
- A modify/delete conflict on a dropped skill means upstream edited it. Read the change, then `git rm` it again unless you want it back.
- A new upstream skill arrives as a new folder. Apply the rules above before keeping it, and add it to the tables here.

Finish by updating "Last synced commit" in this file, committing, and pushing both branches with `git push origin dev upstream`. Then update the other clones (see below).

## Matt Pocock's skills

Four skills come from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT, see `THIRD_PARTY_NOTICES.md`). No branch holds his snapshots. Compare his files at the base commit and at his latest commit, then port the difference by hand.

| Field | Value |
| --- | --- |
| Upstream repo | `https://github.com/mattpocock/skills` |
| Base commit | `6f7f783` (2026-11-25) |

| Ours | His source | What we changed |
| --- | --- | --- |
| `pr` | `skills/engineering/pr/` | Adds a Title section with the title rules from [Theo](https://github.com/t3dotgg)'s [file-pr](https://github.com/t3dotgg/tstack/tree/main/skills/file-pr), and the description says "title or body". |
| `grill-me` | `skills/productivity/grill-me/` and `skills/productivity/grilling/` | `grill-me` is his one-line stub, so `grilling`'s body sits in our `SKILL.md`. Facts get looked up inline, with the sub-agent behind a **Dispatch** note. Adds the `GLOSSARY.md` line. |
| `domain-modeling` | `skills/engineering/domain-modeling/` | None. Files are LF. |
| `wait-what` | `skills/productivity/wait-what/` | None. Files are LF. |

To check for an update:

```bash
W=/tmp/matt-sync && rm -rf $W && git clone -q https://github.com/mattpocock/skills $W
git -C $W log --oneline 6f7f783..HEAD -- skills/engineering/pr skills/productivity/grill-me skills/productivity/grilling skills/engineering/domain-modeling skills/productivity/wait-what
git -C $W diff 6f7f783..HEAD -- <his source path>
```

Port each change into our skill and keep what the table says we changed. Apply the three rules above to new text. Then set "Base commit" to his latest commit. A new skill of his that we take gets a row here.

## Three clones

`~/.claude/skills` (Claude Code), `~/.agents/skills` (Codex and others), and `~/.gemini/config/skills` (Gemini) are separate clones of the same GitHub repo. Work in one, push both branches, then update each of the others before you change anything there:

```bash
cd ~/.gemini/config/skills   # each clone you didn't work in
git pull --ff-only
git fetch origin upstream:upstream
```
