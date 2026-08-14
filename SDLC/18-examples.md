> Part of the [appco SDLC documentation](README.md) - original section 18.

# 18. Practical Examples

### 18.1 Example A — A feature, end to end

> **"Allow store owners to manage their subscription plan."**

#### Step 1 — The idea arrives

The Head of Customer Success files a Feature Request in `appco/product-a-docs`:

```markdown
### Problem
Store owners cannot change their subscription plan themselves. Every upgrade
or downgrade is an email to support, who change it manually in the admin panel.

### Business justification
~40 requests/month, ~15 minutes each = ~10 hours/month of support time.
More importantly, owners who want to upgrade wait up to 2 working days,
and some abandon the upgrade entirely.

### Expected users
All store owners (currently 1,240 active stores). Roughly 3% change plan monthly.

### Business value
Recover ~10 h/month of support capacity; remove friction from upgrades
(the revenue-positive direction); reduce manual data-entry errors.

### Requested priority
High

### Known risks
Billing errors are visible to customers and damage trust. Proration must be
correct. Payment provider integration is involved.

### Dependencies
Payment provider API (Stripe). Finance must agree the proration policy.

### Rough scope
In scope: view current plan, upgrade, downgrade, see change history.
Out of scope: cancellation, refunds, custom/enterprise plans, currency changes.

### Deadline
No hard deadline. Preferred before the Q4 pricing update in November.
```

PO triage: not a duplicate, clear value, fits product direction, **Large** → Epic + discovery.
Fields set: `Priority: P1 High`, `Work Type: Feature`, `Status: Needs Analysis`, `Product: Product A`.

#### Step 2 — Discovery output (summarised)

| Item | Result |
|---|---|
| **Business goal** | Self-service plan management; recover support capacity; remove upgrade friction |
| **Success metrics** | (1) ≥ 70% of plan changes self-service within 30 days. (2) Median upgrade time from ~2 days to < 5 min. (3) `billing-change` support tickets down ≥ 50% in 60 days |
| **Constraints** | Must use the existing Stripe integration; proration policy fixed by Finance; no downtime |
| **Assumptions** | Stripe subscription API supports mid-period changes with proration (**verified by spike**); all active stores have a valid payment method (**false — 4% do not**, so this became a requirement) |
| **Risks** | Incorrect billing (High); webhook failures leaving state inconsistent (Medium); users downgrading and losing data (Medium) |
| **Spike** | 2 days: "Can Stripe handle immediate upgrade with proration and scheduled downgrade?" → **Yes**, using subscription update with `proration_behavior: create_prorations` for upgrades and a scheduled phase for downgrades. Written up as ADR-0012 |
| **Rough size** | ~1.5 sprints |

#### Step 3 — The Epic

```markdown
Title: [Epic] Subscription plan self-management
Labels: type: epic, priority: P1, area: billing
Project fields: Product = Product A, Work Type = Feature,
                Start Date = 2026-08-17, Target Date = 2026-09-11

# Goal
Store owners can view, upgrade, and downgrade their subscription plan
without contacting support.

# Business value
Recovers ~10 h/month of support time and removes a 2-day delay from the
revenue-positive upgrade path.

# Success metrics
1. ≥ 70% of plan changes are self-service within 30 days of release
2. Median upgrade completion time < 5 minutes
3. `billing-change` support tickets down ≥ 50% within 60 days

# Out of scope
Cancellation, refunds, enterprise/custom plans, currency changes,
payment method management (already exists).

# Child issues
- [ ] product-a-backend#142 — API: change subscription plan
- [ ] product-a-backend#143 — Proration calculation and business rules
- [ ] product-a-backend#144 — Stripe webhook handling for plan changes
- [ ] product-a-backend#145 — Plan change audit log
- [ ] product-a-frontend#88 — Plan comparison and selection UI
- [ ] product-a-frontend#89 — Change confirmation and result flow
- [ ] product-a-frontend#90 — Plan change history view
- [ ] product-a-docs#12 — Help article: changing your plan

# Business rules
- BR-01: Upgrades take effect immediately with a prorated charge
- BR-02: Downgrades take effect at the end of the current billing period
- BR-03: A store with a failed payment cannot change plan until resolved
- BR-04: A store may change plan at most 3 times per billing period
- BR-05: Downgrading below current usage limits is blocked with a clear message

# ADRs
- ADR-0012: Proration strategy for subscription plan changes
```

#### Step 4 — A user story with acceptance criteria

```markdown
Title: As a store owner, I want to upgrade my subscription plan so that I get
       access to higher limits immediately
Repository: appco/product-a-backend
Issue: #142
Labels: type: story, priority: P1, area: billing
Parent Epic: #140
Estimate: 5

# Story
As a store owner,
I want to upgrade my subscription plan immediately,
so that I can use the higher limits without waiting for support.

# Acceptance criteria

### AC-1: Successful upgrade
Given I am a store owner with an active "Basic" plan
And I have a valid payment method
When I select the "Pro" plan and confirm
Then my subscription changes to "Pro" immediately
And I am charged a prorated amount for the remaining period
And I receive a confirmation email
And an audit record is written with actor, old plan, new plan, timestamp, IP

### AC-2: Failed payment
Given I am a store owner with an expired payment method
When I attempt to upgrade
Then the upgrade is rejected
And I see: "We could not process the payment. Please update your payment method."
And my current plan is unchanged
And no audit record of a successful change is written

### AC-3: Permission
Given I am store staff, not the owner
When I open the subscription page
Then I can see the current plan
But the change controls are not available
And a direct POST to the plan change endpoint returns 403

### AC-4: Rate limit (BR-04)
Given I have already changed plan 3 times this billing period
When I attempt another change
Then the request is rejected with a clear explanation and the date I may retry

# Non-functional requirements
- Response time < 2 s p95 (external payment provider call included)
- Authorisation enforced server-side, verified by an automated test
- Audit record retained 7 years
- All state transitions logged with the Stripe event ID for reconciliation
- Idempotent: a repeated request with the same idempotency key must not double-charge

# Dependencies
- Blocked by: #143 (proration calculation)
- Related: appco/product-a-frontend#88

# Design
- API contract: docs/api/openapi.yaml#/subscriptions/plan
- ADR-0012: Proration strategy
```

#### Step 5 — Technical tasks

| Issue | Repo | Title | Estimate |
|---|---|---|---|
| #142 | backend | API: change subscription plan | 5 |
| #143 | backend | Proration calculation service + unit tests | 3 |
| #144 | backend | Stripe webhook handler (idempotent) | 3 |
| #145 | backend | Plan change audit log + migration | 2 |
| #88 | frontend | Plan comparison and selection UI | 5 |
| #89 | frontend | Confirmation and result flow | 3 |
| #90 | frontend | Plan change history view | 2 |
| #12 | docs | Help article | 1 |

**Deployment order recorded on the Epic:** backend first (the endpoint is additive and unused until the frontend ships), then frontend. Both can ride the same release; if split, backend must go first.

#### Step 6 — GitHub Project fields for #142

| Field | Value |
|---|---|
| Status | `Ready for Sprint` |
| Priority | `P1 High` |
| Work Type | `Feature` |
| Product | `Product A` |
| Repository | `product-a-backend` (automatic) |
| Sprint | `Sprint 12 (Aug 17–28)` |
| Estimate | `5` |
| Assignee | Backend developer |
| Parent Epic | `#140` |
| Start Date | `2026-08-17` |
| Target Date | `2026-08-21` |
| Milestone | `v1.5.0` |
| Release | `v1.5.0` |
| Risk | `High` (money) |
| Blocked | `No` |

#### Step 7 — Development

```bash
git checkout nightly && git pull origin nightly
git checkout -b feature/142-subscription-plan-change
```

Commits along the way:

```
feat(billing): add subscription plan change endpoint

POST /api/subscriptions/{id}/plan. Upgrades apply immediately with
proration; downgrades are scheduled to the period end per BR-02.
Requires the store-owner role; enforced by SubscriptionPolicy.

Refs #142
```

```
test(billing): cover plan change permission and rate limit cases

Adds feature tests for AC-2, AC-3 and AC-4 including the 403 for
store staff and the 3-changes-per-period limit.

Refs #142
```

```
docs(api): document the plan change endpoint

Refs #142
```

#### Step 8 — Pull request

```markdown
Title: feat(billing): add subscription plan change endpoint

# Related issue
Closes #142

# Summary
Adds `POST /api/subscriptions/{id}/plan`, allowing a store owner to change
their subscription plan. Upgrades apply immediately with proration (BR-01);
downgrades are scheduled for the end of the billing period (BR-02).
Authorisation is enforced by `SubscriptionPolicy`; the endpoint is
idempotent on the `Idempotency-Key` header.

# Changes
- `SubscriptionPlanController::update` — new endpoint
- `PlanChangeService` — orchestration, calls `ProrationCalculator` (#143)
- `SubscriptionPolicy::changePlan` — store-owner only
- Migration: `plan_change_logs` table (additive, reversible)
- OpenAPI contract updated

# How this was tested
- Unit: proration for 12 upgrade/downgrade combinations
- Feature: all four acceptance criteria, including 403 for staff and the
  rate limit
- Manual: full upgrade against the Stripe sandbox on my machine
- Not yet tested: concurrent double-submit — covered by the idempotency key,
  and a load test is tracked in #148

# Screenshots
N/A — backend only.

# Migration impact
Adds `plan_change_logs`. Additive and reversible. ~0.1 s on current data.

# Deployment impact
New environment variable `BILLING_WEBHOOK_SECRET` — added to `.env.example`
and to the Preview and Production environment secrets. **Must be set before
this deploys**, or webhook verification will fail.

# Security considerations
- Authorisation server-side via policy; verified by test AC-3
- Rate limited to 3 changes per billing period (BR-04)
- Stripe webhook signature verified
- No card data touches our servers; only the Stripe customer ID is stored
- Audit log records actor, old plan, new plan, timestamp, IP

# Rollback considerations
Redeploy the previous tag. The migration is additive so it can stay; the
`plan_change_logs` table is simply unused by the old code. No data loss.

# Checklist
- [x] Acceptance criteria met
- [x] Tests added and passing
- [x] Documentation updated
- [x] Self-reviewed the diff
- [x] No secrets committed
```

#### Step 9 — Automated checks

| Check | Result |
|---|---|
| Format | ✅ 4 s |
| Lint | ✅ 18 s |
| Type check | ✅ 22 s |
| Unit tests | ✅ 142 passed, 1 min 4 s |
| Integration tests | ✅ 38 passed, 2 min 11 s |
| Build | ✅ 48 s |
| Dependency review | ✅ no new vulnerabilities |
| CodeQL | ✅ no findings |
| **Total** | **✅ 5 min 12 s** |

Review: Tech Lead required (CODEOWNERS covers `/app/Policies/`). Two comments — one `nit:` on a variable name, one blocking: *"AC-4 counts changes per calendar month, but BR-04 says per billing period. These differ for stores that signed up mid-month."* Genuine bug, caught in review. Fixed, re-reviewed, approved.

#### Step 10 — Merge, Preview, QA

Squash merged into `nightly`. `deploy-preview.yml` runs: build (2 min) → deploy (1 min) → migrations → smoke tests → E2E. Status auto-moves to `Ready for QA`; team chat notified.

Tester works through the acceptance criteria on `https://preview.product-a.appco.com`:

| AC | Result |
|---|---|
| AC-1 upgrade | ✅ prorated charge correct against the Stripe sandbox |
| AC-2 failed payment | ✅ correct message, plan unchanged |
| AC-3 permission | ✅ 403 confirmed via direct API call |
| AC-4 rate limit | ⚠️ error message says "3 times per month" — inconsistent with BR-04 wording |
| Exploratory | Found: browser back button after confirmation re-submits the form |

The wording issue is fixed on the same issue (it never left the sprint). The back-button issue is filed as a new bug (#151, `P2`) because it is a separate frontend concern. Both fixes land on `nightly` and redeploy to Preview automatically. Re-tested → **pass** → `Ready for Deployment`.

#### Step 11 — Release

At the end of Sprint 12, seven items are `Ready for Deployment`. The release PR is opened:

```markdown
Title: release: v1.5.0
Base: main   ←   Head: nightly

# Included
- #142 Plan change API endpoint
- #143 Proration calculation
- #144 Stripe webhook handling
- #145 Plan change audit log
- #88 Plan selection UI
- #89 Confirmation flow
- #151 Fix back-button resubmission

# Release checklist
- [x] Preview regression pass complete — signed off by @tester 2026-08-28
- [x] UAT sign-off from @product-owner
- [x] No open Critical/High defects
- [x] Migrations reviewed — additive, reversible
- [x] Backup taken and restore-verified 2026-08-28 09:14 UTC
- [x] BILLING_WEBHOOK_SECRET set in the production environment
- [x] Rollback plan: redeploy v1.4.3; migration is additive and can remain
- [x] Stakeholders notified — release window Tue 2026-09-01, 09:30
- [x] @backend-dev available for 1 h after deployment
```

Approved by the Tech Lead and PO → merged with a merge commit → tag `v1.5.0` → GitHub Release published with generated notes.

`deploy-production.yml` starts, rebuilds from the tag, re-runs the full suite, packages the artifact, and **pauses**: *"Deployment to production is waiting for approval from @appco/release-approvers."*

The approver checks the release checklist and approves. Deployment: backup verified → artifact downloaded and checksum-verified → unpacked → migrations → symlink switched → service reloaded → health check ✅ → smoke tests ✅ → team notified. Elapsed: 4 minutes 20 seconds.

#### Step 12 — Monitoring and follow-up

First 30 minutes: error rate flat, latency on the new endpoint 340 ms p95 (target < 2 s), 3 real plan changes succeeded.

Follow-up items filed:

| Issue | Type | Priority | Source |
|---|---|---|---|
| #152 Load test the plan change endpoint | `type: task` | P2 | Deferred from PR review |
| #153 Reconcile Stripe webhook events nightly | `type: task` | P2 | Design review |
| #154 Analytics event for plan change funnel | `type: task` | P2 | Success metric measurement |
| #155 Support the 4% of stores with no payment method | `type: story` | P1 | Discovery assumption proved false |

**30-day metric review:** 78% of plan changes self-service (target 70% ✅); median upgrade time 3 min 40 s (target < 5 min ✅); `billing-change` tickets down 61% (target 50% ✅). Recorded on the Epic, which is then closed.

---

### 18.2 Example B — A normal bug

> **"Downgrades charge the full new-plan price instead of prorating."**

| Step | What happens |
|---|---|
| **1. Report** | Support files a bug: a store downgraded from Pro to Basic and was charged the full Basic price immediately instead of at the period end. Two stores affected, ~£40 overcharged |
| **2. Triage** | Tech Lead reproduces on Preview. Money is involved and users are affected, but the workaround (manual refund) exists and the volume is small → **`P1` High**, not `P0`. Assigned to the current sprint using the unplanned-work reserve |
| **3. Issue** | `type: bug`, `priority: P1`, `area: billing`, linked to #143. Steps, expected, actual, store IDs, Stripe event IDs, screenshots of both invoices |
| **4. Branch** | `git checkout nightly && git pull` → `git checkout -b fix/187-downgrade-proration` |
| **5. Root cause** | `ProrationCalculator` used the period **start** date rather than the change date when computing the credit. Grep of callers shows the same helper is used by the scheduled-change path — so the bug affects both, not just the reported one. **Fixed in the shared helper, not at the call site** |
| **6. Regression test** | `test(billing): downgrade prorates from the change date, not the period start` — written first, confirmed failing, then fixed |
| **7. Commit** | `fix(billing): prorate downgrades from the change date` + body explaining the miscalculation + `Closes #187` |
| **8. PR** | Small (31 lines changed). Reviewed within 2 h. Tech Lead required (billing area). Approved |
| **9. Preview** | Merged into `nightly`, auto-deployed. Tester verifies both the reported case and the scheduled-change path |
| **10. Release** | Ships in the next sprint release, `v1.5.1`. Not urgent enough for a hotfix — the two affected stores are refunded manually the same day |
| **11. Follow-up** | Retro notes that proration lacked property-based tests. Issue #190 filed: "Add table-driven proration tests across all plan-pair and date combinations" |

**What this example demonstrates:** the fix went to the *shared* function after checking every caller, not to the path the ticket named. The reported symptom was one of two affected paths.

---

### 18.3 Example C — An emergency hotfix

> **"Checkout returns 500 for all users."** — 14:32, Tuesday

| Time | Action |
|---|---|
| **14:32** | Sentry alert: `TypeError: Cannot read property 'id' of null` in `CheckoutController`, 340 events in 4 minutes. Uptime monitor still green — the health endpoint does not touch checkout |
| **14:33** | Backend developer acknowledges in the team channel: *"I'm on it."* |
| **14:34** | Tech Lead becomes **Incident Lead**, declares **`P0`**, opens incident issue #203 (`type: incident`, `priority: P0`) and starts the timeline. PO begins stakeholder comms |
| **14:36** | **Triage:** all checkouts failing. Last production deploy was `v1.5.0` at 09:30 — five hours earlier, so a deploy is not the obvious cause. Error started at 14:28 |
| **14:38** | Correlation found: a payment-provider webhook at 14:27 set `payment_method` to `null` for customers whose card was removed provider-side. The checkout path assumed it was never null |
| **14:40** | **Mitigation decision.** Rollback would not help — the bug is latent in `v1.5.0` and older. No feature flag. → **Hotfix required** |
| **14:41** | `git checkout main && git pull && git checkout -b hotfix/203-checkout-null-payment-method` |
| **14:52** | Fix: null guard in the checkout path with a clear user message ("Please add a payment method"), plus a regression test. 14 lines changed. Nothing else touched |
| **14:55** | PR into `main`, labelled `hotfix`. CI runs — **not skipped** |
| **15:01** | CI green. Expedited review by another senior developer (1 approval per §12.5): *"Fixes it, cannot break anything else. Approved."* |
| **15:03** | Merged. Tagged `v1.5.1`. GitHub Release published |
| **15:04** | `deploy-production.yml` runs; pauses for approval. Incident Lead approves immediately |
| **15:08** | Deployed. Health checks ✅, smoke tests ✅ |
| **15:10** | Verified: error rate returns to zero; two test checkouts succeed |
| **15:15** | PO announces resolution to stakeholders. **Total user impact: 43 minutes, ~60 failed checkouts** |
| **15:40** | Monitoring stable for 30 minutes → incident issue closed |
| **15:45** | **Back-merge:** `git checkout nightly && git merge main && git push` — the fix is now in the integration branch and cannot be lost by the next release |
| **Next day** | Blameless post-incident review |

**Post-incident review outcomes:**

| Finding | Action | Owner |
|---|---|---|
| Health check did not cover checkout — we learned from Sentry, not from monitoring | Extend the health endpoint into a synthetic checkout probe running every 5 min | DevOps |
| Nullable provider fields were not defensively handled anywhere in the payment path | Audit every payment-path field for nullability; add tests | Backend dev |
| No E2E test covered "customer with no payment method" | Add that fixture to the E2E suite | QA |
| Stakeholders were told at 14:34 but not updated until 15:15 | Incident template now requires a status update every 30 min for `P0` | Scrum Facilitator |
| Detection took 4 minutes, response was good | Keep — noted as what went well | — |

**No action item says "be more careful."** Every one changes the system so the same failure is caught automatically (§12.8).

