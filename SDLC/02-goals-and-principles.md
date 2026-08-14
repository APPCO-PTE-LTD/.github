> Part of the [appco SDLC documentation](README.md) - original section 2.

# 2. SDLC Goals and Principles

These are the principles that justify every specific rule later in the document. When a rule and a principle conflict, the principle wins and the rule gets fixed.

### 2.1 The eleven principles

#### P1 — Deliver work incrementally

Ship the smallest slice that a user can actually use, then the next one. A feature split into five issues delivered over three weeks is better than the same feature delivered as one 4,000-line pull request, because each slice gets reviewed, tested, and validated while it is still small enough to reason about.

*Practical rule:* if an issue cannot plausibly be finished within one sprint, it is an Epic and **MUST** be broken down before it enters a sprint (§5 Phase 5).

#### P2 — Prioritize working software

A running staging environment demonstrating a feature is worth more than a document describing it. We write documents to *reduce risk before building* (§5 Phases 2–4) and to *preserve knowledge after building* (§14) — not as a deliverable in themselves.

*Practical rule:* design artefacts are sized to the risk. A one-line UI copy change gets no design doc. A payments integration gets an ADR, a sequence diagram, and a rollback plan.

#### P3 — Keep requirements traceable

Any line of code in production **SHOULD** be answerable: which PR introduced it, which issue asked for it, which business need justified it, and which release shipped it.

*Practical rule:* every PR links an issue (`Closes #123`). Every issue states its value. Exceptions (typo fixes, dependency bumps) are fine but should be the visible minority.

#### P4 — Review changes before merging

No change reaches a shared branch without a second pair of eyes. This is not about distrust; it is about knowledge spread and catching the class of mistake that authors cannot see in their own work.

*Practical rule:* `main` has branch protection requiring at least one approval (§5 Phase 7). Solo-developer repositories still use PRs — self-merge is permitted only where no second reviewer exists, and the PR still records the change (§7.6).

#### P5 — Automate repeatable processes

If a human does the same sequence of steps more than a handful of times, automate it. Humans are bad at repetition and excellent at judgement; give each the work it is suited to.

*Practical rule:* formatting, linting, type checking, tests, builds, and deployment are automated. Approving a production release is a human decision and stays human.

#### P6 — Test before deployment

Every deployment to a shared environment is preceded by automated verification. The pipeline, not a person's memory, decides whether a change is safe to promote.

*Practical rule:* failing required checks block merge, mechanically (§5 Phase 8).

#### P7 — Separate development, preview, and production

Environments are isolated: separate databases, separate credentials, separate infrastructure. A developer experimenting **MUST NOT** be able to affect production data by accident.

Concretely, we run **two shared environments** (§10): **Preview**, deployed from the `nightly` branch, and **Production**, deployed from `main`. Preview is where testers validate; nothing reaches `main` that has not passed there.

*Practical rule:* production secrets live only in the GitHub `production` Environment, which is access-restricted and approval-gated (§9).

#### P8 — Prefer small and reversible deployments

Deploy often, in small pieces. Small deployments have small blast radius, are diagnosable in minutes, and can be rolled back cleanly. Large quarterly releases concentrate risk into the worst possible moment.

*Practical rule:* every production deployment has a documented rollback path *before* it starts (§5 Phase 9). If a change cannot be rolled back (e.g. a destructive migration), that is called out explicitly in the PR and reviewed with extra care.

#### P9 — Document important decisions

Record decisions that were hard, expensive, or non-obvious — and specifically *why the alternatives were rejected*. Future-us will otherwise repeat the analysis or, worse, undo the decision without knowing the reason.

*Practical rule:* Architecture Decision Records (§14.4) for decisions that are costly to reverse. Not for everything.

#### P10 — Continuously improve through retrospectives

The process described here is a starting point, not a constitution. The retrospective is where it changes. Every retrospective **SHOULD** produce at most 1–3 concrete, owned actions — more than that and none of them happen.

#### P11 — Avoid unnecessary meetings and process overhead

Every ceremony must earn its place. If a meeting has no decision and no output, cancel it. If a field on the board is never filtered on, delete it. If a template section is always "N/A", remove it.

*Practical rule:* the retrospective **SHOULD** periodically ask "what part of our process can we delete?" — not only "what can we add?".

### 2.2 Keeping it lightweight

The single largest failure mode for a team adopting formal process is **process weight exceeding process value**. Symptoms to watch for:

| Symptom | Likely cause | Correction |
|---|---|---|
| Issues are created but never updated | Board has too many statuses, or updating is manual busywork | Cut statuses; add Project automation (§7.8) |
| Estimation takes longer than the task | Team is estimating in false precision | Switch to T-shirt sizes; timebox estimation to 2 min/item |
| PRs sit unreviewed for days | Review is not treated as first-class work | Review-before-new-work rule (§5 Phase 7.4) |
| Templates filled with "N/A" | Template asks for irrelevant things | Shrink the template |
| Meetings overrun consistently | Wrong participants, or no prepared inputs | Enforce timeboxes; move detail to async |
| Nobody reads the docs | Docs are far from the code | Move docs into the repo (§14) |

**Structure without accountability is theatre; accountability without structure is blame.** The goal is a system where the right thing to do next is obvious, and where anyone can see the state of the work without asking anyone.

