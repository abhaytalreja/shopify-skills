# Shopify Build Reference - Complete API & Patterns

## GraphQL Admin API

### Endpoint
`POST https://{shop}.myshopify.com/admin/api/{version}/graphql.json`
Header: `X-Shopify-Access-Token: {token}`

### Key Queries

```graphql
# Products with pagination
query GetProducts($first: Int!, $after: String) {
  products(first: $first, after: $after) {
    nodes { id title handle status vendor productType }
    pageInfo { hasNextPage endCursor }
  }
}

# Orders with filtering
query GetOrders($first: Int!, $query: String) {
  orders(first: $first, query: $query, sortKey: CREATED_AT, reverse: true) {
    nodes {
      id name createdAt totalPriceSet { shopMoney { amount currencyCode } }
      customer { id email firstName lastName }
      lineItems(first: 50) { nodes { title quantity } }
    }
    pageInfo { hasNextPage endCursor }
  }
}

# Shop info
query { shop { name email myshopifyDomain plan { displayName } } }

# ShopifyQL analytics
query { shopifyqlQuery(query: "FROM sales SINCE -30d UNTIL today SHOW sum(gross_sales) GROUP BY day ORDER BY day") {
  __typename
  ... on TableResponse { tableData { columns { name dataType } rowData } }
}}
```

### Key Mutations

```graphql
# Set metafields
mutation MetafieldsSet($metafields: [MetafieldsSetInput!]!) {
  metafieldsSet(metafields: $metafields) {
    metafields { id namespace key value }
    userErrors { field message }
  }
}

# Create subscription
mutation AppSubscriptionCreate($name: String!, $returnUrl: URL!, $lineItems: [AppSubscriptionLineItemInput!]!, $test: Boolean, $trialDays: Int) {
  appSubscriptionCreate(name: $name, returnUrl: $returnUrl, lineItems: $lineItems, test: $test, trialDays: $trialDays) {
    appSubscription { id }
    confirmationUrl
    userErrors { field message }
  }
}

# Webhook subscription
mutation WebhookSubscriptionCreate($topic: WebhookSubscriptionTopic!, $webhookSubscription: WebhookSubscriptionInput!) {
  webhookSubscriptionCreate(topic: $topic, webhookSubscription: $webhookSubscription) {
    webhookSubscription { id }
    userErrors { field message }
  }
}
```

### Error Handling

GraphQL returns HTTP 200 even for errors. Always check:
```typescript
const { data, errors } = await response.json();
if (errors) {
  // Query-level errors (syntax, auth, throttle)
  console.error("GraphQL errors:", errors);
}
if (data?.myMutation?.userErrors?.length) {
  // Business logic errors
  console.error("User errors:", data.myMutation.userErrors);
}
```

Error codes in 200 responses: `THROTTLED`, `ACCESS_DENIED`, `SHOP_INACTIVE`, `INTERNAL_SERVER_ERROR`
HTTP errors: 400, 402 (frozen shop), 403, 404, 423 (locked), 5xx

## Rate Limits Detail

Leaky bucket algorithm based on query cost points.

**Cost calculation:**
- Scalars/Enums: 0 points
- Objects: 1 point
- Mutations: 10 points
- Connections: sized by `first`/`last` arguments

**Response metadata:**
```json
{
  "extensions": {
    "cost": {
      "requestedQueryCost": 12,
      "actualQueryCost": 8,
      "throttleStatus": {
        "maximumAvailable": 1000,
        "currentlyAvailable": 992,
        "restoreRate": 50
      }
    }
  }
}
```

**Strategies:**
1. Request only needed fields
2. Cache frequently used data
3. Backoff 1 second on throttle (check `THROTTLED` error)
4. Use bulk operations for large datasets
5. Match `sortKey` to query filter for performance

**Limits:**
- Input array max: 250 items
- Pagination max: 25,000 objects deep
- Max page size: 250 items per request
- Variant limit: Beyond 50K variants/store, max 1,000 new/day

## Pagination

Cursor-based (Relay spec). Use `nodes` over `edges` unless per-edge cursor needed.

**Forward:** `first` + `after` (using `endCursor`)
**Backward:** `last` + `before` (using `startCursor`)

```graphql
query ($first: Int!, $after: String) {
  products(first: $first, after: $after) {
    nodes { id title }
    pageInfo { hasNextPage hasPreviousPage startCursor endCursor }
  }
}
```

## Bulk Operations

Async processing for large datasets. Bypasses rate limits and pagination.

```graphql
mutation {
  bulkOperationRunQuery(query: """
    { products { edges { node {
      id title
      variants { edges { node { id title price } } }
    } } } }
  """) {
    bulkOperation { id status }
    userErrors { field message }
  }
}
```

**Rules:**
- Remove `first`/`last`/`after`/`before` (auto-handled)
- Results as JSONL (one JSON object per line)
- Must use offline access tokens
- Up to 5 concurrent bulk query ops per shop (API 2026-01+)
- Must complete within 10 days
- Monitor via `bulkOperation(id:)` query or `BULK_OPERATIONS_FINISH` webhook

## Session Token JWT

```json
{
  "iss": "https://{shop}.myshopify.com/admin",
  "dest": "https://{shop}.myshopify.com",
  "aud": "{app_client_id}",
  "sub": "{user_id}",
  "exp": 1234567890,
  "iat": 1234567890,
  "jti": "{uuid}",
  "sid": "{session_id}"
}
```

Session tokens are for **authentication only**. Access tokens (from OAuth/token exchange) are for API calls.

## authenticate.admin() Return Values

- `admin` - GraphQL/REST client
- `session` - User session (shop, accessToken)
- `sessionToken` - Decoded JWT (embedded only)
- `redirect` - Embedded-aware redirect (embedded only)
- `cors` - CORS header helper
- `billing` - Methods: check, require, request, cancel, createUsageRecord
- `scopes` - Methods: query, request, revoke

## Webhook Headers

- `X-Shopify-Topic` - Event type
- `X-Shopify-Hmac-Sha256` - HMAC signature
- `X-Shopify-Shop-Domain` - Store domain
- `X-Shopify-Webhook-Id` - Unique webhook ID
- `X-Shopify-Triggered-At` - Trigger timestamp
- `X-Shopify-Event-Id` - Event ID (for deduplication)

No guaranteed ordering. Use `X-Shopify-Triggered-At` for chronological sorting.

## Metafields

**Ownership types:**
- App-owned (`$app` namespace): Managed by your app
- Merchant-owned (`custom` namespace): Shared across apps
- App-data: Hidden from admin, tied to AppInstallation

**Define in shopify.app.toml:**
```toml
[product.metafields.app.internal_sku]
name = "Internal SKU"
type = "single_line_text_field"
access.admin = "merchant_read_write"
access.storefront = "public_read"
```

## Session Storage

Default: Prisma with SQLite (dev only). Production options:
- `@shopify/shopify-app-session-storage-prisma` (PostgreSQL, MySQL, MongoDB)
- `@shopify/shopify-app-session-storage-redis`
- `@shopify/shopify-app-session-storage-mongodb`

Always replace SQLite for production deployments.

## App Extensions

Types: `ui_extension`, `function`, `theme_app_extension`, `flow_action`, `flow_trigger`, `flow_template`, `web_pixel`, `post_purchase_ui`, `subscription_ui`, `editor_extension_collection`

UI extensions have a strict **64 KB compressed** limit.
Max **5 metafield key/namespace pairs** per extension.
All extensions versioned together, deployed with `shopify app deploy`.
