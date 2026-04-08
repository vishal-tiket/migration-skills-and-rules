# ESLint Flat Config Template

Copy and adapt this template as `eslint.config.cjs` at the project root.

```javascript
const js = require('@eslint/js');
const { fixupPluginRules } = require('@eslint/compat');
const nextPlugin = require('@next/eslint-plugin-next');
const tseslint = require('typescript-eslint');
const globals = require('globals');
const reactPlugin = require('eslint-plugin-react');
const reactHooksPlugin = require('eslint-plugin-react-hooks');

module.exports = [
  // --------------------------------------------------
  // Ignore generated / build output
  // --------------------------------------------------
  {
    ignores: [
      '.next/**',
      'node_modules/**',
      'dist/**',
      'build/**',
      'out/**',
      '.storybook/**',
      // Add project-specific ignores here
    ],
  },

  // --------------------------------------------------
  // Base ESLint recommended
  // --------------------------------------------------
  js.configs.recommended,

  // --------------------------------------------------
  // Next.js
  // --------------------------------------------------
  nextPlugin.configs.recommended,
  nextPlugin.configs['core-web-vitals'],

  // --------------------------------------------------
  // React + React Hooks (ESLint 10 compat via fixupPluginRules)
  // --------------------------------------------------
  {
    files: ['**/*.ts', '**/*.tsx', '**/*.js', '**/*.jsx'],
    plugins: {
      react: fixupPluginRules(reactPlugin),
      'react-hooks': fixupPluginRules(reactHooksPlugin),
    },
    languageOptions: {
      globals: {
        ...globals.browser,
        ...globals.node,
      },
    },
    settings: {
      react: { version: 'detect' },
    },
  },

  // --------------------------------------------------
  // TypeScript
  // --------------------------------------------------
  {
    files: ['**/*.ts', '**/*.tsx'],
    languageOptions: {
      parser: tseslint.parser,
    },
    plugins: {
      '@typescript-eslint': tseslint.plugin,
    },
    rules: {
      'no-undef': 'off',
      'no-unused-vars': 'off',
      'no-useless-assignment': 'warn',

      '@typescript-eslint/no-unused-vars': [
        'warn',
        {
          argsIgnorePattern: '^_',
          varsIgnorePattern: '^_',
          caughtErrorsIgnorePattern: '^_',
        },
      ],
      '@typescript-eslint/no-require-imports': 'off',
      '@typescript-eslint/ban-ts-comment': 'off',
      '@typescript-eslint/no-explicit-any': 'warn',
      '@typescript-eslint/no-empty-object-type': 'off',
      '@typescript-eslint/no-non-null-asserted-optional-chain': 'off',

      '@next/next/no-page-custom-font': 'off',

      // Enforce wrapper usage instead of direct imports
      'no-restricted-imports': [
        'warn',
        {
          paths: [
            {
              name: '@tanstack/react-query',
              importNames: ['useQuery', 'useInfiniteQuery'],
              message:
                'Use our useFetchQuery/useFetchInfiniteQuery wrappers instead.',
            },
            {
              name: 'next/router',
              importNames: ['useRouter'],
              message:
                'Use our useCustomRouter wrapper instead.',
            },
          ],
        },
      ],

      // Restrict dangerouslySetInnerHTML (adapt to your project)
      'no-restricted-syntax': [
        'warn',
        {
          selector: "JSXAttribute[name.name='dangerouslySetInnerHTML']",
          message:
            "Use the HTMLText component instead of dangerouslySetInnerHTML.",
        },
      ],

      'no-prototype-builtins': 'off',
      'no-useless-escape': 'off',
      'no-case-declarations': 'off',
      'no-extra-boolean-cast': 'off',
      'no-unsafe-optional-chaining': 'off',
      'no-redeclare': 'off',
      'prefer-const': 'warn',

      'no-console': ['warn', { allow: ['warn', 'error'] }],

      // React Compiler rules (from eslint-plugin-react-hooks ^7.0.1)
      // Turn off initially; enable incrementally as the codebase is ready.
      'react-hooks/set-state-in-effect': 'off',
      'react-hooks/refs': 'off',
      'react-hooks/immutability': 'off',
      'react-hooks/preserve-manual-memoization': 'off',
      'react-hooks/static-components': 'off',
    },
  },

  // --------------------------------------------------
  // Jest / test files
  // --------------------------------------------------
  {
    files: [
      '**/*.spec.ts',
      '**/*.spec.tsx',
      '**/*.test.ts',
      '**/*.test.tsx',
      '**/jest.setup.js',
      '**/__mocks__/**',
    ],
    languageOptions: {
      globals: {
        ...globals.jest,
        ...globals.node,
      },
    },
  },
];
```

## Customization notes

- **no-restricted-imports**: Adjust the wrapper names (`useFetchQuery`, `useCustomRouter`) to match your project's conventions. Remove if the project does not use wrappers.
- **no-restricted-syntax**: Adjust or remove the `dangerouslySetInnerHTML` rule if your project does not have an `HTMLText` component.
- **React Compiler rules**: These are shipped with `eslint-plugin-react-hooks ^7.0.1`. They are turned off by default. Set to `'warn'` for incremental adoption when the codebase is ready.
- **eslint-config-prettier**: If you need Prettier integration as a flat config entry, add it as the last item:

```javascript
const prettierConfig = require('eslint-config-prettier');
// or for ^10.x:
// const prettierConfig = require('eslint-config-prettier/flat');

module.exports = [
  // ... all other configs ...
  prettierConfig,
];
```

- **eslint-plugin-jsx-a11y**: If the project uses it, wrap with `fixupPluginRules`:

```javascript
const jsxA11yPlugin = require('eslint-plugin-jsx-a11y');

// In the React section:
plugins: {
  react: fixupPluginRules(reactPlugin),
  'react-hooks': fixupPluginRules(reactHooksPlugin),
  'jsx-a11y': fixupPluginRules(jsxA11yPlugin),
},
```

- **Project-specific global variables**: Add to `languageOptions.globals` as needed (e.g., `google`, `Nullish`).
