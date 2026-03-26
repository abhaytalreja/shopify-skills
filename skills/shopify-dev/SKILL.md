---
name: shopify-dev
description: Comprehensive Shopify app development guide covering React Router v7, GraphQL Admin API, authentication, webhooks, billing, compliance, Flow integration, and app store launch. Use when building, debugging, or deploying Shopify apps.
---

# Shopify App Development Skill

## Quick Reference

- Framework: React Router v7 (NOT Remix) via `@shopify/shopify-app-react-router`
- API: GraphQL Admin API only (REST is legacy since Oct 2024)
- Auth: Token Exchange + Session Tokens (embedded apps)
- CLI: `@shopify/cli` (v3.90+) - `npm install -g @shopify/cli`
- App Bridge: CDN script tag (auto-updates, no npm package needed)
- API Version: Use latest stable (currently `2026-01`)

## Architecture

Shopify apps use a hybrid model:
- Your server hosts app pages and backend logic (React Router v7)
- Shopify hosts app extensions (rendered natively in admin/checkout/POS)
- Embedded apps run inside the Shopify admin iframe via App Bridge

## Project Setup

```bash
shopify app init                  # Scaffold new app (React Router v7 template)
shopify app dev                   # Dev server + tunnel + webhook registration
shopify app deploy                # Deploy config + extensions to production
shopify app generate extension    # Generate a new extension
shopify app config link           # Link to existing app
shopify app webhook trigger       # Test a webhook locally
shopify app info                  # Show app details
```

## Project Structure

```
app/
  routes/
    app.tsx              # Layout route - auth wrapper for all app.* routes
    app._index.tsx       # Main dashboard (authenticated)
    app.settings.tsx     # Settings page (authenticated)
    webhooks.tsx         # Webhook handler (no auth wrapper)
  shopify.server.ts      # Core config - exports authenticate, sessionStorage, etc.
  root.tsx               # Root route
extensions/              # Shopify-hosted extensions
  my-extension/
    shopify.extension.toml
    src/index.tsx
shopify.app.toml         # App config (scopes, webhooks, auth, billing)
shopify.web.toml         # Web process config
vite.config.ts           # Vite configuration
prisma/schema.prisma     # Session storage schema (replace SQLite in production)
```

Routes prefixed with `app.` inherit authentication, error boundaries, and embedded app headers from `app.tsx`.

## Authentication

Use Token Exchange (recommended for all embedded apps):

```typescript
// app/shopify.server.ts
import "@shopify/shopify-app-react-router/adapters/node";
import { ApiVersion, AppDistribution, shopifyApp } from "@shopify/shopify-app-react-router/server";
import { PrismaSessionStorage } from "@shopify/shopify-app-session-storage-prisma";

const shopify = shopifyApp({
  apiKey: process.env.SHOPIFY_API_KEY!,
  apiSecretKey: process.env.SHOPIFY_API_SECRET!,
  appUrl: process.env.SHOPIFY_APP_URL!,
  scopes: process.env.SCOPES?.split(","),
  apiVersion: ApiVersion.January26,
  sessionStorage: new PrismaSessionStorage(prisma),
  distribution: AppDistribution.AppStore,
  useOnlineTokens: true,
  hooks: {
    afterAuth: async ({ session }) => {
      await shopify.registerWebhooks({ session });
    },
  },
});

export default shopify;
export const authenticate = shopify.authenticate;
```

In routes - always call authenticate in loaders and actions:

```typescript
import type { LoaderFunctionArgs, ActionFunctionArgs } from "react-router";
import { useLoaderData, useFetcher } from "react-router";
import { authenticate } from "../shopify.server";

export const loader = async ({ request }: LoaderFunctionArgs) => {
  const { admin, session } = await authenticate.admin(request);
  const response = await admin.graphql(`query { shop { name } }`);
  const data = await response.json();
  return { shopName: data.data.shop.name };
};

export const action = async ({ request }: ActionFunctionArgs) => {
  const { admin, redirect } = await authenticate.admin(request);
  // IMPORTANT: Use redirect from authenticate, NOT react-router redirect
  return redirect("/app");
};
```

## GraphQL API Usage

Server-side (in loaders/actions):

```typescript
const { admin } = await authenticate.admin(request);
const response = await admin.graphql(
  `query GetProducts($first: Int!) {
    products(first: $first) {
      nodes { id title handle }
      pageInfo { hasNextPage endCursor }
    }
  }`,
  { variables: { first: 10 } }
);
const { data } = await response.json();
```

Frontend direct API access (requires `embedded_app_direct_api_access = true` in TOML):

```typescript
const response = await fetch("shopify:admin/api/graphql.json", {
  method: "POST",
  body: JSON.stringify({ query: `query { shop { name } }` }),
});
```

Global IDs use format `gid://shopify/{Type}/{ID}` (e.g., `gid://shopify/Product/123`).

For rate limits, pagination, bulk operations, cost calculation, and key mutations:
- [references/graphql-patterns.md](references/graphql-patterns.md)

## Webhooks

Configure declaratively in `shopify.app.toml`:

```toml
[webhooks]
api_version = "2026-01"

[[webhooks.subscriptions]]
topics = ["app/uninstalled", "products/update", "orders/create"]
uri = "/webhooks"

[[webhooks.subscriptions]]
compliance_topics = ["customers/data_request", "customers/redact", "shop/redact"]
uri = "/webhooks"
```

Handle in a route:

```typescript
// app/routes/webhooks.tsx
export const action = async ({ request }: ActionFunctionArgs) => {
  const { topic, shop, payload } = await authenticate.webhook(request);
  switch (topic) {
    case "APP_UNINSTALLED":
      await prisma.session.deleteMany({ where: { shop } });
      break;
    case "CUSTOMERS_DATA_REQUEST":
    case "CUSTOMERS_REDACT":
    case "SHOP_REDACT":
      // Handle GDPR compliance - MANDATORY for App Store
      break;
  }
  return new Response();
};
```

For webhook verification, retry handling, GDPR details:
- [references/webhooks-compliance.md](references/webhooks-compliance.md)

## Billing

Configure in `shopify.server.ts`:

```typescript
const shopify = shopifyApp({
  billing: {
    monthly: {
      amount: 29.0,
      currencyCode: "USD",
      interval: "EVERY_30_DAYS",
      trialDays: 7,
    },
    annual: {
      amount: 290.0,
      currencyCode: "USD",
      interval: "ANNUAL",
    },
  },
});
```

Check and require payment in routes:

```typescript
const { billing } = await authenticate.admin(request);
const { hasActivePayment } = await billing.check({ plans: ["monthly"] });
if (!hasActivePayment) {
  await billing.require({
    plans: ["monthly"],
    onFailure: async () => billing.request({ plan: "monthly" }),
  });
}
```

For usage billing, managed pricing, GraphQL billing mutations:
- [references/billing-reference.md](references/billing-reference.md)

## shopify.app.toml Configuration

```toml
name = "My App"
client_id = "YOUR_CLIENT_ID"
application_url = "https://myapp.example.com"
embedded = true

[access_scopes]
scopes = "read_products,read_orders,read_customers"

[access.admin]
direct_api_mode = "online"
embedded_app_direct_api_access = true

[auth]
redirect_urls = ["https://myapp.example.com/auth/callback"]

[webhooks]
api_version = "2026-01"

[[webhooks.subscriptions]]
topics = ["app/uninstalled"]
uri = "/webhooks"

[[webhooks.subscriptions]]
compliance_topics = ["customers/data_request", "customers/redact", "shop/redact"]
uri = "/webhooks"
```

For the complete TOML field reference:
- [references/shopify-app-toml.md](references/shopify-app-toml.md)

## Shopify Flow Integration

For Flow triggers, actions, templates, and TOML configuration:
- [references/flow-reference.md](references/flow-reference.md)

## App Store Launch

For complete submission checklist, Built for Shopify requirements, and common rejection reasons:
- [references/launch-checklist.md](references/launch-checklist.md)

Critical requirements (rejection if missed):
- GraphQL Admin API only (no REST)
- Latest App Bridge (CDN script tag, no npm)
- Embedded in Shopify Admin
- Session token auth (no third-party cookies)
- 3 mandatory GDPR compliance webhooks implemented
- Billing API for all paid features (no external payment)
- Privacy policy linked
- OAuth initiates immediately on install
- TLS/SSL on all endpoints

## Common Mistakes

1. Using REST API - Legacy since Oct 2024. Use GraphQL exclusively.
2. Using react-router `redirect` - Use `authenticate.admin`'s redirect for embedded apps.
3. Using `<a>` tags for navigation - Breaks embedded nav. Use React Router `Link` or Polaris `Link`.
4. Not deduplicating webhooks - Always check `X-Shopify-Event-Id` header.
5. Online tokens for bulk operations - Use offline tokens (bulk ops can exceed 24hr).
6. Ignoring 64KB extension size limit - UI extensions are strictly limited.
7. Changing `handle` in shopify.app.toml after deployment - Breaks embedded admin links.
8. Not implementing compliance webhooks - Instant App Store rejection.
9. Not calling `addDocumentResponseHeaders()` - CSP headers required for embedded apps.
10. Using pagination args in bulk operations - `first`/`last`/`after`/`before` are ignored.
11. Hardcoding shop domain - Always get it from session or authenticate context.
12. Not handling AppSubscriptionCreate user cancellation - Check for null confirmation URL.
13. Storing sensitive data in metafields - Metafields are publicly readable via Storefront API.
14. Not testing with different Shopify plans - Feature availability varies by merchant plan.
15. Skipping idempotency on mutations - Use `idempotencyKey` on financial mutations.

## Rate Limits Quick Reference

| Plan     | Restore Rate | Bucket |
|----------|-------------|--------|
| Standard | 100 pts/sec | 1,000  |
| Advanced | 200 pts/sec | -      |
| Plus     | 1,000 pts/sec | -    |

Single query max: 1,000 points. Use `Shopify-GraphQL-Cost-Debug: 1` header to inspect costs.
For large datasets, use Bulk Operations (bypasses rate limits entirely).
