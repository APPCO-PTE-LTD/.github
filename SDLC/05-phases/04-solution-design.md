> Part of the [appco SDLC documentation](../../README.md) - [phase overview](00-overview.md), Phase 4.

# Phase 4: Solution and Technical Design

**Goal:** decide the technical approach before writing code, at a level of detail proportional to the risk. This phase prevents the two most expensive failures: building the wrong structure, and discovering an integration or migration problem after the code is written.

#### 4.1 Design depth by risk

| Risk level | Triggers | Required design artefacts |
|---|---|---|
| **Low** | UI-only change, existing patterns, no schema change | None. Approach described in the PR. |
| **Medium** | New endpoint, new table/column, new screen, new job | Short technical note in the issue: approach, affected components, migration, test plan |
| **High** | New subsystem, external integration, auth/permission change, money, PII, destructive migration, performance-critical path | ADR + diagrams (sequence/ER as relevant) + API contract + rollback plan + test strategy, reviewed by Tech Lead before implementation starts |

**MUST:** high-risk designs are reviewed and approved (a comment from the Tech Lead on the issue is sufficient) **before** implementation begins.

#### 4.2 Design activities

| Activity | What to produce | Notes |
|---|---|---|
| **Architecture review** | Where the change fits; new components; alternatives considered | For high-risk: an ADR (§19.8) |
| **Database design** | Tables, columns, types, indexes, constraints, relations; ER diagram | Name migrations; state whether reversible |
| **API design** | Endpoints, methods, request/response schemas, status codes, error shapes, pagination, versioning, auth | Write the contract **before** implementing; frontend can start against it |
| **UI/UX design** | Wireframes/mockups incl. loading, empty, error, and permission-denied states | Handover before development (DoR item) |
| **Integration design** | Protocol, auth, retries, timeouts, idempotency, failure behaviour, sandbox availability | Always specify what happens when the third party is down |
| **Permission & authorisation design** | Role → action matrix; where it is enforced | **MUST** be enforced server-side; UI hiding is not authorisation |
| **Security considerations** | Input validation, output encoding, secrets handling, rate limiting, data exposure in logs, OWASP-relevant risks | See §8 security testing |
| **Performance considerations** | Expected volumes, query plans, N+1 risks, caching, pagination, background processing | State the expected data volume; most performance bugs are volume assumptions |
| **Deployment impact** | Config/env vars, new services, downtime, order of deployment across repos | Cross-repo order matters — write it down |
| **Migration & rollback plan** | Migration steps, data backfill, and the exact rollback procedure | If rollback is impossible, say so loudly |
| **Testing strategy** | Which layers are tested and what the risky cases are | Feeds §8 |
| **Monitoring requirements** | Metrics, logs, alerts to add with the feature | Ship observability with the feature, not after the first incident |

#### 4.3 Backwards-compatible change patterns

Because we deploy small and often (P8), schema and API changes **SHOULD** follow expand/contract:

1. **Expand** — add the new column/field/endpoint; make it optional; deploy.
2. **Migrate** — backfill data; write to both old and new; deploy.
3. **Switch** — read from the new; deploy and observe.
4. **Contract** — remove the old, in a later release once nothing references it.

This makes every step individually reversible. A single deploy that renames a column and changes the code that reads it cannot be rolled back without data loss.

#### 4.4 When to create each artefact

| Artefact | Create when | Lives in |
|---|---|---|
| **Architecture Decision Record (ADR)** | The decision is expensive to reverse, or a future reader will ask "why this way?": framework choice, data store, auth model, integration strategy, deployment topology | `docs/adr/NNNN-title.md` in the affected repo, or the product `-docs` repo for cross-repo decisions |
| **Mermaid flow/architecture diagram** | Any process with more than ~4 steps or 3 actors | Inline in issue, ADR, or README |
| **Database / ER diagram** | Any schema change touching more than one table, or introducing relations | `docs/database.md` |
| **Sequence diagram** | Multi-service or third-party interactions, async flows, webhooks | ADR or `docs/architecture/` |
| **API contract** | Every new or changed endpoint | OpenAPI in the backend repo (`docs/api/openapi.yaml`) or, minimally, a table in the issue |
| **UI wireframes** | Any new screen or significant layout change | Design tool; link from the issue |
| **Technical implementation plan** | Epic-sized work, or work split across repositories | Epic issue body or `-docs` repo |

#### 4.5 Output of Phase 4

- Approved technical approach (comment, note, or ADR proportional to risk)
- API contracts and schema changes agreed
- Designs handed over
- Migration and rollback plan for anything touching data
- Technical tasks identified for Phase 5 breakdown
- Test and monitoring requirements noted on the issues

