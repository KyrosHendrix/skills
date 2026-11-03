# Agent skills for Claude Code, Codex, and Cursor

Lauren Tan's [pstack](https://github.com/cursor/plugins/tree/main/pstack/skills), rebuilt to run inline in one context on one plan.

- **Inline by default.** A skill does every step itself. It spawns subagents only when you ask in the session, so a task doesn't multiply its token cost by fanning out.
- **Works in any harness.** No model slugs or vendor setup. [HARNESSES.md](HARNESSES.md) maps each tool, model role, and path to Claude Code, Codex, and Cursor.
- **Routes itself.** [let-him-cook](let-him-cook/SKILL.md) matches a coding task to a recipe (investigation, bug fix, feature, refactoring, or perf issue) and runs the right skills in order.

## Install

Clone into the user skill folder for your harness. Use an empty folder, or back up the one you have first.

| Harness | Command |
| --- | --- |
| Claude Code | `git clone https://github.com/KyrosHendrix/skills ~/.claude/skills` |
| Codex | `git clone https://github.com/KyrosHendrix/skills ~/.agents/skills` |
| Cursor | `git clone https://github.com/KyrosHendrix/skills ~/.cursor/skills` |

To take a single skill, copy its folder. The pstack skills also link to `HARNESSES.md` and the `principles` folder, so copy those with it.

Most pstack skills run only when you type their slash command, such as `/how` or `/why`. Start a coding task with `/let-him-cook`, or let it load on its own, and it picks the rest.

## From Lauren Tan's pstack

These 30 skills are adapted from [pstack](https://github.com/cursor/plugins/tree/main/pstack/skills) by Lauren Tan, MIT licensed. [UPSTREAM.md](UPSTREAM.md) lists what changed from pstack and how to merge pstack updates.

| Skill | Use it to |
| --- | --- |
| [architect](architect/SKILL.md) | Sketch types, signatures, and modules before writing code, then fill in against the sketch. |
| [arena](arena/SKILL.md) | Write competing candidates for one task, pick a base, and graft in the best parts of the rest. |
| [benchmark-checklist](benchmark-checklist/SKILL.md) | Vet a performance measurement before reporting or acting on it. |
| [blast-radius](blast-radius/SKILL.md) | Find what a change could break outside the diff, and prove it's safe by running code. |
| [bro](bro/SKILL.md) | Restate the last message in plain language. |
| [correct](correct/SKILL.md) | Find mistakes agents keep repeating in a repo and make each one impossible. |
| [create-formal-verification-skill](create-formal-verification-skill/SKILL.md) | Generate a project skill that formally verifies the riskiest code. |
| [create-verification-skill](create-verification-skill/SKILL.md) | Generate a project skill that drives the app the way a user does. |
| [figure-it-out](figure-it-out/SKILL.md) | Design an auditable playbook for a large migration or multi-part change. |
| [how](how/SKILL.md) | Explain how code works, and where new code should live. |
| [in-your-own-words](in-your-own-words/SKILL.md) | Restate the user's goals and problem to check understanding. |
| [interrogate](interrogate/SKILL.md) | Review code or a design adversarially to find blind spots. |
| [ketchup](ketchup/SKILL.md) | Catch the user up on what changed in the chat since their last message. |
| [ketchup-video](ketchup-video/SKILL.md) | Same as ketchup, as a short subtitled slide video. |
| [maintain-formal-verification-skill](maintain-formal-verification-skill/SKILL.md) | Bring formal verification back in line after the code changes. |
| [maintain-verification-skill](maintain-verification-skill/SKILL.md) | Keep a project's verification skill and feature map accurate. |
| [no-comments](no-comments/SKILL.md) | Run a Comment Sicko pass and fix the comments it flags. |
| [principles](principles/SKILL.md) | Engineering principles, one file each. pstack ships these as 24 separate `principle-*` skills. |
| [recall](recall/SKILL.md) | Rebuild recent working context from chat history and shared records. |
| [reflect](reflect/SKILL.md) | Review the transcript for learnings and turn each into an edit to a skill. |
| [sharpen](sharpen/SKILL.md) | Make a task deeper, not bigger, and rewrite the prompt. |
| [show-me-your-work](show-me-your-work/SKILL.md) | Keep a TSV decision log for long or unattended work. |
| [statechart](statechart/SKILL.md) | Check a TLA+ model of a stateful system and draw it as a Mermaid statechart. |
| [swarm](swarm/SKILL.md) | Fan out parallel workers and return one report. The only skill that always spawns subagents, so it runs only when invoked. |
| [tdd](tdd/SKILL.md) | Write the failing test first, when asked or when the test target is cheap and obvious. |
| [teach](teach/SKILL.md) | Explain a body of work plainly, using `how` and `why`. |
| [technical-writing](technical-writing/SKILL.md) | Write or review docs, RFCs, READMEs, and PR text to a layered style standard. |
| [typescript-best-practices](typescript-best-practices/SKILL.md) | Follow TypeScript practices when reading or editing `.ts` and `.tsx` files. |
| [unslop](unslop/SKILL.md) | Cut AI tells from writing. |
| [why](why/SKILL.md) | Find the reasons behind code, decisions, and regressions across every available source. |

pstack skills not included here: `automate-me`, `make-bot-ui`, `poteto-help`, `poteto-mode`, and `setup-pstack`. UPSTREAM.md gives the reasons.

## Other

| Skill | Use it to | Source |
| --- | --- | --- |
| [let-him-cook](let-him-cook/SKILL.md) | Pick the recipe for a coding task and run its skills in order, inline. | Kyros, from the ideas in pstack's `poteto-mode` playbooks, without the subagent delegation. |
| [pr](pr/SKILL.md) | Write a PR title and body. | Kyros combined two PR skills: Matt Pocock's [pr](https://github.com/mattpocock/skills/tree/main/skills/engineering/pr) and the title naming rules from Theo's [file-pr](https://github.com/t3dotgg/tstack/tree/main/skills/file-pr). |

Pairs well with [emilkowalski/skills](https://github.com/emilkowalski/skills) for motion and UI work. Install it alongside, and `let-him-cook` routes animation and UI tasks to it.

## Branches

- `main` holds the skills.
- `upstream` is pstack exactly as published. Only new pstack snapshots go there. See [UPSTREAM.md](UPSTREAM.md).

## License

MIT. The root [LICENSE](LICENSE) covers this repo's own work. [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) holds the MIT notices for Lauren Tan, Matt Pocock, and Theo Browne, and lists the folders each one covers.
