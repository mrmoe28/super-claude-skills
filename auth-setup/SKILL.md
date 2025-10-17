---
name: auth-setup
description: Quick, error-free authentication setup for Next.js using NextAuth, Clerk, or custom solutions with proper TypeScript, middleware, and database integration
license: Apache-2.0
allowed-tools:
  - Bash(npm*:*)
  - Bash(npx*:*)
  - Read
  - Write
  - Edit
metadata:
  version: "1.0.0"
  solutions: "NextAuth, Clerk, Custom JWT"
---

# Auth Setup

Quick, error-free authentication setup for Next.js applications.

## Choosing Auth Solution

**NextAuth (Auth.js) - Best for:**
- Full control over auth flow
- Self-hosted solution
- Custom database
- Multiple providers (Google, GitHub, etc.)
- Email/password auth

**Clerk - Best for:**
- Fastest setup (10 minutes)
- Beautiful pre-built UI
- User management dashboard
- Multi-tenancy support
- Don't want to manage auth infra

**Custom JWT - Best for:**
- API-only applications
- Mobile apps
- Very specific requirements
- Learning purposes

## NextAuth Setup (Recommended)

### Step 1: Installation

```bash
npm install next-auth@beta @auth/drizzle-adapter
```

### Step 2: Database Schema

```typescript
// lib/db/schema/auth.ts
import { pgTable, text, timestamp, primaryKey, integer } from 'drizzle-orm/pg-core'

export const users = pgTable('users', {
  id: text('id').primaryKey(),
  name: text('name'),
  email: text('email').unique().notNull(),
  emailVerified: timestamp('email_verified'),
  image: text('image'),
  password: text('password'), // For credentials provider
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow()
})

export const accounts = pgTable('accounts', {
  userId: text('user_id').notNull().references(() => users.id, { onDelete: 'cascade' }),
  type: text('type').notNull(),
  provider: text('provider').notNull(),
  providerAccountId: text('provider_account_id').notNull(),
  refresh_token: text('refresh_token'),
  access_token: text('access_token'),
  expires_at: integer('expires_at'),
  token_type: text('token_type'),
  scope: text('scope'),
  id_token: text('id_token'),
  session_state: text('session_state'),
}, (account) => ({
  compoundKey: primaryKey({ columns: [account.provider, account.providerAccountId] })
}))

export const sessions = pgTable('sessions', {
  sessionToken: text('session_token').primaryKey(),
  userId: text('user_id').notNull().references(() => users.id, { onDelete: 'cascade' }),
  expires: timestamp('expires').notNull()
})

export const verificationTokens = pgTable('verification_tokens', {
  identifier: text('identifier').notNull(),
  token: text('token').notNull(),
  expires: timestamp('expires').notNull()
}, (token) => ({
  compoundKey: primaryKey({ columns: [token.identifier, token.token] })
}))
```

### Step 3: Auth Configuration

```typescript
// lib/auth/config.ts
import NextAuth from 'next-auth'
import GitHub from 'next-auth/providers/github'
import Google from 'next-auth/providers/google'
import Credentials from 'next-auth/providers/credentials'
import { DrizzleAdapter } from '@auth/drizzle-adapter'
import { db } from '@/lib/db'
import { users } from '@/lib/db/schema'
import { eq } from 'drizzle-orm'
import bcrypt from 'bcryptjs'

export const { handlers, signIn, signOut, auth } = NextAuth({
  adapter: DrizzleAdapter(db),
  session: { strategy: 'jwt' },
  pages: {
    signIn: '/login',
    signOut: '/logout',
    error: '/auth/error',
    verifyRequest: '/auth/verify',
    newUser: '/onboarding'
  },
  providers: [
    GitHub({
      clientId: process.env.GITHUB_ID!,
      clientSecret: process.env.GITHUB_SECRET!
    }),
    Google({
      clientId: process.env.GOOGLE_ID!,
      clientSecret: process.env.GOOGLE_SECRET!
    }),
    Credentials({
      name: 'Credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' }
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null
        }

        const user = await db.query.users.findFirst({
          where: eq(users.email, credentials.email as string)
        })

        if (!user || !user.password) {
          return null
        }

        const isValid = await bcrypt.compare(
          credentials.password as string,
          user.password
        )

        if (!isValid) {
          return null
        }

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          image: user.image
        }
      }
    })
  ],
  callbacks: {
    async jwt({ token, user, account }) {
      if (user) {
        token.id = user.id
      }
      return token
    },
    async session({ session, token }) {
      if (session.user) {
        session.user.id = token.id as string
      }
      return session
    }
  }
})
```

### Step 4: API Routes

```typescript
// app/api/auth/[...nextauth]/route.ts
import { handlers } from '@/lib/auth/config'

export const { GET, POST } = handlers
```

### Step 5: Middleware

```typescript
// middleware.ts
import { auth } from '@/lib/auth/config'
import { NextResponse } from 'next/server'

export default auth((req) => {
  const isLoggedIn = !!req.auth
  const isAuthPage = req.nextUrl.pathname.startsWith('/login') ||
                     req.nextUrl.pathname.startsWith('/register')
  const isProtectedRoute = req.nextUrl.pathname.startsWith('/dashboard')

  // Redirect logged-in users away from auth pages
  if (isAuthPage && isLoggedIn) {
    return NextResponse.redirect(new URL('/dashboard', req.url))
  }

  // Redirect non-logged-in users to login
  if (isProtectedRoute && !isLoggedIn) {
    return NextResponse.redirect(new URL('/login', req.url))
  }

  return NextResponse.next()
})

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)']
}
```

### Step 6: Environment Variables

```bash
# .env.local
# NextAuth
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-secret-here-min-32-chars

# GitHub OAuth
GITHUB_ID=your-github-client-id
GITHUB_SECRET=your-github-client-secret

# Google OAuth
GOOGLE_ID=your-google-client-id
GOOGLE_SECRET=your-google-client-secret

# Database
DATABASE_URL=your-database-url
```

**Generate NEXTAUTH_SECRET:**
```bash
openssl rand -base64 32
```

### Step 7: Login Page

```typescript
// app/login/page.tsx
import { signIn } from '@/lib/auth/config'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'

export default function LoginPage() {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <div className="w-full max-w-md space-y-6 rounded-lg border p-8">
        <h1 className="text-2xl font-bold">Sign In</h1>

        <form
          action={async (formData) => {
            'use server'
            await signIn('credentials', formData)
          }}
          className="space-y-4"
        >
          <Input
            name="email"
            type="email"
            placeholder="Email"
            required
          />
          <Input
            name="password"
            type="password"
            placeholder="Password"
            required
          />
          <Button type="submit" className="w-full">
            Sign In
          </Button>
        </form>

        <div className="relative">
          <div className="absolute inset-0 flex items-center">
            <span className="w-full border-t" />
          </div>
          <div className="relative flex justify-center text-xs uppercase">
            <span className="bg-background px-2 text-muted-foreground">
              Or continue with
            </span>
          </div>
        </div>

        <div className="grid gap-2">
          <form
            action={async () => {
              'use server'
              await signIn('github')
            }}
          >
            <Button variant="outline" className="w-full">
              GitHub
            </Button>
          </form>

          <form
            action={async () => {
              'use server'
              await signIn('google')
            }}
          >
            <Button variant="outline" className="w-full">
              Google
            </Button>
          </form>
        </div>
      </div>
    </div>
  )
}
```

### Step 8: Protect Pages

```typescript
// app/dashboard/page.tsx
import { auth } from '@/lib/auth/config'
import { redirect } from 'next/navigation'

export default async function DashboardPage() {
  const session = await auth()

  if (!session) {
    redirect('/login')
  }

  return (
    <div>
      <h1>Welcome, {session.user?.name}!</h1>
    </div>
  )
}
```

### Step 9: Client-Side Session

```typescript
// providers/session-provider.tsx
'use client'

import { SessionProvider as NextAuthSessionProvider } from 'next-auth/react'

export function SessionProvider({ children }: { children: React.ReactNode }) {
  return <NextAuthSessionProvider>{children}</NextAuthSessionProvider>
}

// app/layout.tsx
import { SessionProvider } from '@/providers/session-provider'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <SessionProvider>
          {children}
        </SessionProvider>
      </body>
    </html>
  )
}

// Use in client components
'use client'

import { useSession, signOut } from 'next-auth/react'

export function UserButton() {
  const { data: session } = useSession()

  if (!session) return <div>Not logged in</div>

  return (
    <div>
      <p>{session.user?.name}</p>
      <button onClick={() => signOut()}>Sign Out</button>
    </div>
  )
}
```

## Clerk Setup (Fastest)

### Step 1: Installation

```bash
npm install @clerk/nextjs
```

### Step 2: Environment Variables

```bash
# .env.local
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...

# Sign-in/up routes
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/onboarding
```

### Step 3: Wrap App

```typescript
// app/layout.tsx
import { ClerkProvider } from '@clerk/nextjs'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <ClerkProvider>
      <html>
        <body>{children}</body>
      </html>
    </ClerkProvider>
  )
}
```

### Step 4: Middleware

```typescript
// middleware.ts
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server'

const isProtectedRoute = createRouteMatcher(['/dashboard(.*)'])

export default clerkMiddleware((auth, req) => {
  if (isProtectedRoute(req)) auth().protect()
})

export const config = {
  matcher: ['/((?!.*\\..*|_next).*)', '/', '/(api|trpc)(.*)']
}
```

### Step 5: Sign In/Up Pages

```typescript
// app/sign-in/[[...sign-in]]/page.tsx
import { SignIn } from '@clerk/nextjs'

export default function SignInPage() {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <SignIn />
    </div>
  )
}

// app/sign-up/[[...sign-up]]/page.tsx
import { SignUp } from '@clerk/nextjs'

export default function SignUpPage() {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <SignUp />
    </div>
  )
}
```

### Step 6: Protect Pages

```typescript
// app/dashboard/page.tsx
import { auth } from '@clerk/nextjs/server'
import { redirect } from 'next/navigation'

export default async function DashboardPage() {
  const { userId } = await auth()

  if (!userId) {
    redirect('/sign-in')
  }

  return <div>Dashboard</div>
}
```

### Step 7: User Button

```typescript
// components/UserButton.tsx
import { UserButton as ClerkUserButton } from '@clerk/nextjs'

export function UserButton() {
  return <ClerkUserButton afterSignOutUrl="/" />
}
```

## Common Errors & Solutions

### Error 1: NEXTAUTH_SECRET not set

```bash
# Generate secret
openssl rand -base64 32

# Add to .env.local
NEXTAUTH_SECRET=generated-secret-here
```

### Error 2: Redirect URI mismatch

**Solution:**
- In GitHub/Google OAuth settings:
  - Authorized redirect URI: `http://localhost:3000/api/auth/callback/github`
  - For production: `https://yourdomain.com/api/auth/callback/github`

### Error 3: Session not persisting

**Solution:**
```typescript
// Ensure session strategy matches
export const { auth } = NextAuth({
  session: { strategy: 'jwt' }, // or 'database'
  // ...
})
```

### Error 4: Type errors with session

```typescript
// types/next-auth.d.ts
import { DefaultSession } from 'next-auth'

declare module 'next-auth' {
  interface Session {
    user: {
      id: string
    } & DefaultSession['user']
  }
}
```

### Error 5: Middleware not protecting routes

**Solution:**
```typescript
// middleware.ts - Correct matcher
export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico).*)'
  ]
}
```

## Password Hashing (Credentials Provider)

```typescript
// lib/auth/password.ts
import bcrypt from 'bcryptjs'

export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, 12)
}

export async function verifyPassword(
  password: string,
  hashedPassword: string
): Promise<boolean> {
  return bcrypt.compare(password, hashedPassword)
}
```

## Registration Handler

```typescript
// app/api/auth/register/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { db } from '@/lib/db'
import { users } from '@/lib/db/schema'
import { hashPassword } from '@/lib/auth/password'
import { z } from 'zod'

const registerSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  password: z.string().min(8)
})

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()
    const { name, email, password } = registerSchema.parse(body)

    // Check if user exists
    const existing = await db.query.users.findFirst({
      where: eq(users.email, email)
    })

    if (existing) {
      return NextResponse.json(
        { error: 'Email already registered' },
        { status: 400 }
      )
    }

    // Hash password
    const hashedPassword = await hashPassword(password)

    // Create user
    const user = await db.insert(users).values({
      id: generateId(),
      name,
      email,
      password: hashedPassword
    }).returning()

    return NextResponse.json({
      success: true,
      userId: user[0].id
    })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Invalid input', details: error.errors },
        { status: 422 }
      )
    }

    return NextResponse.json(
      { error: 'Registration failed' },
      { status: 500 }
    )
  }
}
```

## Setup Checklist

**Before deployment:**
- ✅ Environment variables set
- ✅ OAuth apps created and configured
- ✅ Redirect URIs match exactly
- ✅ Database schema created
- ✅ NEXTAUTH_SECRET generated (32+ chars)
- ✅ NEXTAUTH_URL matches domain
- ✅ Middleware configured
- ✅ Session strategy chosen
- ✅ Protected routes tested
- ✅ Sign out tested

## When to Use This Skill

- Adding auth to new app
- Setting up OAuth providers
- Implementing email/password auth
- Protecting routes
- Session management
- User registration
- Password reset
- Multi-factor auth setup

## Best Practices

1. **Use NextAuth for most cases** - Battle-tested, flexible
2. **Use Clerk for speed** - If you want managed auth
3. **Always hash passwords** - Never store plain text
4. **Validate email format** - Prevent invalid registrations
5. **Rate limit auth endpoints** - Prevent brute force
6. **Use HTTPS in production** - Always
7. **Test logout thoroughly** - Sessions must clear properly
8. **Handle OAuth errors** - Users will see them
