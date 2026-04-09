---
name: migrate-ci-docker
description: >-
  Update CI workflows, Docker, and infrastructure for Node 24. Use when
  upgrading CI pipelines, when the user says "update CI", "upgrade Docker",
  "Node 24 in CI", "GitHub Actions versions", or "update workflows".
---

# Migrate CI, Docker, and Infrastructure for Node 24

Update all CI workflows, Dockerfiles, and infrastructure configuration to use Node 24.

## Step 1: Update Volta in package.json

```jsonc
{
  "volta": {
    "node": "24.14.1"
  }
}
```

If the project has `.nvmrc`, update it as well:

```
24.14.1
```

## Step 2: Update Dockerfiles

Update only Dockerfiles that exist in the project. Skip any that are not present.

### Main Dockerfile (if exists)

```dockerfile
# Before
FROM asia-southeast1-docker.pkg.dev/tk-dev-micro/base-image/node:20.10.0-alpine AS base

# After
FROM asia-southeast1-docker.pkg.dev/tk-dev-micro/base-image/node:24.14.1-alpine AS base
```

### Sonar Dockerfile (if `docker/Sonar.Dockerfile` exists)

```dockerfile
# Before
FROM node:20.9.0-alpine

# After
FROM node:24-alpine
# Or with custom registry:
FROM asia-southeast1-docker.pkg.dev/tk-dev-micro/base-image/node:24.14.1-alpine
```

### Test Dockerfile (if `docker/Test.Dockerfile` exists)

```dockerfile
# Before
FROM node:20.10.0-alpine AS base

# After
FROM node:24-alpine AS base
```

## Step 3: Update GitHub Actions

Apply these version bumps across **all** workflow files under `.github/workflows/`:

| Action | Before | After |
|--------|--------|-------|
| `actions/checkout` | `@v2` / `@v3` | `@v4` or `@v6` |
| `actions/setup-node` | `@v3` / `@v4` | `@v6` |
| `pnpm/action-setup` | `@v2` / `@v4` | `@v5` |
| `actions/cache` | `@v3` | `@v4` |
| `peter-evans/find-comment` | `@v2` / `@v3` | `@v4` |
| `peter-evans/create-or-update-comment` | `@v3` / `@v4` | `@v5` |

Update Node version in all workflows:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v6
  with:
    node-version: 24.14.1
    cache: 'pnpm'
```

Pin pnpm version:

```yaml
- name: Setup pnpm
  uses: pnpm/action-setup@v5
  with:
    version: 10
```

## Step 4: Workflow-specific changes

Apply these only to workflow files that exist in the project. Skip any that are not present.

### pull_request_actions.yml (if exists)

Add explicit `setup-node` step if missing (some older workflows relied on system Node):

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v6
  with:
    node-version: 24.14.1
    cache: 'pnpm'
```

### nextjs_bundle_analysis.yml (if exists)

Update build command:

```yaml
# Before
run: ./node_modules/.bin/next build

# After
run: pnpm run build
```

### release.yml (if exists)

Update preparation step:

```yaml
# Before
run: npm pkg set scripts.prepare="echo '[Skip] Husky and MSW Init'"

# After
run: pnpm pkg set scripts.prepare "echo '[Skip] Husky and MSW Init'"
```

### staging_build_autotag.yml (if exists)

Update matrix node version:

```yaml
strategy:
  matrix:
    node-version:
      - 24  # was 18 or 20
```

### chromatic.yml (if exists)

Pin pnpm version:

```yaml
- name: Setup pnpm
  uses: pnpm/action-setup@v5
  with:
    version: 10
```

## Step 5: Update Jenkinsfile.dev (if present)

```groovy
// Before
agent { docker { image 'node:20.10.0-alpine' } }

// After
agent { docker { image 'asia-southeast1-docker.pkg.dev/tk-dev-micro/base-image/node:24.14.1-alpine' } }
```

Update pnpm setup:

```groovy
// Before
sh 'corepack prepare pnpm@latest'

// After
sh 'corepack prepare pnpm@10'
```

## Step 6: Update Sonar config (if file exists)

If `sonar-project.properties` exists, update it. Otherwise skip this step.

In `sonar-project.properties`:

```properties
# Before
sonar.eslint.eslintconfigpath=.eslintrc.json

# After
sonar.eslint.eslintconfigpath=eslint.config.cjs
```

## Checklist

- [ ] Volta node pinned to 24.14.1 in package.json
- [ ] `.nvmrc` updated to 24.14.1 (if present)
- [ ] Main Dockerfile uses node:24.14.1-alpine (if exists)
- [ ] Sonar Dockerfile uses node:24-alpine (if exists)
- [ ] Test Dockerfile uses node:24-alpine (if exists)
- [ ] All GitHub Actions bumped to latest versions
- [ ] Node version set to 24.14.1 in all workflows
- [ ] pnpm version pinned in all workflows
- [ ] Build commands use `pnpm run build`
- [ ] Release uses `pnpm pkg set` instead of `npm pkg set`
- [ ] Staging autotag matrix updated to Node 24
- [ ] Jenkinsfile.dev updated (if present)
- [ ] `sonar-project.properties` points to `eslint.config.cjs` (if file exists)
