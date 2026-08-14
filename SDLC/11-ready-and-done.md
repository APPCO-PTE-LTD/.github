> Part of the [appco SDLC documentation](README.md) - original section 11.

# 11. Definitions of Ready and Done

These two checklists are the highest-value, lowest-cost artefacts in this document. They are the team's shared answer to "can we start?" and "are we finished?" — the two questions that cause the most rework when left implicit.

**Both are team-owned.** They are reviewed in retrospectives and changed by the team, not imposed. A DoD nobody agreed to is a DoD nobody follows.

### 11.1 Definition of Ready (DoR)

An item **MUST** meet these before it can be pulled into a sprint:

| # | Criterion | How to check |
|---|---|---|
| 1 | **The goal is understandable** | Any team member can restate what is being built and for whom, without asking the author |
| 2 | **Business value is identified** | The "so that…" clause is filled in and is not circular ("so that the feature exists") |
| 3 | **Acceptance criteria exist** | Written, testable, including at least one error/negative case |
| 4 | **Dependencies are identified** | Blocking issues linked; external dependencies named with an owner |
| 5 | **Required designs are available** | For UI work: mockups exist and include empty, loading, and error states |
| 6 | **Technical uncertainty is acceptable** | The team can describe an approach. If not — spike first (§5 Phase 2.3) |
| 7 | **It fits in a sprint** | Estimated at or below the team's split threshold (8 points / size L) |
| 8 | **It is estimated** | The team has agreed a size |
| 9 | **Non-functional needs are noted** | Permissions, performance, audit, retention — or explicitly "none" |
| 10 | **The right people are available** | If it needs the designer or a specific API, that is confirmed |

**Enforcement:** at sprint planning, an item failing the DoR is **not** pulled in. Not "pulled in and clarified later" — that is exactly how a sprint fails.

**Reasonable exception:** `P0` incidents skip the DoR. Nobody writes acceptance criteria while production is down.

### 11.2 Definition of Done (DoD)

An item is **Done** only when **all** of these are true:

| # | Criterion | Verified by |
|---|---|---|
| 1 | **All acceptance criteria are satisfied** | Tester on Preview, against the issue |
| 2 | **Code is implemented and merged into `nightly`** | GitHub |
| 3 | **Automated tests written and passing** | CI green on the PR |
| 4 | **Code review approved** | GitHub PR approval |
| 5 | **All review conversations resolved** | GitHub branch protection |
| 6 | **Documentation updated** | README, `.env.example`, API docs, runbook — whichever the change touched |
| 7 | **Security and permission concerns addressed** | Server-side authorisation verified; no secrets in code; input validated |
| 8 | **Migrations tested and reversible** (or irreversibility explicitly accepted) | Ran on Preview against production-like data |
| 9 | **Deployed to Preview** | `deploy-preview.yml` succeeded |
| 10 | **Validated by the tester on Preview** | Status moved to `Ready for Deployment` |
| 11 | **PO/stakeholder acceptance** (Epic-level features) | Comment on the issue |
| 12 | **No unresolved Critical or High defects** | Bug list for the item is clear |
| 13 | **Monitoring/alerting in place where required** | Metric or alert exists for anything user-facing and failure-prone |
| 14 | **Deployed to Production and verified** | Release tagged; smoke tests passed; author or tester confirmed |
| 15 | **GitHub Issue and Project status updated** | `Status: Done` |

#### Two-stage Done

Criteria 1–13 make an item **`Ready for Deployment`** — finished from the team's perspective, waiting for the next release. Criterion 14–15 make it **`Done`**.

This distinction matters because releases are periodic: an item can be genuinely complete for a week before it ships. **SHOULD:** count `Ready for Deployment` items toward sprint goal completion; report `Done` separately as "shipped".

#### Start smaller

For adoption Phase 1, a six-item DoD is enough:

```markdown
# Definition of Done (v1)
- [ ] Acceptance criteria met
- [ ] Tests written and passing in CI
- [ ] Code reviewed and approved
- [ ] Merged into nightly and deployed to Preview
- [ ] Verified on Preview by someone other than the author
- [ ] Issue status updated
```

Add items 6–14 as the practices become real. A 15-item DoD adopted on day one gets ignored by day three.

#### Where to put it

**MUST:** the DoD is visible where work happens — pinned as an issue in the product repository, included in the PR template as a reminder, and linked from the Project README. A DoD in a document nobody opens is decoration.

