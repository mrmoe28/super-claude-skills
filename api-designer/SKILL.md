---
name: api-designer
description: Expert API design for scalable RESTful and GraphQL APIs with versioning, authentication, rate limiting, documentation, and Next.js App Router patterns
license: Apache-2.0
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
metadata:
  version: "1.0.0"
  api-types: "REST, GraphQL"
---

# API Designer

Expert API design following REST principles, scalability patterns, and modern best practices.

## API Design Principles

### Core Principles
1. **Consistency** - Same patterns across all endpoints
2. **Predictability** - Behave as users expect
3. **Discoverability** - Self-documenting
4. **Versioning** - Support multiple versions
5. **Security** - Auth, validation, rate limiting
6. **Performance** - Caching, pagination, optimization

## REST API Design

### Resource Naming

**Rules:**
- Use nouns, not verbs
- Use plural for collections
- Use kebab-case for URLs
- Hierarchical for relationships

**Examples:**
```
✅ Good:
GET    /api/v1/users
GET    /api/v1/users/123
GET    /api/v1/users/123/posts
POST   /api/v1/users
PUT    /api/v1/users/123
DELETE /api/v1/users/123

❌ Bad:
GET  /api/getUsers
POST /api/createUser
GET  /api/user/123/getPosts
```

### HTTP Methods

**Standard CRUD Operations:**
```
GET    /resources       - List all (with pagination)
GET    /resources/:id   - Get one
POST   /resources       - Create new
PUT    /resources/:id   - Update entire resource
PATCH  /resources/:id   - Partial update
DELETE /resources/:id   - Delete resource
```

**Advanced Patterns:**
```
GET    /resources/search?q=query  - Search
POST   /resources/:id/actions     - Resource actions
GET    /resources/:id/relationships - Related resources
```

### HTTP Status Codes

**Success (2xx):**
- `200 OK` - Successful GET, PUT, PATCH, DELETE
- `201 Created` - Successful POST
- `202 Accepted` - Async processing started
- `204 No Content` - Successful DELETE with no body

**Client Errors (4xx):**
- `400 Bad Request` - Invalid input
- `401 Unauthorized` - Not authenticated
- `403 Forbidden` - Not authorized
- `404 Not Found` - Resource doesn't exist
- `422 Unprocessable Entity` - Validation failed
- `429 Too Many Requests` - Rate limit exceeded

**Server Errors (5xx):**
- `500 Internal Server Error` - Server error
- `503 Service Unavailable` - Temporary outage

### Request/Response Format

**Consistent Response Structure:**
```typescript
// Success Response
{
  "success": true,
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "timestamp": "2025-01-17T10:00:00Z",
    "version": "1.0"
  }
}

// Error Response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": [
      {
        "field": "email",
        "message": "Must be valid email address"
      }
    ]
  },
  "meta": {
    "timestamp": "2025-01-17T10:00:00Z",
    "requestId": "req_abc123"
  }
}

// Collection Response with Pagination
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "total": 100,
    "totalPages": 5,
    "hasNext": true,
    "hasPrev": false
  },
  "meta": {
    "timestamp": "2025-01-17T10:00:00Z"
  }
}
```

## Next.js App Router API Patterns

### Basic API Route
```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { z } from 'zod'
import { db } from '@/lib/db'
import { users } from '@/lib/db/schema'

const createUserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
})

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url)
    const page = parseInt(searchParams.get('page') || '1')
    const pageSize = parseInt(searchParams.get('pageSize') || '20')

    const allUsers = await db
      .select()
      .from(users)
      .limit(pageSize)
      .offset((page - 1) * pageSize)

    const total = await db.$count(users)

    return NextResponse.json({
      success: true,
      data: allUsers,
      pagination: {
        page,
        pageSize,
        total,
        totalPages: Math.ceil(total / pageSize),
        hasNext: page * pageSize < total,
        hasPrev: page > 1
      }
    })
  } catch (error) {
    return NextResponse.json(
      {
        success: false,
        error: {
          code: 'INTERNAL_ERROR',
          message: 'Failed to fetch users'
        }
      },
      { status: 500 }
    )
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    const validatedData = createUserSchema.parse(body)

    const newUser = await db
      .insert(users)
      .values(validatedData)
      .returning()

    return NextResponse.json(
      {
        success: true,
        data: newUser[0]
      },
      { status: 201 }
    )
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        {
          success: false,
          error: {
            code: 'VALIDATION_ERROR',
            message: 'Invalid input',
            details: error.errors
          }
        },
        { status: 422 }
      )
    }

    return NextResponse.json(
      {
        success: false,
        error: {
          code: 'INTERNAL_ERROR',
          message: 'Failed to create user'
        }
      },
      { status: 500 }
    )
  }
}
```

### Dynamic Route with ID
```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { db } from '@/lib/db'
import { users } from '@/lib/db/schema'
import { eq } from 'drizzle-orm'

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const user = await db.query.users.findFirst({
      where: eq(users.id, params.id)
    })

    if (!user) {
      return NextResponse.json(
        {
          success: false,
          error: {
            code: 'NOT_FOUND',
            message: 'User not found'
          }
        },
        { status: 404 }
      )
    }

    return NextResponse.json({
      success: true,
      data: user
    })
  } catch (error) {
    return NextResponse.json(
      {
        success: false,
        error: {
          code: 'INTERNAL_ERROR',
          message: 'Failed to fetch user'
        }
      },
      { status: 500 }
    )
  }
}

export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const body = await request.json()

    const updatedUser = await db
      .update(users)
      .set(body)
      .where(eq(users.id, params.id))
      .returning()

    if (!updatedUser.length) {
      return NextResponse.json(
        {
          success: false,
          error: {
            code: 'NOT_FOUND',
            message: 'User not found'
          }
        },
        { status: 404 }
      )
    }

    return NextResponse.json({
      success: true,
      data: updatedUser[0]
    })
  } catch (error) {
    return NextResponse.json(
      {
        success: false,
        error: {
          code: 'INTERNAL_ERROR',
          message: 'Failed to update user'
        }
      },
      { status: 500 }
    )
  }
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    await db.delete(users).where(eq(users.id, params.id))

    return new NextResponse(null, { status: 204 })
  } catch (error) {
    return NextResponse.json(
      {
        success: false,
        error: {
          code: 'INTERNAL_ERROR',
          message: 'Failed to delete user'
        }
      },
      { status: 500 }
    )
  }
}
```

## API Versioning

### URL Versioning (Recommended)
```typescript
// app/api/v1/users/route.ts
// app/api/v2/users/route.ts

// Shared logic
// lib/api/users/v1.ts
// lib/api/users/v2.ts
```

### Version Migration
```typescript
// lib/api/users/base.ts - Shared logic
export async function getUsers(version: 'v1' | 'v2') {
  const users = await db.select().from(usersTable)

  if (version === 'v1') {
    // Transform for v1 format
    return users.map(user => ({
      id: user.id,
      name: user.name,
      email: user.email
    }))
  }

  // v2 includes additional fields
  return users
}
```

## Authentication & Authorization

### JWT Middleware
```typescript
// lib/middleware/auth.ts
import { NextRequest } from 'next/server'
import { verify } from 'jsonwebtoken'

export async function requireAuth(request: NextRequest) {
  const token = request.headers.get('authorization')?.replace('Bearer ', '')

  if (!token) {
    return {
      error: {
        code: 'UNAUTHORIZED',
        message: 'No token provided'
      },
      status: 401
    }
  }

  try {
    const decoded = verify(token, process.env.JWT_SECRET!)
    return { user: decoded }
  } catch (error) {
    return {
      error: {
        code: 'UNAUTHORIZED',
        message: 'Invalid token'
      },
      status: 401
    }
  }
}

// Usage in route
export async function GET(request: NextRequest) {
  const auth = await requireAuth(request)
  if ('error' in auth) {
    return NextResponse.json(auth.error, { status: auth.status })
  }

  // Continue with authorized user
  const user = auth.user
}
```

### Role-Based Access Control
```typescript
// lib/middleware/rbac.ts
export function requireRole(allowedRoles: string[]) {
  return async (request: NextRequest) => {
    const auth = await requireAuth(request)
    if ('error' in auth) return auth

    if (!allowedRoles.includes(auth.user.role)) {
      return {
        error: {
          code: 'FORBIDDEN',
          message: 'Insufficient permissions'
        },
        status: 403
      }
    }

    return auth
  }
}

// Usage
export async function DELETE(request: NextRequest) {
  const auth = await requireRole(['admin'])(request)
  if ('error' in auth) {
    return NextResponse.json(auth.error, { status: auth.status })
  }

  // Admin-only logic
}
```

## Rate Limiting

### Upstash Redis Rate Limiting
```typescript
// lib/middleware/rate-limit.ts
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'
import { NextRequest, NextResponse } from 'next/server'

const redis = Redis.fromEnv()

export const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '10 s'), // 10 requests per 10 seconds
  analytics: true,
})

export async function withRateLimit(
  request: NextRequest,
  handler: () => Promise<NextResponse>
) {
  const identifier = request.ip ?? 'anonymous'
  const { success, limit, reset, remaining } = await ratelimit.limit(identifier)

  if (!success) {
    return NextResponse.json(
      {
        success: false,
        error: {
          code: 'RATE_LIMIT_EXCEEDED',
          message: 'Too many requests'
        }
      },
      {
        status: 429,
        headers: {
          'X-RateLimit-Limit': limit.toString(),
          'X-RateLimit-Remaining': remaining.toString(),
          'X-RateLimit-Reset': reset.toString()
        }
      }
    )
  }

  return handler()
}

// Usage
export async function GET(request: NextRequest) {
  return withRateLimit(request, async () => {
    // Your API logic
    return NextResponse.json({ data: 'success' })
  })
}
```

## Pagination Strategies

### Offset-Based Pagination
```typescript
// Good for small datasets
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url)
  const page = parseInt(searchParams.get('page') || '1')
  const pageSize = parseInt(searchParams.get('pageSize') || '20')

  const items = await db
    .select()
    .from(table)
    .limit(pageSize)
    .offset((page - 1) * pageSize)

  return NextResponse.json({ data: items })
}
```

### Cursor-Based Pagination
```typescript
// Better for large datasets
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url)
  const cursor = searchParams.get('cursor')
  const limit = parseInt(searchParams.get('limit') || '20')

  const items = await db
    .select()
    .from(table)
    .where(cursor ? gt(table.id, cursor) : undefined)
    .limit(limit + 1) // Fetch one extra to check if there's more
    .orderBy(table.id)

  const hasMore = items.length > limit
  const results = hasMore ? items.slice(0, limit) : items
  const nextCursor = hasMore ? results[results.length - 1].id : null

  return NextResponse.json({
    data: results,
    pagination: {
      nextCursor,
      hasMore
    }
  })
}
```

## Error Handling

### Centralized Error Handler
```typescript
// lib/api/error-handler.ts
export class ApiError extends Error {
  constructor(
    public code: string,
    public message: string,
    public status: number = 500,
    public details?: any
  ) {
    super(message)
  }
}

export function handleApiError(error: unknown) {
  if (error instanceof ApiError) {
    return NextResponse.json(
      {
        success: false,
        error: {
          code: error.code,
          message: error.message,
          details: error.details
        }
      },
      { status: error.status }
    )
  }

  if (error instanceof z.ZodError) {
    return NextResponse.json(
      {
        success: false,
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Invalid input',
          details: error.errors
        }
      },
      { status: 422 }
    )
  }

  // Log unexpected errors
  console.error('Unexpected API error:', error)

  return NextResponse.json(
    {
      success: false,
      error: {
        code: 'INTERNAL_ERROR',
        message: 'An unexpected error occurred'
      }
    },
    { status: 500 }
  )
}

// Usage
export async function POST(request: NextRequest) {
  try {
    // Your logic
    if (someCondition) {
      throw new ApiError('INVALID_INPUT', 'Invalid data provided', 400)
    }

    return NextResponse.json({ success: true })
  } catch (error) {
    return handleApiError(error)
  }
}
```

## API Documentation

### OpenAPI/Swagger Setup
```typescript
// lib/api/swagger.ts
import { createSwaggerSpec } from 'next-swagger-doc'

export const getApiDocs = () => {
  return createSwaggerSpec({
    apiFolder: 'app/api',
    definition: {
      openapi: '3.0.0',
      info: {
        title: 'Your API',
        version: '1.0.0',
        description: 'API documentation'
      },
      servers: [
        {
          url: 'https://api.example.com',
          description: 'Production'
        },
        {
          url: 'http://localhost:3000',
          description: 'Development'
        }
      ]
    }
  })
}
```

## When to Use This Skill

- Designing new APIs
- API refactoring
- Implementing versioning
- Adding authentication
- Setting up rate limiting
- API documentation
- Performance optimization
- Scaling concerns
