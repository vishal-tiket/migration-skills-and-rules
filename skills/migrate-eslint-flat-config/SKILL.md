---
name: migrate-eslint-flat-config
description: >-
  Migrate to ESLint 10 flat config with Prettier. Use when upgrading ESLint,
  when the user says "migrate eslint", "flat config", "eslint 10", "replace
  oxlint", "replace eslintrc", or "upgrade linting".
---

# Migrate to ESLint 10 Flat Config + Prettier

Replace legacy ESLint config (`.eslintrc.json` / `.eslintrc`) or oxlint/oxfmt with ESLint 10 flat config and Prettier.

## Step 1: Remove old config and packages

Delete only those files that exist in the project. Skip any that are not present:

- `.eslintrc.json`
- `.eslintrc`
- `.eslintrc.js`
- `.oxlintrc.json`
- `.oxfmtrc.json`

Remove only packages that are currently in `devDependencies`:

```
eslint-config-next
oxlint
@oxlint/binding-darwin-arm64
oxfmt
@oxfmt/binding-darwin-arm64
```

## Step 2: Add new packages

Add to `devDependencies`:

```jsonc
{
  "eslint": "^10.0.3",
  "@eslint/js": "^10.0.1",
  "@eslint/compat": "^2.0.2",
  "@next/eslint-plugin-next": "^16.2.2",
  "eslint-plugin-react": "^7.37.5",
  "eslint-plugin-react-hooks": "^7.0.1",
  "eslint-config-prettier": "^9.0.0",
  "typescript-eslint": "^8.57.0",
  "globals": "^17.2.0",
  "prettier": "^3.2.5",
  "@ianvs/prettier-plugin-sort-imports": "^4.3.1"
}
```

Optional additions based on project needs:

```jsonc
{
  "eslint-plugin-jsx-a11y": "^6.10.2",
  "eslint-plugin-storybook": "^9.1.10"
}
```

## Step 3: Create eslint.config.cjs

Create the flat config file. See [eslint-config-template.md](eslint-config-template.md) for the full template.

The template includes:
- Ignores for `.next/`, `node_modules/`, etc.
- `@eslint/js` recommended rules
- `@next/eslint-plugin-next` recommended + core-web-vitals
- React and React Hooks via `fixupPluginRules` (ESLint 10 compat)
- TypeScript via `typescript-eslint`
- Custom restricted imports (use wrappers for useQuery, useRouter)
- React Compiler rules turned off
- Jest globals for test files

## Step 4: Create Prettier config (if applicable)

If the project already has a Prettier config (`.prettierrc`, `prettier.config.js`, etc.), update it instead of creating a new one. If the project does not use Prettier and is not adding it in this migration, skip this step entirely. If Prettier was added in Step 2, create these files:

Create `.prettierrc`:

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "plugins": ["@ianvs/prettier-plugin-sort-imports"]
}
```

Create `.prettierignore`:

```
.next
node_modules
dist
build
out
coverage
pnpm-lock.yaml
```

## Step 5: Update lint-staged (if file exists)

If `.lintstagedrc.js` (or equivalent lint-staged config) exists, update it. Otherwise skip this step.

Update `.lintstagedrc.js`:

```javascript
const path = require('path');

module.exports = {
  '**/*.ts?(x)': () =>
    'bash -c \'OUTPUT=$(tsc --noEmit --pretty 2>&1); ERRORS=$(echo "$OUTPUT" | grep -v node_modules | grep "error TS"); if [ -n "$ERRORS" ]; then echo "$OUTPUT" | grep -v node_modules; exit 1; fi\'',

  '**/*.{js,jsx,ts,tsx}': (filenames) => {
    const files = filenames.map((f) => `"${path.relative(process.cwd(), f)}"`);
    const chunkSize = 30;
    const chunks = [];
    for (let i = 0; i < files.length; i += chunkSize) {
      chunks.push(files.slice(i, i + chunkSize));
    }
    return chunks.map((chunk) => `eslint --fix ${chunk.join(' ')}`);
  },

  '**/*.{js,jsx,ts,tsx,css,scss,md,mdx}': 'prettier --write',

  '**/*.{css,scss}': 'stylelint --fix',
};
```

## Step 6: Update sonar-project.properties (if file exists)

If `sonar-project.properties` exists, update it. Otherwise skip this step:

```properties
sonar.eslint.eslintconfigpath=eslint.config.cjs
```

## Step 7: Update ESLint local rules (if present)

If the project has custom ESLint rules, update the ESLint 10 API:

```javascript
// Before (ESLint 8)
context.getSourceCode()

// After (ESLint 10)
context.sourceCode
```

Also update inline disable comments if plugin IDs changed:

```typescript
// Before
// eslint-disable-next-line local-rules/no-dangerously-set-inner-html

// After
// eslint-disable-next-line local/no-dangerously-set-inner-html
```

## Step 8: Update package.json scripts

```jsonc
{
  "lint:js": "eslint .",
  "format": "prettier . --write || true"
}
```

## Checklist

- [ ] Old config files deleted (`.eslintrc*`, `.oxlintrc.json`, `.oxfmtrc.json`)
- [ ] Old packages removed (`eslint-config-next`, `oxlint`, `oxfmt`)
- [ ] New ESLint 10 packages added
- [ ] `eslint.config.cjs` created
- [ ] `.prettierrc` and `.prettierignore` created or updated (if applicable)
- [ ] `.lintstagedrc.js` updated (if file exists)
- [ ] `sonar-project.properties` updated (if file exists)
- [ ] ESLint local rules updated for ESLint 10 API (if applicable)
- [ ] `lint:js` and `format` scripts updated
- [ ] `pnpm run lint:js` passes (or shows only warnings)
