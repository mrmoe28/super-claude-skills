---
name: legacy-refactor
description: Safe legacy code refactoring strategies with incremental improvement, testing, and risk mitigation for modernizing old codebases without breaking functionality
license: Apache-2.0
allowed-tools:
  - Bash(npm*:*)
  - Bash(git*:*)
  - Read
  - Write
  - Edit
  - Grep
  - Glob
metadata:
  version: "1.0.0"
  approach: "incremental"
---

# Legacy Refactor

Safe refactoring strategies for modernizing legacy code without breaking functionality.

## Refactoring Philosophy

**Golden Rules:**
1. **Never change behavior** - Refactoring changes structure, not functionality
2. **Test first** - Add tests before refactoring
3. **Small steps** - Incremental changes are safer
4. **One thing at a time** - Don't mix refactoring with features
5. **Version control** - Commit frequently
6. **Reversible** - Always have a rollback plan

## Assessment Phase

### Step 1: Understand the Code

**Ask Critical Questions:**
- What does this code do?
- Why was it written this way?
- What are the dependencies?
- Are there existing tests?
- What would break if changed?

**Mapping Tools:**
```bash
# Find all usages of a function
grep -r "functionName" .

# Check git history
git log --follow path/to/file.ts
git blame path/to/file.ts

# Find dependencies
npm list
npx depcheck

# Check for dead code
npx unimported
```

### Step 2: Identify Code Smells

**Common Issues:**
- Long functions (>50 lines)
- Deep nesting (>3 levels)
- Duplicate code
- Magic numbers/strings
- God objects (too many responsibilities)
- Tight coupling
- No error handling
- Global state
- Callback hell
- Missing types

### Step 3: Prioritize Refactoring

**Impact vs Effort Matrix:**
```
High Impact, Low Effort:
- Add TypeScript types
- Extract constants
- Add error handling
- Split long functions

High Impact, High Effort:
- Rewrite core logic
- Change architecture
- Update dependencies
- Migrate to new framework

Low Impact, Low Effort:
- Rename variables
- Format code
- Add comments
- Fix typos

Low Impact, High Effort:
- Premature optimization
- Over-engineering
- Unnecessary abstractions
```

## Refactoring Strategies

### Strategy 1: Strangler Fig Pattern

**Gradually replace old code with new:**
```typescript
// Phase 1: Add new implementation alongside old
function legacyCalculation(data: any): any {
  // Old complex code
}

function newCalculation(data: DataType): ResultType {
  // New clean implementation
}

// Phase 2: Route through feature flag
function calculate(data: any): any {
  if (process.env.USE_NEW_CALCULATION === 'true') {
    return newCalculation(data)
  }
  return legacyCalculation(data)
}

// Phase 3: Remove old code once verified
function calculate(data: DataType): ResultType {
  return newCalculation(data)
}
```

### Strategy 2: Extract and Test

**Break down monolithic functions:**
```typescript
// ❌ Before: 200 line function
function processOrder(order: any) {
  // validate order
  // calculate totals
  // apply discounts
  // check inventory
  // create invoice
  // send email
  // update database
  // return result
}

// ✅ After: Extracted functions
function processOrder(order: Order): OrderResult {
  validateOrder(order)
  const totals = calculateTotals(order)
  const discounted = applyDiscounts(totals)
  checkInventory(order.items)
  const invoice = createInvoice(discounted)
  sendOrderEmail(invoice)
  const result = saveOrder(order, invoice)
  return result
}

// Now each function can be tested independently
```

### Strategy 3: Add TypeScript Types

**From any to typed:**
```typescript
// ❌ Before: No types
function getUser(id) {
  return fetch(`/api/users/${id}`).then(res => res.json())
}

// ✅ After: Fully typed
interface User {
  id: string
  name: string
  email: string
  createdAt: string
}

async function getUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`)
  if (!response.ok) {
    throw new Error(`Failed to fetch user: ${response.status}`)
  }
  return response.json()
}
```

### Strategy 4: Replace Callbacks with Async/Await

**Modernize async code:**
```typescript
// ❌ Before: Callback hell
function getData(id, callback) {
  db.query('SELECT * FROM users WHERE id = ?', [id], (err, user) => {
    if (err) return callback(err)

    db.query('SELECT * FROM posts WHERE user_id = ?', [id], (err, posts) => {
      if (err) return callback(err)

      db.query('SELECT * FROM comments WHERE user_id = ?', [id], (err, comments) => {
        if (err) return callback(err)
        callback(null, { user, posts, comments })
      })
    })
  })
}

// ✅ After: Clean async/await
async function getData(id: string) {
  const user = await db.query.users.findFirst({
    where: eq(users.id, id)
  })

  const [posts, comments] = await Promise.all([
    db.select().from(posts).where(eq(posts.userId, id)),
    db.select().from(comments).where(eq(comments.userId, id))
  ])

  return { user, posts, comments }
}
```

### Strategy 5: Eliminate Global State

**Move to proper state management:**
```typescript
// ❌ Before: Global variables
let currentUser = null
let isAuthenticated = false

function login(credentials) {
  currentUser = authenticateUser(credentials)
  isAuthenticated = true
}

// ✅ After: Context/Store
interface AuthState {
  user: User | null
  isAuthenticated: boolean
}

const AuthContext = createContext<AuthState>({
  user: null,
  isAuthenticated: false
})

function useAuth() {
  return useContext(AuthContext)
}
```

## Incremental Refactoring Process

### Phase 1: Add Tests

**Before changing anything:**
```typescript
// Add characterization tests (capture current behavior)
describe('legacyFunction', () => {
  it('should handle case A', () => {
    const result = legacyFunction(inputA)
    expect(result).toEqual(expectedA)
  })

  it('should handle case B', () => {
    const result = legacyFunction(inputB)
    expect(result).toEqual(expectedB)
  })

  // Test edge cases and errors
  it('should handle null input', () => {
    expect(() => legacyFunction(null)).toThrow()
  })
})
```

### Phase 2: Refactor with Safety Net

**Make small, testable changes:**
```typescript
// Commit 1: Extract constant
- const MAX_RETRIES = 3
+ const maxRetries = config.maxRetries ?? 3

// Commit 2: Extract variable
- if (user.role === 'admin' || user.role === 'superadmin')
+ const isAdminUser = user.role === 'admin' || user.role === 'superadmin'
+ if (isAdminUser)

// Commit 3: Extract function
- // complex calculation inline
+ const result = calculateDiscount(order)

// Run tests after EACH commit
npm test
```

### Phase 3: Improve Readability

**Make intent clear:**
```typescript
// ❌ Before: Unclear
if (u.s === 1 && u.r > 0) {
  // do something
}

// ✅ After: Self-documenting
const isActiveUserWithBalance =
  user.status === UserStatus.Active &&
  user.balance > 0

if (isActiveUserWithBalance) {
  // do something
}
```

### Phase 4: Add Error Handling

**Robust error management:**
```typescript
// ❌ Before: Silent failures
function processData(data) {
  const result = transformData(data)
  saveToDatabase(result)
}

// ✅ After: Proper error handling
async function processData(data: DataInput): Promise<DataOutput> {
  try {
    const validatedData = validateData(data)
    const result = transformData(validatedData)
    await saveToDatabase(result)
    return result
  } catch (error) {
    if (error instanceof ValidationError) {
      throw new ProcessingError('Invalid data format', { cause: error })
    }
    if (error instanceof DatabaseError) {
      throw new ProcessingError('Failed to save data', { cause: error })
    }
    throw error
  }
}
```

## Dealing with Legacy Dependencies

### Strategy: Gradual Upgrade

**Don't upgrade everything at once:**
```bash
# 1. Check for updates
npm outdated

# 2. Update dev dependencies first (safer)
npm update --save-dev

# 3. Update minor versions
npm update

# 4. Major upgrades one at a time
npm install package@latest

# 5. Test thoroughly after each update
npm test
npm run build
```

### Creating Adapters

**Wrap legacy code:**
```typescript
// Legacy third-party library with bad API
import * as LegacyLib from 'old-library'

// Create adapter with clean interface
export class DataAdapter {
  async getData(id: string): Promise<Data> {
    // Wrap ugly legacy API
    const rawData = await LegacyLib.fetchDataSync(id)
    return this.transformToModernFormat(rawData)
  }

  private transformToModernFormat(raw: any): Data {
    return {
      id: raw.ID,
      name: raw.NAME,
      createdAt: new Date(raw.CREATED_AT)
    }
  }
}

// Use clean adapter in your code
const adapter = new DataAdapter()
const data = await adapter.getData('123')
```

## Code Patterns to Replace

### Pattern 1: Replace var with const/let

```typescript
// ❌ Before
var count = 0
var name = 'test'

// ✅ After
let count = 0
const name = 'test'
```

### Pattern 2: Replace == with ===

```typescript
// ❌ Before
if (value == null)
if (count == 0)

// ✅ After
if (value === null || value === undefined)
if (count === 0)
```

### Pattern 3: Replace for loops with array methods

```typescript
// ❌ Before
const results = []
for (let i = 0; i < items.length; i++) {
  if (items[i].active) {
    results.push(items[i].name)
  }
}

// ✅ After
const results = items
  .filter(item => item.active)
  .map(item => item.name)
```

### Pattern 4: Replace null checks with optional chaining

```typescript
// ❌ Before
const city = user && user.address && user.address.city

// ✅ After
const city = user?.address?.city
```

### Pattern 5: Replace string concatenation with templates

```typescript
// ❌ Before
const message = 'Hello ' + name + ', you have ' + count + ' messages'

// ✅ After
const message = `Hello ${name}, you have ${count} messages`
```

## Testing Legacy Code

### Add Tests First (Characterization Tests)

```typescript
// Test current behavior before changing
describe('LegacyCalculator', () => {
  // Test what it currently does, even if wrong
  it('current behavior: returns 0 for negative input', () => {
    expect(legacyCalc(-5)).toBe(0)
  })

  // After refactor, decide if behavior should change
  it('new behavior: throws error for negative input', () => {
    expect(() => newCalc(-5)).toThrow('Input must be positive')
  })
})
```

### Test Coverage

```bash
# Check current coverage
npm run test -- --coverage

# Aim for:
# - 80%+ coverage before refactoring
# - 100% coverage of refactored code
```

## Git Workflow for Refactoring

### Small, Focused Commits

```bash
# Each commit should be one logical change
git commit -m "refactor: extract calculateDiscount function"
git commit -m "refactor: add TypeScript types to Order"
git commit -m "refactor: replace callbacks with async/await"
git commit -m "test: add unit tests for calculateDiscount"

# NOT this:
git commit -m "refactor: massive cleanup"
```

### Feature Branches

```bash
# Create branch for refactoring
git checkout -b refactor/modernize-payment-system

# Make incremental commits
# Each commit passes tests

# Merge when complete and tested
git checkout main
git merge refactor/modernize-payment-system
```

## Risk Mitigation

### Always Have Rollback Plan

1. **Feature flags** - Can disable new code
2. **Git revert** - Can undo commits
3. **Monitoring** - Detect issues quickly
4. **Gradual rollout** - Test with small percentage first

### Validation Strategy

```typescript
// Run old and new code in parallel (temporarily)
async function processWithValidation(data: Data) {
  const [oldResult, newResult] = await Promise.all([
    legacyProcess(data),
    newProcess(data)
  ])

  // Log differences
  if (!isEqual(oldResult, newResult)) {
    console.error('Results differ!', { oldResult, newResult })
  }

  // Return old result (safe)
  return oldResult

  // After validation period, switch to new
  // return newResult
}
```

## Refactoring Checklist

Before starting:
- ✅ Understand what the code does
- ✅ Add tests for current behavior
- ✅ Tests pass
- ✅ Code is in version control
- ✅ Have rollback plan

During refactoring:
- ✅ Make one change at a time
- ✅ Run tests after each change
- ✅ Commit frequently
- ✅ Keep builds green
- ✅ Document changes

After refactoring:
- ✅ All tests pass
- ✅ No functionality changed
- ✅ Code is more readable
- ✅ Technical debt reduced
- ✅ Documented in PR/commit

## When NOT to Refactor

**Stop if:**
- No tests exist and can't add them
- Code works and is rarely changed
- System is being replaced soon
- No business value
- High risk, low reward
- Time constraints are tight

## When to Use This Skill

- Modernizing legacy code
- Improving code quality
- Reducing technical debt
- Before adding new features
- Making code testable
- Improving maintainability
- Upgrading dependencies
- Onboarding new developers
