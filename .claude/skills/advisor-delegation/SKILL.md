---
name: advisor-delegation
description: Opus-advisor / Sonnet-Haiku-executor workflow for asili-prototype. Use for multi-step work that can be delegated to subagents — re-syncing many screens from brillio_official, link or hotspot sweeps across the 62 screens, accessibility passes. Not for a single-screen fix.
---

# Advisor delegation

Use for any multi-step task that can be handed to subagents. Skip it for single-turn questions or small edits — the overhead isn't worth it.

This is the orchestrator pattern with an advisor on call: Opus comprehends and briefs up front, cheaper models execute, and Opus comes back only when an escalation trigger fires. Claude Code has no native advisor tool, so the handoff is a subagent returning early and the parent answering.

## Roles

- **Parent (Opus, or Fable for the hardest problems): comprehension and judgment.** Reads the inputs, writes the brief, answers escalations, owns the final output. Does not do bulk work itself.
- **Executor (Sonnet):** runs a brief end to end — drafting, research, synthesis, tool-heavy work.
- **Bulk (Haiku):** mechanical, high-volume, low-judgment steps — tagging, extraction, normalization, first-pass summaries.
- **Gate (Jev if a Jev tool is connected, otherwise Haiku):** yes/no or multiple-choice checks with a confidence score.

## Step 1 — Write the brief (parent, before spawning anything)

Keep it under 300 words. The brief is the weak point: a wrong brief produces confident work on the wrong problem, so the parent reads the inputs itself before writing it.

```
BRIEF
Goal:        one sentence; an outcome, not an activity
Context:     only what the executor needs; link or give paths rather than pasting
Constraints: scope limits, allowed sources, format, audience
Plan:        3–6 steps
Done when:   acceptance criteria the executor can check itself
Escalate if: default triggers below + this repo's triggers + any task-specific ones
Return:      exact shape of what comes back (table columns, file path, ≤200-word summary)
```

## Step 2 — Delegate

- Spawn subagents with the Agent tool, setting `model: "sonnet"` or `model: "haiku"`, with the brief verbatim as the prompt.
- Run independent pieces in parallel, one brief per subagent, in a single message.
- Add to every brief: "If an escalation trigger fires, stop and return `ESCALATE: <trigger> — <what you know> — <decision needed>`. Don't guess."
- A subagent can't pause mid-run to ask. ESCALATE means it returns early; the parent answers by continuing that agent with SendMessage if the session has it (keeps its context), and otherwise by re-spawning with the brief, the executor's findings so far and the PLAN or CORRECTION.

## Step 3 — Escalation (the parent acts as advisor)

Default triggers:

1. Requirements or sources conflict.
2. The step is hard to reverse (see this repo's list below).
3. Low confidence: the executor is unsure, or a gate returns confidence below 0.7.
4. Two failed attempts at the same step.
5. The output is exec-facing or customer-facing.

The parent answers an ESCALATE with exactly one of:

- **PLAN** — revised steps
- **CORRECTION** — what's wrong and the fix
- **STOP** — abandon, with the reason

Keep the answer short (roughly 400–700 tokens), then resume or re-spawn the executor. The parent can't approve a hard-to-reverse write. For that trigger it answers PLAN or STOP only, and the write waits for the user's own confirmation; parent approval is not user approval.

## Step 4 — Gate checks

Use a gate for any decision with a fixed answer set. If a Jev tool is available, send it the state plus the questions (only what fits in the request; irrelevant context lowers its accuracy, and never send restricted data to it without review). Otherwise spawn Haiku and tell it to answer only from the given options, with a 0–1 confidence. Anything below 0.7 escalates.

## Step 5 — Parent review

The parent checks the output against "Done when" before presenting it. Exec-facing work always gets a parent review.

## Cost hygiene and measurement

- Cap advisor rounds at 3 per task. If a fourth is needed, stop and ask the user.
- End with one line: subagents spawned by tier, escalation count, triggers hit. A healthy escalation rate is roughly 10–25% (an assumption to calibrate): near zero means the triggers are too loose; above 40% means the brief is weak.
## This repo's triggers (always in the brief)

This repo is a public, published copy served by GitHub Pages. Hard to reverse here, so the executor stops and returns ESCALATE before any of these:

- Pushing or merging to `main`. It publishes immediately to a public URL.
- Editing a screen's design here instead of in `design/prototype/` in `asili-inc/brillio_official`, which is the source of truth. Changes made only here are overwritten on the next export.
- Adding anything not already public: real names or photos of children (DS-16 allows illustrated avatars only), trainer or parent data, internal docs, Supabase or Stripe identifiers.
- Deleting screens or files.
