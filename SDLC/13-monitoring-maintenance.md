> Part of the [appco SDLC documentation](README.md) - original section 13.

# 13. Production Monitoring and Maintenance

### 13.1 Why this phase exists

Deployment is not the end of the lifecycle. Most of a system's life is spent running, and most of what we learn about a feature comes from watching it run. A team that ships and stops looking finds out about problems from customers — which is the most expensive possible detection channel.

**The goal: we know about a problem before our users tell us.**

### 13.2 What to monitor

| Area | What | Signals to alert on | Tooling (examples) |
|---|---|---|---|
| **Application health** | Is the app up and responding? | Health endpoint fails 2× in a row | Uptime monitor (UptimeRobot, Better Stack, Healthchecks.io) |
| **Error tracking** | Unhandled exceptions, stack traces, affected users | New error type in production; error rate > baseline; any error affecting > 5 users | Sentry, Bugsnag, Rollbar |
| **HTTP metrics** | Request rate, error rate (4xx/5xx), latency | 5xx rate > 1% over 5 min; p95 latency > 2× baseline | Reverse proxy logs, APM |
| **Server** | CPU, memory, disk, network | Disk > 85%; memory > 90% sustained; load > cores × 2 | Netdata, Prometheus + Grafana, provider monitoring |
| **Database** | Connections, slow queries, replication lag, size growth | Connections > 80% of pool; queries > 1 s; disk growth anomalies | Built-in stats, pgHero, slow query log |
| **Queues / background jobs** | Depth, processing time, failure rate, dead-letter count | Depth > threshold for 5 min; any job failing repeatedly; **worker not running** | Horizon, Sidekiq UI, custom metric |
| **Scheduled jobs (cron)** | Did each run, and succeed? | A job did not check in within its window | Healthchecks.io-style dead-man's switch |
| **Certificates & domains** | TLS expiry | < 14 days to expiry | Uptime monitor / cert check |
| **Backups** | Did it run? Does it restore? | Backup missing or older than the SLA; restore test failed | `maintenance.yml` (§8.6) |
| **Security** | Dependency CVEs, secret leaks, failed logins, unusual access | Any high/critical CVE; secret detected; auth failure spike | Dependabot, secret scanning, log alerts |
| **Business metrics** | Signups, orders, plan changes, revenue | Volume drops to zero or far below normal for the time of day | Product analytics / dashboard |

**The most under-rated alert on this list is the last one.** A technical monitor says the app is up; a business metric says the app is *working*. An app returning 200s while zero orders are placed is broken in a way no health check catches.

### 13.3 Alerting rules

| Rule | Detail |
|---|---|
| **Alert on symptoms, not causes** | Alert on "users cannot check out", not on "CPU is 80%". High CPU with a working site is not an emergency |
| **Every alert must be actionable** | If nobody would do anything about it at 3 a.m., it is a dashboard metric, not an alert |
| **Route by severity** | `P0` → phone/push + team channel; `P1` → team channel; everything else → daily digest |
| **Tune ruthlessly** | An alert that fires often and is usually ignored trains the team to ignore all alerts. Fix it or delete it — this is the fastest way to make monitoring worthless |
| **Test the alerting path** | **SHOULD** quarterly: deliberately trigger an alert and confirm a human receives it. Silent broken alerting is common and only discovered during an incident |
| **On-call, honestly** | For a small team, "best effort during working hours, plus a named person reachable out-of-hours by rotation" is realistic. Do not write a 24/7 on-call policy you will not staff |

### 13.4 Logging

| Rule | Priority |
|---|---|
| Structured logs (JSON) with a request/correlation ID | **SHOULD** |
| Never log passwords, tokens, full card numbers, or personal data beyond what is needed | **MUST** |
| Log levels used meaningfully: `error` = needs attention, `warn` = suspicious, `info` = notable events, `debug` = off in production | **SHOULD** |
| Centralised collection (all servers, searchable in one place) | **SHOULD** — the first thing you want during an incident |
| Retention: 30 days searchable, 1 year archived (adjust to legal requirements) | **SHOULD** |
| Audit log for sensitive actions (permission changes, plan changes, deletions, admin access) kept separately and longer | **MUST** where §5 Phase 3 required it |

### 13.5 Maintenance work

| Activity | Frequency | Owner |
|---|---|---|
| Dependency updates (grouped Dependabot PRs into `nightly`) | Weekly review, merged within 2 weeks | Developers on rotation |
| Security patches (high/critical CVEs) | **Within 7 days**; `P0` if actively exploited | Tech Lead |
| OS and runtime patching (app servers + runner hosts) | Monthly | DevOps |
| Backup restore verification | Weekly automated, quarterly full manual drill | DevOps |
| Log/disk cleanup, old release pruning | Automated daily | DevOps |
| Certificate renewal | Automated; alert at 14 days | DevOps |
| Access review (org members, teams, deploy keys, PATs) | Quarterly and on every departure | Tech Lead |
| Secret rotation | Every 90 days for production credentials | DevOps |
| Technical debt review | Every sprint at refinement | Tech Lead |
| Runner cleanup and updates | Monthly | DevOps |

**Dependency policy:** patch and minor updates are merged routinely once CI is green. Major updates get their own issue, their own PR, and a note about breaking changes. **MUST NOT** let dependencies sit un-updated for more than a quarter — the longer the gap, the more the upgrade becomes a project rather than a task.

### 13.6 Technical debt

Technical debt is a normal consequence of shipping, not a moral failing. It becomes a problem only when it is invisible.

| Rule | Detail |
|---|---|
| **Record it** | When taking a shortcut deliberately, open a `type: tech-debt` issue describing what was skipped and what it will cost |
| **Mark it in code** | A `TODO:` comment referencing the issue number, so the code and the backlog agree |
| **Budget it** | **SHOULD** allocate ~10–20% of each sprint to debt and maintenance. Not a rule to enforce mechanically, but a target to defend when pressure builds |
| **Justify it in business terms** | "This will slow every future billing change by roughly a day" persuades; "the code is ugly" does not |
| **Re-evaluate it** | Some debt never needs paying because the code is deleted or never touched again. Review the debt list quarterly and close what no longer matters |

### 13.7 Feeding production findings back into the backlog

**This is the loop that closes the SDLC.** Every production signal has a defined path back into planning:

| Source | Becomes | Labels | Who files it |
|---|---|---|---|
| Error tracker: new/frequent error | Bug issue with stack trace and occurrence count | `type: bug`, `source: production` | Whoever triages errors (rotate weekly) |
| Post-incident review actions | Prevention issues | `source: incident` | Incident Lead |
| Performance degradation | Tech debt or bug issue with the measurement | `type: tech-debt`, `source: production` | DevOps |
| User feedback / support tickets | Feature request or bug | `source: support` | PO (batched weekly) |
| Security scan finding | Security issue or private advisory | `type: security` | Tech Lead |
| Failed scheduled job | Auto-created issue (§8.6) | `source: automation` | Automation |
| Usage analytics showing a feature is unused | Discussion → possible removal | `type: tech-debt` | PO |

**MUST:** a weekly, timeboxed (~20 min) **production review** — Tech Lead + DevOps + PO — that walks the error tracker, uptime report, and support themes, and files issues. Without a scheduled slot, this always loses to feature work, and the feedback loop quietly stops existing.

**Deleting features is legitimate work.** If analytics show nobody uses something, removing it reduces maintenance, testing, and cognitive load permanently. It should be as respectable a backlog item as building something new.

