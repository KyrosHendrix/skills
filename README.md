# skills

Kyros's agent skills. They load from `~/.claude/skills` in Claude Code, and a mirror in `~/.agents/skills` serves other harnesses.

Every skill runs inline. A skill spawns subagents only when the user asks for it in the session. [HARNESSES.md](HARNESSES.md) maps tool, model, and path terms to Claude Code, Codex, and Cursor.

## From Lauren Tan's pstack

These 30 skills are adapted from [pstack](https://github.com/cursor/plugins/tree/main/pstack/skills) by Lauren Tan, MIT licensed (see [LICENSE](LICENSE)). [UPSTREAM.md](UPSTREAM.md) lists what changed from pstack and how to merge pstack updates.

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

## From Emil Kowalski's skills

These 16 skills come from [emilkowalski/skills](https://github.com/emilkowalski/skills/tree/main/skills) by Emil Kowalski, MIT licensed. They cover motion, UI polish, and frontend craft.

| Skill | Use it to |
| --- | --- |
| [animate](animate/SKILL.md) | Build a web animation, deciding first whether it should animate at all. |
| [animate-expo](animate-expo/SKILL.md) | Build animations and gestures in React Native and Expo. |
| [animation-performance](animation-performance/SKILL.md) | Keep web animations inside the frame budget and find dropped frames. |
| [animation-vocabulary](animation-vocabulary/SKILL.md) | Name a motion effect from a vague description. |
| [apple-design](apple-design/SKILL.md) | Apply Apple's interface and motion design to the web. |
| [ask-sonner](ask-sonner/SKILL.md) | Set up and debug Sonner toasts in React. |
| [break-ui](break-ui/SKILL.md) | Stress-test a UI with worst-case data and report what breaks. |
| [emil-design-eng](emil-design-eng/SKILL.md) | Apply Emil Kowalski's UI polish and component design standards. |
| [find-animation-opportunities](find-animation-opportunities/SKILL.md) | Find places that should animate, without changing code. |
| [improve-animations](improve-animations/SKILL.md) | Audit a codebase's motion and write implementation plans. |
| [mobile-native](mobile-native/SKILL.md) | Make a web app feel native on a phone. |
| [pick-ui-library](pick-ui-library/SKILL.md) | Pick a frontend library for a task from a curated list. |
| [prototype](prototype/SKILL.md) | Build several different versions of a UI piece behind a live picker. |
| [remove-animations](remove-animations/SKILL.md) | Remove motion an interface doesn't need. |
| [review-animations](review-animations/SKILL.md) | Review animation code against a high craft bar. |
| [write-swift](write-swift/SKILL.md) | Write modern Swift, including Swift 6 concurrency. |

## Other

| Skill | Use it to | Source |
| --- | --- | --- |
| [let-him-cook](let-him-cook/SKILL.md) | Pick the recipe for a coding task and run its skills in order, inline. | Kyros, from the ideas in pstack's `poteto-mode` playbooks, without the subagent delegation. |
| [pr](pr/SKILL.md) | Write a PR title and body. | Kyros combined two PR skills: Matt Pocock's [pr](https://github.com/mattpocock/skills/tree/main/skills/engineering/pr) and the title naming rules from Theo's [file-pr](https://github.com/t3dotgg/tstack/tree/main/skills/file-pr). |

## Branches

- `main` is this folder.
- `upstream` is pstack exactly as published. Only new pstack snapshots go there. See UPSTREAM.md.

## Keeping the mirror in sync

`~/.claude/skills` is the source. After any change, mirror it to `~/.agents/skills`. This deletes files in the copy that no longer exist here:

```powershell
robocopy "$env:USERPROFILE\.claude\skills" "$env:USERPROFILE\.agents\skills" /MIR /XD .git synced .trash
```
