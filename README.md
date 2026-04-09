# Cursor Rules Organization

This directory contains modular Cursor AI rules that guide code generation and modifications.

## 📁 Directory Structure

```
.cursor/
├── README.md                                  # This file
└── rules/
    ├── navigation.cursorrules.md              # Navigation hooks & URL restructure
    ├── testing-navigation.cursorrules.md      # Testing navigation components
    └── testing-type-safety.cursorrules.md     # Type safety & linter compliance (MANDATORY)
```

## 🎯 Purpose

Cursor Rules help the AI understand:

- **Project-specific patterns**: How we structure code in THIS project
- **Standards & conventions**: Naming, file organization, best practices
- **Testing patterns**: How to write and fix tests
- **Migration guides**: Deprecated vs current approaches

## 📖 How Cursor Rules Work

### 1. **File Format**

- Markdown (`.md`) files for better readability
- Can use `.cursorrules` or `.cursorrules.md` extension
- Both extensions work identically in Cursor

### 2. **File Location**

- **Root-level**: `.cursorrules` - Main entry point (required)
- **Modular**: `.cursor/rules/*.cursorrules.md` - Topic-specific rules
- Cursor AI reads the main `.cursorrules` file, which references modular files

### 3. **How AI Uses Them**

- AI reads these files when you ask for code help
- Follows patterns and standards defined here
- Applies rules automatically when generating/modifying code
- References specific sections when fixing tests or refactoring

## 📚 Current Rules

### Main Rules File

**Location**: `/.cursorrules`

- Core language and framework rules
- General testing patterns
- CSS/styling conventions
- TypeScript requirements
- **References**: Points to modular rules in `.cursor/rules/`

### Navigation Rules

**Location**: `.cursor/rules/navigation.cursorrules.md`

**Covers**:

- `useNavigationWindow` hook usage and API
- `useNavigationRouter` hook usage
- `useMiddlewareData` integration
- URL structure patterns (old vs new)
- Migration guide from deprecated patterns
- Common navigation scenarios with examples
- Best practices and anti-patterns

**Use When**:

- Creating components with navigation
- Refactoring navigation code
- Fixing deprecated navigation patterns
- Adding new navigation features

### Testing Navigation Rules

**Location**: `.cursor/rules/testing-navigation.cursorrules.md`

**Covers**:

- How to mock `useNavigationWindow`
- How to mock `useNavigationRouter`
- When to mock `useMiddlewareData`
- Test patterns for different scenarios
- Complete test file templates
- Common test failures and solutions
- Quick reference checklist

**Use When**:

- Writing tests for components with navigation
- Fixing failing navigation tests
- Understanding why tests fail
- Creating test templates for new components

### Testing Type Safety & Linter Compliance Rules 🚨

**Location**: `.cursor/rules/testing-type-safety.cursorrules.md`

**Covers**:

- **MANDATORY**: All tests must be type-safe and lint-clean
- Zero TypeScript errors and zero ESLint errors required
- Explicit types for all mock data
- Proper type imports using `import type` syntax
- Mock data must match interface definitions exactly
- Typed Jest mocks and mock functions
- Common type errors and how to fix them
- Pre-completion checklist for test generation
- Test generation workflow step-by-step
- DO and DON'T patterns with examples

**Use When**:

- **ALWAYS** when generating ANY unit test
- Before committing test files
- Fixing TypeScript errors in tests
- Fixing ESLint/linter errors in tests
- Creating mock data for tests
- Ensuring test quality and maintainability
- Reviewing test pull requests

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

### Step 1: Determine Scope

Is this rule:

- **Project-wide**? → Add to main `.cursorrules`
- **Topic-specific**? → Create new file in `.cursor/rules/`

### Step 2: Create File

```bash
# For new topic
touch .cursor/rules/your-topic.cursorrules.md
```

### Step 3: Structure Your Rules

```markdown
# Topic Name

## Overview

Brief description of what this covers

## Core Concepts

Main ideas and patterns

## Usage Examples

Real code examples

## Best Practices

DO ✅ and DON'T ❌ patterns

## Common Mistakes

Errors and how to fix them

## Quick Reference

Cheat sheet for common scenarios
```

### Step 4: Reference in Main File

Update `.cursorrules` to reference your new file:

```markdown
- **Your Topic**: See `.cursor/rules/your-topic.cursorrules.md`
  - Brief description
  - What it covers
```

## 📝 Rule Writing Guidelines

### Good Rule Example

```markdown
## Navigation Hook Usage

### ✅ Correct Pattern

\`\`\`typescript
import { useNavigationWindow } from '@/lib/hooks/redirection/useNavigationWindow';

const MyComponent = () => {
const { redirectToUrl } = useNavigationWindow();

const handleClick = () => {
redirectToUrl('/to-do/product', { type: 'push' });
};

return <button onClick={handleClick}>Navigate</button>;
};
\`\`\`

### ❌ Wrong Pattern (Deprecated)

\`\`\`typescript
// Don't do this - window.location is not managed
const handleClick = () => {
window.location.href = '/to-do/product';
};
\`\`\`

### Why?

The hook manages URL structure, forward params, and tracking automatically.
```

### Bad Rule Example (Too Vague)

```markdown
## Navigation

Use the navigation hooks.
```

## 🔍 Finding Rules

### By Topic

1. Check main `.cursorrules` for topic list
2. Navigate to specific rule file
3. Use table of contents in rule file

### By Search

```bash
# Search all rules
grep -r "useNavigationWindow" .cursor/rules/

# Search specific file
grep "mock" .cursor/rules/testing-navigation.cursorrules.md
```

## 🚀 Quick Start

### For New Team Members

1. Read main `.cursorrules` - get overview
2. Read navigation rules - understand core patterns
3. Read testing rules - learn how to test
4. Write code with Cursor AI - see rules in action

### For Cursor AI

When user asks for navigation code:

1. Check `.cursor/rules/navigation.cursorrules.md`
2. Apply current patterns (not deprecated ones)
3. Generate code following examples
4. Add proper mocks if it's a test file

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

## 🎓 Learning Path

### Beginner

1. Read main `.cursorrules`
2. Use Cursor AI to generate component
3. See how rules are applied
4. Compare generated code with rules

### Intermediate

1. Read topic-specific rules
2. Fix failing tests using test rules
3. Refactor deprecated patterns
4. Suggest rule improvements

### Advanced

1. Write new rules for new patterns
2. Maintain existing rules
3. Help others understand rules
4. Optimize rule organization

## 💡 Pro Tips

### For Developers

- **Bookmark** the rules directory
- **Reference** specific rules in PRs
- **Suggest** rule updates when you find gaps
- **Follow** rules consistently

### For Cursor AI

- **Quote** specific rules when explaining decisions
- **Reference** rule files by name
- **Suggest** when code violates rules
- **Apply** rules automatically when generating code

## 📞 Need Help?

### Understanding Rules

- Read the examples in the rule file
- Check the "Common Mistakes" section
- Ask Cursor AI: "Show me example of [pattern] from rules"

### Updating Rules

- Create a new rule file in `.cursor/rules/`
- Follow the structure of existing files
- Update main `.cursorrules` with reference
- Get team review before merging

## 🔗 Related Resources

- Main Cursor Rules: `/.cursorrules`
- Navigation Standards: `.cursor/rules/navigation.cursorrules.md`
- Testing Navigation: `.cursor/rules/testing-navigation.cursorrules.md`
- Testing Type Safety: `.cursor/rules/testing-type-safety.cursorrules.md` 🚨
- Project README: `/README.md`
- Testing Guide: `/docs/testing.md`

---

**Last Updated**: 2025-11-05
**Maintained By**: Development Team
**Review Schedule**: Monthly
