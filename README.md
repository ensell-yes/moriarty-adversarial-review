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

## The two-model review loop

The harness is most effective when the **author** and the **reviewer** are *different models in
separate sessions*. Independence is the point: a model reviewing its own output shares its own
blind spots and tends to rationalize rather than refute. Two different models — with a fresh
reviewer context that sees only the artifact and its sources of truth, not the author's reasoning —
surface defects a self-review misses.

Run the loop by passing text between the two sessions:

1. **Author model — produce.** Write (or revise) the artifact. Copy it.
2. **Reviewer model — review and score.** Paste the harness, the artifact, and its sources of truth
   (spec, code, tests). The reviewer works all eight dimensions, emits findings in the
   `Behavior / Evidence / Fix / Test` format, scores each dimension by its worst unresolved finding,
   then computes the overall (arithmetic mean of the eight). **If the overall is below 9.2 — or any
   single dimension is below 9.2 — it lists the improvements needed.** Copy the findings.
3. **Author model — address.** Paste the findings. Address each one *only if it is valid and
   valuable* — evaluate the finding, fix what's real, and push back (with reasons) on anything wrong
   or low-value. Produce the revised artifact and copy a short summary of what changed and why.
4. **Reviewer model — re-review.** Paste the revised artifact and the change summary, then **loop
   from step 2.** Continue until the score clears the bar: **overall ≥ 9.2 and no single dimension
   below 9.2** (the weakest dimension binds the gate).

### Why 9.2

9.2 is the bottom of the harness's **"Clean"** band — a dimension only reaches ≥ 9.2 when it has
**no unresolved finding** — so "every dimension ≥ 9.2" means every dimension is genuinely clean,
not merely "mostly fine." The bar is set here because, in practice, quality gains flatten out past
this mark: further looping tends to produce churn rather than material improvement. Raise or lower
it to match how much you're willing to spend chasing the last increment.

### Tuning the dimensions

The eight dimensions are a sensible default, not a fixed law. **Swap, drop, or add dimensions to
fit the usage context** — e.g. a pure-prose spec may not need Performance, while a data-migration
plan might add a dedicated "Data integrity" dimension. Keep the scoring mechanics (severity → band,
overall = mean, weakest dimension binds the gate) and adjust the dimension *set* and the *bar* to
the artifact at hand.

## Using the harness in your tools

In every tool the pattern is the same: load `moriarty-adversarial-review-harness.md` as the
reviewer's standing instructions, then assign the **author** and **reviewer** roles to *different
models* so the loop above has real independence.

### Claude Code

- Save the harness as a **Skill** — `.claude/skills/adversarial-review/SKILL.md` (paste the harness
  body under a short `name`/`description` frontmatter) — or as a slash command in
  `.claude/commands/review.md`. Invoke it on the reviewer's turn.
- For two models, define a **reviewer subagent** in `.claude/agents/reviewer.md` with a different
  `model:` in its frontmatter, or run two sessions and switch with `/model`. Hand the artifact to
  the subagent; it returns findings.

### Codex

- Put the harness in **`AGENTS.md`** at the repo root (or a scoped subdirectory) so it loads as
  standing instructions, or save it as a custom prompt under `~/.codex/prompts/review.md` and call
  it on demand.
- For two models, run the author and reviewer as separate Codex sessions with different `--model`
  values (or two config profiles).

### Cursor

- Add the harness as a **project rule**: `.cursor/rules/adversarial-review.mdc` (set it to apply on
  request and invoke it in the review chat). A User Rule works too if you want it across all
  projects.
- For two models, use Cursor's **model picker**: author in one chat/model, open a separate chat,
  switch the model, and run the review there.

### GitHub Copilot

- Add the harness as **repository custom instructions** in `.github/copilot-instructions.md`, or as
  a reusable **prompt file** at `.github/prompts/review.prompt.md` that you run in Copilot Chat.
- For two models, use the Copilot Chat **model picker** to author with one model and review with
  another in a separate chat.

### Gemini (as a Gem)

- In the Gemini app, open the **Gem manager → New Gem**. Name it (e.g. "Adversarial Reviewer") and
  **paste the harness into the Instructions field**; optionally attach the harness file (and your
  spec/standards) as **knowledge**. Save it.
- Use the Gem as your **reviewer**, and author in standard Gemini (or another tool/model) so the two
  roles run on independent contexts. Paste the artifact into the Gem to get scored findings.

## License

[CC0 1.0 Universal](./LICENSE) — this work is dedicated to the public domain. You may copy,
modify, distribute, and use it, including for commercial purposes, all without asking permission
or providing attribution.
