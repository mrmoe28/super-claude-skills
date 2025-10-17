---
name: debug-protocol
description: Systematic debugging protocol for diagnosing, researching, fixing, and preventing errors with root cause analysis and comprehensive documentation
license: Apache-2.0
allowed-tools:
  - Bash(vercel*:*)
  - Bash(npm*:*)
  - Bash(npx*:*)
  - WebSearch
  - Read
  - Write
  - Edit
  - Grep
  - Glob
metadata:
  version: "1.0.0"
  protocol: "systematic"
---

# Debug Protocol

Systematic debugging protocol for diagnosing, fixing, and preventing errors through root cause analysis.

## Core Philosophy

**Never guess. Never ignore. Always investigate.**

- Errors are symptoms, not the disease
- Find root causes, not just symptoms
- Fix properly, never use workarounds
- Document learnings to prevent recurrence
- Build knowledge progressively

## Systematic Debugging Protocol

### Phase 1: Diagnosis (NEVER Skip)

#### DEBUG-1: Check Logs FIRST
**Gather evidence before making assumptions**

```bash
# Vercel logs
vercel ls                    # List deployments
vercel inspect [url]         # Inspect specific deployment
vercel logs [deployment-id]  # View deployment logs

# Build logs
npm run build                # Local build check

# Browser console
# Open DevTools → Console tab
# Check for JavaScript errors, warnings, network failures

# Server logs (if applicable)
npm run dev                  # Check dev server output
```

#### DEBUG-2: Copy EXACT Error Information
**Precision matters**

Capture:
- ✅ Complete error message (not paraphrased)
- ✅ File path and line number
- ✅ Stack trace (full, not truncated)
- ✅ Error code (if applicable)
- ✅ Timestamp
- ✅ Environment (dev/staging/prod)

Example:
```
Error: Cannot find module './utils'
  at Object.<anonymous> (/app/src/lib/helpers.ts:3:1)
  at Module._compile (node:internal/modules/cjs/loader:1241:14)
  at Module._extensions..js (node:internal/modules/cjs/loader:1295:10)
Environment: Production (Vercel)
Timestamp: 2025-10-17 14:32:18 UTC
```

#### DEBUG-3: Find Root Cause
**Ask "why" five times**

Questions to ask:
1. **What changed?** - Recent commits, deployments, config changes
2. **Why did it fail?** - What's the immediate cause
3. **Why did that happen?** - What allowed that condition
4. **Why wasn't it caught?** - Missing validation/tests
5. **Why is this pattern present?** - Architectural issue

```bash
# Check recent changes
git log --oneline -10
git diff HEAD~5

# Check when it last worked
git bisect start
git bisect bad          # Current broken state
git bisect good <hash>  # Last known good commit
```

### Phase 2: Research & Design

#### DEBUG-4: Research Error
**Leverage collective knowledge**

Research checklist:
1. **Search exact error message** (in quotes)
2. **Check official documentation**
3. **Search GitHub issues** for the package
4. **Check Stack Overflow**
5. **Verify version compatibility**
6. **Check recent breaking changes**

```bash
# Version checks
npm list [package-name]
npm outdated

# Check package info
npm view [package-name] versions
npm view [package-name] peerDependencies
```

Research sources (in order):
1. Official documentation
2. GitHub repository issues
3. Stack Overflow
4. Release notes / CHANGELOG
5. Community forums

#### DEBUG-5: Design Minimal Fix
**One change at a time**

Fix design principles:
- ✅ **Minimal** - Change only what's necessary
- ✅ **Testable** - Can verify it works
- ✅ **Reversible** - Can rollback if needed
- ✅ **Root cause** - Fixes the real issue
- ✅ **No side effects** - Doesn't break other things

Planning:
```markdown
## Fix Plan

### Root Cause
[Exact cause of the error]

### Proposed Solution
[What will be changed]

### Files to Modify
- file1.ts (line 23) - Change X to Y
- file2.ts (line 45) - Add validation

### Rollback Plan
If this doesn't work:
1. Revert commit: git revert <hash>
2. Alternative approach: [describe]

### Verification Steps
1. Run build: npm run build
2. Test locally: npm run dev
3. Check error is gone: [specific test]
```

### Phase 3: Apply & Validate

#### DEBUG-6: Track with Todo
**Maintain visibility**

```typescript
TodoWrite:
- Diagnose error and find root cause
- Research solution and design fix
- Apply fix and verify it works
- Update documentation/tests
- Commit with detailed message
```

#### DEBUG-7: Run Failing Command Again
**Verify the fix works**

```bash
# Run the command that was failing
npm run build
# or
vercel deploy
# or
npm test

# Check for:
# ✅ Original error is gone
# ✅ No new errors introduced
# ✅ All tests pass
# ✅ Build succeeds
```

#### DEBUG-8: Commit with Context
**Document the fix**

Commit message format:
```bash
git commit -m "$(cat <<'EOF'
fix(component): resolve import path error

Root Cause:
- Relative import path was incorrect after refactor
- Module resolution failed in production build

Solution:
- Updated import to use absolute path with @/ alias
- Added path to tsconfig paths

Verification:
- Build succeeds locally
- Vercel deployment passes
- Error no longer appears in logs

Refs: #123
EOF
)"
```

### Phase 4: Prevention

#### DEBUG-9: Document in Knowledge Base
**Build institutional memory**

Add to CLAUDE.md or CONTEXT.md:
```markdown
## Error Fix: [Date] - [Error Type]

### Error
[Exact error message]

### Root Cause
[What actually caused it]

### Solution
[How it was fixed]

### Prevention
[How to avoid in future]
- Add test for this case
- Add validation
- Update documentation
- Add CI check
```

#### DEBUG-10: Add Safeguards
**Prevent recurrence**

Options:
1. **Add Tests**
```typescript
describe('ImportPaths', () => {
  it('should resolve module paths correctly', () => {
    expect(() => import('./utils')).not.toThrow()
  })
})
```

2. **Add Validation**
```typescript
if (!config.apiKey) {
  throw new Error('API_KEY is required in environment variables')
}
```

3. **Add CI Check**
```yaml
# .github/workflows/ci.yml
- name: Verify build
  run: npm run build
```

4. **Update Documentation**
```markdown
## Environment Variables

Required variables:
- `API_KEY` - API authentication key (required)
```

## Common Error Patterns

### Build Errors

**Module not found:**
```
Error: Cannot find module './utils'

Checklist:
□ File actually exists
□ Import path is correct
□ File extension included (if needed)
□ Case sensitivity (utils vs Utils)
□ TypeScript paths configured
□ Package installed (npm install)
```

**Type errors:**
```
Error: Type 'string' is not assignable to type 'number'

Checklist:
□ Check type definitions
□ Verify function signatures
□ Check generic constraints
□ Update interface if needed
□ Add type assertion (last resort)
```

### Runtime Errors

**Undefined reference:**
```
TypeError: Cannot read property 'name' of undefined

Checklist:
□ Add null check
□ Use optional chaining (?.)
□ Provide default value
□ Check data flow
□ Add validation upstream
```

**Network errors:**
```
Error: Network request failed

Checklist:
□ Check API endpoint URL
□ Verify CORS settings
□ Check authentication
□ Review rate limits
□ Check network tab in DevTools
□ Verify environment variables
```

### Deployment Errors

**Vercel build failure:**
```
Error: Build failed

Checklist:
□ Local build succeeds: npm run build
□ Check Vercel logs: vercel logs
□ Environment variables set
□ Dependencies installed
□ No .env.local in repo
□ Build command correct
□ Node version compatible
```

**Environment variable issues:**
```
Error: process.env.API_KEY is undefined

Checklist:
□ Variable set in Vercel dashboard
□ Variable name correct (case-sensitive)
□ Prefixed with NEXT_PUBLIC_ (if client-side)
□ Deployment triggered after setting
□ .env.local not committed to repo
```

## Anti-Patterns (FORBIDDEN)

### Diagnostic Anti-Patterns
- ❌ Guessing without checking logs
- ❌ Assuming the error message
- ❌ Skipping stack trace analysis
- ❌ Not checking what changed
- ❌ Ignoring environment differences

### Fix Anti-Patterns
- ❌ Trying random solutions
- ❌ Making multiple changes at once
- ❌ Using workarounds instead of fixes
- ❌ Ignoring warnings
- ❌ Not testing the fix
- ❌ Copy-pasting without understanding

### Process Anti-Patterns
- ❌ Not documenting the fix
- ❌ Not adding safeguards
- ❌ Moving on with errors present
- ❌ Fixing symptoms, not root cause
- ❌ Not researching before trying

## Debugging Tools

### Vercel
```bash
vercel ls                      # List deployments
vercel inspect [url]           # Inspect deployment
vercel logs [deployment-id]    # View logs
vercel env ls                  # List env variables
vercel env pull .env.local     # Pull env variables
```

### Git
```bash
git log --oneline -n 10        # Recent changes
git diff HEAD~5                # Compare with 5 commits ago
git bisect                     # Find breaking commit
git blame file.ts              # See who changed what
```

### npm
```bash
npm run build                  # Test build
npm run lint                   # Check for issues
npm outdated                   # Check outdated packages
npm list [package]             # Check installed version
```

### Browser DevTools
- Console: JavaScript errors
- Network: API calls, failed requests
- Sources: Debug with breakpoints
- Application: LocalStorage, cookies
- Performance: Performance issues

## When to Use This Skill

- Encountering any error
- Build failures
- Deployment issues
- Runtime exceptions
- Type errors
- Test failures
- Performance problems
- Mysterious bugs
- Production incidents

## Success Criteria

Error is resolved when:
- ✅ Root cause identified
- ✅ Proper fix applied (no workarounds)
- ✅ Error no longer occurs
- ✅ No new errors introduced
- ✅ Tests pass
- ✅ Build succeeds
- ✅ Fix documented
- ✅ Safeguards added
- ✅ Knowledge base updated
