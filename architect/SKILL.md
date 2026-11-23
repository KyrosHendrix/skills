---
name: architect
description: "Sketch types, signatures, and module structure before code, then stay in the loop while implementation fills in. Use for /architect, 'architect this', 'design this', or non-trivial work where jumping to code would lock in the wrong shape."
disable-model-invocation: true
---

# Architect

Before writing code, trace every system the change touches, then sketch two different shapes for it (types, signatures, module boundaries) and pick the one where each piece of state has one owner, each task has one way to do it, and callers need to know least. Redesign instead of patching if workarounds repeat.

If the project has a `GLOSSARY.md`, name types and modules with its terms. Add a new name worth keeping through [domain-modeling](../domain-modeling/SKILL.md).

For a large or contested design, run the [arena](../arena/SKILL.md) skill instead of sketching alone.
