---
name: migrate-react19-code-fixes
description: >-
  Apply React 19 component and type fixes across the codebase. Use when fixing
  TypeScript errors after upgrading to React 19, when the user says "fix React
  19 types", "RefObject errors", "child.props typing", "useRef undefined",
  "hydration mismatch", or "React 19 breaking changes".
---

# React 19 Component and Type Fixes

Fix TypeScript errors and runtime issues introduced by React 19's stricter types and behavior changes.

## 1. RefObject type: add `| null`

React 19 changed `RefObject<T>` to include `null` in `current`. Update all ref types:

```typescript
// Before
ref: RefObject<HTMLDivElement>
ref: RefObject<ImperativeHandle>

// After
ref: RefObject<HTMLDivElement | null>
ref: RefObject<ImperativeHandle | null>
```

**Search pattern:**

```
rg "RefObject<" --type ts --type tsx
```

This affects stores, types files, and any component accepting refs as props.

## 2. useRef: explicit initial value

React 19 requires explicit initial values for `useRef`:

```typescript
// Before
const ref = useRef<number>();
const timerRef = useRef<NodeJS.Timeout>();

// After
const ref = useRef<number | undefined>(undefined);
const timerRef = useRef<NodeJS.Timeout | undefined>(undefined);
```

**Search pattern:**

```
rg "useRef<.*>\(\)" --type ts --type tsx
```

## 3. child.props typing

React 19 makes `child.props` access stricter on `ReactElement`. Cast to `Record<string, unknown>`:

```typescript
// Before
if (isValidElement(child) && typeof child.props['stackKey'] === 'string') {
  const key = child.props['stackKey'];
}

// After
if (
  isValidElement(child) &&
  typeof (child.props as Record<string, unknown>)['stackKey'] === 'string'
) {
  const key = (child.props as Record<string, unknown>)['stackKey'] as string;
}
```

## 4. cloneElement typing

When using `cloneElement`, explicitly type the props argument:

```typescript
// Before
cloneElement(child, { isActive: true, onClick: handler })

// After
type ChildProps = { isActive: boolean; onClick: () => void };
cloneElement(child, { isActive: true, onClick: handler } as Partial<ChildProps>)
```

## 5. Image src type safety

Next 16's `Image` component has stricter `src` typing:

```typescript
// Before
<Image src={src} alt={alt} fill />

// After
<Image src={typeof src === 'string' ? src : ''} alt={alt} fill />
```

## 6. @loadable/component to React.lazy + Suspense

Replace `@loadable/component` with React's built-in lazy loading:

```typescript
// Before
import loadable from '@loadable/component';
const Component = loadable(() => import('./module'), {
  resolveComponent: (m) => m.NamedExport,
});
<Component />

// After
import { lazy, Suspense } from 'react';
const Component = lazy(() =>
  import('./module').then((m) => ({ default: m.NamedExport })),
);
<Suspense fallback={null}>
  <Component />
</Suspense>
```

Remove `@loadable/component` and `@types/loadable__component` from dependencies.

## 7. Hydration fixes

### Move side effects from render to useEffect

```typescript
// Before (runs during render, causes SSR/client mismatch)
const isInitialLoad = useRef(false);
if (!isInitialLoad.current && typeof window !== 'undefined') {
  handleResize();
  isInitialLoad.current = true;
}

// After (runs after mount only)
useEffect(() => {
  handleResize();
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

### Use factory functions for Date-based defaults

```typescript
// Before (module-level, evaluated once — SSR/client mismatch)
export const DEFAULT_DATE = { start: new Date(), end: addDays(new Date(), 1) };

// After (factory function for fresh dates)
export const getDefaultDate = () => ({
  start: new Date(),
  end: addDays(new Date(), 1),
});

// In component: useState(getDefaultDate)
```

## 8. useReducer: remove explicit generic

```typescript
// Before (may cause type mismatch with React 19's updated signatures)
const [state, dispatch] = useReducer<MyReducerType>(reducer, initialState);

// After (let TypeScript infer from the reducer function)
const [state, dispatch] = useReducer(reducer, initialState);
```

## 9. Boolean coercion in className

```typescript
// Before (falsy value 0 would render "0" in DOM)
className={rightSlot && styles.has_icon}

// After
className={!!rightSlot && styles.has_icon}
```

## 10. Remove unused import React

With `jsx: "react-jsx"`, files that only use JSX no longer need the React import:

```typescript
// Before
import React from 'react';

// After (remove if only JSX is used; keep if using React.FC, React.ReactNode, etc.)
```

**Search pattern:**

```
rg "import React from 'react'" --type ts --type tsx
```

Keep the import only if the file references `React.*` (e.g., `React.ReactNode`, `React.FC`, `React.memo`).

## 11. Design system / TDS type gaps

If the design system types don't yet include certain tokens (e.g., color `C100`), add `@ts-expect-error` with a TODO:

```typescript
// @ts-expect-error - TODO: confirm with TDS why C100 is not supported
bgColor="C100"
```

Remove these once the design system publishes updated types.

## 12. NodeJS.Timer type

If using `@types/node` v24, `NodeJS.Timer` no longer exists:

```typescript
// Before
let interval: NodeJS.Timer;

// After
let interval: NodeJS.Timeout | null;
```

## Checklist

- [ ] `RefObject<T>` updated to `RefObject<T | null>` across the codebase
- [ ] `useRef<T>()` updated to `useRef<T | undefined>(undefined)`
- [ ] `child.props` access casted where needed
- [ ] `cloneElement` calls have explicit type casts
- [ ] `Image src` type guarded
- [ ] `@loadable/component` replaced with `React.lazy` + `Suspense` (if applicable)
- [ ] Hydration issues fixed (side effects moved to useEffect)
- [ ] `useReducer` explicit generics removed
- [ ] Boolean coercion fixed in className
- [ ] Unused `import React` removed
- [ ] TDS type gaps documented with `@ts-expect-error`
- [ ] `NodeJS.Timer` replaced with `NodeJS.Timeout`
- [ ] `tsc --noEmit` passes
