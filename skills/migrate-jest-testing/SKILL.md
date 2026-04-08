---
name: migrate-jest-testing
description: >-
  Update Jest, jsdom, and Testing Library for React 19 and Node 24. Use when
  fixing test failures after upgrading, when the user says "fix tests",
  "jest upgrade", "jsdom patch", "testing library migration", or "window.location
  mock broken".
---

# Migrate Jest and Testing for React 19

Update Jest, jsdom, and Testing Library to work with React 19, Node 24, and pnpm.

## Step 1: Update test dependencies

Update only packages that already exist in `devDependencies`. Add new ones only where noted:

```jsonc
{
  "jest-environment-jsdom": "^30.2.0",
  "jest-fixed-jsdom": "^0.0.11",            // add only if project uses jsdom test environment
  "@testing-library/react": "^16.3.2",      // upgrade only if already present
  "@testing-library/dom": "^10.4.0",        // add only if @testing-library/react is in the project
  "moo-color": "^1.0.2"                     // add only if jest-canvas-mock is in devDependencies
}
```

Remove (if present):

```
@testing-library/react-hooks
```

Use `renderHook` and `waitFor` from `@testing-library/react` instead.

## Step 2: Add pnpm overrides (if using jsdom)

Only add these if the project uses Jest with a jsdom test environment. In `package.json`, add:

```jsonc
{
  "pnpm": {
    "overrides": {
      "jsdom": "26.1.0",
      "html-encoding-sniffer": "4.0.0",
      "@exodus/bytes": "1.9.0"
    },
    "patchedDependencies": {
      "jsdom@26.1.0": "patches/jsdom@26.1.0.patch"
    }
  }
}
```

## Step 3: Create the jsdom patch (if using jsdom)

If the project uses jsdom and pnpm overrides were added in Step 2, create `patches/jsdom@26.1.0.patch` to make `window.location` mockable. Skip if the project already has a jsdom patch or does not use jsdom. See [patches-and-mocks.md](patches-and-mocks.md) for the full patch content.

This patch changes `window.location` and all `Location` prototype properties from `configurable: false` to `configurable: true`, which allows tests to `delete window.location` and assign a mock.

## Step 4: Create moo-color mock (if needed)

Only create `lib/__mocks__/moo-color.js` if `jest-canvas-mock` is in devDependencies and `moo-color` was added in Step 1. Skip entirely if the project does not use `jest-canvas-mock`. See [patches-and-mocks.md](patches-and-mocks.md) for the content.

This mock is needed because `jest-canvas-mock` depends on `moo-color`, which can fail to resolve under pnpm.

## Step 5: Update jest.config.mjs

Update the existing `jest.config.mjs` (or `jest.config.js`/`jest.config.ts`). If the moo-color mock was created in Step 4, add the moo-color module name mapping. Otherwise, skip the moo-color lines.

Key changes:

```javascript
import path from 'path';
import { fileURLToPath } from 'url';
import nextJest from 'next/jest.js';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
// Only include moo-color mapping if the mock was created in Step 4
const mooColorPath = path.join(__dirname, 'lib', '__mocks__', 'moo-color.js');

const config = {
  // ... existing config ...
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/$1',
    '^moo-color$': mooColorPath,       // only if moo-color mock exists
  },
  coveragePathIgnorePatterns: [
    // ... existing patterns ...
    'lib/__mocks__/*',                  // only if __mocks__ dir was created
  ],
};

export default async function finalConfig() {
  const nextJestConfig = await createJestConfig(config)();
  return {
    ...nextJestConfig,
    // Re-apply moo-color mapping only if moo-color mock exists
    moduleNameMapper: {
      ...nextJestConfig.moduleNameMapper,
      '^moo-color$': mooColorPath,     // only if moo-color mock exists
    },
    transformIgnorePatterns: [`<rootDir>/node_modules/(?!.pnpm)(?!(@tiket))`],
  };
}
```

## Step 6: Update jest.polyfills.js

Add the `performance.markResourceTiming` polyfill at the top:

```javascript
if (typeof globalThis.performance?.markResourceTiming !== 'function') {
  globalThis.performance = globalThis.performance || {};
  globalThis.performance.markResourceTiming = () => {};
}
```

Remove any duplicate `PerformanceObserver` or `markResourceTiming` polyfills from `jest.setup.js`.

## Step 7: Update jest.setup.js

If `@tiket/react-common-remote` is in the project's dependencies, add a global mock to prevent `__webpack_init_sharing__` errors. Skip if the project does not use this package:

```javascript
jest.mock('@tiket/react-common-remote', () => ({
  RemoteFooter: () => null,
  RemoteHeader: () => null,
  RemoteScripts: () => null,
  ReactRemoteProvider: ({ children }) => children,
  getReactRemoteProps: async () => ({ remoteModuleSsrProps: {}, remoteModuleVersion: '' }),
}));
```

## Step 8: React 19 test patterns

### Hoisted `<link>` and `<meta>` tags

React 19 hoists `<link>` and `<meta>` into `document.head`. Update assertions:

```typescript
// Before
expect(container.querySelector('link[rel="stylesheet"]')).toBeInTheDocument();

// After
expect(document.head.querySelector('link[rel="stylesheet"]')).toBeInTheDocument();
```

Add cleanup in `afterEach`:

```typescript
afterEach(() => {
  cleanup();
  document.head.innerHTML = '';
});
```

### Async APIs (cookies, headers)

If Next.js `cookies()` or `headers()` are now async, update mocks:

```typescript
// Before
jest.mocked(cookies).mockReturnValue(mockCookies);

// After
jest.mocked(cookies).mockResolvedValue(mockCookies);
```

### renderHook migration

```typescript
// Before
import { renderHook, waitFor } from '@testing-library/react-hooks';

// After
import { renderHook, waitFor } from '@testing-library/react';
```

## Checklist

- [ ] `jest-environment-jsdom` upgraded to ^30.2.0 (if already present)
- [ ] `jest-fixed-jsdom` added (if using jsdom environment)
- [ ] `@testing-library/react` upgraded to ^16.3.2 (if already present)
- [ ] `@testing-library/react-hooks` removed (if present)
- [ ] pnpm overrides added for jsdom 26.1.0 (if using jsdom)
- [ ] jsdom patch created at `patches/jsdom@26.1.0.patch` (if using jsdom)
- [ ] `lib/__mocks__/moo-color.js` created (if jest-canvas-mock is in project)
- [ ] `jest.config.mjs` updated with moo-color mapping (if moo-color mock was created)
- [ ] `jest.polyfills.js` has `markResourceTiming` polyfill (if file exists)
- [ ] `jest.setup.js` has `@tiket/react-common-remote` mock (if package is in dependencies)
- [ ] Test assertions updated for `document.head` hoisting
- [ ] Async API mocks use `mockResolvedValue`
- [ ] `renderHook` imported from `@testing-library/react`
