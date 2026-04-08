---
name: migrate-core-deps
description: >-
  Upgrade core framework dependencies for the React 19, Next 16, and Node 24
  migration. Use when starting a Tiket project migration, when the user says
  "upgrade dependencies", "migrate to React 19", "upgrade to Next 16", or
  "update core packages".
---

# Migrate Core Dependencies

Upgrade `package.json` dependencies for the React 19 / Next 16 / Node 24 stack.

## Step 1: Scan current versions

Read `package.json` and identify which of these packages exist and their current versions. Not every project uses every package.

## Step 2: Upgrade core framework

Update these in `dependencies`:

```jsonc
{
  "react": "^19.2.4",
  "react-dom": "^19.2.4",
  "next": "^16.2.2",
  "@next/bundle-analyzer": "^16.2.2",  // if present
  "webpack": "^5.90.3"                  // add only if project will use --webpack flag in build scripts (see migrate-next-config)
}
```

## Step 3: Upgrade type definitions

Update these in `devDependencies`:

```jsonc
{
  "@types/react": "^19.2.9",
  "@types/react-dom": "^19.2.3",
  "@types/node": "^24.0.3"
}
```

## Step 4: Add compatibility packages

If the project uses Zustand, add to `dependencies`:

```jsonc
{
  "use-sync-external-store": "^1.6.0"
}
```

And in `devDependencies`:

```jsonc
{
  "@types/use-sync-external-store": "^1.5.0"
}
```

## Step 5: Remove deprecated packages

Remove from `dependencies` (if present):

- `react-query` -- replaced by `@tanstack/react-query` (see `migrate-react-query-v5` skill)

Remove from `devDependencies` (if present):

- `@testing-library/react-hooks` -- `renderHook` is now in `@testing-library/react`

## Step 6: Upgrade testing library

Update in `devDependencies`:

```jsonc
{
  "@testing-library/react": "^16.3.2",
  "@testing-library/dom": "^10.4.0"    // add only if @testing-library/react is already in the project
}
```

## Step 7: Update Volta

In `package.json`, update the Volta pin:

```jsonc
{
  "volta": {
    "node": "24.14.1"
  }
}
```

## Step 8: Install

Run `pnpm install` to update the lockfile. Fix any peer dependency warnings.

## Checklist

- [ ] `react` and `react-dom` upgraded to ^19.2.4
- [ ] `next` upgraded to ^16.2.2
- [ ] `@next/bundle-analyzer` upgraded to ^16.2.2 (if present)
- [ ] `webpack ^5.90.3` added as explicit dependency (if using --webpack)
- [ ] Type definitions upgraded (`@types/react`, `@types/react-dom`, `@types/node`)
- [ ] `use-sync-external-store` added (if using Zustand)
- [ ] `react-query` removed (if present)
- [ ] `@testing-library/react-hooks` removed (if present)
- [ ] `@testing-library/react` upgraded to ^16.3.2
- [ ] Volta node pinned to 24.14.1
- [ ] `pnpm install` runs without errors

## Related skills

- `migrate-tiket-packages` -- upgrade @tiket/* internal packages
- `migrate-next-config` -- update next.config for Next 16
- `migrate-typescript-jsx` -- update tsconfig and JSX transform
