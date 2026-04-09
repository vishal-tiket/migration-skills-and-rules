# Cursor Rules Organization

This directory contains modular Cursor AI rules that guide code generation and modifications.

## 📁 Directory Structure

```
.cursor/
├── README.md
├── rules/
│   └── migration.mdc                          # Migration orchestrator (React 19 / Next 16 / Node 24)
└── skills/
    ├── migrate-core-deps/SKILL.md             # Phase 1: Core framework dependencies
    ├── migrate-tiket-packages/SKILL.md        # Phase 1: @tiket/* alpha packages
    ├── migrate-next-config/SKILL.md           # Phase 2: next.config.js → next.config.ts
    ├── migrate-typescript-jsx/SKILL.md        # Phase 2: tsconfig + JSX compat layer
    ├── migrate-react-query-v5/
    │   ├── SKILL.md                           # Phase 3: react-query v3 → TanStack v5
    │   └── transforms.md                      # Detailed code transform patterns
    ├── migrate-zustand-v5/SKILL.md            # Phase 3: Zustand v4 → v5
    ├── migrate-eslint-flat-config/
    │   ├── SKILL.md                           # Phase 4: ESLint 10 flat config + Prettier
    │   └── eslint-config-template.md          # Full eslint.config.cjs template
    ├── migrate-jest-testing/
    │   ├── SKILL.md                           # Phase 5: Jest 30 + jsdom 26 + Testing Library 16
    │   └── patches-and-mocks.md               # jsdom patch + moo-color mock
    ├── migrate-react19-code-fixes/SKILL.md    # Phase 6: React 19 type and code fixes
    └── migrate-ci-docker/SKILL.md             # Phase 7: Dockerfiles, GitHub Actions, Jenkins
```

## 🎯 Purpose

Cursor Rules help the AI understand:

- **Project-specific patterns**: How we structure code in THIS project
- **Standards & conventions**: Naming, file organization, best practices
- **Testing patterns**: How to write and fix tests
- **Migration guides**: Deprecated vs current approaches

## 📖 How Cursor Rules Work

### 1. **File Format**

- Rules use the `.mdc` extension (Cursor's native rule format with YAML frontmatter)
- The root `.cursorrules` file uses plain Markdown
- Skills use standard `.md` Markdown files

### 2. **File Location**

- **Root-level**: `.cursorrules` -- always-applied project rules
- **Modular rules**: `.cursor/rules/*.mdc` -- conditional rules triggered by description keywords
- **Skills**: `.cursor/skills/*/SKILL.md` -- step-by-step procedural guides

### 3. **How AI Uses Them**

- `.cursorrules` is read automatically on every interaction
- `.mdc` rules are activated when the user's prompt matches their trigger description
- Skills are invoked explicitly or by matching trigger keywords

## 📚 Current Rules

### Main Rules File

**Location**: `/.cursorrules`

- Core language and framework rules
- General testing patterns
- CSS/styling conventions
- TypeScript requirements

## 🔄 Stack Migration (React 19 / Next 16 / Node 24)

The `.cursor/skills/` directory contains **10 migration skills** orchestrated by a single rule that upgrades the project from React 18 + Next 14 + Node 20 to React 19 + Next 16 + Node 24.

### Migration Orchestrator

**Location**: `.cursor/rules/migration.mdc`

**Trigger keywords**: "migrate", "migration", "upgrade React", "upgrade Next", "React 19", "Next 16", "Node 24", "upgrade stack", "upgrade dependencies"

When triggered, the orchestrator reads and executes each skill in order, skipping phases that do not apply to the project (e.g., Zustand if not used, React Query if not used).

**Adaptive rule**: Before executing any step, the orchestrator checks whether the package, file, or config already exists. It never adds packages the project does not depend on, and never creates files the project does not need.

### Execution Phases

| Phase | Skill | What It Does |
|-------|-------|-------------|
| **1 -- Dependencies** | `migrate-core-deps` | Upgrades React, Next, Node, type definitions, Testing Library; removes deprecated packages |
| **1 -- Dependencies** | `migrate-tiket-packages` | Switches all `@tiket/*` packages to React 19 / Next 16 compatible alpha versions; updates import paths |
| **2 -- Configuration** | `migrate-next-config` | Converts `next.config.js` to `.ts`; applies `remotePatterns`, `cssChunking`, `--webpack` flag, stricter rewrite syntax |
| **2 -- Configuration** | `migrate-typescript-jsx` | Sets `jsx: "react-jsx"` in tsconfig; creates `react-jsx.d.ts` compatibility layer; updates `next-env.d.ts` |
| **3 -- Data & State** | `migrate-react-query-v5` | Replaces `react-query` v3 with `@tanstack/react-query` v5; converts all hooks to object form; removes `onSuccess`/`onError` from `useQuery` |
| **3 -- Data & State** | `migrate-zustand-v5` | Migrates stores from `create` to `createWithEqualityFn` + `shallow` to preserve re-render behavior |
| **4 -- Linting** | `migrate-eslint-flat-config` | Replaces `.eslintrc` / oxlint with ESLint 10 flat config (`eslint.config.cjs`) + Prettier |
| **5 -- Testing** | `migrate-jest-testing` | Upgrades Jest 30, jsdom 26, Testing Library 16; adds pnpm overrides and jsdom patch for `window.location` mocking |
| **6 -- Code Fixes** | `migrate-react19-code-fixes` | Fixes `RefObject<T \| null>`, `useRef` initial values, `child.props` casts, callback refs, hydration issues, and 10+ other React 19 patterns |
| **7 -- Infrastructure** | `migrate-ci-docker` | Updates all Dockerfiles, GitHub Actions, Jenkinsfile, and Sonar config to Node 24 |

### Supplementary Files

Three skills include companion documents with detailed reference content:

| Skill | File | Content |
|-------|------|---------|
| `migrate-react-query-v5` | `transforms.md` | 12 before/after code transform patterns (positional to object form, `cacheTime` to `gcTime`, `onSuccess` removal, `keepPreviousData`, error casting, etc.) |
| `migrate-eslint-flat-config` | `eslint-config-template.md` | Complete `eslint.config.cjs` template with TypeScript, React, Next.js, and Jest support |
| `migrate-jest-testing` | `patches-and-mocks.md` | Full `jsdom@26.1.0.patch` diff and `moo-color.js` mock (both conditional) |

### Execution Notes

- Run `pnpm install` after Phase 1 completes before continuing
- Verify the project builds (`pnpm run build`) or type-checks (`pnpm run type-check`) after each phase
- If a phase fails, fix the issues before proceeding to the next phase
- Each skill has a checklist at the bottom for verification
- The orchestrator reports which phases were applied and which were skipped

### Using Individual Skills

Each skill can also be triggered independently without the orchestrator:

- "upgrade dependencies" or "migrate to React 19" triggers `migrate-core-deps`
- "update tiket packages" or "alpha versions" triggers `migrate-tiket-packages`
- "update next config" or "Next 16 config" triggers `migrate-next-config`
- "fix JSX types" or "update tsconfig" triggers `migrate-typescript-jsx`
- "migrate react-query" or "v3 to v5" triggers `migrate-react-query-v5`
- "migrate zustand" or "zustand v5" triggers `migrate-zustand-v5`
- "migrate eslint" or "flat config" triggers `migrate-eslint-flat-config`
- "fix tests" or "jest upgrade" triggers `migrate-jest-testing`
- "fix React 19 types" or "RefObject errors" triggers `migrate-react19-code-fixes`
- "update CI" or "upgrade Docker" triggers `migrate-ci-docker`

## ✅ Best Practices

### For Developers

1. **Read Before You Code**

   - Check relevant rules before implementing navigation
   - Follow established patterns
   - Don't reinvent the wheel

2. **Keep Rules Updated**

   - Update rules when patterns change
   - Document new patterns as they're established
   - Remove outdated patterns

3. **Be Specific**

   - Include real code examples
   - Show both ❌ wrong and ✅ correct patterns
   - Explain WHY, not just WHAT

4. **Organize by Topic**
   - Keep related rules together
   - Use clear section headings
   - Include a table of contents for long files

### For AI (Cursor)

1. **Follow ALL Rules**

   - Apply patterns from both main and modular rules
   - Prefer newer patterns over deprecated ones
   - Reference specific examples when generating code

2. **Suggest Improvements**

   - When you see patterns not in the rules, suggest adding them
   - Point out deprecated patterns in existing code
   - Recommend refactoring when appropriate

3. **Be Consistent**
   - Apply the same patterns across the codebase
   - Don't mix old and new patterns
   - Follow the migration guides

## 🔧 How to Add New Rules

- **Project-wide conventions** → Edit `/.cursorrules`
- **Conditional / topic-specific rules** → Create a `.mdc` file in `.cursor/rules/` with YAML frontmatter (`description`, `alwaysApply`, `globs`)
- **Step-by-step procedural guides** → Create a `SKILL.md` in `.cursor/skills/<skill-name>/`

## 🔍 Finding Rules

1. Check main `.cursorrules` for project-wide conventions
2. Check `.cursor/rules/` for conditional `.mdc` rules
3. Check `.cursor/skills/` for migration step-by-step guides

## 🚀 Quick Start

1. Read `/.cursorrules` for project conventions
2. Review the migration skills in `.cursor/skills/` if upgrading the stack
3. Use trigger keywords (e.g., "migrate", "upgrade React") to activate skills automatically

## 📊 Rule Maintenance

### When to Update

- ✅ New pattern is established (3+ uses in codebase)
- ✅ Old pattern is deprecated
- ✅ Team agrees on new standard
- ✅ Common mistake needs documentation
- ❌ One-off solution (don't add to rules)
- ❌ Experimental code (wait until proven)

### Review Cycle

- **Weekly**: Check for outdated patterns
- **Monthly**: Review and consolidate rules
- **Per Sprint**: Update based on new features
- **Per Refactor**: Document new patterns

## 🔗 Related Resources

- Main Cursor Rules: `/.cursorrules`
- Migration Orchestrator: `.cursor/rules/migration.mdc`
- Migration Skills: `.cursor/skills/` (10 skills across 7 phases)
- Project README: `/README.md`

---

**Last Updated**: 2026-04-09
**Maintained By**: Development Team
**Review Schedule**: Monthly
