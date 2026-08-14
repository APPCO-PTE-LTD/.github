> Part of the [appco SDLC documentation](README.md) - original section 17.

# 17. Diagram Reference

Every diagram in this document is Mermaid, so it lives in version control, renders on GitHub, and can be reviewed in a pull request like code (§14.3).

### 17.1 Index

| # | Diagram | Location |
|---|---|---|
| **A** | Complete SDLC flowchart | §17.2 (phase summary also in §5.0) |
| **B** | Agile Scrum sprint cycle | §17.3 |
| **C** | GitHub Issue-to-Production workflow | §17.4 (compact version in §1.3) |
| **D** | GitHub Organization and repository structure | [§6.13](#613-how-multiple-repositories-connect-to-one-project) |
| **E** | CI/CD pipeline | [§8.2](#82-pipeline-overview) |
| **F** | GitHub Project status flow | [§7.4](#74-status-flow-and-wip-limits) |
| **G** | Environment promotion | [§9.2 of Phase 9](#92-environment-promotion) |
| **H** | Incident and hotfix workflow | [§12.4](#124-production-incident-flow) |
| **I** | Responsibility / communication flow | [§4.4](#44-communication-flow) |
| **J** | Example roadmap / Gantt | §17.5 (adoption roadmap Gantt in §16.1) |
| — | Work breakdown hierarchy | §5 Phase 5.1 |
| — | Hybrid Scrum-Kanban intake | §3.4 |
| — | Testing pyramid | §5 Phase 8.1 |
| — | Triage decision flow | §5 Phase 1.3 |
| — | QA verification loop | §5 Phase 8.5 |
| — | Release process | §5 Phase 9.3 |
| — | Runner and network architecture | §9.5 |
| — | Branching model (gitGraph) | §10.2 |
| — | Improvement loop | §15.5 |

### 17.2 Diagram A — Complete SDLC flowchart

```mermaid
flowchart TD
    subgraph INTAKE["1. Intake"]
        I1["Stakeholder request"] --> T["Triage<br/>PO + Tech Lead, weekly"]
        I2["Customer support"] --> T
        I3["Production monitoring"] --> T
        I4["Team improvement idea"] --> T
        T -->|"reject or duplicate"| X["Closed with reason"]
    end

    T -->|"accepted"| DISC["2. Discovery<br/>goals, metrics, risks,<br/>constraints, spikes"]
    DISC -->|"no-go"| X
    DISC -->|"go"| REQ["3. Requirements<br/>user stories, acceptance criteria,<br/>NFRs, business rules"]
    REQ --> DES["4. Design<br/>architecture, API, DB, UI,<br/>ADR, rollback plan"]
    DES --> BL["5. Backlog prep<br/>epics, stories, estimates,<br/>Definition of Ready"]
    BL --> SP["Sprint planning<br/>goal + committed items"]

    SP --> DEV["6. Development<br/>branch from nightly,<br/>implement, test, commit"]
    DEV --> PR["7. Pull request<br/>CI checks + peer review"]
    PR -->|"changes requested"| DEV
    PR -->|"approved"| MRG["Merge into nightly"]
    MRG --> PRV["Auto-deploy to Preview"]
    PRV --> QA["8. QA on Preview<br/>acceptance criteria,<br/>regression, UAT"]
    QA -->|"defects"| DEV
    QA -->|"passed"| RDY["Ready for Deployment"]

    RDY --> REL["9. Release<br/>release PR nightly to main,<br/>tag, GitHub Release"]
    REL --> APP{"Production approval<br/>required reviewer"}
    APP -->|"rejected"| RDY
    APP -->|"approved"| PROD["Deploy to Production<br/>migrations, health checks"]
    PROD -->|"health checks fail"| RBK["Automatic rollback<br/>+ incident"]
    RBK --> INC
    PROD -->|"healthy"| MON["10. Monitoring<br/>errors, performance,<br/>business metrics, feedback"]

    MON --> INC["Incidents and defects"]
    MON --> FB["User feedback<br/>and usage data"]
    MON --> DEBT["Technical debt<br/>and maintenance"]
    INC --> T
    FB --> T
    DEBT --> T

    RET["Sprint retrospective"] -.->|"process improvements"| DEV
    MON -.->|"metrics"| RET
```

*The whole lifecycle, with its two feedback loops made explicit: **product** feedback (monitoring → intake) and **process** feedback (metrics → retrospective → how we work). A lifecycle without both is a one-way pipeline, and one-way pipelines never get better.*

### 17.3 Diagram B — Agile Scrum sprint cycle

```mermaid
flowchart LR
    PB["Product Backlog<br/>ordered by the PO"]

    subgraph REF["Mid-sprint"]
        RF["Backlog Refinement<br/>45-60 min<br/>clarify, split, estimate"]
    end

    subgraph SPRINT["Sprint - 2 weeks"]
        SP["Sprint Planning<br/>1-2 h<br/>goal + sprint backlog"]
        SP --> DAILY["Daily Stand-up<br/>10-15 min"]
        DAILY --> WORK["Development<br/>code, review, merge,<br/>deploy to Preview"]
        WORK --> TEST["QA validation<br/>on Preview"]
        TEST -->|"defects"| WORK
        TEST --> DONE["Increment<br/>meets Definition of Done"]
        DONE --> DAILY
        DONE --> SR["Sprint Review<br/>45-60 min<br/>demo + feedback"]
        SR --> RETRO["Retrospective<br/>45 min<br/>1-3 owned actions"]
    end

    PB --> RF
    RF -->|"items meeting<br/>Definition of Ready"| PB
    PB --> SP
    SR -->|"feedback becomes<br/>new backlog items"| PB
    RETRO -->|"process improvements"| SP
    DONE -->|"released to production<br/>at least once per sprint"| PROD["Production"]
    PROD -.->|"monitoring findings"| PB
```

*Refinement sits deliberately outside the sprint box: it prepares the work for the *next* sprint, which is why planning can be selection rather than discovery. Note the two arrows returning to the backlog — review feedback and monitoring findings — and the one returning to planning: retrospective actions.*

### 17.4 Diagram C — GitHub Issue to Production

```mermaid
sequenceDiagram
    actor Dev as Developer
    participant GH as GitHub Issue<br/>+ Project
    participant Br as Branch
    participant PR as Pull Request
    participant CI as GitHub Actions
    actor Rev as Reviewer
    participant N as nightly
    participant Prev as Preview env
    actor QA as Tester
    participant M as main
    actor Appr as Release Approver
    participant Prod as Production

    Dev->>GH: Pick issue, check Definition of Ready
    GH-->>Dev: Acceptance criteria
    Dev->>GH: Status = In Progress
    Dev->>Br: git checkout -b feature/142-plan-change
    Dev->>Br: Commit - feat(billing): add plan change endpoint
    Dev->>PR: Open PR into nightly, "Closes #142"
    PR->>CI: Trigger pr-validation.yml
    CI-->>PR: format, lint, types, unit, integration, build, security
    PR->>Rev: Request review
    Rev-->>PR: Comments / approval
    Dev->>PR: Address comments, push fixes
    Rev->>PR: Approve
    Dev->>N: Squash merge
    N->>CI: Trigger deploy-preview.yml
    CI->>CI: Build artifact on GitHub-hosted runner
    CI->>Prev: Deploy via self-hosted preview runner
    Prev-->>CI: Migrations, smoke tests, E2E pass
    CI->>GH: Status = Ready for QA, notify team
    QA->>Prev: Verify acceptance criteria
    QA-->>GH: Pass, Status = Ready for Deployment
    Dev->>M: Open release PR - nightly to main
    Rev->>M: Approve and merge, tag v1.5.0
    M->>CI: GitHub Release published, deploy-production.yml
    CI->>CI: Rebuild from tag, re-run tests, package artifact
    CI->>Appr: Production environment - waiting for approval
    Appr-->>CI: Approve
    CI->>Prod: Deploy artifact via self-hosted production runner
    Prod-->>CI: Backup verified, migrations, health checks pass
    CI->>GH: Deployment recorded, team notified
    Dev->>GH: Status = Done
    Prod-->>GH: Monitoring findings become new issues
```

*Read this as the answer to "what actually happens to my work?". Every handoff is a GitHub artefact — no step depends on someone remembering to tell someone else.*

### 17.5 Diagram J — Example feature roadmap (Gantt)

Sample: the "subscription plan management" feature from §18.

```mermaid
gantt
    title Feature - Store owners manage their subscription plan
    dateFormat YYYY-MM-DD
    axisFormat %b %d
    excludes weekends

    section Discovery
    Stakeholder interviews          :done, d1, 2026-08-10, 3d
    Success metrics and constraints :done, d2, after d1, 2d
    Payment provider spike          :done, d3, after d1, 3d
    Go / no-go decision             :milestone, m0, after d3, 0d

    section Requirements
    User stories and acceptance criteria :active, r1, after m0, 3d
    Business rules and permissions       :active, r2, after m0, 2d
    Non-functional requirements          :r3, after r2, 2d

    section Design
    API contract              :de1, after r1, 2d
    Database schema and ERD   :de2, after r1, 2d
    UI wireframes and states  :de3, after r1, 4d
    ADR - proration strategy  :de4, after de1, 1d
    Design review and sign-off :milestone, m1, after de3, 0d

    section Development - Sprint 12
    Backend - plan change endpoint :dv1, after m1, 4d
    Backend - proration logic      :dv2, after dv1, 3d
    Frontend - plan selection UI   :dv3, after m1, 5d
    Frontend - confirmation flow   :dv4, after dv3, 3d
    Webhook handling               :dv5, after dv2, 2d

    section Testing
    Code review and CI          :t1, after dv2, 2d
    QA on Preview               :t2, after dv5, 3d
    Regression pass             :t3, after t2, 2d
    UAT with the Product Owner  :t4, after t3, 2d
    Sign-off                    :milestone, m2, after t4, 0d

    section Release
    Release PR and tag v1.5.0 :rel1, after m2, 1d
    Production approval        :rel2, after rel1, 1d
    Production deployment      :milestone, m3, after rel2, 0d

    section Post-release
    Smoke tests and monitoring watch :pr1, after m3, 1d
    30-day success metric review     :pr2, after pr1, 20d
    Follow-up improvements           :pr3, after pr1, 10d
```

*Useful for communicating shape and sequence to stakeholders. **Read it with §7.6 in mind:** dates beyond the current sprint are forecasts, not commitments, and the discovery/requirements/design bars will move as the team learns. Update the chart when reality changes rather than defending it — a Gantt chart that is never updated is a work of fiction that people plan around.*

