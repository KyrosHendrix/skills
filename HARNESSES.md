# Harness map

These skills use plain terms for tools and models. Map each term to your harness with the tables below. If your harness has no row for a term, use the closest tool it has and say so in one line. Never pass a field the tool schema rejects.

## Delegation

Every skill runs inline by default. Do each role a skill names, such as explorer, investigator, candidate, judge, adversary, reviewer, or worker, yourself, one after another, in this conversation. Finish and write down one role's result before starting the next, so one pass does not shape another.

Spawn subagents only when the user asks for it in this session. Asking looks like "dispatch", "use subagents", "fan out", "in parallel", or invoking `/swarm`. A request covers the task it was made for, not later tasks. Never spawn a subagent because a task is large, slow, or would fill the context. Offer it in one line instead and keep working inline.

When the user does ask, follow the skill's **Dispatch** note for how to split the work, the Actions table below for the tool, and the model roles for which model runs each part.

## Actions

| Term in a skill | Claude Code | Codex | Cursor |
| --- | --- | --- | --- |
| Spawn a subagent | `Agent` tool, `subagent_type: "general-purpose"` | Spawn a sub-agent if the session offers one. Otherwise do the brief yourself, in sequence, and say so. | `Task` tool, `subagent_type: generalPurpose` |
| Read-only subagent | `subagent_type: "Explore"`. Its brief also says "do not edit files". Check `git status` afterward. | Same fallback as above, with a "do not edit files" brief | `Task` with `readonly: true`. Readonly strips MCP access, so use agent mode when the subagent needs MCP. |
| Isolated subagent | `isolation: "worktree"`. Name the branch or commit to check out. | Make a git worktree yourself and point the worker at it | `environment: "cloud"`, plus `cloud_base_branch` for a non-default branch |
| Spawn in parallel | Several `Agent` calls in one message. They run in the background and notify you. | Same as spawn | Several `Task` calls in one message with `run_in_background: true` |
| Ask the user | `AskUserQuestion`. In `-p` mode, ask in plain text. | Ask in plain text | `AskQuestion` |
| Keep a todo list | `TodoWrite` when offered. Otherwise keep the list in your reply. | `update_plan` | The built-in todo list |
| Load another skill | Read `../<name>/SKILL.md` from the calling skill's folder and follow it. The `Skill` tool refuses user-only skills. | Same | Same |

## Model roles

Model roles matter only under dispatch. Inline work runs on the session model. Skills name a role, not a model. Pick the model per role from what your harness offers:

- **Judgment model.** The strongest model available, for design, prose, synthesis, and hard changes.
- **Fast model.** A cheaper, faster model for mechanical edits, bulk reading, and parallel workers.
- **Another model family.** A model from a different vendor than the one doing the work, for judges, adversaries, and second opinions. Different families catch different bugs.

| Role | Claude Code | Codex | Cursor |
| --- | --- | --- | --- |
| Judgment | `model: "opus"`, `effort: "high"` or above | The session model at high reasoning | A Claude Opus or GPT top-tier slug |
| Fast | `model: "sonnet"` or `"haiku"` | The session model at low reasoning | A fast slug such as Grok or Composer |
| Another family | Not available. Use the session model in a fresh subagent and say the panel was single-family. | Not available, same fallback | Any slug from another vendor |

When a role has no matching model, omit the model so the subagent runs on the session model, and say so.

## Paths

| Term in a skill | Claude Code | Codex | Cursor |
| --- | --- | --- | --- |
| Project skill folder | `.claude/skills/` | `.agents/skills/` | `.cursor/skills/` |
| User skill folder | `~/.claude/skills/` | `~/.agents/skills/` | `~/.cursor/skills/` |
| Always-loaded agent instructions | `CLAUDE.md` | `AGENTS.md` | `AGENTS.md` or `.cursor/rules/` |
| Session transcripts | `~/.claude/projects/<slug>/<session>.jsonl`, where `<slug>` is the workspace path with each `/`, `\`, `:` and `.` turned into `-`. Subagent transcripts sit in `<session>/subagents/`. | `~/.codex/sessions/` | `~/.cursor/projects/<slug>/agent-transcripts/` |

Read only the current workspace's transcripts. Never glob across other projects.

## Harness-only features

Some steps assume a feature your harness lacks, such as a review bot or a cloud agent dashboard. Write one line, `Skipped in <harness>: <step>. <reason>.`, and go on to the next step.

## Origin

Adapted from [pstack](https://github.com/cursor/plugins/tree/main/pstack) by [Lauren Tan](https://github.com/poteto), MIT licensed. See `LICENSE`.
