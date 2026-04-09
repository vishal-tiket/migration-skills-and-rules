---
name: migrate-next-config
description: >-
  Convert next.config.js to next.config.ts and apply Next 16 configuration
  changes. Use when upgrading Next.js config, when the user says "update next
  config", "migrate next.config", "convert to TypeScript config", or "Next 16
  config changes".
---

# Migrate Next.js Configuration

Convert `next.config.js` to `next.config.ts` and apply all Next 16 required changes.

## Step 1: Rename config file

If `next.config.js` exists, rename to `next.config.ts`. If the project already has `next.config.ts` or `next.config.mjs`, skip the rename and adapt the existing file in the following steps instead.

```bash
mv next.config.js next.config.ts
```

## Step 2: Convert to TypeScript ESM

Replace `require()` calls with ESM imports and add types:

```typescript
// Before (CommonJS)
// @ts-check
const bundleAnalyzer = require('@next/bundle-analyzer');
const webpack = require('webpack');
const pkg = require('./package.json');

// After (TypeScript ESM)
import bundleAnalyzer from '@next/bundle-analyzer';
import webpack from 'webpack';
import type { NextConfig } from 'next';
import pkg from './package.json';

// Keep require() for packages that only expose CJS or have no type declarations.
// Known untyped/CJS-only packages: next-build-id, @tiket/next/*, @tiket/passport-assets/*
const nextBuildId = require('next-build-id');
const SubresourceIntegrity = require('@tiket/next/webpack-subresource-integrity');
const { getAssetPrefix, getRewrites } = require('@tiket/next/configs/config.cjs');
```

**Important:** After converting imports, verify each package has TypeScript types (bundled `index.d.ts` or a matching `@types/*` package). If a package has no types, keep it as `require()` to avoid "Could not find a declaration file" errors in the `.ts` config file.

## Step 3: Update config function signature

```typescript
// Before
const nextConfig = (phase, { defaultConfig }) => { ... };
module.exports = nextConfig;

// After
const nextConfig = (phase: string) => {
  const nextConfigObj: NextConfig = { ... };
  return withBundleAnalyzer(nextConfigObj);
};
export default nextConfig;
```

Next 16 no longer passes `defaultConfig` as the second argument.

## Step 4: Replace images.domains with remotePatterns

```typescript
// Before
images: {
  domains: ['lightroom.tiket.com', 'cdn.tiket.com'],
},

// After
images: {
  remotePatterns: [
    { protocol: 'https' as const, hostname: 'lightroom.tiket.com' },
    { protocol: 'https' as const, hostname: 'cdn.tiket.com' },
  ],
  // or dynamically:
  remotePatterns: TIKET_LIGHTROOM_HOSTNAMES.map((hostname: string) => ({
    protocol: 'https' as const,
    hostname,
  })),
},
```

## Step 5: Update experimental options

```typescript
experimental: {
  // REMOVE instrumentationHook (no longer supported)
  // instrumentationHook: true,  <-- delete this

  // ADD cssChunking for consistent CSS loading order
  cssChunking: 'strict',

  // Keep existing options like optimizePackageImports
  optimizePackageImports: passportPackages,
},
```

## Step 6: Add allowedDevOrigins (if needed)

For Next 16 dev server CORS when using custom local domains:

```typescript
allowedDevOrigins: ['local.tiket.com'],
```

## Step 7: Fix rewrite path syntax

Next 16 uses a stricter `path-to-regexp`. Update any bare repeat params:

```typescript
// Before
{ source: '/workbox-:path*.js', destination: '...' }

// After
{ source: '/workbox-:path(.*).js', destination: '...' }
```

Apply this to all rewrites/redirects using `:param*`.

## Step 8: Update package.json scripts

```jsonc
{
  "build": "next build --webpack",
  "dev": "source .env.local && next dev --experimental-https ... --webpack",
  "start": "next start --webpack",
  "lint:js": "eslint ."
}
```

The `--webpack` flag is required because Next 16 defaults to Turbopack, which does not support Module Federation or SRI plugins.

The `lint:js` script changes from `next lint` (or `oxlint`) to `eslint .` for the new flat config.

## Step 9: Update tsconfig include

Only add `next.config.ts` to `tsconfig.json` `include` if the file was renamed in Step 1 and is not already covered by `**/*.ts`:

```jsonc
"include": [
  "next-env.d.ts",
  "**/*.ts",
  "**/*.tsx",
  ".next/types/**/*.ts",
  "next.config.ts"
]
```

## Checklist

- [ ] `next.config.js` renamed to `next.config.ts` (or existing .ts/.mjs adapted)
- [ ] `require()` converted to ESM imports with types
- [ ] Untyped/CJS-only packages (e.g. `next-build-id`) kept as `require()`
- [ ] Config function signature updated (no `defaultConfig`)
- [ ] `images.domains` replaced with `images.remotePatterns`
- [ ] `experimental.instrumentationHook` removed
- [ ] `experimental.cssChunking: 'strict'` added
- [ ] `allowedDevOrigins` added (if using custom local domains)
- [ ] Rewrite syntax updated (`:path*` to `:path(.*)`)
- [ ] `--webpack` flag added to build/dev/start scripts
- [ ] `lint:js` script updated to `eslint .`
