---
name: how
description: "Use for \"how does X work\", code walkthroughs before changing something, and placement / ownership / layering questions (\"where should this live\", \"which package owns this\", \"is this the right layer\"). Explains subsystem architecture, runtime flow, onboarding mental models. Use why for motivation."
---

# How

This skill runs inline, so you do every step yourself. Spawn subagents only when the user asks for it, per the Delegation section of [HARNESSES.md](../HARNESSES.md), which also maps tool and path terms.

Explore the codebase to answer "how does X work?" questions. Edit nothing.

If the scope is ambiguous, state your interpretation and explore. The user can redirect.

## 1. Explain

If the project has a `GLOSSARY.md`, read it first and use its terms in the explanation.

Explore and explain in one pass, to this brief:

> You are writing an architectural explanation for a senior engineer. Explore the codebase and write an explanation that a senior engineer unfamiliar with this area could read and walk away with a solid mental model. They should understand the architecture well enough to start working in it confidently. You have read-only access to the codebase.

**Dispatch** (only when the user asks): spawn one read-only subagent on the judgment model with the user's question and that brief.

## 2. Present

Present the explanation. When a subagent wrote it, light edits for clarity or context from the conversation are fine. Do not substantially rewrite it.
