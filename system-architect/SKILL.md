---
name: system-architect
description: Expert system architecture design for scalable applications including microservices, monoliths, serverless, database design, caching, and infrastructure decisions
license: Apache-2.0
allowed-tools:
  - Read
  - Write
  - Edit
metadata:
  version: "1.0.0"
  focus: "scalability, reliability, performance"
---

# System Architect

Expert guidance for designing scalable, reliable, and maintainable system architectures.

## Architecture Decision Framework

### Step 1: Understand Requirements

**Functional Requirements:**
- What features are needed?
- What user flows exist?
- What data needs to be stored?
- What integrations are required?

**Non-Functional Requirements:**
- Expected traffic (users/day, requests/second)
- Latency requirements (p50, p95, p99)
- Availability needs (99.9%, 99.99%?)
- Data consistency requirements
- Budget constraints
- Team size and expertise

### Step 2: Choose Architecture Style

**Monolith vs Microservices:**
```
Choose Monolith when:
✅ Small team (<10 developers)
✅ MVP or early stage product
✅ Simple domain
✅ Need to move fast
✅ Limited resources

Choose Microservices when:
✅ Large team (>20 developers)
✅ Complex domain with clear boundaries
✅ Independent scaling needs
✅ Different tech stack requirements
✅ Multiple teams working in parallel
```

## Next.js Application Architecture

### Small to Medium Apps (Monolith)

```
nextjs-app/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (auth)/            # Auth routes
│   │   ├── (marketing)/       # Public pages
│   │   ├── (dashboard)/       # Protected pages
│   │   ├── api/               # API routes
│   │   │   ├── v1/
│   │   │   └── webhooks/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   │   ├── ui/                # ShadCN components
│   │   ├── features/          # Feature components
│   │   └── layouts/           # Layout components
│   ├── lib/
│   │   ├── db/                # Database
│   │   │   ├── schema.ts
│   │   │   └── queries.ts
│   │   ├── auth/              # Authentication
│   │   ├── payments/          # Payment logic
│   │   ├── email/             # Email service
│   │   └── utils.ts
│   ├── hooks/                 # Custom hooks
│   ├── types/                 # TypeScript types
│   └── config/                # Configuration
└── .env.local
```

### Large Apps (Modular Monolith)

```
apps/
├── web/                       # Main Next.js app
├── admin/                     # Admin dashboard
└── api/                       # API server (optional)

packages/
├── database/                  # Shared database layer
│   ├── schema/
│   ├── migrations/
│   └── queries/
├── auth/                      # Auth module
├── payments/                  # Payment module
├── ui/                        # Shared UI components
├── email/                     # Email templates
└── config/                    # Shared config

```

## Database Architecture

### Single Database Pattern

**When to use:**
- Small to medium apps
- Simple data relationships
- ACID transactions needed
- Team familiar with SQL

**Structure:**
```sql
-- User management
users
user_sessions
user_profiles

-- Core business logic
products
orders
order_items

-- Payments
payment_methods
transactions
invoices

-- Subscriptions
subscriptions
subscription_plans
```

### Database per Service

**When to use:**
- Microservices architecture
- Services need different DB types
- Independent scaling
- Team autonomy

**Structure:**
```
User Service     → PostgreSQL (relational data)
Product Service  → PostgreSQL (structured data)
Cart Service     → Redis (temporary data)
Search Service   → Elasticsearch (full-text search)
Analytics Service → ClickHouse (time-series data)
```

### Shared Database with Schemas

**When to use:**
- Monolith with modules
- Need some isolation
- Cross-module queries needed

```sql
-- Schema per domain
CREATE SCHEMA auth;
CREATE SCHEMA products;
CREATE SCHEMA orders;

-- Access with schema prefix
SELECT * FROM auth.users;
SELECT * FROM products.items;
```

## Caching Strategy

### Multi-Layer Caching

```typescript
// Layer 1: In-Memory Cache (Fastest)
const memoryCache = new Map()

// Layer 2: Redis Cache (Fast, Shared)
import { Redis } from '@upstash/redis'
const redis = Redis.fromEnv()

// Layer 3: Database (Slowest)
import { db } from './db'

async function getData(id: string) {
  // Check memory cache
  if (memoryCache.has(id)) {
    return memoryCache.get(id)
  }

  // Check Redis
  const cached = await redis.get(`data:${id}`)
  if (cached) {
    memoryCache.set(id, cached)
    return cached
  }

  // Query database
  const data = await db.query.data.findFirst({
    where: eq(data.id, id)
  })

  // Cache in both layers
  if (data) {
    await redis.set(`data:${id}`, data, { ex: 3600 })
    memoryCache.set(id, data)
  }

  return data
}
```

### Cache Invalidation Patterns

```typescript
// Write-Through Cache
async function updateData(id: string, updates: any) {
  // Update database
  const updated = await db
    .update(data)
    .set(updates)
    .where(eq(data.id, id))
    .returning()

  // Update cache immediately
  await redis.set(`data:${id}`, updated[0], { ex: 3600 })

  return updated[0]
}

// Cache-Aside Pattern
async function getData(id: string) {
  const cached = await redis.get(`data:${id}`)
  if (cached) return cached

  const data = await db.query.data.findFirst({
    where: eq(data.id, id)
  })

  if (data) {
    await redis.set(`data:${id}`, data, { ex: 3600 })
  }

  return data
}

// Delete cache on write
async function deleteData(id: string) {
  await db.delete(data).where(eq(data.id, id))
  await redis.del(`data:${id}`)
}
```

## API Gateway Pattern

### Centralized API Gateway

```typescript
// lib/api-gateway/router.ts
import { NextRequest, NextResponse } from 'next/server'

export async function handleApiRequest(request: NextRequest) {
  const { pathname } = new URL(request.url)

  // Authentication
  const user = await authenticate(request)
  if (!user && requiresAuth(pathname)) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // Rate limiting
  const allowed = await checkRateLimit(user?.id || request.ip)
  if (!allowed) {
    return NextResponse.json({ error: 'Rate limit exceeded' }, { status: 429 })
  }

  // Logging
  await logRequest({ pathname, user: user?.id, timestamp: new Date() })

  // Route to service
  if (pathname.startsWith('/api/users')) {
    return handleUsersRequest(request, user)
  }
  if (pathname.startsWith('/api/products')) {
    return handleProductsRequest(request, user)
  }

  return NextResponse.json({ error: 'Not found' }, { status: 404 })
}
```

## Background Jobs Architecture

### Queue-Based Processing

```typescript
// Using QStash for background jobs
import { Client } from '@upstash/qstash'

const qstash = new Client({ token: process.env.QSTASH_TOKEN! })

// Enqueue job
export async function enqueueEmailJob(emailData: EmailData) {
  await qstash.publishJSON({
    url: `${process.env.NEXT_PUBLIC_URL}/api/jobs/send-email`,
    body: emailData,
    retries: 3
  })
}

// Process job
// app/api/jobs/send-email/route.ts
export async function POST(request: NextRequest) {
  const { recipient, subject, body } = await request.json()

  await sendEmail({ recipient, subject, body })

  return NextResponse.json({ success: true })
}
```

## Event-Driven Architecture

### Event Bus Pattern

```typescript
// lib/events/event-bus.ts
type EventHandler<T = any> = (data: T) => Promise<void>

class EventBus {
  private handlers = new Map<string, EventHandler[]>()

  subscribe<T>(event: string, handler: EventHandler<T>) {
    if (!this.handlers.has(event)) {
      this.handlers.set(event, [])
    }
    this.handlers.get(event)!.push(handler)
  }

  async publish<T>(event: string, data: T) {
    const handlers = this.handlers.get(event) || []
    await Promise.all(handlers.map(handler => handler(data)))
  }
}

export const eventBus = new EventBus()

// Subscribe to events
eventBus.subscribe('user.created', async (user) => {
  await sendWelcomeEmail(user)
  await createUserProfile(user)
  await trackSignup(user)
})

// Publish events
await eventBus.publish('user.created', newUser)
```

## Scaling Patterns

### Vertical Scaling (Scale Up)
```
✅ Pros:
- Simpler (no code changes)
- Faster initially
- No data distribution needed

❌ Cons:
- Has limits
- Downtime for upgrades
- Single point of failure
```

### Horizontal Scaling (Scale Out)
```
✅ Pros:
- Nearly unlimited scaling
- Better fault tolerance
- No downtime for adding nodes

❌ Cons:
- More complex
- Need load balancer
- Stateless architecture required
```

### Auto-Scaling Strategy

**Vercel (Automatic):**
- Scales automatically with traffic
- Pay per execution
- No configuration needed

**Custom Scaling Metrics:**
- CPU usage > 70%
- Memory usage > 80%
- Request queue > 100
- Response time > 1s

## Security Architecture

### Defense in Depth

```
Layer 1: Network
- HTTPS only
- CORS configuration
- DDoS protection

Layer 2: Application
- Input validation
- Rate limiting
- CSRF protection

Layer 3: Authentication
- JWT/Session tokens
- MFA support
- Secure password storage

Layer 4: Authorization
- Role-based access control
- Resource-level permissions
- API scoping

Layer 5: Data
- Encryption at rest
- Encryption in transit
- Database access controls
```

### Environment Security

```typescript
// config/env.ts
import { z } from 'zod'

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  STRIPE_WEBHOOK_SECRET: z.string().startsWith('whsec_'),
})

export const env = envSchema.parse(process.env)

// Use env.DATABASE_URL (validated)
// Not process.env.DATABASE_URL (unvalidated)
```

## Observability

### Logging Strategy

```typescript
// lib/logger.ts
import pino from 'pino'

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label })
  }
})

// Usage
logger.info({ userId: user.id }, 'User logged in')
logger.error({ error: err, userId: user.id }, 'Payment failed')
```

### Monitoring Metrics

```typescript
// Key metrics to track
interface Metrics {
  // Performance
  requestDuration: number
  dbQueryDuration: number
  cacheHitRate: number

  // Business
  activeUsers: number
  newSignups: number
  revenue: number

  // System
  errorRate: number
  memoryUsage: number
  cpuUsage: number
}
```

## Data Flow Patterns

### Request → Response
```
Client Request
  → Next.js API Route
    → Validation (Zod)
      → Business Logic
        → Database Query
          → Cache Update
            → Response to Client
```

### Background Processing
```
User Action
  → Enqueue Job
    → Return to User (fast)

Background Worker
  → Process Job
    → Update Database
      → Send Notification
```

## Architecture Decision Records

**Document key decisions:**
```markdown
# ADR 001: Database Choice

## Status
Accepted

## Context
Need to choose database for user data and transactions.

## Decision
Use PostgreSQL (NeonDB) for all data.

## Consequences
Pros:
- ACID transactions
- Rich query capabilities
- Team expertise

Cons:
- Single DB to scale
- Need connection pooling
```

## When to Use This Skill

- Starting new projects
- Scaling existing apps
- Architecture reviews
- Technology decisions
- Performance problems
- Designing new features
- Microservices planning
- Infrastructure setup
