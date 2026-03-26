---
description: "Use when building, configuring, deploying, or debugging Shopify apps. Covers React Router v7 architecture, GraphQL Admin API, authentication, webhooks, billing, Shopify Flow, compliance, CLI commands, and App Store launch. Trigger when code imports @shopify/*, uses shopify CLI, or user mentions Shopify app development."
---

# Shopify App Development Skill

## Quick Reference

**Framework:** React Router v7 (NOT Remix) via `@shopify/shopify-app-react-router`
**API:** GraphQL Admin API only (REST is legacy since Oct 2024)
**Auth:** Token Exchange + Session Tokens (for embedded apps)
**CLI:** `@shopify/cli` (v3.90+) - install globally: `npm install -g @shopify/cli`
**App Bridge:** CDN script tag (auto-updates, no npm package)
**API Version:** Use latest stable (currently `2026-01`)

## Architecture Overview

Shopify apps use a hybrid model:
- **Your server** hosts the app's pages and backend (React Router v7)
- **Shopify** hosts app extensions (rendered natively in admin/checkout/etc.)
- **Embedded apps** run inside the Shopify admin iframe via App Bridge

## Project Setup

```bash
# Scaffold new app (uses React Router v7 template)
shopify app init

# Start development (creates tunnel, registers webhooks)
shopify app dev

# Deploy config + extensions to production
shopify app deploy

# Generate extension
shopify app generate extension
```

## Project Structure

```
app/
  routes/
    app.tsx              # Layout route (auth wrapper for all app.* routes)
    app._index.tsx       # Main dashboard (authenticated)
    app.settings.tsx     # Settings page (authenticated)
    webhooks.tsx         # Webhook handler (no auth wrapper needed)
  shopify.server.ts      # Core Shopify config (exports authenticate, etc.)
  root.tsx               # Root route
extensions/              # Shopify-hosted extensions
  my-extension/
    shopify.extension.toml
    src/index.tsx
shopify.app.toml         # App config (scopes, webhooks, auth)
shopify.web.toml         # Web process config
vite.config.ts
prisma/schema.prisma     # Session storage (replace SQLite in production)
```

Routes with `app.` prefix inherit authentication, error boundaries, and embedded app headers.

## Authentication Pattern

Use Token Exchange (recommended for embedded apps):

```typescript
// app/shopify.server.ts
import { shopifyApp } from "@shopify/shopify-app-react-router/server";
import { PrismaSessionStorage } from "@shopify/shopify-app-session-storage-prisma";

const shopify = shopifyApp({
  apiKey: process.env.SHOPIFY_API_KEY!,
  apiSecretKey: process.env.SHOPIFY_API_SECRET!,
  appUrl: process.env.SHOPIFY_APP_URL!,
  scopes: process.env.SCOPES?.split(","),
  apiVersion: "2026-01",
  sessionStorage: new PrismaSessionStorage(prisma),
  distribution: "app_store",
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

**In routes** - always authenticate in loaders and actions:

```typescript
export const loader = async ({ request }: LoaderFunctionArgs) => {
  const { admin, session } = await authenticate.admin(request);
  const response = await admin.graphql(`query { shop { name } }`);
  const { data } = await response.json();
  return json({ shop: data.data.shop });
};

export const action = async ({ request }: ActionFunctionArgs) => {
  const { admin, redirect } = await authenticate.admin(request);
  // Use redirect from authenticate (NOT react-router redirect) for embedded apps
  return redirect("/app");
};
```

## GraphQL API Usage

```typescript
// Server-side (in loaders/actions)
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

// Frontend direct API access (requires embedded_app_direct_api_access = true)
const response = await fetch("shopify:admin/api/graphql.json", {
  method: "POST",
  body: JSON.stringify({ query: `query { shop { name } }` }),
});
```

**Global IDs:** Format `gid://shopify/{Type}/{ID}` (e.g., `gid://shopify/Product/123`)

For complete API reference, rate limits, pagination, and bulk operations, read:
- [references/build-reference.md](references/build-reference.md)

## Webhooks

Configure declaratively in `shopify.app.toml` (recommended):

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

Handle in route:

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
      // Handle GDPR - MANDATORY for App Store
      break;
  }
  return new Response();
};
```

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

## shopify.app.toml Configuration

```toml
name = "My App"
client_id = "YOUR_CLIENT_ID"
application_url = "https://myapp.example.com"
embedded = true

[access_scopes]
scopes = "read_products,read_orders,read_customers,read_analytics"

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

## Shopify Flow Integration

For Flow triggers, actions, and templates, read:
- [references/flow-reference.md](references/flow-reference.md)

## App Store Launch Checklist

For complete launch requirements, review process, and common rejection reasons, read:
- [references/launch-reference.md](references/launch-reference.md)

**Critical requirements (will cause rejection if missed):**
- [ ] GraphQL Admin API only (no REST)
- [ ] Latest App Bridge (CDN script tag)
- [ ] Embedded in Shopify Admin
- [ ] Session token auth (no third-party cookies)
- [ ] 3 mandatory GDPR compliance webhooks implemented
- [ ] Billing API for all paid features (no external payment)
- [ ] Privacy policy linked
- [ ] OAuth initiates immediately on install
- [ ] TLS/SSL on all endpoints
- [ ] English demo screencast with subtitles

## Common Mistakes to Avoid

1. **Using REST API** - Legacy since Oct 2024. Always GraphQL.
2. **Using react-router `redirect`** - Use `authenticate.admin`'s redirect for embedded apps.
3. **Using `<a>` tags** - Breaks embedded nav. Use React Router `Link` or Polaris `Link`.
4. **Not deduplicating webhooks** - Check `X-Shopify-Event-Id`.
5. **Online tokens for bulk operations** - Use offline tokens (bulk ops can exceed 24hr).
6. **Ignoring 64KB extension size limit** - UI extensions are strictly limited.
7. **Changing `handle` in shopify.app.toml** - Breaks embedded app admin links.
8. **Not implementing compliance webhooks** - Instant App Store rejection.
9. **Not calling `addDocumentResponseHeaders()`** - CSP headers required for embedded.
10. **Using pagination args in bulk operations** - `first`/`last`/`after`/`before` are ignored.

## Rate Limits Quick Reference

| Plan | Restore Rate | Bucket |
|------|-------------|--------|
| Standard | 100 pts/sec | 1,000 |
| Advanced | 200 pts/sec | - |
| Plus | 1,000 pts/sec | - |

Single query max: 1,000 points. Use `Shopify-GraphQL-Cost-Debug: 1` header.
For large datasets, use Bulk Operations (bypasses rate limits).

## CLI Commands

```bash
shopify app init              # New app
shopify app dev               # Dev server + tunnel
shopify app deploy            # Deploy to production
shopify app generate extension # New extension
shopify app config link       # Link existing app
shopify app webhook trigger   # Test webhook
shopify app info              # App info
```
