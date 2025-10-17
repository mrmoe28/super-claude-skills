---
name: subscription-builder
description: Add subscription and recurring billing features to apps using Stripe, Square, and other payment platforms with proper webhooks, trial management, and cancellation flows
license: Apache-2.0
allowed-tools:
  - Bash(npm*:*)
  - Read
  - Write
  - Edit
metadata:
  version: "1.0.0"
  platforms: "Stripe, Square"
---

# Subscription Builder

Expert guidance for adding subscription billing to applications with proper implementation patterns.

## Subscription System Architecture

### Core Components

**Required Elements:**
1. **Subscription Plans** - Define pricing tiers
2. **Customer Management** - Link users to payment platform
3. **Subscription State** - Track active/cancelled/past_due
4. **Webhook Handling** - Process payment events
5. **Usage Tracking** - For metered billing
6. **Billing Portal** - User self-service

## Database Schema

### Complete Subscription Schema

```typescript
// lib/db/schema/subscriptions.ts
import { pgTable, text, timestamp, integer, boolean, jsonb } from 'drizzle-orm/pg-core'

export const subscriptionPlans = pgTable('subscription_plans', {
  id: text('id').primaryKey(),
  name: text('name').notNull(),
  description: text('description'),
  // Pricing
  price: integer('price').notNull(), // in cents
  currency: text('currency').notNull().default('usd'),
  interval: text('interval').notNull(), // month, year
  intervalCount: integer('interval_count').notNull().default(1),
  trialDays: integer('trial_days').default(0),
  // Metadata
  features: jsonb('features').$type<string[]>(),
  // Platform IDs
  stripeProductId: text('stripe_product_id'),
  stripePriceId: text('stripe_price_id'),
  squareSubscriptionPlanId: text('square_subscription_plan_id'),
  // Status
  active: boolean('active').notNull().default(true),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow()
})

export const customers = pgTable('customers', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull().references(() => users.id),
  // Platform customer IDs
  stripeCustomerId: text('stripe_customer_id'),
  squareCustomerId: text('square_customer_id'),
  // Metadata
  email: text('email').notNull(),
  name: text('name'),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow()
})

export const subscriptions = pgTable('subscriptions', {
  id: text('id').primaryKey(),
  customerId: text('customer_id').notNull().references(() => customers.id),
  planId: text('plan_id').notNull().references(() => subscriptionPlans.id),
  // Platform subscription IDs
  stripeSubscriptionId: text('stripe_subscription_id'),
  squareSubscriptionId: text('square_subscription_id'),
  // Status
  status: text('status').notNull(), // active, past_due, cancelled, trialing
  // Dates
  currentPeriodStart: timestamp('current_period_start').notNull(),
  currentPeriodEnd: timestamp('current_period_end').notNull(),
  trialStart: timestamp('trial_start'),
  trialEnd: timestamp('trial_end'),
  cancelAt: timestamp('cancel_at'),
  canceledAt: timestamp('canceled_at'),
  endedAt: timestamp('ended_at'),
  // Metadata
  metadata: jsonb('metadata').$type<Record<string, any>>(),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow()
})

export const invoices = pgTable('invoices', {
  id: text('id').primaryKey(),
  subscriptionId: text('subscription_id').references(() => subscriptions.id),
  customerId: text('customer_id').notNull().references(() => customers.id),
  // Platform invoice IDs
  stripeInvoiceId: text('stripe_invoice_id'),
  squareInvoiceId: text('square_invoice_id'),
  // Amounts
  amountDue: integer('amount_due').notNull(),
  amountPaid: integer('amount_paid').notNull().default(0),
  currency: text('currency').notNull().default('usd'),
  // Status
  status: text('status').notNull(), // draft, open, paid, void, uncollectible
  // Dates
  dueDate: timestamp('due_date'),
  paidAt: timestamp('paid_at'),
  // Metadata
  hostedInvoiceUrl: text('hosted_invoice_url'),
  invoicePdf: text('invoice_pdf'),
  createdAt: timestamp('created_at').notNull().defaultNow()
})
```

## Stripe Implementation

### Setup

```bash
npm install stripe @stripe/stripe-js
```

```typescript
// lib/stripe/client.ts
import Stripe from 'stripe'

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-12-18.acacia',
  typescript: true
})
```

### Create Subscription Plans

```typescript
// lib/stripe/plans.ts
export async function createStripePlans() {
  // Create product
  const product = await stripe.products.create({
    name: 'Pro Plan',
    description: 'Pro features for power users'
  })

  // Create price
  const price = await stripe.prices.create({
    product: product.id,
    unit_amount: 2900, // $29.00
    currency: 'usd',
    recurring: {
      interval: 'month',
      trial_period_days: 14
    }
  })

  // Save to database
  await db.insert(subscriptionPlans).values({
    id: generateId(),
    name: 'Pro Plan',
    price: 2900,
    currency: 'usd',
    interval: 'month',
    intervalCount: 1,
    trialDays: 14,
    stripeProductId: product.id,
    stripePriceId: price.id,
    features: ['feature1', 'feature2'],
    active: true
  })
}
```

### Create Customer & Subscription

```typescript
// app/api/subscriptions/create/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe/client'
import { db } from '@/lib/db'
import { customers, subscriptions } from '@/lib/db/schema'

export async function POST(request: NextRequest) {
  const { userId, planId, paymentMethodId } = await request.json()

  // Get user
  const user = await db.query.users.findFirst({
    where: eq(users.id, userId)
  })

  // Create or get Stripe customer
  let customer = await db.query.customers.findFirst({
    where: eq(customers.userId, userId)
  })

  if (!customer) {
    const stripeCustomer = await stripe.customers.create({
      email: user.email,
      name: user.name,
      metadata: { userId }
    })

    customer = await db.insert(customers).values({
      id: generateId(),
      userId,
      email: user.email,
      name: user.name,
      stripeCustomerId: stripeCustomer.id
    }).returning()
  }

  // Get plan
  const plan = await db.query.subscriptionPlans.findFirst({
    where: eq(subscriptionPlans.id, planId)
  })

  // Attach payment method
  await stripe.paymentMethods.attach(paymentMethodId, {
    customer: customer.stripeCustomerId!
  })

  // Set as default payment method
  await stripe.customers.update(customer.stripeCustomerId!, {
    invoice_settings: {
      default_payment_method: paymentMethodId
    }
  })

  // Create subscription
  const stripeSubscription = await stripe.subscriptions.create({
    customer: customer.stripeCustomerId!,
    items: [{ price: plan.stripePriceId! }],
    trial_period_days: plan.trialDays || undefined,
    payment_behavior: 'default_incomplete',
    expand: ['latest_invoice.payment_intent'],
    metadata: { userId, planId }
  })

  // Save to database
  const subscription = await db.insert(subscriptions).values({
    id: generateId(),
    customerId: customer.id,
    planId: plan.id,
    stripeSubscriptionId: stripeSubscription.id,
    status: stripeSubscription.status,
    currentPeriodStart: new Date(stripeSubscription.current_period_start * 1000),
    currentPeriodEnd: new Date(stripeSubscription.current_period_end * 1000),
    trialStart: stripeSubscription.trial_start
      ? new Date(stripeSubscription.trial_start * 1000)
      : null,
    trialEnd: stripeSubscription.trial_end
      ? new Date(stripeSubscription.trial_end * 1000)
      : null
  }).returning()

  return NextResponse.json({
    subscriptionId: subscription[0].id,
    clientSecret: (stripeSubscription.latest_invoice as any).payment_intent.client_secret
  })
}
```

### Webhook Handler

```typescript
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe/client'
import { headers } from 'next/headers'

const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!

export async function POST(request: NextRequest) {
  const body = await request.text()
  const signature = headers().get('stripe-signature')!

  let event: Stripe.Event

  try {
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret)
  } catch (err) {
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 })
  }

  switch (event.type) {
    case 'customer.subscription.created':
    case 'customer.subscription.updated':
      await handleSubscriptionUpdate(event.data.object as Stripe.Subscription)
      break

    case 'customer.subscription.deleted':
      await handleSubscriptionCancelled(event.data.object as Stripe.Subscription)
      break

    case 'invoice.paid':
      await handleInvoicePaid(event.data.object as Stripe.Invoice)
      break

    case 'invoice.payment_failed':
      await handlePaymentFailed(event.data.object as Stripe.Invoice)
      break
  }

  return NextResponse.json({ received: true })
}

async function handleSubscriptionUpdate(stripeSubscription: Stripe.Subscription) {
  await db
    .update(subscriptions)
    .set({
      status: stripeSubscription.status,
      currentPeriodStart: new Date(stripeSubscription.current_period_start * 1000),
      currentPeriodEnd: new Date(stripeSubscription.current_period_end * 1000),
      cancelAt: stripeSubscription.cancel_at
        ? new Date(stripeSubscription.cancel_at * 1000)
        : null,
      updatedAt: new Date()
    })
    .where(eq(subscriptions.stripeSubscriptionId, stripeSubscription.id))
}

async function handleSubscriptionCancelled(stripeSubscription: Stripe.Subscription) {
  await db
    .update(subscriptions)
    .set({
      status: 'cancelled',
      endedAt: new Date(),
      updatedAt: new Date()
    })
    .where(eq(subscriptions.stripeSubscriptionId, stripeSubscription.id))
}

async function handleInvoicePaid(invoice: Stripe.Invoice) {
  await db.insert(invoices).values({
    id: generateId(),
    subscriptionId: invoice.subscription as string,
    customerId: invoice.customer as string,
    stripeInvoiceId: invoice.id,
    amountDue: invoice.amount_due,
    amountPaid: invoice.amount_paid,
    status: 'paid',
    paidAt: new Date(),
    hostedInvoiceUrl: invoice.hosted_invoice_url!,
    invoicePdf: invoice.invoice_pdf!
  })
}

async function handlePaymentFailed(invoice: Stripe.Invoice) {
  const subscription = await db.query.subscriptions.findFirst({
    where: eq(subscriptions.stripeSubscriptionId, invoice.subscription as string)
  })

  // Send payment failed email
  await sendPaymentFailedEmail(subscription)
}
```

### Customer Portal

```typescript
// app/api/billing/portal/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe/client'

export async function POST(request: NextRequest) {
  const { customerId } = await request.json()

  const customer = await db.query.customers.findFirst({
    where: eq(customers.id, customerId)
  })

  const session = await stripe.billingPortal.sessions.create({
    customer: customer.stripeCustomerId!,
    return_url: `${process.env.NEXT_PUBLIC_URL}/dashboard/billing`
  })

  return NextResponse.json({ url: session.url })
}
```

## Square Implementation

### Setup

```bash
npm install square
```

```typescript
// lib/square/client.ts
import { Client, Environment } from 'square'

export const squareClient = new Client({
  accessToken: process.env.SQUARE_ACCESS_TOKEN!,
  environment: process.env.NODE_ENV === 'production'
    ? Environment.Production
    : Environment.Sandbox
})
```

### Create Subscription

```typescript
// app/api/subscriptions/square/create/route.ts
import { squareClient } from '@/lib/square/client'

export async function POST(request: NextRequest) {
  const { userId, planId, cardId, locationId } = await request.json()

  // Get or create customer
  let customer = await db.query.customers.findFirst({
    where: eq(customers.userId, userId)
  })

  if (!customer) {
    const result = await squareClient.customersApi.createCustomer({
      emailAddress: user.email,
      givenName: user.name,
      referenceId: userId
    })

    customer = await db.insert(customers).values({
      id: generateId(),
      userId,
      email: user.email,
      squareCustomerId: result.result.customer!.id
    }).returning()
  }

  // Create subscription
  const plan = await db.query.subscriptionPlans.findFirst({
    where: eq(subscriptionPlans.id, planId)
  })

  const result = await squareClient.subscriptionsApi.createSubscription({
    locationId,
    planId: plan.squareSubscriptionPlanId!,
    customerId: customer.squareCustomerId!,
    cardId,
    startDate: new Date().toISOString().split('T')[0]
  })

  // Save to database
  await db.insert(subscriptions).values({
    id: generateId(),
    customerId: customer.id,
    planId: plan.id,
    squareSubscriptionId: result.result.subscription!.id,
    status: result.result.subscription!.status!,
    currentPeriodStart: new Date(result.result.subscription!.startDate!),
    currentPeriodEnd: new Date(result.result.subscription!.chargedThroughDate!)
  })

  return NextResponse.json({ subscription: result.result.subscription })
}
```

## Frontend Components

### Subscription Plans Display

```typescript
// components/SubscriptionPlans.tsx
'use client'

import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'

export function SubscriptionPlans({ plans }: { plans: SubscriptionPlan[] }) {
  async function handleSubscribe(planId: string) {
    // Show payment modal
    const paymentMethodId = await showPaymentModal()

    // Create subscription
    const response = await fetch('/api/subscriptions/create', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ planId, paymentMethodId })
    })

    const { subscriptionId } = await response.json()
    // Redirect to success page
    window.location.href = '/dashboard/billing/success'
  }

  return (
    <div className="grid gap-6 md:grid-cols-3">
      {plans.map(plan => (
        <Card key={plan.id}>
          <CardHeader>
            <CardTitle>{plan.name}</CardTitle>
            <div className="text-3xl font-bold">
              ${plan.price / 100}
              <span className="text-sm font-normal text-muted-foreground">
                /{plan.interval}
              </span>
            </div>
          </CardHeader>
          <CardContent>
            <ul className="space-y-2">
              {plan.features.map(feature => (
                <li key={feature} className="flex items-center">
                  ✓ {feature}
                </li>
              ))}
            </ul>
            <Button
              className="mt-6 w-full"
              onClick={() => handleSubscribe(plan.id)}
            >
              Subscribe
            </Button>
          </CardContent>
        </Card>
      ))}
    </div>
  )
}
```

## Access Control

### Check Subscription Status

```typescript
// lib/subscription/check-access.ts
export async function checkSubscriptionAccess(userId: string): Promise<boolean> {
  const customer = await db.query.customers.findFirst({
    where: eq(customers.userId, userId)
  })

  if (!customer) return false

  const subscription = await db.query.subscriptions.findFirst({
    where: and(
      eq(subscriptions.customerId, customer.id),
      eq(subscriptions.status, 'active')
    )
  })

  return !!subscription
}

// Middleware
export async function requireSubscription(userId: string) {
  const hasAccess = await checkSubscriptionAccess(userId)

  if (!hasAccess) {
    throw new Error('Active subscription required')
  }
}
```

## Best Practices

1. **Always use webhooks** - Don't rely on client-side confirmation
2. **Handle trial periods** - Clearly communicate trial end dates
3. **Graceful degradation** - Don't immediately block on payment failure
4. **Email notifications** - Inform users of payment events
5. **Proration** - Handle plan upgrades/downgrades correctly
6. **Cancellation flow** - Make it easy but ask for feedback
7. **Invoice storage** - Keep records for accounting
8. **Idempotency** - Handle duplicate webhook events

## When to Use This Skill

- Adding subscriptions to existing app
- Building SaaS billing
- Implementing trials
- Managing recurring payments
- Customer portal setup
- Invoice generation
- Usage-based billing
