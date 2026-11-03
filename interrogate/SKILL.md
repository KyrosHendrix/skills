---
name: interrogate
description: "Use for \"interrogate\", \"adversarial review\", \"multi-model review\", \"challenge this\", \"stress test this code\", \"find blind spots\", or \"tear this apart\". You attack a change or design as an adversary, and only findings proven by a run survive."
disable-model-invocation: true
---

# Interrogate

This skill runs inline, so you do every step yourself. Spawn subagents only when the user asks for it, per the Delegation section of [HARNESSES.md](../HARNESSES.md), which also maps tool and path terms.

Act as an adversary and try to break a change or a design. A finding survives only when a run proves it. Report the findings. Do not apply fixes.

## Frame

The scope is what the user names. Otherwise it is the branch diff from its merge base with the base branch, through the working tree. State the intent with the [in-your-own-words](../in-your-own-words/SKILL.md) skill, from the user's message, the commits, and the PR description. Ask the user when the intent is unclear.

## Attack

Work from the scope and the intent, and follow this brief:

> Break this change. Prove each finding with a run against the real code, such as a failing test, a repro script, or command output that shows the wrong behavior. For a design or plan, run a prototype or probe the current tree. Keep repros out of the scope's files, in a scratch location or your own worktree. Report each finding with a title, the exact command, the output it printed, and the output the intent requires. Leave out anything you did not run. That covers style, naming, structure, and risks that need a future change to matter. No findings is a valid result.

Take at least two distinct angles of attack, one after another, such as bad inputs and edge cases first, then state, concurrency, and failure paths. Write down each angle's findings before starting the next.

**Dispatch** (only when the user asks): run Phase A steps 3 and 4 and Phase B of the [arena](../arena/SKILL.md) skill instead, with three changes. Copy the scope's uncommitted changes into each worktree. Spawn each adversary as a subagent that can write and run code. Give each adversary the scope, the intent, and the brief above in place of the arena task and rationale.

## Verify

Verify each finding per arena Phase F by rerunning its command. Keep a finding when the output shows the failure and the failure breaks the intent. Drop the rest. Merge findings that share a root cause.

Skip arena Phases C, D, and E. Different angles find different failures, so the verdict keeps every verified finding, not one angle's list.

## Output

- **Intent**: the restatement.
- **Findings**: for each verified finding, the command, its output, how it breaks the intent, and the angle or adversary that found it. Say "no verified findings" when none survive.
- **Dropped**: for each angle or adversary, the count of dropped findings and why.
