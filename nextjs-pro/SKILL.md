---
name: nextjs-pro
description: Expert Next.js 15 and React 19 development following 2025 best practices with App Router, Server Components, TypeScript, and modern tooling
license: Apache-2.0
allowed-tools:
  - Bash(npm*:*)
  - Bash(npx*:*)
  - Read
  - Write
  - Edit
  - Glob
  - Grep
metadata:
  version: "1.0.0"
  stack: "Next.js 15, React 19, TypeScript, Tailwind 4"
---

# Next.js Pro

Expert guidance for Next.js 15 and React 19 development following 2025 best practices.

## Core Principles

### Next.js 15 Standards (2025)
- **App Router ONLY** - Never use Pages Router
- **React 19** - Leverage latest features and Server Components
- **TypeScript** - Always use next.config.ts (not .js)
- **No Turbopack in Production** - Standard builds for maximum compatibility
- **Server Components Default** - Use 'use client' only when necessary
- **Typed Routes** - Enable typedRoutes in next.config.ts

### Mandatory Project Structure
```
src/
├── app/                    # App Router (MANDATORY)
│   ├── layout.tsx         # Root layout with React 19
│   ├── page.tsx           # Homepage
│   ├── loading.tsx        # Loading UI
│   ├── error.tsx          # Error boundaries
│   ├── not-found.tsx      # 404 page
│   ├── (auth)/            # Route groups
│   ├── (main)/            # Main app routes
│   └── api/               # API routes
│       └── [version]/     # API versioning
├── components/
│   ├── ui/                # ShadCN components (MANDATORY)
│   ├── features/          # Feature-specific components
│   └── layouts/           # Layout components
├── lib/
│   ├── utils.ts           # General utilities
│   ├── validations.ts     # Zod schemas
│   └── api.ts             # API configuration
├── hooks/                 # Custom React hooks
├── types/                 # TypeScript definitions
├── providers/             # Context providers
└── styles/                # Global styles
```

## Required Dependencies
```json
{
  "dependencies": {
    "next": "^15.5.3",
    "react": "^19.1.1",
    "react-dom": "^19.1.1",
    "typescript": "^5.9.2",
    "@shadcn/ui": "latest",
    "tailwindcss": "^4.1.13",
    "zod": "^3.23.8",
    "next-themes": "^0.3.0"
  }
}
```

## Next.js Config Template
```typescript
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  typescript: {
    ignoreBuildErrors: false,
  },
  eslint: {
    ignoreDuringBuilds: false,
  },
  typedRoutes: true,
  experimental: {
    turbo: {
      rules: {
        '*.svg': {
          loaders: ['@svgr/webpack'],
          as: '*.js',
        },
      },
    },
  },
}

export default nextConfig
```

## Component Templates

### Server Component (Default)
```typescript
import { Metadata } from 'next'

interface PageProps {
  params: { slug: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export const metadata: Metadata = {
  title: 'Page Title',
  description: 'Page description',
}

export default async function ServerPage({ params, searchParams }: PageProps) {
  const data = await fetch('https://api.example.com/data', {
    cache: 'force-cache',
  })

  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-6">{/* Title */}</h1>
    </div>
  )
}
```

### Client Component
```typescript
'use client'

import { useState } from 'react'
import { Button } from '@/components/ui/button'

interface ClientComponentProps {
  initialValue?: string
}

export default function ClientComponent({ initialValue = '' }: ClientComponentProps) {
  const [value, setValue] = useState(initialValue)

  return (
    <div className="space-y-4">
      <Button onClick={() => setValue('new value')}>
        Click me
      </Button>
    </div>
  )
}
```

### API Route with Validation
```typescript
import { NextRequest, NextResponse } from 'next/server'
import { z } from 'zod'

const userSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
})

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    const validatedData = userSchema.parse(body)

    const result = await createUser(validatedData)

    return NextResponse.json({ success: true, data: result })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      )
    }

    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}
```

## Performance & SEO Requirements
- **Images**: Always use next/image with proper sizing
- **Fonts**: Use next/font for Google Fonts
- **Metadata**: Export metadata from every page
- **Loading States**: Implement loading.tsx
- **Error Boundaries**: Add error.tsx
- **Streaming**: Use Suspense for progressive loading

## Development Commands
```bash
# Development
npm run dev

# Type checking
npx tsc --noEmit

# Linting (mandatory after changes)
npm run lint

# Build (must pass without errors)
npm run build

# ShadCN setup
npx shadcn@latest init
npx shadcn@latest add button input form
```

## Best Practices
1. **Server Components First** - Default to Server Components
2. **Client Components Sparingly** - Only for interactivity
3. **Data Fetching** - Use native fetch with caching strategies
4. **Type Safety** - Full TypeScript coverage
5. **Error Handling** - Comprehensive error boundaries
6. **Loading States** - Streaming and Suspense
7. **Route Groups** - Organize routes logically
8. **Metadata** - SEO optimization on every page

## Anti-Patterns to Avoid
- ❌ Using Pages Router
- ❌ Using Turbopack in production builds
- ❌ Missing metadata exports
- ❌ Not using TypeScript config file
- ❌ Ignoring ESLint/TypeScript errors
- ❌ Overusing 'use client'
- ❌ Not implementing loading/error states

## When to Use This Skill
- Creating new Next.js projects
- Building React 19 applications
- Implementing App Router patterns
- Setting up Server Components
- Configuring TypeScript with Next.js
- Optimizing performance and SEO
- Following 2025 best practices
