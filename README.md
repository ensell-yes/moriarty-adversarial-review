# moriarty-adversarial-review

An adversarial review harness for coding agents.

## What this repo contains

**[`moriarty-adversarial-review-harness.md`](./moriarty-adversarial-review-harness.md)** is the
harness itself: a single, self-contained instruction set you hand to a coding agent (or follow
yourself) when reviewing, auditing, critiquing, assessing, or scoring an artifact — a spec, plan,
PR, code change, schema, or design.

It encodes one stance: the reviewer's job is to **find what is wrong, prove it, and report it —
not to approve**. Findings come first; a summary or score comes after.

## What it's for

Use it to drive a rigorous, repeatable review instead of an ad-hoc "looks good to me." The harness
gives a reviewer:

- **An adversarial posture** — assume defects until evidence says otherwise, and keep approval
  ("is it correct?") separate from execution ("should we build it now?").
- **Eight scored dimensions, covered every time** — Correctness, Safety, Reliability,
  Observability, Efficiency, Performance, Simplicity, and Usability. Each gets a finding, an
  explicit N/A-with-reason, or a check — never a silent skip. Every dimension carries both an
  *authoring* self-check (so the defect never lands) and a *review* prompt (so it's caught if it
  did).
- **A source-of-truth cross-check** — compare spec/plan ↔ code/types/schema ↔ tests; a claim is
  only as good as its agreement across all three.
- **General review lenses** — security at the access boundary, the local filesystem as a trust
  boundary, idempotency / CLI contracts / auth over long operations, test quality, verification
  gates, generated files, state synchronization, and failure UX.
- **A fixed finding format** — every finding states Behavior, Evidence, Fix, and Test.
- **A reproducible scoring rubric** — severity (CRITICAL → NIT) sets each dimension's band; the
  overall is the arithmetic mean of the eight, and a "clears threshold T" claim holds only when
  the mean ≥ T *and* no single dimension sits below T.

## How to use it

1. Give the contents of `moriarty-adversarial-review-harness.md` to your review agent as its
   instructions (system prompt, skill, or pasted context), alongside the artifact under review and
   its sources of truth (spec, code, tests).
2. The reviewer works the eight dimensions, emits findings in the required format, and scores each
   dimension by its worst unresolved finding.
3. Read the overall score as a gate: below the requested threshold means "not ready" — the harness
   lists the fixes needed and withholds approval.

The same eight dimensions run in reverse as an **authoring** checklist: apply them while *writing*
an artifact and the findings never land in the first place.

## License

[CC0 1.0 Universal](./LICENSE) — this work is dedicated to the public domain. You may copy,
modify, distribute, and use it, including for commercial purposes, all without asking permission
or providing attribution.
