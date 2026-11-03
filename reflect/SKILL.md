---
name: reflect
description: Review the active transcript through three lenses, surface learnings, and route each to a concrete edit on an existing skill. Use when the user says reflect.
disable-model-invocation: true
---

# Reflect

This skill runs inline, so you do every step yourself. Spawn subagents only when the user asks for it, per the Delegation section of [HARNESSES.md](../HARNESSES.md), which also maps tool and path terms.

Mine the current conversation for durable learnings, then route them into skill edits.

Invoke when the user says "reflect" or "/reflect". Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## 1. Locate the active transcript

First, find this conversation's transcript in the current workspace's session transcript folder, per [HARNESSES.md](../HARNESSES.md). Never read across other projects' folders. That crosses workspace boundaries and reads private chats from unrelated projects.

```bash
ls -t <transcripts>/*.jsonl <transcripts>/*/*.jsonl <transcripts>/*/subagents/*.jsonl 2>/dev/null | head -10
```

That covers flat, nested, and subagent layouts. For each candidate, read the first JSONL line. Take the file whose `message.content[0].text` contains the conversation's opening user prompt. If none matches, pass a tight digest of the session instead.

## 2. Review through three lenses

Read the transcript once per lens, following each template in turn. Write down each lens's findings before starting the next, so one lens does not shape another. **Dispatch** (only when the user asks): spawn the three reviewers in parallel. Each needs MCP access to look up the tickets, chat threads, and observability traces the transcript references, so never use a read-only mode that strips MCP.

Under dispatch, pick each subagent's model role and reasoning level for the job, per [HARNESSES.md](../HARNESSES.md).

| Lens | Model role under dispatch | Prompt template |
|---|---|---|
| Judgment | Judgment | `references/judgment-reviewer.md` |
| Tooling | Fast | `references/tooling-reviewer.md` |
| Divergent | Another family, else judgment | `references/divergent-reviewer.md` |

Under dispatch, pass each template verbatim, substituting the transcript path or digest where marked.

## 3. Synthesize

Follow `references/synthesizer.md` with all three lenses' findings, and spot-verify citations through MCP. Under dispatch, spawn one subagent on the judgment model with MCP access, and pass the template verbatim with each reviewer's full output inlined where marked. The result is an Accepted / Rejected / Backlog list.

## 4. Structural enforcement check

Move any Accepted item that a lint rule, script, metadata flag, or runtime check would enforce more reliably to Backlog. See the [encode-lessons-in-structure](../principles/encode-lessons-in-structure.md) principle.

## 5. Apply

Present the full Accepted / Rejected / Backlog synthesis and wait for explicit approval before applying any Accepted edit. The user picks the subset and may redirect routings. Skill changes affect every future agent in the org. Do not auto-apply.

File each Backlog item to your team's devex or backlog tracker without waiting. Only the Accepted list waits for approval.

Never edit a skill inside a plugin cache. Before approval, show its destination in the project or user skill folder. After approval, copy the skill folder there and edit the copy. Never open a PR or push.

Follow each approved row's Routing exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): the parent does it directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): hand to the harness's skill-authoring skill if it has one (`skill-creator` in Claude Code, `create-skill` in Cursor) and run its draft / test / iterate loop. Otherwise draft, test on a real prompt, and iterate yourself.
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): hand to the skill-authoring skill and run its description-optimization loop, or rewrite the description against prompts that should and should not trigger it.
- `new skill via create-skill: <kebab-name>`: hand creation to the skill-authoring skill. Do not invent the shape ad hoc.

If your environment ships a SKILL.md validator, run it on every touched skill before declaring done.

## 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog filed to the devex tracker: `<issue title>` (`<tags>`). One line each.
- Dropped: one line per rejected finding + reason from the synthesis.
