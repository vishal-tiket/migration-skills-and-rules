# React Query v3 to v5 Code Transforms

Detailed before/after code patterns for every API change.

## 1. useQuery: Positional to object form

```typescript
// Before (v3)
useQuery(queryKey, queryFn, { staleTime: 5000, enabled: true })

// After (v5)
useQuery({ queryKey, queryFn, staleTime: 5000, enabled: true })
```

With types:

```typescript
// Before
useQuery<TData, TError>(queryKey, queryFn, options)

// After
useQuery<TData, TError>({ queryKey, queryFn, ...options })
```

## 2. useMutation: Positional to object form

```typescript
// Before (v3)
useMutation<TData, TError, TVariables>(mutationFn, {
  onSuccess: (data) => { ... },
  onError: (error) => { ... },
})

// After (v5)
useMutation<TData, TError, TVariables>({
  mutationFn,
  onSuccess: (data) => { ... },
  onError: (error) => { ... },
})
```

## 3. cacheTime renamed to gcTime

```typescript
// Before
useQuery(key, fn, { cacheTime: 1000 * 60 * 5 })
new QueryClient({ defaultOptions: { queries: { cacheTime: 0 } } })

// After
useQuery({ queryKey: key, queryFn: fn, gcTime: 1000 * 60 * 5 })
new QueryClient({ defaultOptions: { queries: { gcTime: 0 } } })
```

## 4. onSuccess / onError / onSettled removal from useQuery

v5 removes these callbacks from `useQuery`. Use `useEffect` + `useRef` for stable callbacks:

### Pattern: onSuccess replacement

```typescript
// Before
const { data } = useQuery(key, fn, {
  onSuccess: (data) => {
    doSomething(data);
  },
});

// After
const onSuccessRef = useRef(onSuccess);
onSuccessRef.current = onSuccess;

const { data } = useQuery({ queryKey: key, queryFn: fn });

useEffect(() => {
  if (data !== undefined && onSuccessRef.current) {
    onSuccessRef.current(data);
  }
}, [data]);
```

### Pattern: onError replacement

```typescript
// Before
const { data } = useQuery(key, fn, {
  onError: (error) => {
    logger.error('fetch failed', error);
  },
});

// After
const query = useQuery({ queryKey: key, queryFn: fn });

useEffect(() => {
  if (query.error) {
    logger.error('fetch failed', query.error);
  }
}, [query.error]);
```

### Pattern: Wrapper hook with ref-based callbacks (useFetchQuery style)

For projects with a shared query wrapper, the recommended pattern:

```typescript
export const useFetchQuery = <TData>(
  queryKey: QueryKey,
  queryFn: QueryFunction<TData>,
  options?: { onSuccess?: (data: TData) => void; onError?: (err: Error) => void; /* ... */ },
) => {
  const onSuccessRef = useRef(options?.onSuccess);
  onSuccessRef.current = options?.onSuccess;

  const onErrorRef = useRef(options?.onError);
  onErrorRef.current = options?.onError;

  const stableSelect = useCallback(
    (data: TData) => (options?.select ? options.select(data) : data),
    // eslint-disable-next-line react-hooks/exhaustive-deps
    [],
  );

  const query = useQuery({
    queryKey,
    queryFn,
    ...options,
    select: stableSelect,
  });

  useEffect(() => {
    if (query.data !== undefined) onSuccessRef.current?.(query.data);
  }, [query.data]);

  useEffect(() => {
    if (query.error) onErrorRef.current?.(query.error);
  }, [query.error]);

  return query;
};
```

## 5. query.remove() replaced

```typescript
// Before
const { remove: removeCache } = useQuery(key, fn);
removeCache();

// After
const queryClient = useQueryClient();
const removeCache = useCallback(() => {
  queryClient.removeQueries({ queryKey: key });
}, [queryClient, key]);
```

## 6. isLoading renamed to isPending (mutations)

```typescript
// Before
const { isLoading, mutate } = useMutation(fn);

// After
const { isPending, mutate } = useMutation({ mutationFn: fn });

// If the return interface must stay stable, alias it:
const mutation = useMutation({ mutationFn: fn });
return { ...mutation, isLoading: mutation.isPending };
```

## 7. keepPreviousData

```typescript
// Before
useQuery(key, fn, { keepPreviousData: true })

// After
import { keepPreviousData } from '@tanstack/react-query';
useQuery({ queryKey: key, queryFn: fn, placeholderData: keepPreviousData })
```

## 8. useQueries

```typescript
// Before
useQueries([
  { queryKey: key1, queryFn: fn1 },
  { queryKey: key2, queryFn: fn2 },
])

// After
useQueries({
  queries: [
    { queryKey: key1, queryFn: fn1 },
    { queryKey: key2, queryFn: fn2 },
  ],
})
```

## 9. useInfiniteQuery

```typescript
// Before
useInfiniteQuery(key, fn, {
  getNextPageParam: (lastPage) => lastPage.nextCursor,
})

// After
useInfiniteQuery({
  queryKey: key,
  queryFn: fn,
  initialPageParam: 1,  // required in v5
  getNextPageParam: (lastPage) => lastPage.nextCursor,
})
```

## 10. Import updates for tests

```typescript
// Before
import { QueryClient } from 'react-query';
const queryClient = new QueryClient({ defaultOptions: { queries: { cacheTime: 0 } } });

// After
import { QueryClient } from '@tanstack/react-query';
const queryClient = new QueryClient({ defaultOptions: { queries: { gcTime: 0 } } });
```
