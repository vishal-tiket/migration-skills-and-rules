---
name: migrate-zustand-v5
description: >-
  Migrate Zustand stores from v4 to v5 with shallow equality. Use when
  upgrading Zustand, when the user says "migrate zustand", "upgrade zustand",
  "zustand v5", or "store re-renders after upgrade".
---

# Migrate Zustand 4 to 5

Update all Zustand stores to v5 with `createWithEqualityFn` and `shallow` equality to preserve existing re-render behavior.

## Why this matters

Zustand 5 changed `create` to use **strict reference equality** by default. Selectors that return objects (e.g., `(state) => ({ a: state.a, b: state.b })`) would re-render on every state change, even when the values are the same. Using `createWithEqualityFn` with `shallow` restores Zustand 4's behavior.

## Step 1: Update dependencies

```jsonc
{
  "dependencies": {
    "zustand": "^5.0.10",
    "use-sync-external-store": "^1.6.0"
  },
  "devDependencies": {
    "@types/use-sync-external-store": "^1.5.0"
  }
}
```

## Step 2: Find all stores

Search for all Zustand store files:

```
rg "from 'zustand'" --type ts --type tsx
rg "from \"zustand\"" --type ts --type tsx
rg "import.*create.*from.*zustand" --type ts --type tsx
```

## Step 3: Migrate standalone stores (create pattern)

```typescript
// Before (Zustand 4)
import { create } from 'zustand';
import { devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';

export const useMyStore = create<MyStoreType>()(
  devtools(
    immer((...args) => ({
      // slices
    })),
    { name: 'myStore', enabled: isDev }
  )
);

// After (Zustand 5)
import { devtools } from 'zustand/middleware';
import { immer } from 'zustand/middleware/immer';
import { shallow } from 'zustand/shallow';
import { createWithEqualityFn } from 'zustand/traditional';

export const useMyStore = createWithEqualityFn<MyStoreType>()(
  devtools(
    immer((...args) => ({
      // slices
    })),
    { name: 'myStore', enabled: isDev }
  ),
  shallow  // <-- add as second argument
);
```

## Step 4: Migrate context-based stores (createStore + useStore pattern)

```typescript
// Before (Zustand 4)
import { createStore, useStore } from 'zustand';

const store = createStore<StoreType>()((set) => ({ ... }));

export function useMyStore<T>(selector: (state: StoreType) => T, equalityFn?: (a: T, b: T) => boolean) {
  return useStore(store, selector, equalityFn);
}

// After (Zustand 5)
import { createStore } from 'zustand';
import { shallow } from 'zustand/shallow';
import { useStoreWithEqualityFn } from 'zustand/traditional';

const store = createStore<StoreType>()((set) => ({ ... }));

export function useMyStore<T>(selector: (state: StoreType) => T, equalityFn?: (a: T, b: T) => boolean) {
  return useStoreWithEqualityFn(store, selector, equalityFn ?? shallow);
}
```

> **WARNING**: `createStore` in v5 requires the curried form with empty `()()`. A common mistake:
> ```typescript
> // WRONG -- missing curried ()
> const store = createStore<StoreType>((set) => ({ ... }));
>
> // CORRECT
> const store = createStore<StoreType>()((set) => ({ ... }));
> ```
> Search for this mistake: `rg "createStore<.*>[^(]\(" --type ts --type tsx`

## Step 5: Migrate simple stores without middleware

```typescript
// Before
import { create } from 'zustand';
export const useSimpleStore = create<SimpleType>()((set) => ({
  value: 0,
  setValue: (v: number) => set({ value: v }),
}));

// After
import { shallow } from 'zustand/shallow';
import { createWithEqualityFn } from 'zustand/traditional';

export const useSimpleStore = createWithEqualityFn<SimpleType>()(
  (set) => ({
    value: 0,
    setValue: (v: number) => set({ value: v }),
  }),
  shallow
);
```

## Step 6: Fix Immer middleware set parameter types (if applicable)

Only apply if the project uses `zustand/middleware/immer`. In Zustand 5, if store helper functions accept the `set` parameter directly and pass it around (e.g., to slice creators or helper functions), the type of `set` changes and may not match the old type annotations.

```typescript
// Before (v4): set type was inferred as the store's SetState
type ImmerSetParam = (fn: (state: Draft<MyStore>) => void) => void;
const helper = (set: ImmerSetParam) => { ... };

// After (v5): the set parameter type from immer middleware changed.
// Prefer letting TypeScript infer the type rather than annotating manually.
// If you must annotate, use the store's actual set type:
const helper = (set: Parameters<Parameters<typeof createWithEqualityFn<MyStore>>[0]>[0]) => { ... };

// Or more practically, inline the helper into the store creator:
export const useMyStore = createWithEqualityFn<MyStore>()(
  devtools(
    immer((set, get) => ({
      // define actions here directly instead of splitting into helper functions
      myAction: () => set((state) => { state.value = 1; }),
    })),
  ),
  shallow
);
```

**Search pattern:**

```
rg "ImmerSetParam|immer.*set.*:" --type ts --type tsx
```

If helper functions take `set` as a parameter with a custom type annotation, either:
1. Remove the type annotation and let it be inferred
2. Or inline the helper into the store creator

## Checklist

- [ ] `zustand` upgraded to ^5.0.10
- [ ] `use-sync-external-store` added
- [ ] All `create` calls changed to `createWithEqualityFn` from `zustand/traditional`
- [ ] All `useStore` calls changed to `useStoreWithEqualityFn` from `zustand/traditional`
- [ ] `shallow` imported from `zustand/shallow` and passed as equality function
- [ ] No remaining imports from `'zustand'` top-level `create` (except `createStore` which stays)
- [ ] `createStore` uses curried form `createStore<T>()((set) => ...)`
- [ ] Immer middleware `set` parameter types updated or inferred (if applicable)
- [ ] App tested: no unexpected re-render loops
