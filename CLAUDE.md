# Guidance for Claude Code agents in this repo

Public, published copy of the Asili prototype; the source of truth is `design/prototype/` in `asili-inc/brillio_official`. See `README.md`.

## Advisor delegation (model selection for multi-step work)

For multi-step work, follow the `advisor-delegation` skill (`.claude/skills/advisor-delegation/SKILL.md`):

- The parent session (Opus) reads the inputs and writes a BRIEF (goal, context, constraints, plan, done-when, escalate-if, return shape) before spawning anything.
- Execution goes to subagents: Sonnet for drafting, research and synthesis; Haiku for bulk or mechanical work and for yes/no gate checks (Jev if connected).
- Subagents stop and return `ESCALATE: <trigger>` on: conflicting requirements, irreversible writes (pushes or merges to main (publishes publicly), edits that belong in brillio_official, non-public data, deletes), low confidence (<0.7), two failed attempts, or exec- or customer-facing output.
- The parent answers with PLAN, CORRECTION or STOP, and can't approve an irreversible write; that waits for Andy. Max 3 advisor rounds per task, then ask Andy.
- End each delegated task with one line: subagents by tier, escalation count, triggers hit.
