> Part of the [appco SDLC documentation](../../README.md) - [phase overview](00-overview.md), Phase 2.

# Phase 2: Planning and Discovery

**Goal:** reduce uncertainty before we spend engineering time. Applies to **Medium and Large** items only — running discovery on a small item is exactly the overhead §2.2 warns against.

#### 2.1 Activities

| Activity | Question it answers | Typical effort |
|---|---|---|
| Stakeholder interviews | What do they actually do today, step by step? | 1–3 conversations, 30–45 min each |
| User problem framing | Whose problem is this, and how do they experience it? | Written in one paragraph |
| Business goals | What outcome does the business want? | From the PO |
| Success metrics | How will we know it worked, 30 days after launch? | 1–3 measurable statements |
| Constraints | Budget, deadline, technology, regulation, existing contracts | Listed explicitly |
| Assumptions | What we believe but have not verified | Listed; each one is a risk |
| Risks | What could make this fail or cost far more than expected | Listed with likelihood/impact |
| Dependencies | Other systems, teams, vendors, procurement | Listed with owners |
| Initial timeline | Rough phases and a realistic range, not a single date | Range with confidence |
| Architecture impact | New services, schema changes, integrations, scaling implications | Tech Lead's assessment |
| Resource requirements | People, skills, licences, infrastructure cost | Listed |

#### 2.2 Writing success metrics

Vague success criteria are the main reason features get built, shipped, and never evaluated. Use this shape:

> Within **[time period]** of release, **[metric]** will **[change]** from **[baseline]** to **[target]**.

Examples:
- Within 30 days, the share of subscription changes handled without support involvement rises from 0% to at least 70%.
- Within 60 days, average time to upgrade a plan drops from ~2 working days to under 5 minutes.
- Within 30 days, support tickets tagged `billing-change` fall by at least half.

**SHOULD:** every Epic has at least one success metric. **MUST:** every Epic with a development cost above roughly one sprint has one.

#### 2.3 When to build a proof of concept, prototype, or spike

| Type | Purpose | Timebox | Output | Discard code? |
|---|---|---|---|---|
| **Technical spike** | Answer a specific technical question ("can library X handle our load?") | 1–3 days, **hard stop** | A written answer in the issue + a recommendation | Yes, always |
| **Proof of concept (PoC)** | Prove an approach is feasible end-to-end | 3–5 days | Working throwaway demo + decision | Yes — **MUST NOT** be promoted to production |
| **Prototype (UX)** | Validate a user flow before building it | 1–5 days | Clickable design; user feedback | N/A (design artefact) |

Trigger a spike when:
- The team cannot estimate an item because of unknowns ("we don't know if this is 2 days or 3 weeks").
- A third-party API's behaviour is undocumented or unverified.
- Performance or scale is a genuine open question.
- Two credible architectural options exist and the choice is expensive to reverse.

**MUST:** every spike is an issue with a written *question* and a *timebox*, and it ends with a written answer as a comment or an ADR. A spike that ends with "we ran out of time and learned nothing" is a process failure — split the question smaller next time.

#### 2.4 Output of Phase 2

- Updated **Project Brief** / Epic description containing goals, metrics, constraints, assumptions, risks, dependencies
- A **go / no-go / defer** decision from the PO, recorded as a comment on the Epic
- Rough sizing (in sprints, not days)
- List of items requiring design (Phase 4) or requirements detail (Phase 3)
- Any spike results or ADRs

