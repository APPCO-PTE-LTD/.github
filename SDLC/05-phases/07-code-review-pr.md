> Part of the [appco SDLC documentation](../../README.md) - [phase overview](00-overview.md), Phase 7.

# Phase 7: Code Review and Pull Requests

**Goal:** catch defects early, spread knowledge, and keep the codebase coherent — without becoming a bottleneck.

Code review is **not** a gate where a senior person judges a junior person. It is two people making sure a change is correct and understandable. The reviewer shares responsibility for what gets merged.

#### 7.1 Pull request requirements

Every PR **MUST** contain:

| Element | Why |
|---|---|
| **Title in Conventional Commits format** | It becomes the squash commit message and feeds release notes |
| **Linked issue** (`Closes #142`) | Traceability (P3) |
| **Summary of changes** | The reviewer needs the *intent* before the diff |
| **How it was tested** | Tells the reviewer what is already covered and what to scrutinise |
| **Screenshots / recording** for UI changes | Reviewing UI from a diff is guesswork |
| **Migration impact** | Which migrations run, whether reversible, data volume implications |
| **Deployment impact** | New env vars, new services, ordering constraints, downtime |
| **Security considerations** | Auth, permissions, input handling, secrets, exposed data |
| **Rollback considerations** | How to undo this if it goes wrong in production |

Sections that genuinely do not apply are marked "None" — not deleted, so the reviewer knows the author considered them. Full template in §19.7.

#### 7.2 Size rules

| PR size (changed lines, excl. lockfiles/generated) | Expectation |
|---|---|
| < 200 | Ideal. Review in 10–20 minutes with real attention. |
| 200–400 | Acceptable. |
| 400–800 | **SHOULD** be split. If not possible, explain why in the description and give the reviewer a suggested reading order. |
| > 800 | **SHOULD NOT** exist except for generated code, vendored dependencies, or mechanical refactors — say which in the title. |

Why this matters, plainly: review quality collapses as size grows. A 1,000-line PR does not get 5× the scrutiny of a 200-line PR — it gets *less*, because reviewers skim. Large PRs are approved faster and contain more bugs. Keeping PRs small is the cheapest quality control we have.

Techniques for keeping them small: feature flags for incomplete features; separate refactor PRs from behaviour PRs (**SHOULD** — mixing them makes both unreviewable); API before UI; scaffolding before logic.

#### 7.3 Author responsibilities

Before requesting review — **self-review your own diff on the Files Changed tab.** You will find debug statements, commented-out code, typos, and accidental files. This takes three minutes and saves a review round trip. Then:

- All CI checks green
- PR description complete
- Draft → Ready for review
- Reviewers requested (CODEOWNERS may auto-assign)
- Add inline comments on your own diff wherever a choice needs explaining — pre-empting questions speeds review dramatically

#### 7.4 Reviewer responsibilities and SLA

| Rule | Detail |
|---|---|
| **Timeliness (SHOULD)** | First response within **one working day**. Review is not an interruption of work — it *is* work. |
| **Review before starting new work** | If a PR waits on you, review it before pulling a new issue. |
| **Small PRs first** | Clearing a 50-line PR in 5 minutes unblocks a teammate immediately. |
| **Be specific** | "This is wrong" is not a review comment. Say what and suggest what instead. |
| **Distinguish severity** | Prefix optional comments with `nit:` (style preference) or `question:` (seeking understanding) vs. blocking concerns. This is the single most effective habit for keeping reviews friendly and fast. |
| **Approve when good enough** | Do not block on preference. If the code is correct, tested, and understandable, approve — even if you would have written it differently. |

#### 7.5 Reviewer checklist

**Correctness**
- [ ] Does it do what the linked issue's acceptance criteria say?
- [ ] Edge cases: empty, null, zero, very large, concurrent, duplicate?
- [ ] Are errors handled, and does the user get something useful?

**Tests**
- [ ] Is new logic covered? Does a bug fix have a regression test?
- [ ] Do the tests actually assert behaviour rather than just executing code?

**Security**
- [ ] Is input validated **server-side**?
- [ ] Is authorisation checked on the server for every new action?
- [ ] Any secrets, tokens, or personal data in code, logs, or error messages?
- [ ] Any injection surface (SQL, command, template, XSS)?

**Data**
- [ ] Are migrations reversible? Do they lock large tables?
- [ ] Are indexes present for new query patterns?
- [ ] Is there an N+1 query in a loop?

**Maintainability**
- [ ] Can a new team member understand this in six months?
- [ ] Is it consistent with existing patterns in this repo (or deliberately and explicitly not)?
- [ ] Any duplicated logic that should reuse something existing?
- [ ] Is complexity justified, or is a simpler version available?

**Operations**
- [ ] New config/env vars documented?
- [ ] Anything needing monitoring or an alert?
- [ ] Is rollback possible?

#### 7.6 Approvals, self-review, and drafts

| Situation | Rule |
|---|---|
| **Standard change** | 1 approval (**MUST**) |
| **High-risk change** — auth, payments, migrations, infra, shared libraries, CI/CD workflows | 2 approvals, one from CODEOWNERS/Tech Lead (**MUST**) |
| **Self-merge** | **MUST NOT** when another qualified reviewer exists. Permitted only in single-maintainer repositories, and even then the PR is opened, CI runs, and the description is complete — the record is the point. |
| **Docs-only / typo** | 1 approval; **MAY** be merged by the author after CI if trivial and the repo policy allows |
| **Draft PRs** | Use freely for early feedback and CI. Draft = "not asking for review yet". **MUST NOT** request review while in draft. |
| **Stale approvals** | Dismissed automatically when new commits are pushed (branch protection setting) |
| **Conversation resolution** | All threads **MUST** be resolved before merge. The **reviewer** resolves their own threads where possible; the author may resolve after making the requested change, stating what changed. |
| **Emergency / hotfix** | Expedited path in §12.5 — reduced approvals, never *zero* review, retroactive review within 24 h if a fix was merged solo during an incident |

#### 7.7 Merge strategy

Default **squash and merge**. Reasons: one issue = one commit on `main`, trivially revertable (`git revert <sha>`), clean history, and the PR title (Conventional Commits) generates release notes.

**MUST:** when squashing, check the auto-filled commit message — GitHub concatenates all commit messages into the body by default. Trim it to something meaningful.

#### 7.8 Branch protection rules

Configure on **both `main` and `nightly`** in **every** repository. This is the mechanical enforcement of P4 and P6 — without it, the rules above are suggestions.

| Rule | Setting | Priority |
|---|---|---|
| Require a pull request before merging | On | **MUST** |
| Required approvals | 1 (2 for critical repos) | **MUST** |
| Dismiss stale approvals when new commits are pushed | On | **MUST** |
| Require review from Code Owners | On (where CODEOWNERS exists) | **SHOULD** |
| Require status checks to pass | On; select the CI jobs by name | **MUST** |
| Require branches to be up to date before merging | On for low-traffic repos; consider off for busy repos (causes rebase churn) | **SHOULD** |
| Require conversation resolution before merging | On | **MUST** |
| Block force pushes | On | **MUST** |
| Block deletions | On | **MUST** |
| Require linear history | On if squash-only | **MAY** |
| Require deployments to succeed before merging | Off (we deploy after merge) | **MAY** |
| Require signed commits | Off initially — enable when the team is comfortable with GPG/SSH signing | **MAY** |
| Include administrators | **On** | **SHOULD** |
| Allowed merge methods | Squash only (repo setting) | **SHOULD** |
| Auto-delete head branches | On (repo setting) | **SHOULD** |

Notes:
- *Include administrators*: exempting admins means the rules do not apply under pressure, which is exactly when they matter. Keep it on and use the documented emergency procedure (§12.5) instead of quietly bypassing.
- *Signed commits*: real security value, real onboarding friction. Defer to adoption Phase 5.
- Use **rulesets** (organisation-level) rather than per-repository branch protection once you have more than ~5 repositories — one place to change, consistent everywhere.
- `nightly` and `main` are protected differently: `nightly` takes feature PRs with one approval; `main` accepts **only** release PRs from `nightly` (or `hotfix/*`) and requires the stricter gate. Full settings in §10.4.

