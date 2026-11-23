---
name: sdd
description: "Spec-driven development: write one spec file with testable requirements to docs/specs/ before any code, then stop. Use for /sdd or 'write a spec for X'."
disable-model-invocation: true
---

# SDD

Write one spec file for a feature, then stop. Don't write code, and don't commit. The spec ships with the code under the user's normal git rules.

For a one-line fix or throwaway work, say a spec isn't worth it and stop.

## Steps

1. Restate the goal in one or two lines, per [in-your-own-words](../in-your-own-words/SKILL.md). Don't wait for confirmation.
2. If the feature touches existing code, trace that code per [how](../how/SKILL.md) so Constraints and Requirements match what is really there.
3. List what you would otherwise have to guess. Ask the user up to 3 questions and wait for the answers. Write the answers under Resolved. If more than 3 things are unclear, the request is too vague for a spec: say so and ask the user to narrow it.
4. Find the spec folder. Use the repo's existing spec or design-doc folder if it has one (`specs/`, `docs/specs/`, `docs/rfcs/`, `.kiro/specs/`). Otherwise use `docs/specs/`, and create it only when you write the file.
5. Pick a slug: lowercase kebab-case, 2 to 5 words, naming the thing being built (`password-reset`, not `implement-reset-flow`). No filler such as `spec-`, `feature-`, or `new-`.
6. If a file matching `*-<slug>.md` exists for the same feature, update it in place and keep its name. Otherwise create `YYYY-MM-DD-<slug>.md`, with today's local date. A different feature that collides with an existing name gets a more specific slug, never a `-2` suffix.
7. Fill [spec-template.md](spec-template.md). Delete the placeholder text of any section that doesn't apply, but keep Non-goals: write "None" if there are none.
8. Reply with the spec path, one line per requirement ID, and anything you deliberately left out.

## Writing rules

- Describe what the system does, never how. No file names, functions, libraries, or schemas in Requirements or User stories. Those belong in the implementer's plan.
- Name the entities and states the feature deals with before writing requirements, per [model-the-domain](../principles/model-the-domain.md). Use those names consistently. If the project has a `GLOSSARY.md`, use its terms, and add each new one through [domain-modeling](../domain-modeling/SKILL.md).
- Write each requirement in one EARS pattern: `WHEN <event> THE SYSTEM SHALL <response>`, `IF <condition> THEN THE SYSTEM SHALL <response>`, or `WHILE <state> THE SYSTEM SHALL <response>`. Use a plain `THE SYSTEM SHALL` for a requirement that always holds.
- Give every requirement a `Check:` that observes behavior from outside, per [test-behavior-not-implementation](../principles/test-behavior-not-implementation.md). A requirement you can't check is a wish: rewrite it or cut it.
- Put a number on every non-functional requirement: "p99 under 400 ms", not "fast".
- Rank user stories P1, P2, P3. P1 alone must be a working, testable slice.
- List as non-goals what a reader might wrongly assume is in scope.
- Keep it short enough to read in a few minutes, per [technical-writing](../technical-writing/SKILL.md).

## Status

Every spec starts at `STATUS: READY`. The only valid values are `READY`, `IN-PROGRESS`, and `DONE`. The template carries the rule so the implementer updates it: `IN-PROGRESS` when work starts, `DONE` only when every row in Verification is `PASS`.

When you update an existing spec, keep its status unless the user asks you to change it. Never set `DONE` while any Verification row is empty or not `PASS`.
