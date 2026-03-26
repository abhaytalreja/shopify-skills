# Billing API Reference

## Overview

Shopify requires all paid app features to use the Billing API. External payment processing (Stripe, PayPal, etc.) is prohibited for features accessed within Shopify admin. Shopify takes a revenue share (varies by plan, typically 15-20%).

## Billing Models

### 1. Recurring Subscriptions

Monthly or annual charges:

```typescript
// shopify.server.ts
const shopify = shopifyApp({
  billing: {
    basic_monthly: {
      amount: 9.99,
      currencyCode: "USD",
      interval: "EVERY_30_DAYS",
      trialDays: 14,
    },
    pro_monthly: {
      amount: 29.99,
      currencyCode: "USD",
      interval: "EVERY_30_DAYS",
      trialDays: 14,
    },
    pro_annual: {
      amount: 299.99,
      currencyCode: "USD",
      interval: "ANNUAL",
      trialDays: 14,
    },
  },
});
```

### 2. Usage-Based Billing

Charge based on actual usage (e.g., per API call, per order processed):

```typescript
billing: {
  usage_plan: {
    amount: 0, // Base amount (can be 0)
    currencyCode: "USD",
    interval: "EVERY_30_DAYS",
    usageTerms: "Up to $50/month based on usage",
    trialDays: 7,
  },
}
```

Record usage charges:

```typescript
const { billing } = await authenticate.admin(request);
await billing.createUsageCharge("usage_plan", {
  amount: 0.05,
  description: "Processed 1 order",
});
```

### 3. One-Time Charges

For single purchases:

```graphql
mutation {
  appPurchaseOneTimeCreate(
    name: "Premium Theme Pack"
    price: { amount: "49.99", currencyCode: USD }
    returnUrl: "https://myapp.com/billing/callback"
  ) {
    appPurchaseOneTime { id status }
    confirmationUrl
    userErrors { field message }
  }
}
```

## Billing Flow

### Checking Active Subscriptions

```typescript
export const loader = async ({ request }: LoaderFunctionArgs) => {
  const { billing } = await authenticate.admin(request);

  // Check if merchant has an active subscription
  const { hasActivePayment, appSubscriptions } = await billing.check({
    plans: ["basic_monthly", "pro_monthly", "pro_annual"],
  });

  if (!hasActivePayment) {
    // Redirect to billing page or show upgrade prompt
    return { needsBilling: true, plans: getPlanDetails() };
  }

  // Determine which plan is active
  const activePlan = appSubscriptions[0]?.name;
  return { needsBilling: false, activePlan };
};
```

### Requiring Payment

```typescript
const { billing } = await authenticate.admin(request);
const { hasActivePayment } = await billing.check({ plans: ["pro_monthly"] });

if (!hasActivePayment) {
  await billing.require({
    plans: ["pro_monthly"],
    isTest: process.env.NODE_ENV !== "production",
    onFailure: async () => {
      // Merchant declined the charge
      return billing.request({
        plan: "pro_monthly",
        isTest: process.env.NODE_ENV !== "production",
      });
    },
  });
}
```

### Requesting a Subscription

```typescript
const { billing } = await authenticate.admin(request);

// This redirects the merchant to Shopify's approval page
return billing.request({
  plan: "pro_monthly",
  isTest: process.env.NODE_ENV !== "production",
  returnUrl: "https://myapp.com/app/billing/callback",
});
```

### Handling Billing Callback

After the merchant approves or declines:

```typescript
// app/routes/app.billing.callback.tsx
export const loader = async ({ request }: LoaderFunctionArgs) => {
  const { billing, redirect } = await authenticate.admin(request);
  const url = new URL(request.url);
  const chargeId = url.searchParams.get("charge_id");

  if (chargeId) {
    // Verify the charge was accepted
    const { hasActivePayment } = await billing.check({
      plans: ["pro_monthly"],
    });

    if (hasActivePayment) {
      return redirect("/app?billing=success");
    }
  }

  return redirect("/app?billing=cancelled");
};
```

## GraphQL Billing Mutations

### Create Subscription

```graphql
mutation AppSubscriptionCreate(
  $name: String!
  $lineItems: [AppSubscriptionLineItemInput!]!
  $returnUrl: URL!
  $test: Boolean
) {
  appSubscriptionCreate(
    name: $name
    lineItems: $lineItems
    returnUrl: $returnUrl
    test: $test
  ) {
    appSubscription {
      id
      status
      name
      createdAt
      currentPeriodEnd
      trialDays
      test
      lineItems {
        id
        plan {
          pricingDetails {
            ... on AppRecurringPricing {
              price { amount currencyCode }
              interval
            }
            ... on AppUsagePricing {
              balanceUsed { amount currencyCode }
              cappedAmount { amount currencyCode }
              terms
            }
          }
        }
      }
    }
    confirmationUrl
    userErrors { field message }
  }
}
```

Variables:
```json
{
  "name": "Pro Plan",
  "returnUrl": "https://myapp.com/billing/callback",
  "test": true,
  "lineItems": [{
    "plan": {
      "appRecurringPricingDetails": {
        "price": { "amount": 29.99, "currencyCode": "USD" },
        "interval": "EVERY_30_DAYS"
      }
    }
  }]
}
```

### Cancel Subscription

```graphql
mutation {
  appSubscriptionCancel(id: "gid://shopify/AppSubscription/123") {
    appSubscription { id status }
    userErrors { field message }
  }
}
```

### Create Usage Charge

```graphql
mutation {
  appUsageRecordCreate(
    subscriptionLineItemId: "gid://shopify/AppSubscriptionLineItem/456"
    description: "Processed 50 orders"
    price: { amount: 2.50, currencyCode: USD }
    idempotencyKey: "order-batch-789"
  ) {
    appUsageRecord { id }
    userErrors { field message }
  }
}
```

Always use `idempotencyKey` for usage charges to prevent duplicate billing.

### Query Active Subscriptions

```graphql
query {
  appInstallation {
    activeSubscriptions {
      id name status test createdAt
      currentPeriodEnd trialDays
      lineItems {
        id
        plan {
          pricingDetails {
            ... on AppRecurringPricing { price { amount currencyCode } interval }
            ... on AppUsagePricing { balanceUsed { amount } cappedAmount { amount } terms }
          }
        }
      }
    }
    oneTimePurchases(first: 25) {
      nodes { id name status price { amount currencyCode } createdAt }
    }
  }
}
```

## Managed Pricing

Managed pricing lets Shopify handle plan selection UI. Configure in `shopify.app.toml`:

```toml
[app_settings.pricing]
embedded = true
```

With managed pricing, merchants select plans from your App Store listing page. Check active subscription status in your app but do not handle plan selection UI.

## Testing Billing

- Set `isTest: true` or `test: true` on all charges during development
- Test charges are not real and auto-approve
- Test subscriptions show "(test)" in the Shopify admin
- Always use test mode in development; only use real charges in production

## Billing Webhooks

Subscribe to billing-related webhooks:

```toml
[[webhooks.subscriptions]]
topics = ["app_subscriptions/update"]
uri = "/webhooks"
```

The `app_subscriptions/update` webhook fires when:
- Subscription activated
- Subscription frozen (payment failed)
- Subscription cancelled
- Trial ended

```typescript
case "APP_SUBSCRIPTIONS_UPDATE": {
  const { app_subscription } = payload;
  await db.subscription.upsert({
    where: { shopifySubscriptionId: app_subscription.admin_graphql_api_id },
    update: { status: app_subscription.status },
    create: {
      shopifySubscriptionId: app_subscription.admin_graphql_api_id,
      shop,
      status: app_subscription.status,
      plan: app_subscription.name,
    },
  });
  break;
}
```

## Common Billing Mistakes

1. Not handling `confirmationUrl` being null - merchant may cancel before confirming
2. Using real charges in development - always use test mode
3. Not providing `idempotencyKey` for usage charges - causes duplicate billing
4. Billing for features outside Shopify admin - external features can use external payment
5. Not handling frozen subscriptions - payment failures freeze subscriptions
6. Hardcoding prices - use the billing config object for single source of truth
7. Not subscribing to `app_subscriptions/update` - miss payment failures and cancellations
