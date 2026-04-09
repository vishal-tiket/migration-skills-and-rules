---
name: migrate-react-query-v5
description: >-
  Migrate from react-query v3 to TanStack Query v5. Use when upgrading React
  Query, when the user says "migrate react-query", "upgrade to TanStack Query",
  "useQuery migration", "v3 to v5", or "react-query to tanstack".
---

# Migrate React Query v3 to TanStack Query v5

Replace `react-query` with `@tanstack/react-query` v5 and update all query/mutation call sites.

## Step 1: Update dependencies

### 1a. Swap the core package

```jsonc
// Remove from dependencies
"react-query": "^3.39.3"

// Add to dependencies
"@tanstack/react-query": "^5.90.20",
```

### 1b. Check for devtools usage

Search the codebase for any import of the old devtools entry point:

```
rg "react-query/devtools" --type ts --type tsx
```

**If any match is found**, also add the devtools package:

```jsonc
"@tanstack/react-query-devtools": "^5.91.2",
```

This is a **required** addition whenever the old `react-query/devtools` import existed — do not skip it.

## Step 2: Update provider

```typescript
// Before
import { QueryClient, QueryClientProvider } from 'react-query';

// After
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
```

## Step 3: Update Hydration (if using SSR)

```typescript
// Before
import { Hydrate } from 'react-query';
<Hydrate state={dehydratedState}>

// After
import { HydrationBoundary } from '@tanstack/react-query';
<HydrationBoundary state={dehydratedState}>
```

The `dehydratedState` type changes from `unknown` to `DehydratedState | undefined`.

## Step 4: Update Devtools

```typescript
// Before
import { ReactQueryDevtools } from 'react-query/devtools';
<ReactQueryDevtools position="bottom-left" />

// After
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
<ReactQueryDevtools buttonPosition="bottom-left" />
```

## Step 5: Migrate all query and mutation call sites

> **WARNING**: Do NOT rely only on `rg "from 'react-query'"` to find files -- by this point, Step 1 has already changed all imports to `@tanstack/react-query`. Use `rg "useQuery"` and the callback-specific searches below.

Search the codebase for all usages. Include patterns with generics (`<`) since calls like `useQuery<TData, TError>(key, fn, opts)` are easy to miss:

```
rg "from 'react-query'" --type ts --type tsx
rg "useQuery[<(]" --type ts --type tsx
rg "useMutation[<(]" --type ts --type tsx
rg "useInfiniteQuery[<(]" --type ts --type tsx
rg "useQueries[<(]" --type ts --type tsx
```

Apply the transforms documented in [transforms.md](transforms.md).

**After converting**, verify no positional-form calls remain. Any match here is a missed migration:

```
rg "useQuery<.*>\(\s*\[" --type ts --type tsx --multiline
rg "useQuery\(\s*\[" --type ts --type tsx
```

Key changes:
- **useQuery**: positional args to object form
- **useMutation**: positional args to object form
- **cacheTime** renamed to **gcTime**
- **onSuccess/onError/onSettled** all removed from useQuery (use useEffect to watch `data`/`error` instead; see transforms.md for patterns)
- **query.remove()** replaced with `queryClient.removeQueries()`
- **isLoading** renamed to **isPending** for mutations
- **keepPreviousData** replaced with `placeholderData: keepPreviousData`
- **useQueries**: array to `{ queries: [...] }`
- **useInfiniteQuery**: must add `initialPageParam`

**Type changes**:
- `UseQueryOptions` generics changed from `<TQueryFnData, TError, TData>` to `<TQueryFnData, TError, TData, TQueryKey>`. If any hook uses `as UseQueryOptions<...>` type assertions, update them to match v5 generics or remove the cast and let TypeScript infer.
- The default error type changed from `unknown` to `Error`. Casting `error as SomeType` may need an intermediate `unknown` cast: `(error as unknown) as SomeType`.

### 5b. Remove onSuccess / onError / onSettled from all useQuery calls

**CRITICAL**: These callbacks were removed from `useQuery` in v5. Leaving them causes **silent type degradation** -- `data` resolves to `{}` instead of your real type, cascading into dozens of downstream errors.

**Important**: `onSuccess`/`onError`/`onSettled` are still valid on `useMutation`. Only remove them from `useQuery`, `useInfiniteQuery`, and custom hooks wrapping these.

Search for all files still using these callbacks:

```
rg "onSuccess|onError|onSettled" --type ts --type tsx -l
```

For each match, determine if it is in a `useQuery` context or a `useMutation` context. For `useQuery`:
- Remove the callback from the `useQuery` options object
- Replace with `useEffect` watching `data` or `error` (see transforms.md Section 4)

**Wrapper hooks**: If a custom hook accepts `onSuccess` as a parameter and forwards it to `useQuery`, you must:
1. Remove `onSuccess` from the hook's **type definition / parameter interface**
2. Remove `onSuccess` from the `useQuery({...})` options inside the hook
3. Update all **callers** of that hook to use `useEffect` watching the returned `data` instead

**Verification** -- after migration, confirm zero `onSuccess`/`onError`/`onSettled` remain in any `useQuery` context:

```
rg "onSuccess|onError|onSettled" --type ts --type tsx | grep -v "useMutation\|mutate\|\.spec\.\|__tests__"
```

Review every remaining match. If it is inside a `useQuery` options object or a custom hook type that feeds into `useQuery`, it must be removed.

## Step 6: Update test files

- Change all `import ... from 'react-query'` to `@tanstack/react-query`
- Replace `cacheTime` with `gcTime` in test options
- Update `dehydratedState` defaults from `{}` to `undefined`
- For async APIs, change `mockReturnValue` to `mockResolvedValue`

## Step 7: Update test utilities (if file exists)

Only if `lib/test-utils.tsx` (or equivalent test utils file) exists. In `lib/test-utils.tsx`:

```typescript
// Before
import { QueryClient, QueryClientProvider } from 'react-query';

// After
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
```

## Checklist

- [ ] `react-query` removed, `@tanstack/react-query` and devtools added
- [ ] Provider import updated
- [ ] Hydrate replaced with HydrationBoundary (if applicable)
- [ ] Devtools import and props updated
- [ ] All useQuery calls converted to object form
- [ ] All useMutation calls converted to object form
- [ ] cacheTime renamed to gcTime everywhere
- [ ] `onSuccess`/`onError`/`onSettled` removed from ALL `useQuery` / `useInfiniteQuery` calls
- [ ] Custom wrapper hooks that accepted `onSuccess` as a parameter are refactored (type + implementation + callers)
- [ ] Verification search confirms zero `onSuccess`/`onError`/`onSettled` in `useQuery` contexts
- [ ] query.remove() replaced with queryClient.removeQueries()
- [ ] isLoading replaced with isPending for mutations
- [ ] keepPreviousData replaced with placeholderData
- [ ] useInfiniteQuery has initialPageParam
- [ ] Test files updated
- [ ] Test utilities updated
