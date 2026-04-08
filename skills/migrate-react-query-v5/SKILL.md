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

```jsonc
// Remove from dependencies
"react-query": "^3.39.3"

// Add to dependencies
"@tanstack/react-query": "^5.90.20",
"@tanstack/react-query-devtools": "^5.91.2"  // only if react-query/devtools was previously used
```

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

Search the codebase for all usages:

```
rg "from 'react-query'" --type ts --type tsx
rg "useQuery\(" --type ts --type tsx
rg "useMutation\(" --type ts --type tsx
rg "useInfiniteQuery\(" --type ts --type tsx
rg "useQueries\(" --type ts --type tsx
```

Apply the transforms documented in [transforms.md](transforms.md).

Key changes:
- **useQuery**: positional args to object form
- **useMutation**: positional args to object form
- **cacheTime** renamed to **gcTime**
- **onSuccess/onError/onSettled** removed from useQuery (use useEffect + useRef)
- **query.remove()** replaced with `queryClient.removeQueries()`
- **isLoading** renamed to **isPending** for mutations
- **keepPreviousData** replaced with `placeholderData: keepPreviousData`
- **useQueries**: array to `{ queries: [...] }`
- **useInfiniteQuery**: must add `initialPageParam`

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
- [ ] onSuccess/onError callbacks migrated to useEffect pattern
- [ ] query.remove() replaced with queryClient.removeQueries()
- [ ] isLoading replaced with isPending for mutations
- [ ] keepPreviousData replaced with placeholderData
- [ ] useInfiniteQuery has initialPageParam
- [ ] Test files updated
- [ ] Test utilities updated
