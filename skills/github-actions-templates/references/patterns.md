# Workflow patterns

Five sanitized shapes, each the minimum that still works in production. Replace every
`<PLACEHOLDER>` with a value read from the project; never substitute a plausible-looking fake.

Conventions:
- `actions/*` are GitHub first-party and may use majors. Check the current major when generating.
- Everything else is third-party: `uses: owner/repo@<FULL_SHA>  # <TAG>`. Resolve the SHA from the
  release you want, then record the tag in the comment so a human can read the diff.

---

## 1. Test on pull request

The default PR gate. Read-only token, no secrets, cancels superseded runs.

```yaml
# .github/workflows/test.yml
name: Test

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    name: Node ${{ matrix.node }}
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        node: ['20', '22']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: npm
          cache-dependency-path: package-lock.json
      - run: npm ci
      - run: npm run lint
      - run: npm test
```

Notes:
- Build a matrix only for runtimes the project commits to supporting (`engines`, docs, support
  policy). No stated contract means one version, not three.
- `cache: npm` needs a lockfile path; monorepos point `cache-dependency-path` at each workspace lock.
- Nothing here needs a write scope. A step that asks for one belongs in another workflow.
---

## 2. Build and push an image to the registry

Use when the artifact is a container image. Escalates `packages: write` on the one job that pushes.

```yaml
# .github/workflows/build-image.yml
name: Build image

on:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@<FULL_SHA>  # <TAG>
      - name: Log in to GHCR
        uses: docker/login-action@<FULL_SHA>  # <TAG>
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Metadata
        id: meta
        uses: docker/metadata-action@<FULL_SHA>  # <TAG>
        with:
          images: ghcr.io/<OWNER>/<IMAGE_NAME>
          tags: |
            type=ref,event=branch
            type=sha,prefix=sha-
      - name: Build and push
        uses: docker/build-push-action@<FULL_SHA>  # <TAG>
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

Notes:
- `GITHUB_TOKEN` is enough for GHCR; do not add a PAT with broader scope as a workaround.
- `type=gha` cache is scoped per ref and branch, so a PR cannot poison the default branch's cache.
- Tag by SHA, never `latest` alone — a deploy must name the exact image it rolls back from.

---

## 3. Reusable workflow and its caller

Use when two or more workflows share one gate. Typed inputs make misuse a validation error.

```yaml
# .github/workflows/_test.yml
name: Reusable test gate

on:
  workflow_call:
    inputs:
      node-version:
        type: string
        required: true
      run-e2e:
        type: boolean
        required: false
        default: false
    secrets:
      TEST_API_TOKEN:
        required: false

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: npm
      - run: npm ci
      - run: npm test
        env:
          TEST_API_TOKEN: ${{ secrets.TEST_API_TOKEN }}
      - if: ${{ inputs.run-e2e }}
        run: npm run test:e2e
```

```yaml
# caller
jobs:
  gate:
    uses: <OWNER>/<REPO>/.github/workflows/_test.yml@<REF>
    with:
      node-version: '22'
      run-e2e: true
    secrets:
      TEST_API_TOKEN: ${{ secrets.TEST_API_TOKEN }}
```

Notes:
- Secrets are **not** inherited by default. Name each one, or write `secrets: inherit` — and only
  when the reusable workflow lives inside the same trust boundary (same org).
- The `secrets:` block declares only what the callee needs; passing everything is how one careless
  reusable workflow reads every credential in the org.
- Point the caller at a tag or SHA, not the default branch, once the gate is load-bearing.

---

## 4. Scan and publish findings as SARIF

Use to surface scan results as code-scanning annotations on the PR.

```yaml
# .github/workflows/scan.yml
name: Scan

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read
  security-events: write

jobs:
  fs-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Trivy fs scan
        id: trivy
        continue-on-error: true
        uses: aquasecurity/trivy-action@<FULL_SHA>  # <TAG>
        with:
          scan-type: fs
          scan-ref: .
          format: sarif
          output: trivy-results.sarif
      - name: Upload SARIF
        if: always() && steps.trivy.outcome != 'cancelled'
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: trivy-results.sarif
          category: trivy-fs
```

Notes:
- Pin the scanner to a SHA like any other third-party action: scanners execute code on your runner.
  Replace `<FULL_SHA>` with the release you pick — never ship the placeholder.
- SARIF upload needs `security-events: write`; the scan itself needs nothing.
- `continue-on-error` on the scan keeps the finding stream alive; add a separate severity gate step
  when you want a blocking rule with an explicit threshold.

---

## 5. Production deploy behind an environment

Use for anything that changes production. The `environment:` is where the approval rule lives; the
workflow only points at it.

```yaml
# .github/workflows/deploy.yml
name: Deploy production

on:
  workflow_dispatch:
  push:
    tags: ['v*']

permissions:
  contents: read

concurrency:
  group: deploy-<ENVIRONMENT>
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: <ENVIRONMENT>
      url: <PRODUCTION_URL>
    permissions:
      contents: read
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - name: Assume role (OIDC, short-lived)
        uses: aws-actions/configure-aws-credentials@<FULL_SHA>  # <TAG>
        with:
          role-to-assume: ${{ secrets.<IAM_ROLE_ARN_SECRET> }}
          aws-region: <AWS_REGION>
      - name: Deploy
        run: <DEPLOY_COMMAND>
      - name: Smoke check
        run: curl --fail --retry 5 --retry-delay 10 --silent --show-error <HEALTHCHECK_URL>
      - name: Rollback pointer
        if: failure()
        run: echo "::error::Deploy failed. Follow <ROLLBACK_RUNBOOK_URL> and revert to the last healthy release."
```

Notes:
- `concurrency` with `cancel-in-progress: false` serializes deploys: a second push waits instead of
  stacking on a half-applied release.
- `id-token: write` exists only to mint the OIDC token, and that is what keeps a long-lived cloud key
  out of the repo. The sketch uses AWS; the same shape covers GCP workload identity, Azure AD
  workload identity federation, and any OIDC-trusting target — swap the action and the audience.
- Keep the smoke check inside the job so a failed verification marks the deploy failed, not green.
- The rollback step prints a pointer; the runbook owns the procedure.
