# Turk-OS lab journal

One entry per work session, newest at the bottom. The journal is part of the method, not an extra: it records what
was tried, what broke and why. It gets you unstuck, and it's what you bring along when you ask for help.

## How to keep it

- **Every session:** the date, the phase and task, what you tried, what happened, and what you learned.
- **Bugs that take more than an hour:** write the hypothesis down *before* each attempt, and change one thing at a
  time. Record the evidence (serial log lines, `-d int` output, register values) instead of your memory of it.
- **Assembly phrase book:** when you read compiler output (Phase 0, Task 0.6 and Phase 1), paste the interesting
  snippets and what each instruction does.
- **Milestones:** when a phase's exit criteria are all met, add a short milestone entry, tick the box in the
  [README tracker](../README.md#milestone-tracker) and add a line to the [CHANGELOG](../CHANGELOG.md).
- **Stuck for two sessions?** Follow the getting-unstuck protocol from the roadmap and bring this file along when you
  ask for help.

## Entry template

Copy this block for each session.

```markdown
### YYYY-MM-DD · Phase N, Task N.M: short title

**Goal:** what this session was meant to achieve.

**Did:**
- ...

**Result:** what worked, what didn't. Paste the key serial output or error message.

**Bug log** (only when stuck):
| # | Hypothesis | Change | Evidence | Verdict |
|---|---|---|---|---|
| 1 | ... | ... | ... | confirmed / rejected |

**Learned:** the one thing to remember from today.

**Next:** where to pick up next time.
```

## Milestone template

```markdown
### YYYY-MM-DD · Milestone: Phase N complete, "Milestone name"

- [x] Each exit criterion from the phase document, with how it was verified.
- Time spent: N weeks (planned: N).
- Hardest part: ...
- Would do differently: ...
- Commit: `abc1234`
```

---

## Entries

<!-- New entries go below this line. -->
