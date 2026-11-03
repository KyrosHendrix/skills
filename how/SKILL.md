---
name: how
description: "Use for \"how does X work\", code walkthroughs before changing something, and placement / ownership / layering questions (\"where should this live\", \"which package owns this\", \"is this the right layer\"). Explains subsystem architecture, runtime flow, onboarding mental models. Use why for motivation."
disable-model-invocation: true
---

# How

This skill runs inline, so you do every step yourself. Spawn subagents only when the user asks for it, per the Delegation section of [HARNESSES.md](../HARNESSES.md), which also maps tool and path terms.

Explore the codebase to answer "how does X work?" questions. Produce architectural explanations at the level of a senior engineer onboarding onto a subsystem, enough to build a working mental model, not so much that it reads like annotated source code.

Do every step yourself and edit nothing. **Dispatch** (only when the user asks): each explorer is a read-only subagent on the fast model, and the explainer runs on the judgment model.

## 1. Assess complexity

If the scope is ambiguous, state your interpretation and explore. The user can redirect.

- **Simple** (a single module, a small utility, a narrow question such as "how does function X work"): a single pass. Explore and explain together, following `references/explainer-prompt.md` without the explorer-findings section. Go to step 4.
- **Complex** (a subsystem spanning multiple files or services, a cross-cutting feature, a full architectural overview): go to step 2.

When in doubt, take the simple path.

## 2. Explore (complex only)

Decompose the question into 2 to 4 angles, each a distinct slice of the subsystem. Trace each angle in turn, following `references/explorer-prompt.md`, and write down its findings before starting the next. **Dispatch** (only when the user asks): spawn one explorer per angle in a single message, each with the template and its angle filled in.

## 3. Synthesize (complex only)

Once every angle is traced, write the explanation per `references/explainer-prompt.md` from all the findings. Under dispatch, spawn one explainer with the template and every explorer's findings filled in.

## 4. Present

Present the explanation. When an explainer subagent wrote it, light edits for clarity or context from the conversation are fine, but do not substantially rewrite it.

## Output format

The explanation uses the sections defined in `references/explainer-prompt.md`, dropping any that do not apply: Overview, Key Concepts, How It Works, Where Things Live, Gotchas.
