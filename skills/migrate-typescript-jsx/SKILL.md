---
name: migrate-typescript-jsx
description: >-
  Update TypeScript configuration and add React 19 JSX compatibility layer. Use
  when updating tsconfig for React 19, when the user says "fix JSX types",
  "update tsconfig", "react-jsx transform", or "JSX namespace deprecated".
---

# Migrate TypeScript and JSX Transform

Update TypeScript configuration for React 19's modern JSX transform and add the JSX compatibility layer.

## Step 1: Update tsconfig.json (or tsconfig.base.json)

Change the `jsx` compiler option:

```jsonc
{
  "compilerOptions": {
    "jsx": "react-jsx"  // was "preserve"
  }
}
```

## Step 2: Update include array

Add Next 16 dev types and the JSX compatibility file:

```jsonc
{
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    ".next/dev/types/**/*.ts",   // add this
    "react-jsx.d.ts"              // add this
  ]
}
```

## Step 3: Create react-jsx.d.ts (REQUIRED)

This is a new required file -- without it, existing code using `JSX.Element` and `JSX.IntrinsicElements` will fail to compile after React 19. This is NOT optional.

Create this file at the repository root:

```typescript
/**
 * React 19 JSX Compatibility Layer
 *
 * React 19 deprecates the global JSX namespace in favor of React.JSX.
 * This declaration re-exports React.JSX as global JSX so existing code
 * using JSX.Element, JSX.IntrinsicElements, etc. continues to work.
 */
import type * as React from 'react';

export {};

declare global {
  namespace JSX {
    /** @deprecated Use `React.JSX.Element` instead. */
    type Element = React.JSX.Element;
    /** @deprecated Use `React.JSX.ElementClass` instead. */
    type ElementClass = React.JSX.ElementClass;
    /** @deprecated Use `React.JSX.ElementAttributesProperty` instead. */
    type ElementAttributesProperty = React.JSX.ElementAttributesProperty;
    /** @deprecated Use `React.JSX.ElementChildrenAttribute` instead. */
    type ElementChildrenAttribute = React.JSX.ElementChildrenAttribute;
    /** @deprecated Use `React.JSX.LibraryManagedAttributes` instead. */
    type LibraryManagedAttributes<C, P> = React.JSX.LibraryManagedAttributes<C, P>;
    /** @deprecated Use `React.JSX.IntrinsicAttributes` instead. */
    type IntrinsicAttributes = React.JSX.IntrinsicAttributes;
    /** @deprecated Use `React.JSX.IntrinsicClassAttributes` instead. */
    type IntrinsicClassAttributes<T> = React.JSX.IntrinsicClassAttributes<T>;
    /** @deprecated Use `React.JSX.IntrinsicElements` instead. */
    type IntrinsicElements = React.JSX.IntrinsicElements;
  }
}
```

## Step 4: Update next-env.d.ts (if file exists)

Update `next-env.d.ts` only if the file exists. Next.js will regenerate it on the next build if missing. Add the route types import and update the docs URL:

```typescript
/// <reference types="next" />
/// <reference types="next/image-types/global" />
/// <reference types="next/navigation-types/compat/navigation" />
import "./.next/types/routes.d.ts";

// NOTE: This file should not be edited
// see https://nextjs.org/docs/app/api-reference/config/typescript for more information.
```

## Step 5: Remove deprecated TypeScript options

If `tsconfig.json` contains `importsNotUsedAsValues`, remove it (deprecated in TypeScript 5):

```jsonc
// Remove this line
"importsNotUsedAsValues": "error"
```

## Step 6: For monorepos

If this is a monorepo, each package's `tsconfig.json` should include the root `react-jsx.d.ts`:

```jsonc
{
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "../../react-jsx.d.ts"  // relative path to root
  ]
}
```

Also update build scripts to include `react-jsx.d.ts` in their `include` array if using programmatic tsc builds.

## Checklist

- [ ] `jsx` changed from `"preserve"` to `"react-jsx"` in tsconfig
- [ ] `.next/dev/types/**/*.ts` added to `include`
- [ ] `react-jsx.d.ts` added to `include`
- [ ] `react-jsx.d.ts` file created at repo root
- [ ] `next-env.d.ts` updated with route types import
- [ ] `importsNotUsedAsValues` removed (if present)
- [ ] Monorepo package tsconfigs include root `react-jsx.d.ts` (if applicable)
- [ ] `tsc --noEmit` passes without JSX-related errors
