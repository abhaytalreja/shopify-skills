# Webhooks and Compliance Reference

## Webhook Configuration Methods

### 1. Declarative (shopify.app.toml) - Recommended

```toml
[webhooks]
api_version = "2026-01"

[[webhooks.subscriptions]]
topics = ["app/uninstalled", "products/update", "orders/create", "orders/updated"]
uri = "/webhooks"

[[webhooks.subscriptions]]
topics = ["customers/create", "customers/update"]
uri = "/webhooks/customers"

[[webhooks.subscriptions]]
compliance_topics = ["customers/data_request", "customers/redact", "shop/redact"]
uri = "/webhooks"

# Filter specific webhooks (optional)
[[webhooks.subscriptions]]
topics = ["orders/create"]
uri = "/webhooks/high-value-orders"
filter = "total_price:>=1000"

# Sub-topic filtering
[[webhooks.subscriptions]]
topics = ["products/update"]
uri = "/webhooks/product-changes"
sub_topic = "type:price_change"
```

Declarative webhooks are automatically registered on `shopify app deploy` and during the `afterAuth` hook.

### 2. Programmatic (GraphQL)

```graphql
mutation {
  webhookSubscriptionCreate(
    topic: ORDERS_CREATE
    webhookSubscription: {
      callbackUrl: "https://myapp.com/webhooks"
      format: JSON
    }
  ) {
    webhookSubscription { id topic endpoint { ... on WebhookHttpEndpoint { callbackUrl } } }
    userErrors { field message }
  }
}
```

Verify existing subscriptions:
```graphql
query {
  webhookSubscriptions(first: 25) {
    nodes { id topic endpoint { ... on WebhookHttpEndpoint { callbackUrl } } }
  }
}
```

## Webhook Handler Pattern

```typescript
// app/routes/webhooks.tsx
import type { ActionFunctionArgs } from "react-router";
import { authenticate } from "../shopify.server";
import db from "../db.server";

export const action = async ({ request }: ActionFunctionArgs) => {
  const { topic, shop, payload, session, admin } = await authenticate.webhook(request);

  switch (topic) {
    case "APP_UNINSTALLED":
      // Clean up all shop data
      await db.session.deleteMany({ where: { shop } });
      await db.shopData.deleteMany({ where: { shop } });
      break;

    case "PRODUCTS_UPDATE":
      await handleProductUpdate(shop, payload);
      break;

    case "ORDERS_CREATE":
      await handleNewOrder(shop, payload);
      break;

    // GDPR Mandatory Webhooks
    case "CUSTOMERS_DATA_REQUEST":
      await handleCustomerDataRequest(shop, payload);
      break;

    case "CUSTOMERS_REDACT":
      await handleCustomerRedact(shop, payload);
      break;

    case "SHOP_REDACT":
      await handleShopRedact(shop, payload);
      break;

    default:
      console.warn(`Unhandled webhook topic: ${topic}`);
  }

  // MUST return 200 within 5 seconds
  return new Response();
};
```

## HMAC Verification

The `authenticate.webhook()` function handles HMAC verification automatically. If implementing manually:

```typescript
import crypto from "crypto";

function verifyWebhook(rawBody: string, hmacHeader: string, secret: string): boolean {
  const hash = crypto
    .createHmac("sha256", secret)
    .update(rawBody, "utf8")
    .digest("base64");
  return crypto.timingSafeEqual(
    Buffer.from(hash),
    Buffer.from(hmacHeader)
  );
}

// Headers to check:
// X-Shopify-Hmac-Sha256: HMAC signature
// X-Shopify-Topic: webhook topic
// X-Shopify-Shop-Domain: shop domain
// X-Shopify-Event-Id: unique event ID (for deduplication)
// X-Shopify-Webhook-Id: subscription ID
// X-Shopify-Api-Version: API version
```

## Webhook Deduplication

Shopify may send the same webhook multiple times. Always deduplicate:

```typescript
const eventId = request.headers.get("X-Shopify-Event-Id");

// Check if already processed
const existing = await db.webhookLog.findUnique({ where: { eventId } });
if (existing) {
  return new Response(); // Already processed, return 200
}

// Process the webhook
await processWebhook(topic, payload);

// Mark as processed
await db.webhookLog.create({
  data: { eventId, topic, shop, processedAt: new Date() },
});
```

## Webhook Delivery Rules

- Shopify expects a 2xx response within 5 seconds
- After 5 seconds, Shopify considers delivery failed
- Failed deliveries are retried up to 19 times over 48 hours
- After 19 consecutive failures for a topic, the subscription is removed
- Webhook payload max size: varies by topic, typically under 10MB

### Retry Schedule

Retries use exponential backoff. Approximate schedule:
- Retry 1: 10 seconds
- Retry 2: 30 seconds
- Retry 3-5: minutes apart
- Retry 6+: hours apart
- Final retry: ~48 hours after first attempt

## GDPR Compliance Webhooks (Mandatory)

These three webhooks are REQUIRED for App Store approval. Failure to implement them results in automatic rejection.

### 1. customers/data_request

Sent when a customer requests their data under GDPR. You must provide all stored data about the customer within 30 days.

Payload:
```json
{
  "shop_id": 954889,
  "shop_domain": "shop.myshopify.com",
  "orders_requested": [299938, 280263, 220458],
  "customer": {
    "id": 191167,
    "email": "customer@example.com",
    "phone": "+15551234567"
  },
  "data_request": {
    "id": 9999
  }
}
```

Handler:
```typescript
case "CUSTOMERS_DATA_REQUEST": {
  const { customer, shop_domain } = payload;
  // Gather all data stored about this customer
  const customerData = await db.customerData.findMany({
    where: { shopDomain: shop_domain, customerId: String(customer.id) },
  });
  // Send data to merchant or store for retrieval
  // You have 30 days to respond
  await queueCustomerDataExport(customer.email, customerData);
  break;
}
```

### 2. customers/redact

Sent when a customer requests data deletion. Delete all stored personal data within 30 days.

Payload:
```json
{
  "shop_id": 954889,
  "shop_domain": "shop.myshopify.com",
  "customer": {
    "id": 191167,
    "email": "customer@example.com",
    "phone": "+15551234567"
  },
  "orders_to_redact": [299938, 280263]
}
```

Handler:
```typescript
case "CUSTOMERS_REDACT": {
  const { customer, shop_domain, orders_to_redact } = payload;
  // Delete ALL personal data for this customer
  await db.customerData.deleteMany({
    where: { shopDomain: shop_domain, customerId: String(customer.id) },
  });
  // Also redact any order-specific data
  if (orders_to_redact?.length) {
    await db.orderData.deleteMany({
      where: { orderId: { in: orders_to_redact.map(String) } },
    });
  }
  break;
}
```

### 3. shop/redact

Sent 48 hours after app uninstall. Delete ALL data for the shop.

Payload:
```json
{
  "shop_id": 954889,
  "shop_domain": "shop.myshopify.com"
}
```

Handler:
```typescript
case "SHOP_REDACT": {
  const { shop_domain } = payload;
  // Delete ALL data associated with this shop
  await db.shopData.deleteMany({ where: { shopDomain: shop_domain } });
  await db.customerData.deleteMany({ where: { shopDomain: shop_domain } });
  await db.orderData.deleteMany({ where: { shopDomain: shop_domain } });
  await db.session.deleteMany({ where: { shop: shop_domain } });
  break;
}
```

## Common Webhook Topics

### App Lifecycle
- `app/uninstalled` - App removed from shop (always subscribe)
- `app_subscriptions/update` - Billing subscription changed

### Products
- `products/create` - New product created
- `products/update` - Product modified
- `products/delete` - Product deleted
- `collections/create`, `collections/update`, `collections/delete`

### Orders
- `orders/create` - New order placed
- `orders/updated` - Order modified
- `orders/paid` - Payment received
- `orders/fulfilled` - Order fulfilled
- `orders/cancelled` - Order cancelled
- `refunds/create` - Refund issued

### Customers
- `customers/create` - New customer registered
- `customers/update` - Customer profile modified
- `customers/delete` - Customer deleted

### Inventory
- `inventory_items/create`, `inventory_items/update`, `inventory_items/delete`
- `inventory_levels/connect`, `inventory_levels/update`, `inventory_levels/disconnect`

### Fulfillment
- `fulfillments/create`, `fulfillments/update`
- `fulfillment_events/create`, `fulfillment_events/delete`

### Cart/Checkout
- `carts/create`, `carts/update`
- `checkouts/create`, `checkouts/update`

### Other
- `shop/update` - Shop settings changed
- `themes/create`, `themes/update`, `themes/delete`, `themes/publish`
- `bulk_operations/finish` - Bulk operation completed

## Webhook Payload Format

All webhook payloads include:
- JSON body with resource data
- Headers for verification and metadata
- Topic in `X-Shopify-Topic` header (slash format: `orders/create`)
- Topic in `authenticate.webhook()` uses enum format: `ORDERS_CREATE`

## Testing Webhooks Locally

```bash
# Trigger a test webhook
shopify app webhook trigger --topic orders/create --address http://localhost:3000/webhooks

# List registered webhooks
shopify app webhook list
```

During `shopify app dev`, the CLI creates a tunnel and auto-registers webhook URLs pointing to your local server.

## Event Bridge and Pub/Sub Delivery

Alternative to HTTPS callbacks:

```toml
# Amazon EventBridge
[[webhooks.subscriptions]]
topics = ["orders/create"]
uri = "arn:aws:events:us-east-1::event-source/aws.partner/shopify.com/123/my-event-source"

# Google Pub/Sub
[[webhooks.subscriptions]]
topics = ["orders/create"]
uri = "pubsub://project-id:topic-id"
```

These methods are more reliable for high-volume apps and do not require 5-second response time.
