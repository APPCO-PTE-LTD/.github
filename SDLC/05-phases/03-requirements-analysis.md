> Part of the [appco SDLC documentation](../../README.md) - [phase overview](00-overview.md), Phase 3.

# Phase 3: Requirements Analysis

**Goal:** describe *what the system must do* precisely enough that a developer can build it and a tester can verify it, without describing *how* to build it.

#### 3.1 Functional requirements

What the system does: features, screens, rules, calculations, workflows, states, permissions.

Expressed primarily as **user stories with acceptance criteria** (§3.3–3.4), supplemented by:

| Element | When to add |
|---|---|
| **Use case / flow description** | Multi-step processes with alternate paths (checkout, onboarding, approval chains) |
| **Business rules** | Any rule that is a business decision rather than a technical one — list them numbered, e.g. `BR-01: Downgrades take effect at the end of the current billing period.` |
| **State machine** | An entity with meaningful states (subscription: `trial → active → past_due → cancelled`). Draw it (§17) |
| **Permission matrix** | Any feature where different roles see or do different things |
| **Validation rules** | Per field: required, format, length, range, uniqueness, cross-field rules |
| **Error cases** | What the user sees and what the system does for each failure mode |

#### 3.2 Non-functional requirements (NFRs)

These get forgotten and then cause production incidents. **MUST** be considered for every Medium/Large item; write "not applicable" explicitly rather than leaving blank.

| Category | Questions to answer | Example requirement |
|---|---|---|
| **Performance** | Expected load? Acceptable response time? Data volume growth? | "Plan list page loads in < 500 ms p95 with 10,000 subscriptions" |
| **Security** | Who may access this? What data is sensitive? Injection/XSS/CSRF surface? Rate limiting? | "Only the store owner may view billing data; enforced server-side, verified by an automated test" |
| **Authorisation** | Exact role → permission mapping | "`store_owner` may change plan; `store_staff` may view only" |
| **Audit & logging** | What must be recorded, with what fields, kept how long? | "Every plan change writes an audit record: actor, old plan, new plan, timestamp, IP" |
| **Data retention** | How long is data kept? Deletion or anonymisation rules? Legal basis? | "Invoices retained 7 years; payment tokens never stored locally" |
| **Reporting** | What must be queryable/exportable, by whom, at what freshness? | "Monthly revenue-by-plan export, CSV, admin only" |
| **Availability** | Uptime target, maintenance windows, degradation behaviour | "Billing UI may be read-only during provider outage; must not error" |
| **Compatibility** | Browsers, devices, OS versions, API versions | "Last 2 versions of Chrome/Safari/Firefox/Edge; iOS 16+; Android 10+" |
| **Localisation** | Languages, currencies, time zones, date formats | "Amounts displayed in the store's currency; times in the store's time zone" |
| **Accessibility** | Standard to meet | "Keyboard navigable; WCAG 2.1 AA contrast" |
| **Integration** | External systems, protocols, failure behaviour, idempotency, retries | "Webhook handler is idempotent on `event_id`; retries with exponential backoff up to 24 h" |
| **Monitoring** | What must be observable after release | "Alert if failed plan changes exceed 5 in 10 minutes" |

#### 3.3 User story format

**MUST** use this format for user-facing work:

> **As a** [type of user], **I want** [goal], **so that** [business value].

Rules:
- The "so that" clause is not decoration. If it cannot be written, the value is unclear and the item is not ready.
- Write from the user's perspective, not the system's. ✗ "As a system, I want to store the plan" — that is a technical task, not a story.
- One story = one user-visible outcome. If it contains "and", consider splitting.
- A story **SHOULD** be completable within a single sprint by one or two people.

**Splitting patterns** when a story is too big:

| Pattern | Example |
|---|---|
| By workflow step | Separate "view current plan" from "change plan" from "see change history" |
| By rule complexity | Happy path first; proration and edge-case rules second |
| By user role | Owner first, then staff, then admin |
| By data variation | One payment method first, then the rest |
| By interface | API first, then UI |
| By quality attribute | Working first, then optimised/paginated |

#### 3.4 Acceptance criteria

Acceptance criteria are **the contract**: the developer knows when to stop, QA knows what to test, the PO knows what to accept.

**Given–When–Then** (**SHOULD** for behavioural rules):

```
Given the store owner has an active "Basic" subscription
When they select the "Pro" plan and confirm the upgrade
Then the subscription becomes "Pro" immediately
And a prorated charge for the remaining period is created
And a confirmation email is sent to the billing contact
And an audit record of the change is written
```

**Checklist form** (acceptable for simple or structural items):

```
- [ ] The plan selection page lists all active public plans
- [ ] The current plan is visually marked as current
- [ ] Plans unavailable in the store's country are hidden
- [ ] Page is usable on mobile widths from 360 px
```

Quality bar for acceptance criteria — each one **MUST** be:

| Property | Test |
|---|---|
| Testable | Someone other than the author can verify pass/fail without asking questions |
| Unambiguous | No "fast", "user-friendly", "properly", "as needed" |
| Complete | Covers the failure path, not just the happy path |
| Independent of implementation | Says *what*, not *which function* |

**MUST:** every story includes at least one negative/error criterion (what happens when it fails, when permission is missing, when input is invalid).

#### 3.5 Work item types and their templates

| Type | Label | Template | Key contents |
|---|---|---|---|
| Epic | `type: epic` | §19.2 | Goal, value, success metrics, child issues, out of scope |
| User story | `type: story` | §19.3 | Story sentence, acceptance criteria, design links, NFRs |
| Technical task | `type: task` | §19.4 | Technical goal, approach, affected repos, done criteria |
| Bug | `type: bug` | §19.5 | Steps, expected, actual, environment, severity, evidence |
| Spike | `type: spike` | §19.6 | Question, timebox, method, decision to be made |
| Tech debt | `type: tech-debt` | §19.4 variant | Current problem, cost of keeping it, proposed change |
| Incident | `type: incident` | §19.11 | Impact, timeline, severity, mitigation |

Every issue **MUST** carry: one `type:` label, one `priority:` label, and be added to the product's GitHub Project.

#### 3.6 Output of Phase 3

- Epic with child stories in GitHub Issues
- Each story with acceptance criteria and any relevant NFRs
- Business rules listed and numbered
- Permission matrix where roles differ
- Open questions listed and assigned to a person for answer
- Items marked `Status: Ready for Refinement`

