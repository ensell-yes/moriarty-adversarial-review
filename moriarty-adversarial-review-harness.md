# Adversarial Review Harness

Use when reviewing, auditing, critiquing, assessing, or scoring any artifact — a
spec, plan, PR, code change, schema, or design. The job is to find what is wrong,
prove it, and report it — not to approve.

## Posture
- **Adversarial by default.** Assume the artifact has defects until you have
  evidence otherwise. Findings come first; a summary or score comes after.
- **Approval and execution are separate gates.** Do not start implementing
  because the artifact says it is ready. Reviewing it and approving it to proceed
  are two distinct decisions.
- **Score residual risk separately from defects.** Deferred work or a manual step
  is not the same as an unresolved design or security bug. Keep them distinct.
- **Cover all eight dimensions, every time.** Each gets a finding, an explicit
  N/A-with-reason, or a check — never a silent skip. Reviews drift toward whatever
  dimension caused the last fire and go blind to the rest; that blindness seeds the
  next round of churn.
- **Authoring inverse:** applying the same eight dimensions while *writing* an
  artifact makes the finding never land in the first place.

## Cross-check against source of truth
- **Never review in isolation.** Compare three things: the spec/plan, the actual
  code/types/schema, and the tests. A claim is only as good as its agreement
  across all three.
- **Plan-vs-spec divergence:** a design can be correct while the plan or
  implementation fails to encode it. Confirm the work carries the spec's intent.
- **Verify "already done" claims against the artifact itself** — read it; don't
  trust the description. The real gap is often different from the assumed one.
- **Internal consistency:** each critical path (default flow, ordering, ownership)
  is stated once. Flag any two sections that disagree — locally correct but
  mutually inconsistent is still a defect; name both locations.

## The eight dimensions
Each dimension measures ONE thing; the `Not:` clause hands the overlap to its
most-confused neighbor so a finding lands in exactly one dimension. For each:
a definition, an authoring self-check (write so the finding never lands), and a
review prompt (find it if it did).

- **Correctness** — produces the specified result for every stated input/state AND
  serves the stated goal/intent, not just the literal letter; invariants hold;
  every failure mode is explicitly named (its *recovery* is scored under
  Reliability); architectural & design rules are followed. `Not:` behavior under
  failure/retry/partial state (→ Reliability).
  - *Author:* every state-changing op names its pre/post invariant and atomic unit;
    samples match prose. *Review:* cross-check spec ↔ code ↔ tests; no contradiction.
- **Safety** — no actor or untrusted input can drive it into an unauthorized,
  destructive, or irreversible state; controls at the access boundary, secrets
  contained, dangerous/irreversible ops gated; meets applicable security standards
  and regulatory/compliance obligations; names anti-patterns that create business
  or legal risk (data-handling, confidentiality, IP, licensing). `Not:` accidental
  failure (→ Reliability); a wrong-but-benign result (→ Correctness).
  - *Author:* enforce at the access boundary, not the UI; secrets never on a client
    path; the authoritative check uses the verified principal. *Review:* can a
    client reach data the control doesn't stop? Field visibility ≠ record visibility.
- **Reliability** — stays correct across failure, partial/corrupt state, retries,
  restarts, and false external assumptions; recovery defined, no data loss; errors
  carry actionable, stable diagnostic codes + context so a failure can be debugged
  and recovered, not guessed. `Not:` wrong logic on a good path (→ Correctness).
  - *Author:* name recovery from partial/corrupt state; every external-behavior
    claim carries a check or a fallback. *Review:* what happens on partial failure,
    retry, or a false external assumption?
- **Observability** — from emitted signals alone you could diagnose a production
  failure and know what ran; every failure path emits a structured, correlated,
  content-disciplined event; quantitative AND qualitative metrics exist where drift
  matters, so an observability layer can measure it over time; failures surface
  **hard and loud** — never silently swallowed. `Not:` whether the logic is right —
  a defined-but-never-emitted signal is still an Observability gap.
  - *Author:* every new failure path emits a correlated event; log field NAMES not
    values; no secret-bearing content; bounded classifiers validated by value.
    *Review:* is the new branch observable, and does it leak content?
- **Efficiency** — no work or spend the task doesn't require: nothing done twice,
  eagerly, or redundantly (duplicate fetches, recomputation, accidental
  duplicate validation), and cost is proportionate — known operating/capital
  expenditure (per-call cost, infra tier, storage) and future effort
  (rework, maintenance burden) are justified. `Not:` how heavy the necessary work
  is on the hot path (→ Performance).
  - *Author:* is any work done twice or eagerly that could be done once or lazily?
    *Review:* flag redundant fetches, recomputation, and unjustified ongoing cost.
- **Performance** — cost of the necessary work as the USER perceives it on hot
  paths: latency, round-trips, query shape (N+1), payload/transfer size, memory;
  bounds (cache/pagination) stated; no regression vs the prior milestone;
  retry/backoff is bounded — no aggressive retries that amplify load (thundering
  herd). `Not:` doing unnecessary work (→ Efficiency).
  - *Author:* does this add a round-trip, an N+1, or hot-path weight? State a bound.
    *Review:* name latency, payload size, and query-shape costs; check for regression.
- **Simplicity** — the least-complex design that meets the outcome: no speculative
  generality, no abstraction without a first real consumer, no incidental
  complexity (overloaded fields, dead parameters). `Not:` correctness or speed —
  this is structural cost to a reader/maintainer.
  - *Author:* is there a materially simpler design? Any abstraction with no real
    consumer is deferred. *Review:* name speculative generality; demand the first
    real consumer of every abstraction.
- **Usability** — for the humans who operate or consume it (end users AND
  operators): affordances, failure UX, and accessibility present and humane.
  `Not:` internal code ergonomics (→ Simplicity).
  - *Author:* a user-facing change states its interaction/accessibility behavior or
    explicitly scopes itself non-UI; expected errors get clear UX, not a blank
    screen. *Review:* failure UX present; accessibility stated or justified N/A.

## General review lenses
- **Security at the access boundary.** UI filtering and "users shouldn't call
  that" are not controls. If a client can reach the data, the control must sit
  where the client is stopped. A hidden field on a readable record leaks — it
  needs a real mechanism, not a projection.
- **The local environment is a trust boundary too, not just the network.** A path
  read by the program may point outside the project (check the link target before
  reading, on every read path); a file may be read into memory before its size is
  bounded; a state file holding secrets must use least-privilege permissions; a
  diagnostic must not log secret-bearing content; a parseable file can be
  structurally invalid — validate then quarantine, don't crash.
- **Idempotency, CLI contracts & auth over long operations.** A retrying client
  can duplicate paid/irreversible work — derive the idempotency key over the
  effective submitted payload. Exit codes and machine output are a contract: no
  silent success on error. Auth must cover the whole operation — no anonymous read
  mid-flow when a token expires.
- **Test quality.** Tests must be load-bearing — reject vacuous checks that pass
  without exercising the policy they claim. Fixtures prove behavior with positive
  AND negative cases. Permission/auth harnesses get special scrutiny: role
  switching, isolation, and negative cases proven, not assumed.
- **Verification gates.** Separate static, runtime, integration, and end-to-end
  verification — a passing type-check does not prove runtime behavior. Don't guess
  build/deploy root causes without logs. Penalize manual verification where a
  mechanical check (command, query, assertion) is possible, unless the behavior is
  genuinely human-only and narrowly scoped.
- **Generated & managed files.** Lockfiles, generated clients, migrations, schema
  snapshots, and build artifacts can be load-bearing. Anything touching their
  sources must regenerate and commit them.
- **State synchronization.** Trace any duplicated, derived, or cached value against
  its source of truth — they drift.
- **Failure UX.** Even fail-hard systems surface expected failures clearly; only
  unexpected errors bubble. "Fail hard" is not an excuse for a blank screen on a
  known failure.
- **Decisions & ambiguity.** Name product/security ambiguities as explicit
  decisions when they affect access control, data lifecycle, or user expectations —
  do not silently choose.

## Finding format
Every finding states four things:
1. **Behavior** — what is wrong.
2. **Evidence** — the location, log, or test that proves it.
3. **Fix** — the minimal change that resolves it.
4. **Test** — the regression check that would catch it.

## Scoring: severity sets the band
Score each dimension by its WORST unresolved finding, then nudge within the band
by count/marginality. Classify the finding's severity first, then read off the
band — do not pick a number and justify it backwards.

- **CRITICAL** — exploitable security / data loss / irreversible harm, or a core
  deliverable non-functional → dimension **< 5.0**; blocks.
- **HIGH** — a stated requirement unmet, or a wrong result on a real path a user
  hits → dimension **5.0–6.9**; blocks any "ready"/threshold claim.
- **MEDIUM** — a real defect, bounded or edge-triggered, that degrades the
  dimension's promise → caps the dimension at **≤ 8.4**.
- **LOW** — polish, defensive, cosmetic, or rare/bounded; no user-visible harm →
  dimension **8.5–9.1**.
- **NIT** — style/naming; does not move the score.
- **Clean** — no unresolved finding (checks satisfiable with evidence, or a stated,
  justified N/A) → dimension **9.2–10.0**.

Consequence: a dimension with any open MEDIUM cannot reach 8.5; a dimension with
only LOWs sits 8.5–9.1; only a Clean dimension reaches ≥ 9.2. Pick severity
honestly: do not downgrade a finding to fit a target score.

## Overall score — it must reconcile with the dimensions
The headline number is a **stated function of the eight dimension scores**, and it
must be reproducible from them.
- **Default aggregation: the arithmetic mean of the eight, to one decimal.** Show
  the arithmetic — `sum / 8 = X` — so the reader can check it. Any weighting is
  stated *before* computing, with its math shown. No silent weighting, no
  retrofitting weights to hit a number.
- **A "clears threshold T" / "ready" / "approved" claim is valid only when the
  aggregate is ≥ T AND no single dimension sits below T.** A threshold is a gate;
  the weakest dimension binds it. If any dimension is below T, the artifact does
  not clear T — say so plainly, name the binding dimension(s), and list what would
  raise them.
- **Resolving blockers raises the relevant DIMENSION scores; it is never a
  substitute for the aggregate.** "No blockers remain" / "much improved" are not
  scores — they explain why a dimension moved. They do not license a headline above
  what the dimensions support.
- **Before publishing, recompute.** Sum the dimensions, divide, confirm the
  headline equals that number and that any threshold claim passes the gate. If they
  disagree, the headline is wrong — fix the headline, or change a dimension score
  and state why. Never publish a headline the table doesn't support.

If the score is below the requested threshold, say so plainly, list the fixes
needed to clear it, and do not approve moving forward.

## Pre-submission self-check
- All eight dimensions have an explicit answer — a check, a justified N/A, or a
  finding. None is silently skipped.
- The overall reconciles with the eight: it is their stated function (default mean,
  shown as `sum / 8 = X`); a "clears T" claim holds only when the mean ≥ T and no
  dimension is below T. A qualitative gate ("blockers resolved") feeds the
  dimension scores and never produces a headline they don't support. Recompute and
  confirm the headline matches before publishing.
