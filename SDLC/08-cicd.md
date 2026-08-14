> Part of the [appco SDLC documentation](README.md) - original section 8.

# 8. GitHub Actions and CI/CD Architecture

### 8.1 Strategy

Every repository has its own workflows appropriate to its technology, but they are thin wrappers over **reusable workflows** in `shared-workflows` (§6.11). This gives us consistency without copy-paste drift: fixing a caching bug once fixes it everywhere.

**The four standard workflows every deployable repository MUST have:**

| Workflow | File | Trigger | Runner | Purpose |
|---|---|---|---|---|
| **PR validation** | `pr-validation.yml` | `pull_request` → `nightly`, `main` | GitHub-hosted | Gate on quality before merge |
| **Preview deployment** | `deploy-preview.yml` | `push` → `nightly` | Build: GitHub-hosted · Deploy: self-hosted `preview` | Continuous deployment of the integration branch |
| **Production deployment** | `deploy-production.yml` | `release: published` (+ `workflow_dispatch`) | Build: GitHub-hosted · Deploy: self-hosted `production` | Approved, tagged release to production |
| **Scheduled maintenance** | `maintenance.yml` | `schedule` | GitHub-hosted | Security scans, backup verification, health checks |

Library repositories (`shared-ui`, `shared-utils`) have PR validation plus a **publish** workflow instead of deployments.

### 8.2 Pipeline overview

```mermaid
flowchart TD
    subgraph PR["Pull Request into nightly"]
        A1["Checkout + install<br/>with dependency cache"] --> A2["Format check"]
        A2 --> A3["Lint"]
        A3 --> A4["Type check"]
        A4 --> A5["Unit tests"]
        A5 --> A6["Integration tests<br/>service containers: DB, cache"]
        A6 --> A7["Build validation"]
        A7 --> A8["Dependency review<br/>+ secret scan + CodeQL"]
    end

    A8 -->|"all green + 1 approval"| M1["Merge to nightly"]

    subgraph PRV["Preview pipeline - automatic"]
        M1 --> B1["Build artifact<br/>GitHub-hosted runner"]
        B1 --> B2["Full test suite"]
        B2 --> B3["Upload artifact<br/>name: app-nightly-SHA"]
        B3 --> B4["Deploy to Preview<br/>self-hosted runner, label: preview"]
        B4 --> B5["Run migrations"]
        B5 --> B6["Clear caches + restart services"]
        B6 --> B7["Smoke tests"]
        B7 --> B8["E2E suite against Preview"]
        B8 --> B9["Notify team chat"]
    end

    B9 --> T["Tester validates on Preview"]
    T -->|"sign-off"| R1["Release PR: nightly to main"]
    R1 --> R2["Merge + tag vX.Y.Z<br/>+ GitHub Release"]

    subgraph PRD["Production pipeline - approval gated"]
        R2 --> C1["Build artifact from the tag<br/>GitHub-hosted runner"]
        C1 --> C2["Re-run full test suite"]
        C2 --> C3["Upload versioned artifact<br/>name: app-vX.Y.Z"]
        C3 --> C4{"Environment: production<br/>WAITS for approval"}
        C4 -->|approved| C5["Deploy to Production<br/>self-hosted runner, label: production"]
        C5 --> C6["Backup verification"]
        C6 --> C7["Deploy artifact<br/>atomic symlink switch"]
        C7 --> C8["Run migrations"]
        C8 --> C9["Health checks + smoke tests"]
        C9 -->|pass| C10["Announce release<br/>record deployment"]
        C9 -->|fail| C11["Automatic rollback<br/>previous release symlink"]
        C11 --> C12["Alert + open incident issue"]
    end
```

*One rule holds the whole picture together: **build on GitHub-hosted runners, deploy on self-hosted runners.** Untrusted code (anything from a pull request) is never executed on a machine that can reach production. §9 explains why in detail.*

### 8.3 Workflow 1 — Pull request validation

Runs on every PR into `nightly` or `main`. **MUST** be a required status check in branch protection.

```yaml
# .github/workflows/pr-validation.yml
name: PR Validation

on:
  pull_request:
    branches: [nightly, main]
    types: [opened, synchronize, reopened, ready_for_review]

# Cancel superseded runs on the same PR — saves minutes and gives faster feedback
concurrency:
  group: pr-${{ github.event.pull_request.number }}
  cancel-in-progress: true

permissions:
  contents: read
  pull-requests: write   # for coverage / review comments

jobs:
  validate:
    # Skip drafts to save CI minutes; they still run when marked ready for review
    if: github.event.pull_request.draft == false
    runs-on: ubuntu-latest          # GitHub-hosted: PR code is untrusted
    timeout-minutes: 15

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: app_test
        options: >-
          --health-cmd pg_isready --health-interval 10s
          --health-timeout 5s --health-retries 5
        ports: ['5432:5432']

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Format check
        run: npm run format:check

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run typecheck

      - name: Unit tests
        run: npm run test:unit -- --coverage

      - name: Integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/app_test

      - name: Build
        run: npm run build

  security:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4

      - name: Dependency review
        uses: actions/dependency-review-action@v4
        with:
          fail-on-severity: high

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript-typescript

      - name: CodeQL analysis
        uses: github/codeql-action/analyze@v3
```

Using the shared reusable workflow instead (**SHOULD**, once `shared-workflows` exists):

```yaml
name: PR Validation
on:
  pull_request:
    branches: [nightly, main]

jobs:
  ci:
    uses: appco/shared-workflows/.github/workflows/node-ci.yml@v1
    with:
      node-version: '22'
      run-integration-tests: true
    secrets: inherit
```

**Required status checks to select in branch protection:** `validate`, `security`.

### 8.4 Workflow 2 — Preview deployment (branch `nightly`)

Triggered automatically on every push to `nightly` (i.e. every merged PR). This is the deployment testers exercise.

```yaml
# .github/workflows/deploy-preview.yml
name: Deploy to Preview

on:
  push:
    branches: [nightly]
  workflow_dispatch:        # manual re-deploy of the current nightly

# Never run two preview deploys at once; queue them and keep the latest
concurrency:
  group: deploy-preview
  cancel-in-progress: false

permissions:
  contents: read

jobs:
  build:
    name: Build and test
    runs-on: ubuntu-latest        # GitHub-hosted: build is untrusted-adjacent and needs no server access
    timeout-minutes: 20
    outputs:
      artifact-name: ${{ steps.meta.outputs.artifact }}
    steps:
      - uses: actions/checkout@v4

      - id: meta
        run: echo "artifact=app-nightly-${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"

      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }

      - run: npm ci
      - run: npm run lint && npm run typecheck
      - run: npm test
      - run: npm run build

      - name: Create deployment package
        run: |
          tar -czf "${{ steps.meta.outputs.artifact }}.tar.gz" \
            dist/ package.json package-lock.json migrations/

      - uses: actions/upload-artifact@v4
        with:
          name: ${{ steps.meta.outputs.artifact }}
          path: ${{ steps.meta.outputs.artifact }}.tar.gz
          retention-days: 14

  deploy:
    name: Deploy to Preview
    needs: build
    runs-on: [self-hosted, linux, preview]   # runner inside the preview network
    timeout-minutes: 15
    environment:
      name: preview
      url: https://preview.product-a.appco.com
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: ${{ needs.build.outputs.artifact-name }}

      - name: Deploy release directory
        env:
          DEPLOY_PATH: ${{ vars.DEPLOY_PATH }}
        run: |
          set -euo pipefail
          RELEASE="$DEPLOY_PATH/releases/${GITHUB_SHA::7}"
          mkdir -p "$RELEASE"
          tar -xzf "${{ needs.build.outputs.artifact-name }}.tar.gz" -C "$RELEASE"
          ln -sfn "$DEPLOY_PATH/shared/.env" "$RELEASE/.env"
          ln -sfn "$DEPLOY_PATH/shared/storage" "$RELEASE/storage"
          npm ci --omit=dev --prefix "$RELEASE"

      - name: Run database migrations
        run: npm run migrate --prefix "${{ vars.DEPLOY_PATH }}/releases/${GITHUB_SHA::7}"

      - name: Activate release
        run: |
          set -euo pipefail
          ln -sfn "${{ vars.DEPLOY_PATH }}/releases/${GITHUB_SHA::7}" "${{ vars.DEPLOY_PATH }}/current"
          sudo systemctl reload product-a-preview

      - name: Clear caches
        run: npm run cache:clear --prefix "${{ vars.DEPLOY_PATH }}/current"

      - name: Smoke tests
        run: |
          set -euo pipefail
          curl -fsS --retry 5 --retry-delay 3 https://preview.product-a.appco.com/health
          curl -fsS https://preview.product-a.appco.com/api/version | grep "${GITHUB_SHA::7}"

      - name: Prune old releases
        run: ls -1dt "${{ vars.DEPLOY_PATH }}"/releases/* | tail -n +6 | xargs -r rm -rf

  e2e:
    name: E2E against Preview
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npm run test:e2e
        env:
          BASE_URL: https://preview.product-a.appco.com

  notify:
    needs: [deploy, e2e]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Notify team chat
        run: |
          STATUS="${{ needs.e2e.result }}"
          curl -fsS -X POST -H 'Content-Type: application/json' \
            -d "{\"text\":\"Preview deploy ${GITHUB_SHA::7} — $STATUS — https://preview.product-a.appco.com\"}" \
            "${{ secrets.CHAT_WEBHOOK_URL }}"
```

Notes on choices:

| Choice | Reason |
|---|---|
| `concurrency` without `cancel-in-progress` | Deployments queue rather than cancel — cancelling mid-deploy leaves a half-deployed server |
| Build on GitHub-hosted, deploy on self-hosted | The self-hosted runner only ever runs code that already passed review and merge (§9) |
| Release directories + symlink switch | Atomic activation and instant rollback: repoint the symlink |
| Migrations before symlink switch | The new code is not serving traffic yet, so a failed migration does not take the site down |
| E2E on a GitHub-hosted runner *after* deploy | It tests over the network like a user; it needs no server access |
| Artifact retention 14 days | Long enough to redeploy a recent nightly; short enough not to hoard storage |

### 8.5 Workflow 3 — Production deployment (branch `main`)

Triggered by publishing a GitHub Release (which is created from the tag on `main` after the release PR merges). Manual `workflow_dispatch` exists for re-deploys and rollbacks.

```yaml
# .github/workflows/deploy-production.yml
name: Deploy to Production

on:
  release:
    types: [published]
  workflow_dispatch:
    inputs:
      tag:
        description: 'Tag to deploy (e.g. v1.5.0) — also used for rollback'
        required: true

concurrency:
  group: deploy-production
  cancel-in-progress: false

permissions:
  contents: read

jobs:
  guard:
    name: Verify release preconditions
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.v.outputs.version }}
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.inputs.tag || github.ref }}
          fetch-depth: 0

      - id: v
        run: echo "version=${{ github.event.inputs.tag || github.ref_name }}" >> "$GITHUB_OUTPUT"

      - name: Tag must be on main
        run: |
          set -euo pipefail
          git branch -r --contains HEAD | grep -q 'origin/main' \
            || { echo "::error::Tag is not an ancestor of main. Production deploys only from main."; exit 1; }

  build:
    name: Build versioned artifact
    needs: guard
    runs-on: ubuntu-latest
    timeout-minutes: 20
    steps:
      - uses: actions/checkout@v4
        with: { ref: ${{ needs.guard.outputs.version }} }

      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }

      - run: npm ci
      - run: npm run lint && npm run typecheck
      - run: npm test                       # the exact code being shipped is re-verified
      - run: npm run build

      - name: Package
        run: tar -czf "app-${{ needs.guard.outputs.version }}.tar.gz" dist/ package.json package-lock.json migrations/

      - name: Checksum
        run: sha256sum "app-${{ needs.guard.outputs.version }}.tar.gz" > checksum.txt

      - uses: actions/upload-artifact@v4
        with:
          name: app-${{ needs.guard.outputs.version }}
          path: |
            app-${{ needs.guard.outputs.version }}.tar.gz
            checksum.txt
          retention-days: 90        # long enough to roll back to any recent release

  deploy:
    name: Deploy to Production
    needs: [guard, build]
    runs-on: [self-hosted, linux, production]
    timeout-minutes: 20
    environment:
      name: production            # <-- pauses here for required reviewers
      url: https://product-a.appco.com
    steps:
      - uses: actions/download-artifact@v4
        with: { name: app-${{ needs.guard.outputs.version }} }

      - name: Verify checksum
        run: sha256sum -c checksum.txt

      - name: Verify a recent database backup exists
        run: /opt/appco/bin/verify-backup.sh --max-age-hours 6

      - name: Record current release for rollback
        id: prev
        run: echo "previous=$(readlink -f "${{ vars.DEPLOY_PATH }}/current")" >> "$GITHUB_OUTPUT"

      - name: Unpack release
        env:
          VERSION: ${{ needs.guard.outputs.version }}
        run: |
          set -euo pipefail
          RELEASE="${{ vars.DEPLOY_PATH }}/releases/$VERSION"
          mkdir -p "$RELEASE"
          tar -xzf "app-$VERSION.tar.gz" -C "$RELEASE"
          ln -sfn "${{ vars.DEPLOY_PATH }}/shared/.env" "$RELEASE/.env"
          ln -sfn "${{ vars.DEPLOY_PATH }}/shared/storage" "$RELEASE/storage"
          npm ci --omit=dev --prefix "$RELEASE"

      - name: Run migrations
        run: npm run migrate --prefix "${{ vars.DEPLOY_PATH }}/releases/${{ needs.guard.outputs.version }}"

      - name: Activate release
        run: |
          set -euo pipefail
          ln -sfn "${{ vars.DEPLOY_PATH }}/releases/${{ needs.guard.outputs.version }}" "${{ vars.DEPLOY_PATH }}/current"
          sudo systemctl reload product-a

      - name: Health check and smoke tests
        id: health
        run: |
          set -euo pipefail
          sleep 5
          curl -fsS --retry 10 --retry-delay 3 https://product-a.appco.com/health
          curl -fsS https://product-a.appco.com/api/version | grep "${{ needs.guard.outputs.version }}"
          /opt/appco/bin/smoke-tests.sh production

      - name: Rollback on failure
        if: failure() && steps.health.outcome == 'failure'
        run: |
          set -euo pipefail
          ln -sfn "${{ steps.prev.outputs.previous }}" "${{ vars.DEPLOY_PATH }}/current"
          sudo systemctl reload product-a
          curl -fsS https://product-a.appco.com/health
          echo "::error::Health checks failed — rolled back to ${{ steps.prev.outputs.previous }}"

  announce:
    needs: [guard, deploy]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Notify
        run: |
          curl -fsS -X POST -H 'Content-Type: application/json' \
            -d "{\"text\":\"Production deploy ${{ needs.guard.outputs.version }} — ${{ needs.deploy.result }}\"}" \
            "${{ secrets.CHAT_WEBHOOK_URL }}"

  open-incident:
    needs: deploy
    if: failure()
    runs-on: ubuntu-latest
    permissions: { issues: write }
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `Production deploy failed: ${{ needs.guard.outputs.version }}`,
              labels: ['type: incident', 'priority: P0'],
              body: `Deployment of ${{ needs.guard.outputs.version }} failed health checks and was rolled back.\n\nRun: ${context.serverUrl}/${context.repo.owner}/${context.repo.repo}/actions/runs/${context.runId}`
            })
```

**Production controls this workflow enforces (all MUST):**

| Control | Mechanism |
|---|---|
| Only code that reached `main` can deploy | The `guard` job fails if the tag is not an ancestor of `main` |
| Tests passed on the exact shipped code | The build job re-runs the full suite on the tag |
| Human approval | GitHub `production` Environment with required reviewers — the job *waits* |
| Deployment credentials are unreachable without approval | Secrets are `production` **Environment** secrets, not repository secrets |
| Backup exists | `verify-backup.sh` fails the job if no backup within 6 hours |
| Artifact integrity | SHA-256 checksum verified before unpacking |
| Automatic rollback | Symlink restored on health-check failure |
| Incident raised automatically | `open-incident` job creates a `P0` issue |
| Rollback is one action | Re-run with `workflow_dispatch` and the previous tag |

### 8.6 Workflow 4 — Scheduled maintenance

```yaml
# .github/workflows/maintenance.yml
name: Scheduled Maintenance

on:
  schedule:
    - cron: '0 2 * * *'    # 02:00 UTC daily
    - cron: '0 3 * * 1'    # 03:00 UTC Mondays — weekly items
  workflow_dispatch:

permissions:
  contents: read
  issues: write
  security-events: write

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high
      - uses: github/codeql-action/init@v3
        with: { languages: javascript-typescript }
      - uses: github/codeql-action/analyze@v3

  backup-verification:
    runs-on: [self-hosted, linux, production]
    steps:
      - name: Verify last backup restores into a scratch database
        run: /opt/appco/bin/verify-backup.sh --restore-test

  health-check:
    runs-on: ubuntu-latest
    steps:
      - name: Production health
        run: curl -fsS https://product-a.appco.com/health
      - name: Preview health
        run: curl -fsS https://preview.product-a.appco.com/health

  nightly-e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { ref: nightly }
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci && npx playwright install --with-deps chromium
      - run: npm run test:e2e
        env: { BASE_URL: 'https://preview.product-a.appco.com' }

  report-failure:
    needs: [security-scan, backup-verification, health-check, nightly-e2e]
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.create({
              owner: context.repo.owner, repo: context.repo.repo,
              title: `Scheduled maintenance failure — ${new Date().toISOString().slice(0,10)}`,
              labels: ['type: bug', 'priority: P1', 'source: automation'],
              body: `A scheduled maintenance job failed.\n\n${context.serverUrl}/${context.repo.owner}/${context.repo.repo}/actions/runs/${context.runId}`
            })
```

Also configure **Dependabot** (`.github/dependabot.yml`) for weekly grouped dependency PRs targeting `nightly`:

```yaml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule: { interval: weekly, day: monday }
    target-branch: nightly
    open-pull-requests-limit: 5
    groups:
      minor-and-patch:
        update-types: [minor, patch]
  - package-ecosystem: github-actions
    directory: /
    schedule: { interval: weekly }
    target-branch: nightly
```

### 8.7 Workflow hygiene rules

| Rule | Priority | Reason |
|---|---|---|
| Pin third-party actions to a tag, or a commit SHA for anything privileged | **MUST** | `@main` lets an upstream change run arbitrary code in our pipeline |
| Set explicit `permissions:` on every workflow, least privilege | **MUST** | The default token is broader than most jobs need |
| **Never** use `pull_request_target` with a checkout of PR code | **MUST NOT** | It runs untrusted code with access to secrets — the single most exploited Actions misconfiguration |
| Set `timeout-minutes` on every job | **SHOULD** | A hung job can burn hours of runner time and block deploy queues |
| Use `concurrency` groups for deployments | **MUST** | Prevents two deploys racing on the same server |
| Cache dependencies | **SHOULD** | Typically the largest single CI time saving |
| Keep PR validation under 10 minutes | **SHOULD** | Beyond that, people stop waiting and start context-switching |
| Never echo secrets; avoid passing them through shell interpolation | **MUST** | Use `env:` blocks; GitHub masks known secret values but not derived ones |
| Fail loudly — no `continue-on-error` on quality gates | **MUST** | A green tick that means nothing is worse than a red one |
| `set -euo pipefail` in every multi-line `run:` block | **SHOULD** | Without it, a failing command mid-script does not fail the step |

