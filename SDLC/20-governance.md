> Part of the [appco SDLC documentation](README.md) - original section 20.

# 20. Governance and Review

### 20.1 Who owns this process

| Aspect | Owner |
|---|---|
| **This document** | Technical Lead (final approver) |
| **Day-to-day facilitation** | Scrum Facilitator / Project Manager |
| **Backlog and priority process** | Product Owner |
| **CI/CD, environments, runners, secrets** | DevOps / Infrastructure Owner |
| **Testing strategy** | QA (with the Technical Lead) |
| **Adherence and coaching** | Everyone; the Scrum Facilitator notices drift first |

**Ownership means accountability for the process being fit for purpose — not authority to impose it unilaterally.** Changes are discussed with the team.

### 20.2 Proposing a change

This document lives in `appco/engineering-handbook` and is changed exactly like code:

1. Open an issue labelled `type: process` describing the problem with the current process — the *problem*, not just the proposed rule.
2. Discuss it, ideally at a retrospective where the friction was felt.
3. Open a PR editing this document.
4. At least two approvals, one being the Technical Lead. Changes affecting deployment or security also need DevOps approval.
5. Merge, then announce in the team channel with a one-line summary of what changed and why.
6. Bump the version and date at the top of the document.

**Anyone may propose a change.** The most valuable proposals usually come from whoever is most inconvenienced by the current rule, and from new joiners who have not yet learned to work around the rough edges.

### 20.3 Review cadence

| Review | Frequency | Participants | Scope |
|---|---|---|---|
| **Lightweight process check** | Every retrospective | Delivery team | Is anything in the process actively getting in the way? |
| **Full SDLC review** | **Every 3 months during early adoption** (first year), then every 6 months | Tech Lead, PO, Facilitator, DevOps, QA, 1–2 developers | The whole document |
| **Post-incident triggered review** | After any `P0` | Incident participants | The specific parts that failed |
| **Adoption phase gate** | End of each adoption phase (§16) | Whole team | Readiness for the next phase |

**Full review agenda (90 minutes):**
1. Metrics trend since last review (§15) — 15 min
2. Which rules were bypassed, and why — 20 min *(a rule bypassed repeatedly is a bad rule, not a discipline problem)*
3. Which rules earn their cost — 15 min
4. **What can we delete?** — 15 min
5. What is missing — 15 min
6. Agree changes and owners — 10 min

### 20.4 Versioning this document

| Change | Version bump | Announcement |
|---|---|---|
| Typo, clarification, example | PATCH (1.0.1) | None |
| New guidance, new template, expanded section | MINOR (1.1.0) | Team channel |
| Changed mandatory rule, new phase, changed branching/deployment model | MAJOR (2.0.0) | Team meeting + channel; a transition period where both are acceptable |

Keep a changelog at the end of the document (§20.7).

### 20.5 Exceptions

Not every situation fits. Exceptions are fine — **undocumented exceptions are not**, because they are indistinguishable from the process quietly eroding.

To take an exception:
1. State it where the work is (issue or PR comment): which rule, why, what the risk is, and how it is mitigated.
2. Get agreement from the rule's owner (§20.1). Security and deployment exceptions **MUST** be approved by the Technical Lead.
3. Label the issue/PR `process-exception`.
4. Review exceptions at the quarterly review — **if the same exception recurs, change the rule.**

**Never require an exception for:** the emergency incident path (§12), which is already part of the process. Emergencies do not need permission; they need a defined route, and they have one.

### 20.6 Onboarding new team members

| When | What |
|---|---|
| **Before day 1** | GitHub org invite, team membership, repository access, tooling accounts |
| **Day 1** | Read §1, §2, §5 Phase 6, §11 of this document (~45 min). Meet the team. Get one repository running locally |
| **Day 1–2** | **First PR: fix whatever was wrong or unclear in the setup docs.** This teaches the whole workflow on a change with no risk — and the docs genuinely improve, because nobody else can see those gaps any more |
| **Week 1** | Pair on a small `good-first-issue`. Attend all ceremonies as an observer. Read the product `-docs` overview and the ADRs |
| **Week 2** | Own a small issue end to end. Do a code review with a buddy |
| **Week 3–4** | Normal sprint participation. Read §6–§10 for the GitHub and deployment detail |
| **Month 2** | Shadow a production release. Read an incident post-mortem |
| **Ongoing** | Assigned a buddy for the first month — the named person for "stupid" questions |

**MUST:** every new joiner is assigned a buddy on day one. **SHOULD:** they are invited to propose a change to this document within their first month, while the process still looks strange to them — that perspective is worth more than it feels.

### 20.7 Document changelog

| Version | Date | Change | Author |
|---|---|---|---|
| 1.0 | 2026-08-04 | Initial proposal | Engineering Leadership |

