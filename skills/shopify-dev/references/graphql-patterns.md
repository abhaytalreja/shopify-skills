# GraphQL Admin API Patterns Reference

## Rate Limiting

Every GraphQL query has a calculated cost. Shopify uses a leaky bucket algorithm.

| Plan | Restore Rate | Max Bucket |
|------|-------------|------------|
| Standard | 100 pts/sec | 1,000 pts |
| Shopify Plus (Advanced) | 200 pts/sec | - |
| Shopify Plus (Plus) | 1,000 pts/sec | - |

Single query maximum: 1,000 points.

### Inspecting Query Cost

Add header `Shopify-GraphQL-Cost-Debug: 1` to see actual vs requested cost:

```json
{
  "extensions": {
    "cost": {
      "requestedQueryCost": 12,
      "actualQueryCost": 8,
      "throttleStatus": {
        "maximumAvailable": 1000,
        "currentlyAvailable": 992,
        "restoreRate": 100
      }
    }
  }
}
```

### Cost Calculation Rules

- Each field at root level: 1 point
- Each connection (`first: N`): N points per node
- Nested connections multiply: `products(first: 10) { variants(first: 5) }` = 10 + (10 * 5) = 60
- Mutations: base cost of 10 points

### Throttling Response

When throttled, you receive a `THROTTLED` error:

```json
{
  "errors": [{
    "message": "Throttled",
    "extensions": { "code": "THROTTLED" }
  }]
}
```

Handle with exponential backoff. Check `Retry-After` header or `currentlyAvailable` from cost extension.

## Pagination

Use cursor-based pagination with `pageInfo`:

```graphql
query GetProducts($first: Int!, $after: String) {
  products(first: $first, after: $after) {
    edges {
      cursor
      node {
        id
        title
        handle
        status
        totalInventory
        variants(first: 5) {
          nodes {
            id
            title
            price
            inventoryQuantity
          }
        }
      }
    }
    pageInfo {
      hasNextPage
      endCursor
      hasPreviousPage
      startCursor
    }
  }
}
```

Prefer `nodes` over `edges` when you do not need cursors:

```graphql
query {
  products(first: 50) {
    nodes { id title }
    pageInfo { hasNextPage endCursor }
  }
}
```

### Pagination Loop Pattern

```typescript
let hasNextPage = true;
let cursor: string | null = null;
const allProducts = [];

while (hasNextPage) {
  const response = await admin.graphql(
    `query ($first: Int!, $after: String) {
      products(first: $first, after: $after) {
        nodes { id title }
        pageInfo { hasNextPage endCursor }
      }
    }`,
    { variables: { first: 250, after: cursor } }
  );
  const { data } = await response.json();
  allProducts.push(...data.products.nodes);
  hasNextPage = data.products.pageInfo.hasNextPage;
  cursor = data.products.pageInfo.endCursor;
}
```

Maximum `first` value: 250 per request.

## Bulk Operations

For datasets exceeding a few thousand records, use bulk operations. They bypass rate limits and return results as JSONL.

### Starting a Bulk Query

```graphql
mutation {
  bulkOperationRunQuery(
    query: """
    {
      products {
        edges {
          node {
            id
            title
            variants {
              edges {
                node {
                  id
                  title
                  price
                }
              }
            }
          }
        }
      }
    }
    """
  ) {
    bulkOperation {
      id
      status
    }
    userErrors {
      field
      message
    }
  }
}
```

Rules for bulk operation queries:
- Do NOT use `first`, `last`, `after`, `before` arguments (they are ignored)
- Must use `edges { node { ... } }` syntax (not `nodes`)
- Only one bulk operation can run at a time per app per shop
- Use offline access tokens (operations can run for hours)

### Polling for Completion

```graphql
query {
  currentBulkOperation {
    id
    status
    errorCode
    objectCount
    fileSize
    url
    partialDataUrl
  }
}
```

Status values: `CREATED`, `RUNNING`, `COMPLETED`, `FAILED`, `CANCELED`, `EXPIRED`.

### Webhook for Completion

Subscribe to `bulk_operations/finish` to avoid polling:

```toml
[[webhooks.subscriptions]]
topics = ["bulk_operations/finish"]
uri = "/webhooks"
```

### Processing JSONL Results

Each line is a JSON object. Parent-child relationships use `__parentId`:

```jsonl
{"id":"gid://shopify/Product/123","title":"Widget"}
{"id":"gid://shopify/ProductVariant/456","title":"Small","price":"19.99","__parentId":"gid://shopify/Product/123"}
```

```typescript
const response = await fetch(bulkOperation.url);
const text = await response.text();
const lines = text.trim().split("\n").map(JSON.parse);

// Group children by parent
const products = new Map();
for (const line of lines) {
  if (line.__parentId) {
    const parent = products.get(line.__parentId);
    if (parent) parent.variants.push(line);
  } else {
    products.set(line.id, { ...line, variants: [] });
  }
}
```

### Bulk Mutations

```graphql
mutation {
  bulkOperationRunMutation(
    mutation: "mutation call($input: ProductInput!) { productUpdate(input: $input) { product { id } userErrors { field message } } }",
    stagedUploadPath: "tmp/staged-upload-path"
  ) {
    bulkOperation { id status }
    userErrors { field message }
  }
}
```

Upload input data as JSONL via staged uploads first.

## Key Queries

### Products

```graphql
query GetProduct($id: ID!) {
  product(id: $id) {
    id title handle status description descriptionHtml
    productType vendor tags
    totalInventory tracksInventory
    priceRangeV2 { minVariantPrice { amount currencyCode } maxVariantPrice { amount currencyCode } }
    images(first: 10) { nodes { id url altText width height } }
    variants(first: 100) {
      nodes {
        id title price compareAtPrice sku barcode
        inventoryQuantity selectedOptions { name value }
        inventoryItem { id tracked }
      }
    }
    metafields(first: 20) { nodes { id namespace key value type } }
  }
}
```

### Orders

```graphql
query GetOrders($first: Int!, $query: String) {
  orders(first: $first, query: $query, sortKey: CREATED_AT, reverse: true) {
    nodes {
      id name createdAt displayFinancialStatus displayFulfillmentStatus
      totalPriceSet { shopMoney { amount currencyCode } }
      customer { id displayName email }
      lineItems(first: 50) {
        nodes { title quantity originalUnitPriceSet { shopMoney { amount } } }
      }
      shippingAddress { address1 city province country zip }
    }
    pageInfo { hasNextPage endCursor }
  }
}
```

Order query filter syntax: `"financial_status:paid created_at:>2024-01-01"`.

### Customers

```graphql
query GetCustomers($first: Int!, $query: String) {
  customers(first: $first, query: $query) {
    nodes {
      id displayName email phone
      numberOfOrders amountSpent { amount currencyCode }
      tags state createdAt
      addresses(first: 5) { address1 city province country zip }
    }
    pageInfo { hasNextPage endCursor }
  }
}
```

### Inventory

```graphql
query GetInventoryLevels($inventoryItemId: ID!) {
  inventoryItem(id: $inventoryItemId) {
    id tracked sku
    inventoryLevels(first: 10) {
      nodes {
        id
        quantities(names: ["available", "committed", "on_hand"]) {
          name quantity
        }
        location { id name }
      }
    }
  }
}
```

## Key Mutations

### Product Create

```graphql
mutation CreateProduct($input: ProductInput!) {
  productCreate(input: $input) {
    product { id title handle }
    userErrors { field message code }
  }
}
```

Variables:
```json
{
  "input": {
    "title": "New Product",
    "descriptionHtml": "<p>Product description</p>",
    "productType": "Clothing",
    "vendor": "My Brand",
    "tags": ["sale", "new"],
    "status": "DRAFT"
  }
}
```

### Product Update

```graphql
mutation UpdateProduct($input: ProductInput!) {
  productUpdate(input: $input) {
    product { id title }
    userErrors { field message }
  }
}
```

### Inventory Adjust

```graphql
mutation AdjustInventory($input: InventoryAdjustQuantitiesInput!) {
  inventoryAdjustQuantities(input: $input) {
    inventoryAdjustmentGroup {
      reason
      changes { name delta quantityAfterAdjustment }
    }
    userErrors { field message }
  }
}
```

Variables:
```json
{
  "input": {
    "name": "available",
    "reason": "correction",
    "changes": [{
      "inventoryItemId": "gid://shopify/InventoryItem/123",
      "locationId": "gid://shopify/Location/456",
      "delta": 10
    }]
  }
}
```

### Order Fulfill

```graphql
mutation FulfillOrder($fulfillment: FulfillmentV2Input!) {
  fulfillmentCreateV2(fulfillment: $fulfillment) {
    fulfillment { id status }
    userErrors { field message }
  }
}
```

### Metafield Set

```graphql
mutation SetMetafields($metafields: [MetafieldsSetInput!]!) {
  metafieldsSet(metafields: $metafields) {
    metafields { id namespace key value type }
    userErrors { field message }
  }
}
```

Variables:
```json
{
  "metafields": [{
    "ownerId": "gid://shopify/Product/123",
    "namespace": "custom",
    "key": "rating",
    "value": "4.5",
    "type": "number_decimal"
  }]
}
```

### Webhook Subscription Create (Programmatic)

```graphql
mutation {
  webhookSubscriptionCreate(
    topic: ORDERS_CREATE
    webhookSubscription: {
      callbackUrl: "https://myapp.com/webhooks"
      format: JSON
    }
  ) {
    webhookSubscription { id topic }
    userErrors { field message }
  }
}
```

## Staged Uploads

For uploading files (images, JSONL for bulk mutations):

```graphql
mutation {
  stagedUploadsCreate(input: [{
    filename: "image.jpg"
    mimeType: "image/jpeg"
    resource: PRODUCT_IMAGE
    httpMethod: POST
  }]) {
    stagedTargets {
      url
      resourceUrl
      parameters { name value }
    }
    userErrors { field message }
  }
}
```

Then POST to the returned URL with the file and parameters as multipart form data.

## Idempotency

For financial and critical mutations, use the `idempotencyKey` parameter:

```graphql
mutation($idempotencyKey: String!) {
  orderCreate(input: { ... }, idempotencyKey: $idempotencyKey) {
    order { id }
    userErrors { field message }
  }
}
```

Generate unique keys per operation attempt (e.g., UUID). Shopify deduplicates mutations with the same key within a time window.

## Error Handling Pattern

```typescript
const response = await admin.graphql(mutation, { variables });
const { data } = await response.json();

// Check for userErrors (business logic errors)
if (data.productCreate.userErrors.length > 0) {
  const errors = data.productCreate.userErrors;
  // Handle validation errors - these are expected
  return { errors };
}

// Check for top-level errors (rate limiting, auth, etc.)
// These throw automatically with admin.graphql() in most cases
```

Always check `userErrors` on mutation responses. They indicate business rule violations, not transport errors.

## Global ID Utilities

```typescript
// Parse a global ID
const numericId = globalId.split("/").pop(); // "123" from "gid://shopify/Product/123"

// Construct a global ID
const globalId = `gid://shopify/${type}/${numericId}`;

// Common types: Product, ProductVariant, Order, Customer, Collection,
// InventoryItem, InventoryLevel, Location, FulfillmentOrder, Metafield
```
