> Part of the [appco SDLC documentation](README.md) - original section 19.

# 19. Templates

Copy these into `.github/ISSUE_TEMPLATE/` (as Issue Forms), `.github/pull_request_template.md`, or `docs/templates/` as appropriate. **Shrink any template whose sections are routinely answered "N/A"** — see §2.2.

### 19.1 Project proposal / brief

```markdown
# Project Brief: [Name]

**Status:** Draft | Under review | Approved | Rejected | On hold
**Author:** @username
**Date:** YYYY-MM-DD
**Product:** [Product A / Product B / Platform]
**Sponsor:** [Who is asking for this]

# 1. Problem
What is wrong today. Describe the problem, not the solution.

# 2. Who is affected
Which users or roles, and roughly how many.

# 3. Business justification
Why this is worth doing now: revenue, cost, risk, compliance, retention.
Quantify where possible.

# 4. Proposed outcome
What is true after this is delivered, from the user's point of view.

# 5. Success metrics
Within [period], [metric] will move from [baseline] to [target].
1.
2.

# 6. Scope
**In scope:**
**Explicitly out of scope:**

# 7. Constraints
Budget, deadlines, technology, regulatory, contractual.

# 8. Assumptions
Things believed but not verified. Each one is a risk.

# 9. Risks
| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|

# 10. Dependencies
| Dependency | Owner | Status |
|---|---|---|

# 11. Architecture impact
New components, schema changes, integrations, scaling implications.

# 12. Rough sizing
Estimate in sprints, with a confidence level. Not a date.

# 13. Resources required
People, skills, licences, infrastructure cost.

# 14. Alternatives considered
Including "do nothing" and why it was rejected.

# 15. Decision
- [ ] Approved — proceed to requirements
- [ ] Approved with changes: ...
- [ ] Deferred until: ...
- [ ] Rejected because: ...

Decided by: @username on YYYY-MM-DD
```

### 19.2 Epic

```markdown
Title: [Epic] <short name>
Labels: type: epic, priority: P?, area: ?

# Goal
One or two sentences: what this Epic delivers and for whom.

# Business value
Why it matters. Link the project brief if one exists.

# Success metrics
1.
2.

# Scope
**In scope:**
**Out of scope:**

# Child issues
- [ ] org/repo#NN — description
- [ ] org/repo#NN — description

# Business rules
- BR-01:
- BR-02:

# Dependencies
- Blocked by:
- Blocks:

# Design and decisions
- Wireframes:
- API contract:
- ADR-NNNN:

# Deployment notes
Cross-repository ordering, feature flags, migration sequence.

# Definition of Done for this Epic
- [ ] All child issues Done
- [ ] Documentation updated
- [ ] Success metrics instrumented and measurable
- [ ] Stakeholder demo completed
```

### 19.3 User story

```markdown
Title: As a <user type>, I want <goal> so that <value>
Labels: type: story, priority: P?, area: ?
Parent Epic: #NN

# Story
As a <type of user>,
I want <goal>,
so that <business value>.

# Context
Anything the implementer needs that is not obvious. Links to designs,
related issues, prior discussion.

# Acceptance criteria

### AC-1: <happy path>
Given <context>
When <action>
Then <outcome>
And <additional outcome>

### AC-2: <error / edge case>
Given ...
When ...
Then ...

### AC-3: <permission case>
Given ...
When ...
Then ...

# Non-functional requirements
- Performance:
- Security / authorisation:
- Audit / logging:
- Data retention:
- Accessibility:
(Write "none" rather than leaving blank.)

# Out of scope
What this story deliberately does not cover.

# Definition of Ready
- [ ] Goal is understandable
- [ ] Business value stated
- [ ] Acceptance criteria written, including a negative case
- [ ] Dependencies identified
- [ ] Designs available (if UI)
- [ ] Technical approach is clear enough to estimate
- [ ] Fits within one sprint
- [ ] Estimated
```

### 19.4 Technical task

```markdown
Title: <imperative description of the technical change>
Labels: type: task | type: tech-debt, priority: P?, area: ?
Parent Epic: #NN (if applicable)

# Goal
What technical outcome is required and why it is needed now.

# Current situation
How it works today, and what is wrong or missing.

# Proposed approach
Enough detail for another developer to review the plan before code exists.

# Affected repositories and components
- repo — component

# Done criteria
- [ ]
- [ ]

# Risks
What could break. Which existing behaviour must be preserved.

# Testing approach
How we will know it works and that nothing regressed.

# Deployment impact
Config, migrations, ordering, downtime, rollback.
```

### 19.5 Bug report

```markdown
Title: <what is broken, specifically>
Labels: type: bug, priority: P?, area: ?

# Environment
- Environment: Production | Preview | Local
- Version / release: v?.?.? (or commit SHA)
- Browser / device / OS:
- User role:
- Affected user or record IDs:

# Steps to reproduce
1.
2.
3.

# Expected result
What should have happened, and which requirement says so.

# Actual result
What happened instead.

# Evidence
Screenshots, screen recording, error message, stack trace, log excerpt,
request ID, error-tracker link.

# Impact
- How many users affected:
- Is there a workaround:
- Data correctness or money involved:
- Started at (if known):

# Proposed severity
P0 Critical | P1 High | P2 Medium | P3 Low
(The Technical Lead confirms or changes this.)

# Additional context
Recent related changes, whether it is reproducible every time, when it
last worked.
```

### 19.6 Research spike

```markdown
Title: [Spike] <the question to be answered>
Labels: type: spike, priority: P?

# Question
The single specific question this spike answers. If there is more than one,
split it into separate spikes.

# Why we need to know
What decision or estimate is blocked without this answer.

# Timebox
N days. HARD STOP. At the end we write down what we learned, whether or
not we have a complete answer.

# Approach
How the question will be investigated: prototype, documentation review,
vendor conversation, load test.

# Decision this will inform
What we will decide once we know.

# Output required
- [ ] Written answer in this issue (or an ADR if the decision is significant)
- [ ] Recommendation with trade-offs
- [ ] Follow-up issues created
- [ ] Prototype code discarded or explicitly marked throwaway

# Findings
(Filled in at the end — MUST be completed even if the answer is
"we could not determine this within the timebox".)
```

### 19.7 Pull request

```markdown
# Related issue
Closes #

# Summary
What this changes and why. Two or three sentences.

# Changes
-
-

# How this was tested
- Automated:
- Manual:
- Not tested (and why):

# Screenshots / recording
For UI changes. Before and after where useful. N/A otherwise.

# Migration impact
Which migrations run, whether reversible, expected duration on production
data volume. "None" if none.

# Deployment impact
New environment variables, new services, ordering constraints, downtime,
feature flags. "None" if none.

# Security considerations
Authorisation, input validation, secrets, data exposure, new dependencies.
"None" if none.

# Rollback considerations
How to undo this if it misbehaves in production.

# Checklist
- [ ] PR title follows Conventional Commits
- [ ] Linked to an issue
- [ ] Acceptance criteria met
- [ ] Tests added or updated and passing
- [ ] Documentation updated (README, .env.example, API docs, runbook)
- [ ] Self-reviewed the full diff
- [ ] No secrets, debug output, or commented-out code
- [ ] PR is small and focused, or the size is explained above
```

### 19.8 Architecture Decision Record

```markdown
# ADR-NNNN: <short decision title>

**Status:** Proposed | Accepted | Superseded by ADR-NNNN | Deprecated
**Date:** YYYY-MM-DD
**Deciders:** @username, @username
**Consulted:** @username
**Technical story:** #NN

# Context
The situation forcing a decision. Constraints, requirements, and what is
currently true. Written so someone in two years understands the pressure
we were under — including what we did **not** know.

# Decision
We will <decision>, stated in one or two sentences, in the present tense.

# Alternatives considered

### Option A: <name>
- Pros:
- Cons:
- Why rejected:

### Option B: <name>
- Pros:
- Cons:
- Why rejected:

# Consequences

**Positive:**
-

**Negative / accepted costs:**
-

**Risks:**
-

**What would make us revisit this:**
Concrete conditions — a scale threshold, a vendor change, a cost level.

# Implementation notes
Migration path, rollout order, affected repositories.

# References
Links to documentation, benchmarks, spike results, discussions.
```

### 19.9 Release checklist

```markdown
# Release Checklist — v?.?.?

**Release manager:** @username
**Target date/time:**
**Release PR:** #NN

# Scope
- [ ] All included issues are `Ready for Deployment`
- [ ] Items not ready have been moved out of the milestone
- [ ] Release notes drafted and readable by non-developers

# Quality
- [ ] Preview regression pass complete — signed off by @tester on YYYY-MM-DD
- [ ] UAT sign-off from the Product Owner (if applicable)
- [ ] No open Critical or High defects in scope
- [ ] All CI checks green on the release PR
- [ ] Nightly E2E suite passing

# Data
- [ ] Migrations reviewed by the Technical Lead
- [ ] Migration reversibility confirmed (or irreversibility explicitly accepted)
- [ ] Migration duration estimated against production data volume
- [ ] Database backup taken AND restore-verified — timestamp: ______
- [ ] Large backfills separated from the deployment

# Configuration
- [ ] New environment variables set in the production environment
- [ ] Third-party credentials or webhooks configured
- [ ] Feature flags set to their intended initial state

# Operations
- [ ] Rollback plan written for this specific release
- [ ] Deployment window agreed (avoid Fridays and pre-holiday)
- [ ] Someone familiar with the changes is available for 1 hour after
- [ ] Stakeholders notified of timing and expected impact
- [ ] Monitoring dashboards open and baselines noted

# After deployment
- [ ] Health checks passed
- [ ] Smoke tests passed
- [ ] Error rate normal after 30 minutes
- [ ] Key business metric normal (orders, signups, etc.)
- [ ] Release announced
- [ ] Issues moved to `Done`
- [ ] Post-release follow-ups filed
```

### 19.10 Deployment checklist (per deployment)

```markdown
# Deployment — <environment> — <version>

**Deployed by:** @username
**Approved by:** @username (production only)
**Started:** HH:MM UTC
**Workflow run:** <link>

# Pre-deployment
- [ ] Correct version/tag selected
- [ ] Tag is an ancestor of `main` (production only)
- [ ] All required checks green on that commit
- [ ] Backup verified within the last 6 hours (production only)
- [ ] Previous release recorded for rollback: ______
- [ ] Environment variables current

# Deployment
- [ ] Artifact downloaded and checksum verified
- [ ] Release unpacked
- [ ] Dependencies installed
- [ ] Migrations run — duration: ______
- [ ] Release activated (symlink switched)
- [ ] Services reloaded
- [ ] Caches cleared

# Verification
- [ ] Health endpoint returns 200
- [ ] Deployed version confirmed via /api/version
- [ ] Smoke tests passed
- [ ] Manual check of the primary changed feature
- [ ] Error rate normal after 15 minutes
- [ ] Queue workers running
- [ ] Scheduled jobs running

# Completion
- [ ] Team notified
- [ ] Old releases pruned
- [ ] Issues updated to `Done`
- **Finished:** HH:MM UTC — total duration: ______

# If rolled back
- Reason:
- Rolled back to:
- Incident issue: #
```

### 19.11 Production incident

```markdown
Title: [INCIDENT] <short description of user impact>
Labels: type: incident, priority: P0 | P1

# Status
🔴 Investigating | 🟡 Mitigated | 🟢 Resolved

# Impact
- What users cannot do:
- How many users affected:
- Started at: (UTC)
- Detected at: (UTC)
- Detected by: monitoring | user report | team member
- Data or money at risk: yes/no — detail

# Severity
P0 | P1 — set by @username

# Roles
- Incident Lead: @
- Responders: @
- Communicator: @
- Scribe: @

# Timeline
(Append as things happen. Times in UTC. Facts, not interpretation.)
- HH:MM — Alert fired: ...
- HH:MM — Acknowledged by @
- HH:MM — Observed: ...
- HH:MM — Hypothesis: ...
- HH:MM — Action taken: ...
- HH:MM — Result: ...

# Current hypothesis
What we think is happening and what we are doing about it.

# Mitigation
- [ ] Rollback considered — outcome:
- [ ] Feature flag considered — outcome:
- [ ] Hotfix required — PR #
- [ ] Service restored at: HH:MM

# Communication log
- HH:MM — Stakeholders notified by @
- HH:MM — Status update sent
- HH:MM — Resolution announced

# Resolution
What finally fixed it, and how we verified it.

# Follow-up
- [ ] Hotfix back-merged into `nightly` (MUST, within 24 h)
- [ ] Post-incident review scheduled for: YYYY-MM-DD
```

### 19.12 Post-incident review

```markdown
# Post-Incident Review — <incident title>

**Incident:** #NN
**Date of incident:** YYYY-MM-DD
**Review date:** YYYY-MM-DD
**Participants:** @, @, @
**Facilitator:** @

> This review is blameless. We assume everyone acted reasonably given what
> they knew at the time. We are examining the system, not the people.

# Summary
Two or three sentences a non-engineer can understand.

# Impact
| Measure | Value |
|---|---|
| Duration of user impact | |
| Users affected | |
| Failed transactions / requests | |
| Data loss | |
| Revenue impact | |
| Support tickets generated | |

# Timeline
| Time (UTC) | Event |
|---|---|

Key intervals:
- Time to detect (start → alert):
- Time to acknowledge (alert → human):
- Time to mitigate (acknowledge → service restored):
- Time to resolve (start → fully fixed):

# What happened
The narrative, in plain language.

# Contributing factors
Not "the root cause" — usually there are several.
1.
2.
3.

# Detection
- How did we find out?
- Did monitoring catch it? If not, why not?
- Could we have detected it sooner?

# What went well
-

# What was difficult
-

# Where we got lucky
Things that could have made this much worse but did not. These are the most
valuable findings, because next time the luck may not hold.
-

# Action items
| # | Action | Type | Owner | Target sprint | Issue |
|---|---|---|---|---|---|
| 1 | | Prevent / Detect / Mitigate | | | # |

Prefer actions that make the failure impossible or immediately visible over
actions that ask people to be more careful.

# Lessons for the wider team
What should everyone know as a result of this?
```

### 19.13 Sprint planning notes

```markdown
# Sprint NN Planning — YYYY-MM-DD

**Sprint dates:** YYYY-MM-DD → YYYY-MM-DD
**Participants:**
**Facilitator:**

# Sprint Goal
One sentence. What will be true at the end of this sprint.

# Capacity
| Person | Available days | Notes (leave, support duty, other commitments) |
|---|---|---|
| | | |
| **Total available days** | | |
| **Planning capacity** (× 0.6 × 0.8) | | |
| Previous sprint velocity | | |

# Committed items
| Issue | Title | Estimate | Owner | Repo |
|---|---|---|---|---|

**Total estimate:** ___ / capacity ___

# Unplanned-work reserve
___ points / days held back for bugs and support.

# Known risks and dependencies
| Risk / dependency | Impact | Mitigation | Owner |
|---|---|---|---|

# Items deliberately not included
And why — this is useful context when someone asks later.

# Carried over from last sprint
| Issue | Why it did not finish | Still valid? |
|---|---|---|
```

### 19.14 Sprint review notes

```markdown
# Sprint NN Review — YYYY-MM-DD

**Attendees:**
**Sprint Goal:**
**Goal achieved:** Yes | Partly | No — because ...

# Demonstrated
| Issue | Title | Demoed by | Feedback |
|---|---|---|---|

# Completed but not demoed
Internal or technical work worth mentioning.

# Not completed
| Issue | Status | Reason | Plan |
|---|---|---|---|

# Feedback captured
| Feedback | From | Action | New issue |
|---|---|---|---|

# Metrics
- Items committed / completed:
- Points committed / completed:
- Unplanned work:  ___ % of capacity
- Escaped defects from the previous release:
- Production deployments this sprint:

# What's next
The Product Owner's outline of the coming sprint's priorities.
```

### 19.15 Sprint retrospective notes

```markdown
# Sprint NN Retrospective — YYYY-MM-DD

**Attendees:**
**Facilitator:**
**Format:** Keep / Stop / Start (or other)

# Previous actions — reviewed FIRST
| Action | Owner | Status | Notes |
|---|---|---|---|

# Keep — what worked
-

# Stop — what did not work
-

# Start — what to try
-

# Discussion themes
The two or three topics that generated the most conversation, and what
was concluded.

# Actions for next sprint (maximum 3)
| # | Action | Owner | How we will know it worked | Issue |
|---|---|---|---|---|
| 1 | | | | # |

# What can we delete?
Is any part of our process no longer earning its cost?

# Team mood
Optional 1–5 check-in and a one-line note on the trend.
```

