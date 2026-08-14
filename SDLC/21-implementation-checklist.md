> Part of the [appco SDLC documentation](README.md) - original section 21.

# 21. Implementation Checklist

A condensed, ordered checklist for actually putting this in place. Each line is a task someone can own.

### Week 1–3 — Foundation (adoption Phase 1)

**GitHub organization**
- [ ] Set organization base permission to `Read`
- [ ] Create teams: `engineering`, `tech-leads`, `devops`, `qa`, `design`, `product`, `release-approvers`
- [ ] Ensure at least two organization owners
- [ ] Require 2FA for all members
- [ ] Create the `.github` repository with org-wide issue templates, PR template, and `SECURITY.md`
- [ ] Agree and apply the repository naming convention
- [ ] Fill in descriptions and topics on every existing repository

**Per repository**
- [ ] Add `.github/CODEOWNERS`
- [ ] Add `README.md` following the §14.2 standard
- [ ] Add `.env.example` with every variable documented
- [ ] Enable branch protection on `main`: PR required, 1 approval, no force push, no deletion, conversation resolution
- [ ] Set squash-merge only; enable auto-delete of head branches
- [ ] Enable Dependabot alerts, secret scanning, and push protection

**Process**
- [ ] Create one GitHub Project per product; enable auto-add of new issues
- [ ] Configure the six starting statuses and the built-in automation workflows
- [ ] Add Project fields: `Status`, `Priority`, `Work Type`, `Assignee`
- [ ] Migrate all known in-flight work into issues
- [ ] Publish the Definition of Done v1 as a pinned issue
- [ ] Document branch naming and Conventional Commits in each README
- [ ] Add a minimal `pr-validation.yml` (lint + tests) and set it as a required check

### Week 4–10 — Sprint process (adoption Phase 2)

- [ ] Create the `Sprint` iteration field (2 weeks, 6 iterations ahead)
- [ ] Add `Estimate`, `Parent Epic`, `Target Date`, `Product` fields
- [ ] Create the Backlog, Current Sprint, My Work, and Blocked views
- [ ] Schedule the recurring ceremonies with owners and timeboxes
- [ ] Run the first sprint planning; agree a Sprint Goal
- [ ] Introduce T-shirt estimation at refinement
- [ ] Publish the Definition of Ready
- [ ] Run the first retrospective; record 1–3 owned actions
- [ ] Improve issue templates based on what has actually been missing

### Week 8–16 — Deployment automation (adoption Phase 3)

**Branching**
- [ ] Create `nightly` from `main` in every deployable repository
- [ ] Set `nightly` as the default branch
- [ ] Apply the differentiated protection rules from §10.4
- [ ] Add the scheduled `main` → `nightly` sync PR workflow

**Environments**
- [ ] Provision the Preview server/database, mirroring production configuration
- [ ] Create the `preview` GitHub Environment (no approval)
- [ ] Create the `production` GitHub Environment: required reviewers = `release-approvers`, restricted to protected branches/tags
- [ ] Move all deployment credentials into Environment secrets
- [ ] Remove any deployment secrets from repository or organization scope

**Runners**
- [ ] Provision a self-hosted runner per environment with labels `preview` / `production`
- [ ] Create runner groups restricted to specific repositories and workflows
- [ ] Create dedicated unprivileged deploy users with narrow `sudo` rules
- [ ] Verify no deployment workflow triggers on `pull_request`
- [ ] Confirm runners are attached only to private repositories
- [ ] Add runner offline alerting and scheduled disk cleanup

**Pipelines**
- [ ] Create `shared-workflows` with reusable CI workflows, pinned to a tag
- [ ] Add `deploy-preview.yml` — automatic on merge to `nightly`
- [ ] Add `deploy-production.yml` — on Release published, approval-gated, with health checks and auto-rollback
- [ ] Add `maintenance.yml` — scheduled scans, backup verification, health checks
- [ ] Configure Dependabot to target `nightly`
- [ ] Document the release PR ritual and the deployment runbook
- [ ] **Perform a rollback drill on Preview, then on Production**

### Week 12–22 — Quality and observability (adoption Phase 4)

- [ ] Add integration tests for every API endpoint
- [ ] Add E2E tests for critical user journeys; run them after every Preview deploy
- [ ] Adopt the regression-test-with-every-bug-fix rule
- [ ] Adopt the flaky-test quarantine policy
- [ ] Enable CodeQL on all repositories
- [ ] Set up error tracking on Preview and Production
- [ ] Set up uptime, server, database, queue, and cron monitoring
- [ ] Add a business-metric alert (orders/signups dropping to zero)
- [ ] Configure severity-based alert routing; test the alerting path
- [ ] Start recording DORA metrics and cycle time
- [ ] Start the weekly 20-minute production review

### Week 20+ — Maturity (adoption Phase 5)

- [ ] Create `docs/adr/` and write the first ADRs for existing decisions
- [ ] Move from T-shirt sizes to story points; start tracking velocity
- [ ] Introduce explicit capacity planning
- [ ] Expand to the full Definition of Done and the full status set
- [ ] Add the Roadmap and Timeline views for stakeholders
- [ ] Document cross-repository dependency and deployment ordering practice
- [ ] Run the first quarterly SDLC review
- [ ] Write the onboarding guide and test it on the next joiner

### Always

- [ ] Every change has an issue
- [ ] Every issue has acceptance criteria
- [ ] Every PR has a review
- [ ] Every merge to `nightly` deploys to Preview
- [ ] Every production release is tagged, approved, and reversible
- [ ] Every incident gets a blameless review
- [ ] Every retrospective produces at most three owned actions
- [ ] Every quarter, ask what to delete

