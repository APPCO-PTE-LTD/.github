> Part of the [appco SDLC documentation](README.md) - original section 14.

# 14. Documentation and Decision Management

### 14.1 What goes where

**Principle: documentation lives as close to the code it describes as possible.** Documentation in a distant wiki goes stale because updating it is a separate act; documentation in the repository is updated in the same PR as the change.

| Document | Location | Owner | Updated when |
|---|---|---|---|
| **README** | Repo root | Repo maintainers | Any change to setup or purpose |
| **Setup / local development** | `README.md` or `docs/setup.md` | Developers | Any new dependency or step |
| **Environment configuration** | `.env.example` + `docs/configuration.md` | Developers | **Same PR** as any new variable (**MUST**) |
| **Architecture overview** | `docs/architecture.md` | Tech Lead | Structural changes |
| **API documentation** | `docs/api/openapi.yaml` (generated where possible) | Backend developers | Same PR as any endpoint change |
| **Database documentation** | `docs/database.md` + ER diagram | Backend developers | Schema changes |
| **Deployment procedure** | `docs/deployment.md` | DevOps | Pipeline changes |
| **Rollback procedure** | `docs/deployment.md#rollback` | DevOps | Deployment changes |
| **Troubleshooting / runbook** | `docs/runbook.md` | DevOps + developers | After every incident (**SHOULD**) |
| **Coding standards** | `engineering-handbook` | Tech Lead | Rarely; via PR + team discussion |
| **Security procedures** | `engineering-handbook/security.md` + `SECURITY.md` | Tech Lead | Policy changes |
| **Architecture Decision Records** | `docs/adr/NNNN-*.md` | Decision author | Never edited — superseded instead |
| **Release notes** | GitHub Releases | Release manager | Every production release |
| **User documentation** | Product docs site or `<product>-docs` | PO + Designer | User-visible changes |
| **This SDLC** | `engineering-handbook/SDLC.md` | Tech Lead | Quarterly review (§20) |

### 14.2 README standard

Every repository's README **MUST** answer, in this order:

```markdown
# Repository name
One paragraph: what this is and which product it belongs to.

# Stack
Language, framework, database, notable services.

# Getting started
Prerequisites, then copy-paste commands that take a new developer from
clone to running app. If it is more than ~6 commands, provide a script.

# Environment variables
Point to .env.example; explain any variable that is not self-evident.

# Running tests
The exact commands, including the single "run everything CI runs" command.

# Branching and deployment
Feature branches → PR into `nightly` → auto-deploy to Preview →
release PR into `main` → tagged production deploy. Link to the SDLC.

# Architecture
Short overview and a link to docs/architecture.md.

# Related repositories
The other repos in this product and how they connect.

# Who to ask
Team or people responsible.
```

**The test:** a new developer should get the project running locally, unaided, in under an hour. If they cannot, the README is wrong — and fixing it is the new joiner's first PR (**SHOULD**; nobody sees the gaps as clearly as someone hitting them for the first time).

### 14.3 Diagrams as code

**SHOULD** use Mermaid for diagrams. It lives in Markdown, is version-controlled, diffs meaningfully in PRs, and renders natively on GitHub. Binary images from a drawing tool go stale immediately and cannot be reviewed.

Use images only where Mermaid genuinely cannot express it (UI mockups, infrastructure vendor diagrams, screenshots).

### 14.4 Architecture Decision Records

An ADR captures **one decision**: the context, the choice, the alternatives, and the consequences. It is short — one page — and **immutable**. When a decision changes, write a new ADR that supersedes the old one; never edit history, because the point is understanding what was known *at the time*.

**Write an ADR when:**
- The decision is expensive to reverse (data store, framework, auth model, deployment topology, integration partner)
- Two or more credible options were seriously considered
- Someone will ask "why on earth did we do it this way?" within a year
- The decision constrains future work

**Do not write one for:** naming, formatting, library choices with obvious defaults, or anything reversible in an afternoon.

Numbered sequentially, never renumbered: `docs/adr/0007-use-nightly-branch-for-preview.md`. Status one of `Proposed`, `Accepted`, `Superseded by ADR-00XX`, `Deprecated`. Template: §19.8.

**MUST:** an ADR is reviewed as a PR like code, so the decision gets discussed before it becomes fact.

### 14.5 Documentation review

| Rule | Priority |
|---|---|
| Documentation changes go through PRs like code | **MUST** |
| Docs affected by a change are updated in the **same PR** | **MUST** (it is a DoD item) |
| README setup steps are verified whenever someone new joins | **SHOULD** |
| Runbook is updated after every incident | **SHOULD** |
| Quarterly documentation sweep: delete what is wrong | **SHOULD** — wrong documentation is worse than none, because it is trusted |

