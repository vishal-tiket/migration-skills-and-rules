---
name: migrate-tiket-packages
description: >-
  Switch @tiket/* internal packages to React 19 and Next 16 compatible alpha
  versions. Use when upgrading Tiket passport, react-common, or @tiket/next
  packages, or when the user says "update tiket packages", "upgrade passport",
  "alpha versions", or "internal packages".
---

# Migrate Tiket Internal Packages

Switch all `@tiket/*` dependencies to alpha builds that support React 19 and Next 16.

## Step 1: Scan package.json

Read `package.json` and categorize all `@tiket/*` dependencies into three groups:

| Group | Pattern | Alpha version |
|-------|---------|---------------|
| **@tiket/next** | `@tiket/next` | `0.0.0-alpha-324-20260402105439` |
| **Passport** | `@tiket/passport`, `@tiket/passport-*` | `0.0.0-alpha-3045-20260402093025` |
| **React Common** | `@tiket/react-common-*` | `0.0.0-alpha-1029-20260406145129` |

## Step 2: Update versions

Replace every `@tiket/*` dependency version with its corresponding alpha:

```jsonc
// @tiket/next group
"@tiket/next": "0.0.0-alpha-324-20260402105439",

// Passport group (apply same version to all passport-* packages)
"@tiket/passport": "0.0.0-alpha-3045-20260402093025",
"@tiket/passport-assets": "0.0.0-alpha-3045-20260402093025",
"@tiket/passport-calendar": "0.0.0-alpha-3045-20260402093025",
"@tiket/passport-design-tokens": "0.0.0-alpha-3045-20260402093025",
"@tiket/passport-icons": "0.0.0-alpha-3045-20260402093025",
"@tiket/passport-page-module": "0.0.0-alpha-3045-20260402093025",
"@tiket/passport-page-module-shared": "0.0.0-alpha-3045-20260402093025",
"@tiket/passport-utils": "0.0.0-alpha-3045-20260402093025",
// ... any other @tiket/passport-* packages in the project

// React Common group (apply same version to all react-common-* packages)
"@tiket/react-common-ab-test": "0.0.0-alpha-1029-20260406145129",
"@tiket/react-common-core-attributes": "0.0.0-alpha-1029-20260406145129",
"@tiket/react-common-error-handler": "0.0.0-alpha-1029-20260406145129",
"@tiket/react-common-jsi": "0.0.0-alpha-1029-20260406145129",
"@tiket/react-common-navigator-permission": "0.0.0-alpha-1029-20260406145129",
"@tiket/react-common-remote": "0.0.0-alpha-1029-20260406145129",
"@tiket/react-common-utilities": "0.0.0-alpha-1029-20260406145129",
// ... any other @tiket/react-common-* packages in the project
```

## Step 3: Handle known gaps

- `@tiket/react-common-eagle-eye`: may not have an alpha version yet. If not available, remove temporarily and re-add when stable.
- `@tiket/passport-corporate-interim-module`: kept at its current version in devDependencies (not part of the alpha set).

## Step 4: Import path changes

Search for and update these import paths:

```typescript
// Before
import { createLogger } from '@tiket/next';

// After
import { createLogger } from '@tiket/next/logger';
```

## Step 5: Update transpilePackages

In `next.config.ts`, ensure `transpilePackages` includes all Tiket packages that need compilation:

```typescript
transpilePackages: [
  ...passportPackages,  // dynamically from package.json
  '@tiket/next',
  '@tiket/react-common-remote',
  '@tiket/react-common-utilities',
  '@tiket/react-common-error-handler',
  '@tiket/react-common-ab-test',  // add if not present
],
```

## Step 6: For library repos (monorepos publishing packages)

If this is a library repo (like react-common or passport), also update:

- **peerDependencies**: `react` / `react-dom` to `^19.0.0`
- **peerDependencies**: `next` to `^16.0.0` (where applicable)
- **devDependencies**: `@tiket/passport` and `@tiket/passport-utils` to the alpha versions
- **Build scripts**: add `--skipLibCheck` to `tsc` invocations

## Step 7: Install and verify

Run `pnpm install` and verify the project builds.

## Checklist

- [ ] All `@tiket/next` packages use the correct alpha version
- [ ] All `@tiket/passport*` packages use the correct alpha version
- [ ] All `@tiket/react-common-*` packages use the correct alpha version
- [ ] Import path `@tiket/next` to `@tiket/next/logger` updated (for createLogger)
- [ ] `transpilePackages` updated in next.config
- [ ] `pnpm install` succeeds

## Follow-up

These are pre-release builds. Replace them with stable versions once the Tiket platform packages publish React 19 / Next 16 compatible stable releases.
