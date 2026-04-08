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

## Checklist

- [ ] `zustand` upgraded to ^5.0.10
- [ ] `use-sync-external-store` added
- [ ] All `create` calls changed to `createWithEqualityFn` from `zustand/traditional`
- [ ] All `useStore` calls changed to `useStoreWithEqualityFn` from `zustand/traditional`
- [ ] `shallow` imported from `zustand/shallow` and passed as equality function
- [ ] No remaining imports from `'zustand'` top-level `create` (except `createStore` which stays)
- [ ] App tested: no unexpected re-render loops
