---
name: square-integration
description: Complete Square payment integration including one-time payments, subscriptions, webhooks, and Square POS with proper error handling and testing
license: Apache-2.0
allowed-tools:
  - Bash(npm*:*)
  - Read
  - Write
  - Edit
metadata:
  version: "1.0.0"
  platform: "Square"
---

# Square Integration

Complete guide for integrating Square payment processing into Next.js applications.

## Setup & Configuration

### Install Square SDK

```bash
npm install square
```

### Environment Variables

```bash
# .env.local
# Get from Square Developer Dashboard
SQUARE_ACCESS_TOKEN=your_access_token
SQUARE_LOCATION_ID=your_location_id
SQUARE_WEBHOOK_SIGNATURE_KEY=your_webhook_signature_key
SQUARE_ENVIRONMENT=sandbox # or production
NEXT_PUBLIC_SQUARE_APPLICATION_ID=your_app_id
```

### Initialize Square Client

```typescript
// lib/square/client.ts
import { Client, Environment } from 'square'

export const squareClient = new Client({
  accessToken: process.env.SQUARE_ACCESS_TOKEN!,
  environment: process.env.SQUARE_ENVIRONMENT === 'production'
    ? Environment.Production
    : Environment.Sandbox
})

export const locationId = process.env.SQUARE_LOCATION_ID!
```

## One-Time Payments

### Frontend Payment Form

```typescript
// components/SquarePaymentForm.tsx
'use client'

import { useEffect, useState } from 'react'
import { Button } from '@/components/ui/button'

export function SquarePaymentForm({ amount, onSuccess }: {
  amount: number
  onSuccess: (paymentId: string) => void
}) {
  const [card, setCard] = useState<any>(null)
  const [loading, setLoading] = useState(false)

  useEffect(() => {
    async function initializeSquare() {
      if (!window.Square) {
        throw new Error('Square.js failed to load')
      }

      const payments = window.Square.payments(
        process.env.NEXT_PUBLIC_SQUARE_APPLICATION_ID!,
        process.env.SQUARE_LOCATION_ID!
      )

      try {
        const cardElement = await payments.card()
        await cardElement.attach('#card-container')
        setCard(cardElement)
      } catch (e) {
        console.error('Failed to initialize Square', e)
      }
    }

    initializeSquare()
  }, [])

  async function handlePayment() {
    if (!card) return

    setLoading(true)

    try {
      const result = await card.tokenize()

      if (result.status === 'OK') {
        // Send to your server
        const response = await fetch('/api/payments/square/process', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            sourceId: result.token,
            amount
          })
        })

        const data = await response.json()

        if (data.success) {
          onSuccess(data.paymentId)
        } else {
          alert('Payment failed: ' + data.error)
        }
      } else {
        alert('Tokenization failed')
      }
    } catch (error) {
      console.error('Payment error:', error)
    } finally {
      setLoading(false)
    }
  }

  return (
    <div className="space-y-4">
      <div id="card-container"></div>
      <Button
        onClick={handlePayment}
        disabled={!card || loading}
        className="w-full"
      >
        {loading ? 'Processing...' : `Pay $${(amount / 100).toFixed(2)}`}
      </Button>
    </div>
  )
}
```

### Load Square.js Script

```typescript
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <head>
        <script
          type="text/javascript"
          src="https://sandbox.web.squarecdn.com/v1/square.js"
        />
      </head>
      <body>{children}</body>
    </html>
  )
}
```

### Backend Payment Processing

```typescript
// app/api/payments/square/process/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { squareClient, locationId } from '@/lib/square/client'
import { randomUUID } from 'crypto'

export async function POST(request: NextRequest) {
  try {
    const { sourceId, amount, customerId } = await request.json()

    const { result } = await squareClient.paymentsApi.createPayment({
      sourceId,
      idempotencyKey: randomUUID(),
      amountMoney: {
        amount: BigInt(amount),
        currency: 'USD'
      },
      customerId,
      locationId,
      autocomplete: true
    })

    // Save payment to database
    await db.insert(payments).values({
      id: generateId(),
      customerId,
      squarePaymentId: result.payment!.id,
      amount,
      status: result.payment!.status!,
      createdAt: new Date()
    })

    return NextResponse.json({
      success: true,
      paymentId: result.payment!.id,
      status: result.payment!.status
    })
  } catch (error: any) {
    console.error('Payment error:', error)

    return NextResponse.json(
      {
        success: false,
        error: error.message || 'Payment failed'
      },
      { status: 400 }
    )
  }
}
```

## Customer Management

### Create Customer

```typescript
// lib/square/customers.ts
import { squareClient } from './client'

export async function createSquareCustomer(data: {
  email: string
  givenName?: string
  familyName?: string
  phoneNumber?: string
  referenceId?: string
}) {
  const { result } = await squareClient.customersApi.createCustomer({
    emailAddress: data.email,
    givenName: data.givenName,
    familyName: data.familyName,
    phoneNumber: data.phoneNumber,
    referenceId: data.referenceId // Your internal user ID
  })

  return result.customer
}

export async function getSquareCustomer(customerId: string) {
  const { result } = await squareClient.customersApi.retrieveCustomer(customerId)
  return result.customer
}

export async function updateSquareCustomer(
  customerId: string,
  data: Partial<{
    emailAddress: string
    givenName: string
    familyName: string
    phoneNumber: string
  }>
) {
  const { result } = await squareClient.customersApi.updateCustomer(customerId, data)
  return result.customer
}
```

## Save Card on File

### Store Payment Method

```typescript
// app/api/payments/square/save-card/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { squareClient } from '@/lib/square/client'

export async function POST(request: NextRequest) {
  try {
    const { sourceId, customerId } = await request.json()

    const { result } = await squareClient.cardsApi.createCard({
      idempotencyKey: randomUUID(),
      sourceId,
      card: {
        customerId
      }
    })

    // Save card to database
    await db.insert(paymentMethods).values({
      id: generateId(),
      customerId,
      squareCardId: result.card!.id,
      last4: result.card!.last4!,
      brand: result.card!.cardBrand!,
      expMonth: result.card!.expMonth!,
      expYear: result.card!.expYear!,
      isDefault: true
    })

    return NextResponse.json({
      success: true,
      cardId: result.card!.id
    })
  } catch (error: any) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 400 }
    )
  }
}
```

## Subscriptions

### Create Subscription Plan

```typescript
// lib/square/subscriptions.ts
import { squareClient, locationId } from './client'

export async function createSubscriptionPlan(data: {
  name: string
  phases: Array<{
    cadence: 'DAILY' | 'WEEKLY' | 'MONTHLY' | 'ANNUAL'
    periods?: number
    recurringPriceMoney: { amount: bigint; currency: string }
  }>
}) {
  const { result } = await squareClient.catalogApi.upsertCatalogObject({
    idempotencyKey: randomUUID(),
    object: {
      type: 'SUBSCRIPTION_PLAN',
      id: `#${data.name.toLowerCase().replace(/\s+/g, '-')}`,
      subscriptionPlanData: {
        name: data.name,
        phases: data.phases.map(phase => ({
          cadence: phase.cadence,
          periods: phase.periods,
          recurringPriceMoney: phase.recurringPriceMoney
        }))
      }
    }
  })

  return result.catalogObject
}
```

### Create Subscription

```typescript
// app/api/subscriptions/square/create/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { squareClient, locationId } from '@/lib/square/client'

export async function POST(request: NextRequest) {
  try {
    const { customerId, planId, cardId, startDate } = await request.json()

    const { result } = await squareClient.subscriptionsApi.createSubscription({
      locationId,
      planId,
      customerId,
      cardId,
      startDate, // YYYY-MM-DD format
      timezone: 'America/Los_Angeles'
    })

    // Save to database
    await db.insert(subscriptions).values({
      id: generateId(),
      customerId,
      squareSubscriptionId: result.subscription!.id,
      planId,
      status: result.subscription!.status!,
      startDate: new Date(result.subscription!.startDate!)
    })

    return NextResponse.json({
      success: true,
      subscription: result.subscription
    })
  } catch (error: any) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 400 }
    )
  }
}
```

### Cancel Subscription

```typescript
// app/api/subscriptions/square/cancel/route.ts
export async function POST(request: NextRequest) {
  const { subscriptionId } = await request.json()

  const { result } = await squareClient.subscriptionsApi.cancelSubscription(subscriptionId)

  // Update database
  await db
    .update(subscriptions)
    .set({
      status: 'CANCELED',
      canceledAt: new Date()
    })
    .where(eq(subscriptions.squareSubscriptionId, subscriptionId))

  return NextResponse.json({ success: true })
}
```

## Webhooks

### Setup Webhook Handler

```typescript
// app/api/webhooks/square/route.ts
import { NextRequest, NextResponse } from 'next/server'
import crypto from 'crypto'

const webhookSignatureKey = process.env.SQUARE_WEBHOOK_SIGNATURE_KEY!

function isValidSignature(body: string, signature: string): boolean {
  const hmac = crypto.createHmac('sha256', webhookSignatureKey)
  hmac.update(body)
  const hash = hmac.digest('base64')
  return hash === signature
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.text()
    const signature = request.headers.get('x-square-hmacsha256-signature')!

    if (!isValidSignature(body, signature)) {
      return NextResponse.json({ error: 'Invalid signature' }, { status: 403 })
    }

    const event = JSON.parse(body)

    switch (event.type) {
      case 'payment.created':
        await handlePaymentCreated(event.data.object.payment)
        break

      case 'payment.updated':
        await handlePaymentUpdated(event.data.object.payment)
        break

      case 'subscription.created':
      case 'subscription.updated':
        await handleSubscriptionUpdate(event.data.object.subscription)
        break

      case 'invoice.paid':
        await handleInvoicePaid(event.data.object.invoice)
        break

      case 'invoice.payment_failed':
        await handlePaymentFailed(event.data.object.invoice)
        break
    }

    return NextResponse.json({ success: true })
  } catch (error) {
    console.error('Webhook error:', error)
    return NextResponse.json({ error: 'Webhook processing failed' }, { status: 500 })
  }
}

async function handlePaymentCreated(payment: any) {
  await db.insert(payments).values({
    id: generateId(),
    squarePaymentId: payment.id,
    customerId: payment.customer_id,
    amount: parseInt(payment.amount_money.amount),
    status: payment.status,
    createdAt: new Date(payment.created_at)
  })
}

async function handlePaymentUpdated(payment: any) {
  await db
    .update(payments)
    .set({ status: payment.status })
    .where(eq(payments.squarePaymentId, payment.id))
}
```

## Refunds

### Process Refund

```typescript
// app/api/payments/square/refund/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { squareClient } from '@/lib/square/client'
import { randomUUID } from 'crypto'

export async function POST(request: NextRequest) {
  const { paymentId, amount, reason } = await request.json()

  const { result } = await squareClient.refundsApi.refundPayment({
    idempotencyKey: randomUUID(),
    paymentId,
    amountMoney: {
      amount: BigInt(amount),
      currency: 'USD'
    },
    reason
  })

  // Save refund to database
  await db.insert(refunds).values({
    id: generateId(),
    paymentId,
    squareRefundId: result.refund!.id,
    amount,
    reason,
    status: result.refund!.status!,
    createdAt: new Date()
  })

  return NextResponse.json({
    success: true,
    refundId: result.refund!.id
  })
}
```

## Error Handling

### Common Square Errors

```typescript
// lib/square/errors.ts
export function handleSquareError(error: any) {
  const errors = error.errors || []

  for (const err of errors) {
    switch (err.code) {
      case 'CARD_DECLINED':
        return 'Your card was declined. Please try another payment method.'

      case 'INSUFFICIENT_FUNDS':
        return 'Insufficient funds. Please try another card.'

      case 'CARD_EXPIRED':
        return 'Your card has expired. Please use a different card.'

      case 'CVV_FAILURE':
        return 'Invalid CVV. Please check your security code.'

      case 'ADDRESS_VERIFICATION_FAILURE':
        return 'Address verification failed. Please check your billing address.'

      case 'INVALID_CARD':
        return 'Invalid card information. Please check and try again.'

      default:
        return err.detail || 'Payment processing failed. Please try again.'
    }
  }

  return 'An unexpected error occurred. Please try again.'
}

// Usage in API routes
try {
  const { result } = await squareClient.paymentsApi.createPayment(...)
} catch (error: any) {
  const message = handleSquareError(error)
  return NextResponse.json({ error: message }, { status: 400 })
}
```

## Testing

### Use Sandbox Environment

```typescript
// Test card numbers (Sandbox only)
const testCards = {
  success: '4111 1111 1111 1111',
  declined: '4000 0000 0000 0002',
  cvvFailure: '4000 0000 0000 0101',
  expired: '4000 0000 0000 0069'
}

// CVV: Any 3 digits
// Postal Code: Any 5 digits
// Expiration: Any future date
```

## Database Schema

```typescript
// lib/db/schema/payments.ts
export const payments = pgTable('payments', {
  id: text('id').primaryKey(),
  customerId: text('customer_id').references(() => customers.id),
  squarePaymentId: text('square_payment_id').unique(),
  amount: integer('amount').notNull(),
  currency: text('currency').notNull().default('USD'),
  status: text('status').notNull(), // COMPLETED, PENDING, FAILED, CANCELED
  receiptUrl: text('receipt_url'),
  createdAt: timestamp('created_at').notNull().defaultNow()
})

export const paymentMethods = pgTable('payment_methods', {
  id: text('id').primaryKey(),
  customerId: text('customer_id').references(() => customers.id),
  squareCardId: text('square_card_id').unique(),
  last4: text('last4').notNull(),
  brand: text('brand').notNull(), // VISA, MASTERCARD, etc.
  expMonth: integer('exp_month').notNull(),
  expYear: integer('exp_year').notNull(),
  isDefault: boolean('is_default').notNull().default(false),
  createdAt: timestamp('created_at').notNull().defaultNow()
})

export const refunds = pgTable('refunds', {
  id: text('id').primaryKey(),
  paymentId: text('payment_id').references(() => payments.id),
  squareRefundId: text('square_refund_id').unique(),
  amount: integer('amount').notNull(),
  reason: text('reason'),
  status: text('status').notNull(),
  createdAt: timestamp('created_at').notNull().defaultNow()
})
```

## Best Practices

1. **Always use idempotency keys** - Prevent duplicate charges
2. **Verify webhook signatures** - Ensure requests are from Square
3. **Handle all error cases** - Provide clear messages to users
4. **Store minimal card data** - Never store full card numbers
5. **Use customer IDs** - Link payments to your user records
6. **Test in sandbox** - Thoroughly test before production
7. **Monitor webhooks** - Set up alerts for failures
8. **Implement retries** - Handle transient failures gracefully

## When to Use This Skill

- Adding Square payments
- Implementing subscriptions
- Processing refunds
- Customer management
- Webhook setup
- POS integration
- Testing payment flows
