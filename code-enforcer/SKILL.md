---
name: code-enforcer
description: Enforce code quality standards including ESLint, TypeScript, testing best practices, and coding principles with zero tolerance for errors or workarounds
license: Apache-2.0
allowed-tools:
  - Bash(npm run lint*:*)
  - Bash(npx eslint*:*)
  - Bash(npx tsc*:*)
  - Bash(turbo typecheck lint*:*)
  - Bash(prettier*:*)
  - Read
  - Edit
  - Grep
  - Glob
metadata:
  version: "1.0.0"
  enforcement: "strict"
---

# Code Enforcer

Strict enforcement of code quality standards with zero tolerance for errors, workarounds, or technical debt.

## Core Principles

### ZERO TOLERANCE POLICIES

#### 1. NO WORKAROUNDS POLICY
- **NEVER** use workarounds, hacks, or temporary fixes
- **ALWAYS** implement proper, direct solutions
- **NO EXCEPTIONS**: Find the correct way to solve problems
- **BANNED PHRASES**: "temporary fix", "workaround", "quick hack", "for now"

#### 2. ASK DON'T ASSUME POLICY
- **WHEN UNCERTAIN**: Stop and ask the user for clarification
- **NO GUESSING**: Don't assume what the user wants
- **VERIFY FIRST**: Only proceed when you can verify the approach is correct
- **BE DIRECT**: State exactly what you don't understand

#### 3. ZERO TOLERANCE ERROR POLICY
- **NEVER IGNORE ERRORS**: Every error must be addressed immediately
- **FIX BEFORE PROCEEDING**: Cannot move to next task until all errors are resolved
- **NO SILENT FAILURES**: Always acknowledge and fix errors when they occur
- **ERROR PRIORITY**: Fixing errors takes precedence over new features

#### 4. VERIFICATION REQUIREMENTS
- **CAN'T VERIFY = DON'T PROCEED**: If unsure about implementation, ask first
- **TEST BEFORE CLAIMING**: Don't say something works unless you can verify it
- **HONEST UNCERTAINTY**: Admit when you don't know instead of guessing
- **SPECIFIC QUESTIONS**: Ask targeted questions about unclear requirements

## Implementation Best Practices

### Before Coding (MUST)
- **BP-1 (MUST)** Ask clarifying questions
- **BP-2 (SHOULD)** Draft approach for complex work
- **BP-3 (SHOULD)** List pros/cons if ≥2 approaches

### While Coding
- **C-1** Consider TDD but prioritize user's request
- **C-2 (MUST)** Use existing domain vocabulary
- **C-3** Prefer simple, composable, testable functions over classes
- **C-5 (MUST)** Use branded types for IDs: `type UserId = Brand<string, 'UserId'>`
- **C-6 (MUST)** Use `import type` for type-only imports
- **C-7** Avoid comments; write self-explanatory code
- **C-8** Default to `type`; use `interface` only when needed
- **C-9** Don't extract functions unless reused, testable only that way, or improves readability

### Testing (MUST)
- **T-1 (MUST)** Colocate unit tests: `*.spec.ts` in same directory
- **T-2 (MUST)** Add/extend integration tests for API changes
- **T-3 (MUST)** Separate unit tests from DB integration tests
- **T-4** Prefer integration tests over heavy mocking
- **T-6** Test entire structure in one assertion when possible
- **T-7 (MUST)** Only run tests at end of ALL tasks
- **T-8 (MUST)** Don't run tests unless requested or after completion

### Database
- **D-1 (MUST)** Type as `KyselyDatabase | Transaction<Database>`
- **D-2** Override generated types in `db-types.override.ts`

### Code Organization
- **O-1 (MUST)** Place in `packages/shared` only if used by ≥2 packages

## Quality Gates

### Pre-Commit Gates (ALL MUST PASS)
```bash
# 1. Prettier check
prettier --check .

# 2. TypeScript check
turbo typecheck

# 3. ESLint check
turbo lint

# 4. Build check (if applicable)
npm run build
```

### Mandatory Verification Steps
1. **Read before write** - Always read files before editing
2. **Match patterns** - Follow existing code patterns
3. **One thing at a time** - Complete tasks fully before moving on
4. **Exact paths** - Use absolute paths, never relative
5. **No silent failures** - Report all errors immediately

## ESLint Enforcement

### After Every Code Change
```bash
# Run ESLint
npm run lint

# Or with turbo
turbo lint

# Fix auto-fixable issues
npm run lint -- --fix
```

### Common ESLint Errors & Solutions

**Missing dependency in useEffect:**
```typescript
// ❌ Bad
useEffect(() => {
  fetchData(userId)
}, [])

// ✅ Good
useEffect(() => {
  fetchData(userId)
}, [userId, fetchData])
```

**Unused variables:**
```typescript
// ❌ Bad
import { foo, bar } from './utils'

// ✅ Good - Remove or prefix with underscore
import { foo } from './utils'
// or
const _bar = something // Prefix if needed for future
```

**Missing return types:**
```typescript
// ❌ Bad
function getData() {
  return data
}

// ✅ Good
function getData(): DataType {
  return data
}
```

## TypeScript Enforcement

### Type Safety Rules
- **NO `any` types** - Always use proper types
- **NO `@ts-ignore`** - Fix the actual type issue
- **NO `@ts-expect-error`** - Unless in test cases with explanation
- **Explicit return types** - On all functions
- **Strict mode** - Always enabled

### TypeScript Config Requirements
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  }
}
```

## Writing Functions Checklist

Before completing any function, verify:
1. ✅ Is it easily readable?
2. ✅ Low cyclomatic complexity?
3. ✅ Best data structures used?
4. ✅ No unused parameters?
5. ✅ No unnecessary type casts?
6. ✅ Easily testable without mocking core features?
7. ✅ No hidden dependencies that should be arguments?
8. ✅ Best name vs 3 alternatives?

**Refactor only if**: Reused multiple places, only way to unit test, or drastically improves readability

## Writing Tests Checklist

Before completing any test, verify:
1. ✅ Parameterize inputs (no unexplained literals)
2. ✅ Test must fail for real defects
3. ✅ Description matches assertion
4. ✅ Compare to independent expectations
5. ✅ Follow same lint/type rules as production
6. ✅ Express invariants/axioms when practical
7. ✅ Group under `describe(functionName, ...)`
8. ✅ Use `expect.any(...)` for variable values
9. ✅ Strong assertions over weak ones
10. ✅ Test edge cases & boundaries
11. ✅ Don't test what TypeScript catches

## Task Focus Rules

### MANDATORY WORKFLOW
- **TF-1 (MUST)** Restate user's request before starting. Don't add scope.
- **TF-2 (MUST)** No speculative testing. Only test what's requested.
- **TF-3** Progress updates for tasks >5 minutes
- **TF-4 (MUST)** No unrequested features/refactors
- **TF-5** Clarify ambiguity immediately

### Verification & Error Handling
- **V-1 (MUST)** No assumptions. Verify before claiming success.
- **V-2 (MUST)** Research docs before building
- **V-3 (MUST)** Address all errors directly. No workarounds.
- **V-4 (MUST)** Ask when uncertain
- **V-5 (MUST)** Verify changes work before reporting completion

## Error Response Workflow

**When encountering an error:**
1. **STOP immediately** - Don't proceed past the error
2. **Fix error completely** - No workarounds or temporary fixes
3. **Verify fix works** - Test that error is resolved
4. **Document fix** - Add to knowledge base if novel
5. **Only then proceed** - Continue with original task

## Git Commit Standards

### Conventional Commits (MUST)
```bash
# Format
<type>(<scope>): <subject>

# Types
feat: New feature
fix: Bug fix
refactor: Code refactoring
docs: Documentation changes
test: Test changes
chore: Build/tool changes
style: Code style changes
perf: Performance improvements

# Examples
feat(auth): add JWT token validation
fix(api): resolve race condition in user fetch
refactor(db): simplify query builder logic
```

### Commit Message Requirements
- **MUST** use Conventional Commits format
- **MUST** be concise (1-2 sentences)
- **MUST** focus on "why" not "what"
- **MUST NOT** refer to Claude/Anthropic (remove AI attribution if present)

## Anti-Patterns (FORBIDDEN)

### Code Anti-Patterns
- ❌ Ignoring ESLint errors
- ❌ Using `any` type
- ❌ Using `@ts-ignore`
- ❌ Placeholder/temporary code
- ❌ Commented-out code
- ❌ Console.log in production
- ❌ Magic numbers without constants
- ❌ Deep nesting (>3 levels)

### Process Anti-Patterns
- ❌ Proceeding with errors present
- ❌ Assuming requirements
- ❌ Skipping verification steps
- ❌ Making multiple unrelated changes
- ❌ Committing broken code
- ❌ Ignoring test failures
- ❌ Silent failures

## Success Criteria

Code is complete ONLY when:
- ✅ All ESLint errors fixed
- ✅ All TypeScript errors fixed
- ✅ All tests passing
- ✅ Prettier formatting applied
- ✅ Build succeeds
- ✅ Changes verified working
- ✅ Documentation updated
- ✅ Committed with proper message

## When to Use This Skill

- Before committing code
- During code reviews
- When adding new features
- When refactoring code
- When enforcing standards
- Before marking tasks complete
- When quality is critical
