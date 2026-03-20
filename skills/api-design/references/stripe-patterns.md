# Stripe / Payment Integration Patterns

Complete patterns for Stripe Checkout, subscriptions, webhooks, and billing portal.

## Setup

```bash
npm install stripe @stripe/stripe-js
```

```typescript
// lib/stripe.ts — Server-side client
import Stripe from "stripe";

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-12-18.acacia",
  typescript: true,
});

// lib/stripe-client.ts — Client-side (browser)
import { loadStripe } from "@stripe/stripe-js";

export const getStripeClient = () =>
  loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);
```

## One-Time Checkout

```typescript
// app/api/checkout/route.ts
import { stripe } from "@/lib/stripe";
import { auth } from "@/lib/auth";

export async function POST(req: Request) {
  const user = await auth();
  if (!user) return Response.json({ error: "Unauthorized" }, { status: 401 });

  const { priceId } = await req.json();

  const session = await stripe.checkout.sessions.create({
    mode: "payment",
    payment_method_types: ["card"],
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing`,
    customer_email: user.email,
    metadata: { userId: user.id },
  });

  return Response.json({ url: session.url });
}

// Client-side redirect
async function handleCheckout(priceId: string) {
  const res = await fetch("/api/checkout", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ priceId }),
  });
  const { url } = await res.json();
  window.location.href = url;
}
```

## Subscription Checkout

```typescript
// app/api/subscribe/route.ts
export async function POST(req: Request) {
  const user = await auth();
  if (!user) return Response.json({ error: "Unauthorized" }, { status: 401 });

  const { priceId } = await req.json();

  // Find or create Stripe customer
  let customerId = user.stripeCustomerId;
  if (!customerId) {
    const customer = await stripe.customers.create({
      email: user.email,
      name: user.name,
      metadata: { userId: user.id },
    });
    customerId = customer.id;
    await db.update(users).set({ stripeCustomerId: customerId }).where(eq(users.id, user.id));
  }

  const session = await stripe.checkout.sessions.create({
    mode: "subscription",
    customer: customerId,
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard?upgraded=true`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing`,
    subscription_data: {
      trial_period_days: 14,
      metadata: { userId: user.id },
    },
    allow_promotion_codes: true,
  });

  return Response.json({ url: session.url });
}
```

## Webhook Handler

```typescript
// app/api/webhooks/stripe/route.ts
import { stripe } from "@/lib/stripe";
import { headers } from "next/headers";

export async function POST(req: Request) {
  const body = await req.text();
  const headersList = await headers();
  const signature = headersList.get("stripe-signature")!;

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    console.error("Webhook signature verification failed:", err);
    return Response.json({ error: "Invalid signature" }, { status: 400 });
  }

  try {
    switch (event.type) {
      case "checkout.session.completed": {
        const session = event.data.object as Stripe.Checkout.Session;
        if (session.mode === "subscription") {
          await handleSubscriptionCreated(session);
        } else {
          await handleOneTimePayment(session);
        }
        break;
      }

      case "customer.subscription.updated": {
        const subscription = event.data.object as Stripe.Subscription;
        await db.update(subscriptions).set({
          status: subscription.status,
          priceId: subscription.items.data[0].price.id,
          currentPeriodEnd: new Date(subscription.current_period_end * 1000),
          cancelAtPeriodEnd: subscription.cancel_at_period_end,
        }).where(eq(subscriptions.stripeSubscriptionId, subscription.id));
        break;
      }

      case "customer.subscription.deleted": {
        const subscription = event.data.object as Stripe.Subscription;
        await db.update(subscriptions).set({
          status: "canceled",
          canceledAt: new Date(),
        }).where(eq(subscriptions.stripeSubscriptionId, subscription.id));
        break;
      }

      case "invoice.payment_failed": {
        const invoice = event.data.object as Stripe.Invoice;
        // Notify user about failed payment
        await sendPaymentFailedEmail(invoice.customer as string);
        break;
      }

      case "invoice.paid": {
        const invoice = event.data.object as Stripe.Invoice;
        await db.insert(invoices).values({
          stripeInvoiceId: invoice.id,
          customerId: invoice.customer as string,
          amount: invoice.amount_paid,
          currency: invoice.currency,
          status: "paid",
          paidAt: new Date(),
        });
        break;
      }
    }

    return Response.json({ received: true });
  } catch (error) {
    console.error("Webhook handler error:", error);
    return Response.json({ error: "Handler failed" }, { status: 500 });
  }
}

async function handleSubscriptionCreated(session: Stripe.Checkout.Session) {
  const userId = session.metadata?.userId;
  if (!userId) return;

  const subscription = await stripe.subscriptions.retrieve(session.subscription as string);

  await db.insert(subscriptions).values({
    userId,
    stripeCustomerId: session.customer as string,
    stripeSubscriptionId: subscription.id,
    priceId: subscription.items.data[0].price.id,
    status: subscription.status,
    currentPeriodEnd: new Date(subscription.current_period_end * 1000),
  });
}
```

## Billing Portal (Manage Subscription)

```typescript
// app/api/billing-portal/route.ts
export async function POST() {
  const user = await auth();
  if (!user?.stripeCustomerId) {
    return Response.json({ error: "No subscription" }, { status: 400 });
  }

  const portalSession = await stripe.billingPortal.sessions.create({
    customer: user.stripeCustomerId,
    return_url: `${process.env.NEXT_PUBLIC_APP_URL}/settings`,
  });

  return Response.json({ url: portalSession.url });
}
```

## Check Subscription Status

```typescript
// lib/subscription.ts
export type Plan = "free" | "pro" | "enterprise";

const PRICE_TO_PLAN: Record<string, Plan> = {
  [process.env.STRIPE_PRO_PRICE_ID!]: "pro",
  [process.env.STRIPE_ENTERPRISE_PRICE_ID!]: "enterprise",
};

export async function getUserPlan(userId: string): Promise<Plan> {
  const sub = await db.query.subscriptions.findFirst({
    where: and(
      eq(subscriptions.userId, userId),
      eq(subscriptions.status, "active")
    ),
  });

  if (!sub) return "free";
  return PRICE_TO_PLAN[sub.priceId] ?? "free";
}

// Middleware or server component check
export async function requirePlan(userId: string, required: Plan) {
  const plan = await getUserPlan(userId);
  const planLevel = { free: 0, pro: 1, enterprise: 2 };
  if (planLevel[plan] < planLevel[required]) {
    throw new Error("Upgrade required");
  }
}
```

## Pricing Page Component

```typescript
// components/pricing.tsx
const plans = [
  {
    name: "Free",
    price: "$0",
    priceId: null,
    features: ["5 projects", "Basic analytics", "Community support"],
  },
  {
    name: "Pro",
    price: "$29/mo",
    priceId: process.env.NEXT_PUBLIC_STRIPE_PRO_PRICE_ID,
    popular: true,
    features: ["Unlimited projects", "Advanced analytics", "Priority support", "API access"],
  },
  {
    name: "Enterprise",
    price: "$99/mo",
    priceId: process.env.NEXT_PUBLIC_STRIPE_ENTERPRISE_PRICE_ID,
    features: ["Everything in Pro", "SSO", "Dedicated support", "Custom integrations", "SLA"],
  },
];
```

## Database Schema for Subscriptions

```typescript
// Drizzle schema
export const subscriptions = pgTable("subscriptions", {
  id: uuid("id").defaultRandom().primaryKey(),
  userId: uuid("user_id").notNull().references(() => users.id, { onDelete: "cascade" }),
  stripeCustomerId: varchar("stripe_customer_id", { length: 255 }).notNull(),
  stripeSubscriptionId: varchar("stripe_subscription_id", { length: 255 }).notNull().unique(),
  priceId: varchar("price_id", { length: 255 }).notNull(),
  status: varchar("status", { length: 50 }).notNull(), // active, canceled, past_due, trialing
  currentPeriodEnd: timestamp("current_period_end").notNull(),
  cancelAtPeriodEnd: boolean("cancel_at_period_end").default(false),
  canceledAt: timestamp("canceled_at"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
});
```

## Testing Stripe Locally

```bash
# Install Stripe CLI
brew install stripe/stripe-cli/stripe

# Login
stripe login

# Forward webhooks to local server
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# Trigger test events
stripe trigger checkout.session.completed
stripe trigger customer.subscription.updated
stripe trigger invoice.payment_failed
```

Test card numbers:
- **Success**: `4242 4242 4242 4242`
- **Requires auth**: `4000 0025 0000 3155`
- **Declined**: `4000 0000 0000 0002`
- Expiry: any future date, CVC: any 3 digits

## Environment Variables

```bash
# .env.local
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
NEXT_PUBLIC_STRIPE_PRO_PRICE_ID=price_...
NEXT_PUBLIC_STRIPE_ENTERPRISE_PRICE_ID=price_...
```

## Checklist

- [ ] Stripe account created and API keys set
- [ ] Products and prices created in Stripe Dashboard
- [ ] Webhook endpoint registered (both local and production)
- [ ] Handle all critical webhook events (checkout.completed, subscription.updated/deleted, invoice.failed/paid)
- [ ] Billing portal configured in Stripe Dashboard
- [ ] Test with Stripe CLI locally before deploying
- [ ] Idempotency: webhooks can be retried, so handle duplicates
- [ ] Error handling: log webhook failures, alert on payment issues
