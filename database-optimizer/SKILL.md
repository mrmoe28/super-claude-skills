---
name: database-optimizer
description: Systematic database performance optimization including query analysis, indexing strategies, schema design, and monitoring for PostgreSQL, MySQL, and NeonDB
license: Apache-2.0
allowed-tools:
  - Bash(psql*:*)
  - Bash(npm*:*)
  - Bash(npx*:*)
  - Read
  - Write
  - Edit
  - Grep
  - Glob
metadata:
  version: "1.0.0"
  databases: "PostgreSQL, NeonDB, MySQL"
---

# Database Optimizer

Expert database performance optimization following systematic analysis and optimization protocols.

## Optimization Protocol

### Phase 1: Measure & Profile

**Step 1: Identify Slow Queries**
```sql
-- PostgreSQL: Find slow queries
SELECT
  query,
  calls,
  total_exec_time,
  mean_exec_time,
  max_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;

-- Enable query logging (postgresql.conf)
-- log_min_duration_statement = 1000  # Log queries > 1s
```

**Step 2: Analyze Query Execution**
```sql
-- Use EXPLAIN ANALYZE
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'test@example.com';

-- Look for:
-- - Seq Scan (bad, should be Index Scan)
-- - High execution time
-- - High number of rows processed
```

**Step 3: Check Database Stats**
```sql
-- Table sizes
SELECT
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;

-- Index usage
SELECT
  schemaname,
  tablename,
  indexname,
  idx_scan,
  idx_tup_read,
  idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;
```

### Phase 2: Index Optimization

**Common Index Patterns**
```sql
-- Single column index (for WHERE clauses)
CREATE INDEX idx_users_email ON users(email);

-- Composite index (for multiple WHERE conditions)
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Partial index (for filtered queries)
CREATE INDEX idx_active_users ON users(email)
WHERE status = 'active';

-- Index on JSON fields (PostgreSQL)
CREATE INDEX idx_metadata_type ON products
USING GIN ((metadata->>'type'));

-- Text search index
CREATE INDEX idx_posts_search ON posts
USING GIN (to_tsvector('english', content));
```

**Index Best Practices**
1. Index columns used in WHERE clauses
2. Index foreign keys
3. Index columns used in ORDER BY
4. Index columns used in JOIN conditions
5. Use composite indexes for multiple conditions
6. Don't over-index (slows writes)

### Phase 3: Query Optimization

**N+1 Query Problem**
```typescript
// ❌ Bad: N+1 queries
const users = await db.select().from(users)
for (const user of users) {
  const posts = await db.select().from(posts)
    .where(eq(posts.userId, user.id))
}

// ✅ Good: Single query with join
const usersWithPosts = await db
  .select()
  .from(users)
  .leftJoin(posts, eq(users.id, posts.userId))
```

**Batch Loading**
```typescript
// ❌ Bad: Multiple queries
for (const id of userIds) {
  await db.select().from(users).where(eq(users.id, id))
}

// ✅ Good: Batch query
const users = await db
  .select()
  .from(users)
  .where(inArray(users.id, userIds))
```

**Pagination Optimization**
```typescript
// ❌ Bad: OFFSET for large datasets
const users = await db
  .select()
  .from(users)
  .limit(20)
  .offset(1000) // Scans 1000 rows to skip them

// ✅ Good: Cursor-based pagination
const users = await db
  .select()
  .from(users)
  .where(gt(users.id, lastSeenId))
  .limit(20)
  .orderBy(users.id)
```

**SELECT Only What You Need**
```typescript
// ❌ Bad: Select all columns
const users = await db.select().from(users)

// ✅ Good: Select specific columns
const users = await db
  .select({
    id: users.id,
    email: users.email,
    name: users.name
  })
  .from(users)
```

### Phase 4: Schema Optimization

**Normalization vs Denormalization**
```sql
-- Normalized (reduce redundancy)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email TEXT UNIQUE NOT NULL
);

CREATE TABLE user_profiles (
  user_id INT REFERENCES users(id),
  bio TEXT,
  avatar_url TEXT
);

-- Denormalized (for read performance)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  bio TEXT,
  avatar_url TEXT,
  -- Duplicate data for faster reads
  post_count INT DEFAULT 0,
  last_post_at TIMESTAMP
);
```

**Data Types**
```sql
-- Use appropriate types
email VARCHAR(255) NOT NULL,  -- Not TEXT if max length known
status VARCHAR(20) NOT NULL,   -- Not TEXT for limited values
price DECIMAL(10,2),           -- Not FLOAT for money
is_active BOOLEAN,             -- Not VARCHAR for true/false
created_at TIMESTAMP DEFAULT NOW()
```

**Constraints & Validation**
```sql
-- Database-level constraints
ALTER TABLE users
  ADD CONSTRAINT check_email_format
  CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$');

ALTER TABLE products
  ADD CONSTRAINT check_price_positive
  CHECK (price > 0);
```

### Phase 5: Connection Management

**Connection Pooling (Neon/Vercel)**
```typescript
import { Pool, neonConfig } from '@neondatabase/serverless'
import { drizzle } from 'drizzle-orm/neon-serverless'
import ws from 'ws'

// For serverless: Use pooled connection
neonConfig.webSocketConstructor = ws
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 10, // Connection pool size
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
})

export const db = drizzle(pool)
```

**Connection Limits**
```typescript
// Configure based on your plan
// Neon Free: 100 connections
// Vercel Function: 1 connection per invocation

// Use connection pooling
const pool = new Pool({
  max: 10,                    // Max connections in pool
  min: 2,                     // Min idle connections
  idleTimeoutMillis: 30000,   // Close idle after 30s
  connectionTimeoutMillis: 2000 // Wait max 2s for connection
})
```

### Phase 6: Caching Strategies

**Query Result Caching**
```typescript
import { Redis } from '@upstash/redis'

const redis = Redis.fromEnv()

async function getUserById(id: string) {
  // Check cache first
  const cached = await redis.get(`user:${id}`)
  if (cached) return cached

  // Query database
  const user = await db.query.users.findFirst({
    where: eq(users.id, id)
  })

  // Cache result (TTL: 1 hour)
  if (user) {
    await redis.set(`user:${id}`, user, { ex: 3600 })
  }

  return user
}
```

**Cache Invalidation**
```typescript
async function updateUser(id: string, data: any) {
  // Update database
  await db.update(users)
    .set(data)
    .where(eq(users.id, id))

  // Invalidate cache
  await redis.del(`user:${id}`)
}
```

### Phase 7: Monitoring & Alerts

**Key Metrics to Track**
- Query execution time (p50, p95, p99)
- Connection pool usage
- Slow query count
- Index hit ratio
- Cache hit ratio
- Database size growth
- Lock wait time

**Monitoring Queries**
```sql
-- Active connections
SELECT count(*) FROM pg_stat_activity;

-- Blocked queries
SELECT
  blocked_locks.pid AS blocked_pid,
  blocked_activity.usename AS blocked_user,
  blocking_locks.pid AS blocking_pid,
  blocking_activity.usename AS blocking_user,
  blocked_activity.query AS blocked_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity
  ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks
  ON blocking_locks.locktype = blocked_locks.locktype
WHERE NOT blocked_locks.granted;

-- Cache hit ratio (should be > 99%)
SELECT
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit) as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;
```

## Drizzle ORM Best Practices

**Efficient Queries**
```typescript
// Use prepared statements
const preparedQuery = db
  .select()
  .from(users)
  .where(eq(users.id, sql.placeholder('id')))
  .prepare('get_user_by_id')

const user = await preparedQuery.execute({ id: '123' })

// Use transactions for multiple operations
await db.transaction(async (tx) => {
  await tx.insert(users).values(newUser)
  await tx.insert(profiles).values(newProfile)
})

// Use WITH for complex queries
const sq = db.$with('sq').as(
  db.select().from(users).where(eq(users.active, true))
)
const result = await db.with(sq).select().from(sq)
```

## NeonDB Specific Optimizations

**Branch for Testing**
```bash
# Create branch for testing optimizations
neon branches create --name=optimization-test

# Test queries on branch
psql postgresql://...@...neon.tech/optimization-test

# If successful, merge to main
neon branches merge optimization-test
```

**Autoscaling Configuration**
- Enable autoscaling for variable loads
- Set min/max compute units based on needs
- Monitor scaling patterns
- Adjust during peak/off-peak hours

## Common Performance Issues

### Issue 1: Missing Indexes
**Symptom:** Slow queries with Seq Scan
**Solution:** Add appropriate indexes
```sql
CREATE INDEX idx_column ON table(column);
```

### Issue 2: N+1 Queries
**Symptom:** Many sequential queries
**Solution:** Use JOINs or batch loading

### Issue 3: Large Table Scans
**Symptom:** Full table scans on large tables
**Solution:** Add indexes, use WHERE clauses

### Issue 4: Inefficient Pagination
**Symptom:** Slow OFFSET queries
**Solution:** Use cursor-based pagination

### Issue 5: Connection Pool Exhaustion
**Symptom:** "Too many connections" errors
**Solution:** Proper pooling configuration

## Optimization Checklist

Before deploying:
- ✅ Analyzed slow queries with EXPLAIN
- ✅ Added indexes for common queries
- ✅ Eliminated N+1 query patterns
- ✅ Implemented connection pooling
- ✅ Added caching layer
- ✅ Set up monitoring
- ✅ Tested under load
- ✅ Documented optimizations

## When to Use This Skill

- Slow query performance
- Database connection issues
- High database load
- Scaling concerns
- N+1 query problems
- Index optimization needed
- Schema design decisions
- Monitoring setup
