# FEATURE-002-PHASE02 — `/interview` planning-only + code-review intake (v7) — DONE

## Summary

Refactored the `/interview` command (`.claude/commands/interview.md`) from v6 to **v7**: it is now
**planning-only** — the Phase-4 execute-now/stop-after-plans delivery-timing question and the entire build
flow are removed, so interview never implements, executes, or creates a branch; it always writes roadmap
rows + plan files at `TODO` on the user's current branch/HEAD, prints a planning commit message, and points
to `/build`. Added **code-review intake** (infer + confirm → `CODE-REVIEW-NNN`, findings as severity-ordered
phases) and the **CRLF-is-never-a-question** rule to the beyond-the-draft sweep.

## Files/modules touched

- **Modified** `.claude/commands/interview.md`:
  - Version line → `Using interview v7 by Josué Clément`.
  - Frontmatter description, Role → planning-only, "never implements", handles feature/bug/code review.
  - Phase 1: new step 3 "Detect a code review" (infer + confirm → `CODE-REVIEW-NNN`, findings-as-phases,
    severity-ordered); list renumbered 1→7. Beyond-the-draft sweep step 7 now forbids raising CRLF as a
    question (recommendation-only).
  - "What to ask" bullet: dropped the base-branch question (skill always branches from current HEAD).
  - Phase 3 work-item-breakdown line: notes the `CODE-REVIEW-NNN` findings-as-phases case; "branch name(s)"
    → "suggested branch name(s)".
  - Phase 4 rewritten as "Delivery (planning only)": removed the delivery-timing AskUserQuestion and the
    execute-now build flow; planning flow only, on current branch/HEAD, no `git switch -c`; plan-mode note
    updated to match. Trimmed the skill-defers sentence so it no longer implies DoD/sub-agent-delegation
    govern the planning steps.
- **Modified** `docs/roadmap.md`, `docs/plan/FEATURE-002.md` (PHASE02 status flips).

## Deviations & follow-ups

- No deviations from the plan.
- An adversarial verification workflow (4 lenses + independent verify) raised one minor stylistic point
  about the Phase-4 skill-defers sentence; the verify stage refuted it as a non-defect, but I applied a
  small clarity tightening anyway to reinforce the planning-only scope.
- **Follow-up (unchanged from PHASE01, not done):** repo has no `.gitattributes`; `* text=auto eol=lf` would
  prevent CRLF churn. Recommendation only.

## Build/test evidence

No build step or test suite (prompt/config markdown). Verified by:
- **Adversarial workflow** `verify-interview-v7` — 4 independent review lenses (planning-only completeness,
  code-review intake correctness vs the skill, version/internal consistency, CRLF + regressions) plus an
  independent verification pass. Result: **0 confirmed defects** (1 raw finding, refuted on re-read).
- **Consistency grep**: version is exactly `v7` (no `v6`); the only execute/timing matches are intended
  negations; the only `git switch` mention is the "branching belongs to /build" note; Phase 1 numbered
  cleanly 1–7; CODE-REVIEW (3×) and the CRLF rule present.
