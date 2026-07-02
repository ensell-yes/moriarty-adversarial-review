# The Moriarty Adversarial Review Loop

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

### Three ways to install it

You rarely paste the harness by hand every time — instead, have your **coding agent set it up once**
in one (or all) of three forms, then invoke it per review:

- **Global** — a user-level instruction that applies in *every* project on your machine.
- **Rule** — a project-scoped file committed to the repo, so the whole team's agent uses the same
  rubric.
- **Skill** — a named, on-demand capability you invoke only when you want a review, instead of an
  always-on instruction.

[Using the harness in your tools](#using-the-harness-in-your-tools) below gives a copy-paste **setup
prompt** for Claude Code, Codex, GitHub Copilot, Cursor, and Gemini that builds these files for you
from `moriarty-adversarial-review-harness.md`.

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

### Sample loop prompts

Concrete prompts for each turn. They are tool-agnostic — paste them into whichever session holds
that role. (These assume the harness is already loaded as the reviewer's instructions; if it isn't,
prepend *"Using the rubric in `moriarty-adversarial-review-harness.md`, …"*.)

**Reviewer turn** (steps 2 and 4):

```text
Review the artifact below against the adversarial review harness. Work all eight
dimensions; for each, give findings in Behavior / Evidence / Fix / Test form and a
score with its severity. Then give the overall as `sum / 8 = X`.

Gate: the artifact is approved only when the overall is >= 9.2 AND no single
dimension is below 9.2. If the bar isn't met, list the highest-value improvements
and name the binding dimension(s). Do not approve below the bar.

ARTIFACT:
<paste the artifact>

SOURCES OF TRUTH (cross-check against these):
<paste the spec / code / tests, or links>
```

**Author turn** (step 3):

```text
Below are review findings on the artifact you produced. Address each one ONLY if it
is valid and valuable: fix what's real, and for anything wrong or low-value, push
back with a one-line reason instead of changing it. Then output (a) the revised
artifact and (b) a short change summary mapping each finding to what you did.

FINDINGS:
<paste the reviewer's findings>
```

Hand the revised artifact and change summary back to the reviewer and repeat until the gate is met.

### Why 9.2

9.2 is the bottom of the harness's **"Clean"** band — a dimension only reaches ≥ 9.2 when it has
**no unresolved finding** — so "every dimension ≥ 9.2" means every dimension is genuinely clean,
not merely "mostly fine." The bar is set here because, in practice, quality gains flatten out past
this mark: further looping tends to produce churn rather than material improvement. Raise or lower
it to match how much you're willing to spend chasing the last increment.

### Tuning the dimensions

The eight dimensions are a strong default, not a fixed law — but tune them *deliberately*, never by
silently skipping one. The invariant that keeps a review from going blind is **cover every dimension
every time**, so within the canonical rubric a dimension that doesn't apply is marked **N/A with a
reason** (e.g. Performance on a pure-prose spec) — not dropped. To adapt to a usage context you may
**add** a context-specific dimension (e.g. a "Data integrity" dimension for a data-migration plan)
or deliberately **fork** the rubric into a named variant that redefines the set up front and then
covers *its* dimensions every time. Whichever set you use, keep the scoring mechanics (severity →
band, overall = mean, weakest dimension binds the gate) and adjust the *bar* to the artifact at
hand.

## Using the harness in your tools

In every tool the pattern is the same: load `moriarty-adversarial-review-harness.md` as the
reviewer's standing instructions, then assign the **author** and **reviewer** roles to *different
models* so the loop above has real independence. Each tool below maps the three install forms from
[How to use it](#how-to-use-it) to that tool's files and includes a copy-paste **setup prompt** that
has your coding agent build them from the harness.

> **Harness source.** The setup prompts read the rubric from the in-repo file
> `moriarty-adversarial-review-harness.md`. If it isn't in your working repo, point the agent at the
> raw copy instead:
> `https://raw.githubusercontent.com/ensell-yes/moriarty-adversarial-review/main/moriarty-adversarial-review-harness.md`
> If your agent cannot fetch URLs, download the harness into the repo first and have the agent read
> that local file.

> **Data boundary.** The loop pastes the artifact *and its sources of truth* — which may include
> private code, secrets, or customer data — into one or more external services. Use only models and
> tools approved for that artifact's confidentiality level; never paste secrets or regulated data
> into an unapproved service; and check each provider's retention and admin-access policy first.

> **Verify against current docs.** The setup paths below reflect each vendor's surface as of this
> writing; these conventions (file locations, feature availability, UI labels) change often. Confirm
> the exact path against the tool's current documentation — the *mechanism* (load the harness as the
> reviewer's instructions; run author and reviewer on different models) holds even when a filename
> moves.
>
> **Length-capped fields.** File-based installs can include the harness verbatim. UI text fields, such
> as Copilot personal custom instructions or a Gemini Gem's Instructions field, may have length caps;
> if the full harness does not fit, attach it as a file/knowledge source or point the instruction at a
> local copy or the raw URL instead of pasting the whole text.

### Claude Code

- **Global:** a user-level skill at `~/.claude/skills/adversarial-review/SKILL.md`, and/or a pointer
  in `~/.claude/CLAUDE.md` ("use the adversarial-review skill when reviewing any artifact").
- **Rule:** a project `CLAUDE.md` (or `.claude/CLAUDE.md`) at the repo root that references the
  skill, so the whole team's Claude Code uses the same rubric.
- **Skill:** `.claude/skills/adversarial-review/SKILL.md` — frontmatter `name: adversarial-review`
  plus a one-line `description`, then the harness body. Invoke it on the reviewer's turn.
- **Two models:** define a reviewer subagent in `.claude/agents/reviewer.md` with a different
  `model:`, or run two sessions and switch with `/model`.

```text
Read ./moriarty-adversarial-review-harness.md (or a local downloaded copy; fetch the raw URL above
only if your environment supports web fetch).
Create a Claude Code skill at .claude/skills/adversarial-review/SKILL.md: YAML frontmatter with
name: adversarial-review and a one-line description, then paste the harness body verbatim as the
skill content. Also add to the project CLAUDE.md: "When asked to review, audit, or critique any
artifact, use the adversarial-review skill." Include the harness verbatim — do not summarize it.
```

### Codex

- **Global:** `~/.codex/AGENTS.md` (applies to all Codex sessions), and/or a user skill at
  `~/.agents/skills/adversarial-review/SKILL.md`.
- **Rule:** `AGENTS.md` at the repo root (committed project instructions).
- **Skill:** a Codex repo skill at `.agents/skills/adversarial-review/SKILL.md` — current Codex docs
  list repo skills under `.agents/skills` and user skills under `~/.agents/skills`; a custom prompt
  under `~/.codex/prompts/review.md` is a legacy on-demand fallback.
- **Two models:** run author and reviewer as separate Codex sessions with different `--model` values
  (or two config profiles).

```text
Read ./moriarty-adversarial-review-harness.md (or a local downloaded copy; fetch the raw URL above
only if your environment supports web fetch).
Add an "Adversarial review" section to AGENTS.md at the repo root: "When asked to review, audit, or
critique any artifact, apply this rubric," followed by the harness body verbatim. Also create a
Codex repo skill at .agents/skills/adversarial-review/SKILL.md with name/description frontmatter
and the harness body as its content. Keep the harness text verbatim. If your installed Codex version
documents a different skill path, use the current documented path instead.
```

### Cursor

- **Global:** a **User Rule** (Cursor Settings → Rules → User Rules) — applies in every project.
- **Rule:** `.cursor/rules/adversarial-review.mdc` (committed project rule); set `alwaysApply: true`
  for always-on.
- **Skill (≈ on-demand rule):** the same `.mdc` with `alwaysApply: false`, set to Agent-Requested or
  Manual, so it loads only when you invoke a review.
- **Two models:** use Cursor's **model picker** — author in one chat/model, review in a separate chat
  with a different model.

```text
Read ./moriarty-adversarial-review-harness.md (or a local downloaded copy; fetch the raw URL above
only if your environment supports web fetch).
Create .cursor/rules/adversarial-review.mdc: MDC frontmatter with a description and
alwaysApply: false (Agent-Requested), then the harness body verbatim as the rule content. The rule
instructs: when reviewing, auditing, or critiquing an artifact, apply this rubric and score it.
Keep the harness verbatim.
```

### GitHub Copilot

- **Global:** your **personal custom instructions** (GitHub → Copilot settings, or the IDE's
  user-level Copilot instructions) — applies across all your repos; if the UI field is length-capped,
  use a short pointer to a local file or raw URL instead of pasting the whole harness.
- **Rule:** `.github/copilot-instructions.md` (repo-wide), or a path-scoped
  `.github/instructions/review.instructions.md`.
- **Skill (≈ prompt file):** `.github/prompts/review.prompt.md` — a reusable prompt run from Copilot
  Chat (IDE-only — VS Code, Visual Studio, JetBrains — and in public preview; confirm availability
  for your client).
- **Two models:** use the Copilot Chat **model picker** — author with one model, review with another
  in a separate chat.

```text
Read ./moriarty-adversarial-review-harness.md (or a local downloaded copy; fetch the raw URL above
only if your environment supports web fetch).
Create .github/prompts/review.prompt.md: frontmatter with a description and mode: agent, a body line
"Review the provided artifact against the rubric below and score it," then the harness body verbatim.
Also create .github/copilot-instructions.md noting that artifact reviews follow this rubric. Include
the harness text verbatim, not a summary.
```

### Gemini

- **Global:** in the **Gemini app**, a **Gem** is available across all your chats; for the **Gemini
  CLI**, `~/.gemini/GEMINI.md` is global context.
- **Rule:** for the CLI, `GEMINI.md` at the repo root (committed project context).
- **Skill (≈ a Gem):** in the app, **Gems → New Gem** ("Explore Gems" / Gem manager), name it
  "Adversarial Reviewer", paste the harness into **Instructions** if it fits, and otherwise attach it
  as **knowledge** or point the Instructions field at a local copy or raw URL.
- **Two models:** use the Gem as the reviewer and author in standard Gemini (or another tool) so the
  roles run on independent contexts.

```text
Read ./moriarty-adversarial-review-harness.md (or a local downloaded copy; fetch the raw URL above
only if your environment supports web fetch).
For the Gemini CLI: create GEMINI.md at the repo root stating that artifact reviews follow this
rubric, with the harness body included verbatim. (For the Gemini app instead: create a Gem named
"Adversarial Reviewer" and paste the harness as its Instructions if it fits; otherwise attach it as
knowledge or point the Instructions field at a local copy or raw URL.) Keep the harness verbatim in
file-based installs.
```

## License

[CC0 1.0 Universal](./LICENSE) — this work is dedicated to the public domain. You may copy,
modify, distribute, and use it, including for commercial purposes, all without asking permission
or providing attribution.
