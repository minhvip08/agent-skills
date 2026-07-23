---
name: interview-me
description: Extracts what the user actually wants instead of what they think they should want. Achieves this through one-question-at-a-time interview until ~95% confidence about the underlying intent. Use when an ask is underspecified ("build me X" without "for whom" or "why now"), when the user explicitly invokes ("interview me", "grill me", "are we sure?", "stress-test my thinking"), or when you catch yourself silently filling in ambiguous requirements before any plan, spec, or code exists.
---

# Interview Me

## Overview

What people ask for and what they actually want are often different things. The cheapest moment to find the gap is before any spec or code exists — after that, switching costs are real and the misfit gets locked in. This skill closes the gap by asking one question at a time, each with a best guess attached, until you can predict what the user will say before they say it. `spec-driven-development` is downstream: it assumes you already know what you want and writes it down.

## When to Use

- The ask is missing **who** it's for, **why** now, what **success** looks like, or the binding **constraint**
- The request is conventional rather than specific ("build me X") and you'd be guessing to unpack it
- The user explicitly invokes: "interview me", "grill me", "are we sure?"

**When NOT to use:** the ask is unambiguous and self-contained (rename, typo fix); the user asked for speed over verification; pure information requests; you already have ≥95% confidence (see the stop condition below).

**Loading constraint:** needs a live, responsive user — don't invoke in CI, scheduled runs, or other non-interactive contexts. Flag the ambiguity as a blocker instead.

## The Process

1. **Hypothesize with a confidence number.** Before asking anything, write your best read of what the user wants in one sentence plus an honest 0–100% confidence. Below ~70%, add a one-line reason for what's missing.
2. **Ask one question at a time, each with a guess attached** (`Q: ... / GUESS: ...`), then wait. One at a time because the third question often depends on the answer to the first, and a guess gets a faster, more honest reaction than an open question would.
3. **Listen for "want vs. should want."** Answers that pattern-match best practice ("scalable", "clean architecture", "the standard approach") without specifics are a sign the user is answering what sounds right, not what they want. Ask: *"If you didn't have to justify this to anyone, what would you actually want?"*
4. **Restate the intent** in the user's words once confidence is high — tight, 5–8 lines: Outcome / User / Why now / Success / Constraint / **Out of scope**. The "out of scope" line is non-negotiable; half of misalignment is silent disagreement about what's *not* being built.
5. **Confirm with an explicit yes.** "Whatever you think," "sounds good," and silence are not yes — they mean re-ask with two concrete options, or ask "anything you'd refine?" Loop until the yes is explicit.

**Stop condition:** you're done when you can predict the user's reaction to the next three questions you'd ask. If several rounds pass with no rising confidence, say so directly: "I still can't predict your reactions — something foundational is missing. Want to step back?"

## Output

A confirmed statement of intent — the restate plus an explicit yes. Specs and plans are downstream and consume this intent. Offer to save it (e.g. `docs/intent/[topic].md`) only if the user confirms.

## Example

```
User: build me a dashboard for our metrics

Agent:
HYPOTHESIS: You want a way to answer "how are we doing?" in standup.
CONFIDENCE: ~30% — missing: who it's for, what "metrics" means, what success looks like

Q: Who's asking — you alone, the team in standup, or up the chain?
GUESS: the team, because "we" usually scopes that way.

User: It's actually for me. I keep losing track of which experiments are running.

Agent: Updated read: a personal experiment tracker, not a team dashboard. ~60%.
Q: Is the gap that you don't know which experiments exist, or can't see results in one place?
GUESS: the second — results live in five tools and reconciling by hand is the pain.

User: First one actually. I don't have a list at all.
```

Two questions in, the real ask was "a list," not "a dashboard" — a different artifact and scope entirely.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "The ask is clear enough" | If you can't state the desired outcome in one sentence right now, it isn't clear. |
| "Asking too many questions wastes their time" | A few targeted questions cost little; building the wrong thing costs a lot, and the user pays for it. |
| "They said 'whatever you think,' so I'll decide" | That's delegation, not confirmation. Re-ask with two concrete options. |
| "If I attach my guess, I'm leading them" | Leading is the point — reacting is faster than generating from scratch. Mitigate by being visibly willing to be wrong. |

## Red Flags

- Three or more questions batched into one message
- A question with no hypothesis attached (surveying, not committing)
- Accepting "whatever you think is best" as a terminal answer
- Producing a spec or plan before the user has explicitly confirmed the restate
- Skipping the "Out of scope" line in the restate

## Verification

- [ ] A hypothesis with a confidence number was stated in the first turn
- [ ] Questions were asked one at a time, each with a guess attached
- [ ] A concrete restate (including Out of scope) was written back and confirmed with an explicit yes
- [ ] At the stop point, the agent could predict reactions to the next three questions
