---
name: let-him-cook
description: "Pick the recipe for a coding task and run its skills in order: investigation, bug fix, feature, refactoring, or perf issue. Also maps moments mid-task to the skill that handles them. Use at the start of any nontrivial coding task, when unsure which skill applies, or for /let-him-cook."
---

# Let him cook

This skill runs inline, so you do every step yourself. Spawn subagents only when the user asks for it, per the Delegation section of [HARNESSES.md](../HARNESSES.md), which also maps tool and path terms.

This skill holds no technique of its own. It picks which of the other skills to run, and in what order. To run one, read `../<name>/SKILL.md` and follow it. Reading works for skills marked slash-command only, which the `Skill` tool refuses.

## Start

1. Restate the goal and the smallest change you expect to meet it, per [in-your-own-words](../in-your-own-words/SKILL.md). Don't wait for confirmation.
2. Match the task to one recipe below. A large or cross-cutting effort, or work the user will review after stepping away, goes to [figure-it-out](../figure-it-out/SKILL.md) instead.
3. Open a todo list whose first items are the recipe's steps. A step you skip stays in the list as `skip: <reason>`.

Commit, push, or open a PR only when the user asked for it. The always-loaded instructions win over any step here.

## Recipes

### Investigation

A question about how or why something works, or which option to pick. Read-only: edit nothing.

1. Run [how](../how/SKILL.md). For a question about motivation or history, also run [why](../why/SKILL.md).
2. For a choice between options, end with a recommendation and a tradeoffs table.

If the answer leads to a code change, stop and offer it. On a yes, switch to Bug fix or Feature.

**Reply:** the explanation or recommendation, with your real judgment. Push back when the premise is wrong.

### Bug fix

1. Reproduce it yourself with a focused case on the surface where it was reported.
2. Find the cause. Run [how](../how/SKILL.md) over the affected code and [why](../why/SKILL.md) for regression history. Rule hypotheses out with runs, not by reading. When two fixes on one premise fail, apply [attack-the-premise](../principles/attack-the-premise.md).
3. If the fix crosses a function boundary, run [architect](../architect/SKILL.md) first.
4. When the bug has a cheap local test target, write the failing test first per [tdd](../tdd/SKILL.md). Otherwise keep the repro script.
5. Fix the root cause with the smallest change, per [fix-root-causes](../principles/fix-root-causes.md).
6. Rerun the same repro. For a wide change, run [blast-radius](../blast-radius/SKILL.md).

**Reply:** what broke, the root cause, the fix, and the repro output before and after.

### Feature

1. Run [how](../how/SKILL.md) over the code the feature touches.
2. Name the data shape first, per [model-the-domain](../principles/model-the-domain.md).
3. If the code crosses a function boundary, run [architect](../architect/SKILL.md).
4. Build in small units, verifying each before the next, per [sequence-verifiable-units](../principles/sequence-verifiable-units.md).
5. Verify on the real surface, per [prove-it-works](../principles/prove-it-works.md). Use the project's `verify-<app>` skill when one exists. When none exists and the app has a user surface, offer [create-verification-skill](../create-verification-skill/SKILL.md).
6. Settle a contested design with a check that runs, not with review opinions. For logic that moves between states (a protocol, sync, payments, a multi-step flow), use [create-formal-verification-skill](../create-formal-verification-skill/SKILL.md) and read its results with [statechart](../statechart/SKILL.md). Run [interrogate](../interrogate/SKILL.md) only when nothing can run yet, and keep only the findings a run proves.
7. For a wide change, run [blast-radius](../blast-radius/SKILL.md).

**Reply:** what you built, what you chose and why, and the decisions still open.

### Refactoring

The structure changes. The behavior does not. A real bug or missing feature found on the way gets split out.

1. Run [how](../how/SKILL.md) to learn the current contract, then pin it with a characterization test or snapshot before moving anything.
2. Delete dead code and one-caller wrappers first, per [subtract-before-you-add](../principles/subtract-before-you-add.md).
3. Name the target shape, per [redesign-from-first-principles](../principles/redesign-from-first-principles.md). If it crosses a function boundary, run [architect](../architect/SKILL.md).
4. Move in small steps that keep the pin green. For an API change, migrate every caller and delete the old API in the same pass, per [migrate-callers-then-delete-legacy-apis](../principles/migrate-callers-then-delete-legacy-apis.md).
5. Prove behavior is unchanged. If the result doesn't lower reader load, per [minimize-reader-load](../principles/minimize-reader-load.md), revert it.

**Reply:** what structure changed, the pin it held against, the proof, and the reader-load difference.

### Perf issue

1. Measure a baseline. Vet it and every later number with [benchmark-checklist](../benchmark-checklist/SKILL.md).
2. Run [how](../how/SKILL.md) to ground hypotheses in the code.
3. Try fixes in this order, cheapest first, and stop at the first that meets the target:
   1. Don't do it.
   2. Do it, but don't do it again.
   3. Do it less.
   4. Do it later.
   5. Do it when they're not looking.
   6. Do it concurrently.
   7. Do it cheaper.
4. If a fix crosses a function boundary, run [architect](../architect/SKILL.md). Measure each attempt before trying the next.

**Reply:** the baseline number, the new number, the difference, and where the measurement lives.

## Mid-task

| When | Run |
| --- | --- |
| A fork you could settle by running something | Run it instead of asking. |
| Tempted to add a layer or abstraction | [laziness-protocol](../principles/laziness-protocol.md) |
| Designing types | [type-system-discipline](../principles/type-system-discipline.md), plus [typescript-best-practices](../typescript-best-practices/SKILL.md) in `.ts` files |
| A contested design | A check that runs: the project's `verify-<app>` skill, or [create-formal-verification-skill](../create-formal-verification-skill/SKILL.md) for logic that moves between states. [interrogate](../interrogate/SKILL.md) only when nothing can run yet. |
| Reporting a number you measured | [benchmark-checklist](../benchmark-checklist/SKILL.md) |
| Long, unattended, or multi-phase work | [show-me-your-work](../show-me-your-work/SKILL.md) |
| Agents keep making the same mistake in this repo | [correct](../correct/SKILL.md) |
| Code with comments, before review | [no-comments](../no-comments/SKILL.md) |
| Docs, READMEs, or commit messages | [technical-writing](../technical-writing/SKILL.md) |
| A PR title or body | [pr](../pr/SKILL.md) |
| Parallel workers or competing drafts | [swarm](../swarm/SKILL.md) or [arena](../arena/SKILL.md), only when the user asks |
| Any reply to the user | [unslop](../unslop/SKILL.md) |

In the reply, name each principle that changed a choice, and the choice it changed.
