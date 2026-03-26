# shopify.app.toml Complete Reference

## Overview

`shopify.app.toml` is the primary configuration file for Shopify apps. It defines scopes, webhooks, authentication, billing behavior, extensions, and more. Changes are applied when you run `shopify app deploy`.

## Top-Level Fields

```toml
# App identity
name = "My App"                                    # Display name
client_id = "abc123def456"                          # App client ID (from Partners dashboard)
handle = "my-app"                                   # URL-safe handle (do NOT change after deploy)
application_url = "https://myapp.example.com"       # Base URL for the app
embedded = true                                     # Whether app is embedded in admin

# Organization (auto-populated by CLI)
organization_id = "123456"
```

**IMPORTANT:** Never change `handle` after initial deploy - it breaks admin URLs for existing installs.

## Access Scopes

```toml
[access_scopes]
# Comma-separated list of required scopes
scopes = "read_products,write_products,read_orders,read_customers,read_analytics"

# Optional: scopes that are useful but not required
# Merchants can choose to deny optional scopes
optional_scopes = "write_customers,read_inventory"

# Use after deploy to keep existing scopes
use_legacy_install_flow = false
```

### Common Scopes

| Scope | Access |
|-------|--------|
| `read_products` / `write_products` | Products, variants, collections |
| `read_orders` / `write_orders` | Orders, transactions |
| `read_customers` / `write_customers` | Customer data |
| `read_inventory` / `write_inventory` | Inventory levels |
| `read_fulfillments` / `write_fulfillments` | Fulfillment data |
| `read_analytics` | Store analytics |
| `read_content` / `write_content` | Pages, blogs, articles |
| `read_themes` / `write_themes` | Theme files |
| `read_shipping` / `write_shipping` | Shipping rates |
| `read_discounts` / `write_discounts` | Discount codes/rules |
| `read_metaobjects` / `write_metaobjects` | Metaobjects |

Request only scopes your app needs. Excessive scopes are a common rejection reason.

## Admin API Access

```toml
[access.admin]
# How to access the Admin API from the browser
# "online" - uses online (user-specific) tokens
# "offline" - uses offline (app-level) tokens
direct_api_mode = "online"

# Enable direct API calls from the frontend via shopify:admin/api/graphql.json
embedded_app_direct_api_access = true
```

## Authentication

```toml
[auth]
# OAuth redirect URLs (for non-embedded or OAuth callback)
redirect_urls = [
  "https://myapp.example.com/auth/callback",
  "https://myapp.example.com/auth/shopify/callback"
]
```

## Webhooks

```toml
[webhooks]
# API version for webhook payloads
api_version = "2026-01"

# Privacy compliance webhook URL (shared endpoint for all compliance topics)
privacy_compliance_url = "https://myapp.example.com/webhooks/compliance"

# Regular webhooks
[[webhooks.subscriptions]]
topics = ["app/uninstalled"]
uri = "/webhooks"

[[webhooks.subscriptions]]
topics = ["products/create", "products/update", "products/delete"]
uri = "/webhooks"

[[webhooks.subscriptions]]
topics = ["orders/create", "orders/updated", "orders/paid"]
uri = "/webhooks"

# Compliance webhooks (mandatory for App Store)
[[webhooks.subscriptions]]
compliance_topics = ["customers/data_request", "customers/redact", "shop/redact"]
uri = "/webhooks"

# Filtered webhooks
[[webhooks.subscriptions]]
topics = ["orders/create"]
uri = "/webhooks/high-value"
filter = "total_price:>=1000"

# Sub-topic filtering
[[webhooks.subscriptions]]
topics = ["products/update"]
uri = "/webhooks/price-changes"
sub_topic = "type:price_change"

# EventBridge delivery
[[webhooks.subscriptions]]
topics = ["orders/create"]
uri = "arn:aws:events:us-east-1::event-source/aws.partner/shopify.com/123/source"

# Google Pub/Sub delivery
[[webhooks.subscriptions]]
topics = ["orders/create"]
uri = "pubsub://project-id:topic-id"
```

## App Proxy

Route requests through the Shopify storefront to your app:

```toml
[app_proxy]
url = "https://myapp.example.com/api/proxy"
subpath = "apps"
prefix = "my-app"
# Accessible at: https://store.myshopify.com/apps/my-app/*
```

## Application Charge (Managed Pricing)

```toml
[app_settings.pricing]
embedded = true
```

## Build Configuration

```toml
[build]
# Automatically update the app on deploy
automatically_update_urls_on_dev = true

# Dev store URL
dev_store_url = "my-dev-store.myshopify.com"

# Include config deploy confirmation
include_config_on_deploy = true
```

## Extensions

### UI Extension

```toml
[[extensions]]
type = "ui_extension"
name = "Product Review Widget"
handle = "product-review-widget"

[extensions.settings]
# Extension target (where it renders in Shopify admin)
[[extensions.targeting]]
target = "admin.product-details.block.render"
module = "./src/index.tsx"

# Extension configuration schema
[[extensions.settings.fields]]
key = "show_rating"
type = "boolean"
name = "Show rating"
```

### Theme App Extension

```toml
[[extensions]]
type = "theme_app_extension"
name = "My Theme Extension"
handle = "my-theme-ext"

[[extensions.embedded_blocks]]
template = "product"
```

### Checkout UI Extension

```toml
[[extensions]]
type = "checkout_ui_extension"
name = "Custom Checkout"
handle = "custom-checkout"

[extensions.capabilities]
api_access = true
network_access = true
block_progress = true

[[extensions.targeting]]
target = "purchase.checkout.block.render"
module = "./src/Checkout.tsx"
```

### Post-Purchase Extension

```toml
[[extensions]]
type = "post_purchase_ui_extension"
name = "Post Purchase Upsell"
handle = "post-purchase-upsell"

[[extensions.targeting]]
target = "purchase.post.render"
module = "./src/index.tsx"
```

### Flow Extension

```toml
# Flow Trigger
[[extensions]]
type = "flow_trigger"
name = "Custom Event"
handle = "custom-event"

[extensions.settings]
description = "Triggers when a custom event occurs"

[[extensions.settings.fields]]
key = "event_type"
name = "Event Type"
type = "string_field"

# Flow Action
[[extensions]]
type = "flow_action"
name = "Custom Action"
handle = "custom-action"

[extensions.settings]
description = "Performs a custom action"
url = "https://myapp.example.com/api/flow/action"

[[extensions.settings.fields]]
key = "input_data"
name = "Input"
type = "string_field"
required = true
```

### POS UI Extension

```toml
[[extensions]]
type = "pos_ui_extension"
name = "POS Widget"
handle = "pos-widget"

[[extensions.targeting]]
target = "pos.home.tile.render"
module = "./src/Tile.tsx"

[[extensions.targeting]]
target = "pos.home.modal.render"
module = "./src/Modal.tsx"
```

## Distribution

```toml
# App distribution type
# "app_store" - Public app on Shopify App Store
# "custom" - Custom app for specific merchants
# "single_merchant" - Single merchant install
distribution = "app_store"
```

## GDPR Endpoints

```toml
[gdpr]
# These can also be configured via compliance_topics in webhooks
customer_data_request_url = "https://myapp.example.com/webhooks/gdpr/data-request"
customer_deletion_url = "https://myapp.example.com/webhooks/gdpr/customer-delete"
shop_deletion_url = "https://myapp.example.com/webhooks/gdpr/shop-delete"
```

## Environment Variables Referenced

The following environment variables are typically used alongside the TOML:

```bash
SHOPIFY_API_KEY=         # Same as client_id in TOML
SHOPIFY_API_SECRET=      # API secret key (never in TOML)
SHOPIFY_APP_URL=         # Same as application_url
SCOPES=                  # Same as access_scopes.scopes
```

## Multiple Environments

Use separate TOML files for different environments:

```bash
shopify.app.toml          # Production config
shopify.app.staging.toml  # Staging config
shopify.app.dev.toml      # Development config
```

Switch environments:
```bash
shopify app config use staging
shopify app deploy --config staging
```

## Web Process Configuration (shopify.web.toml)

```toml
# shopify.web.toml
type = "frontend"
[commands]
dev = "npm run dev"
build = "npm run build"
```

Or for separate frontend/backend:

```toml
# web/frontend/shopify.web.toml
type = "frontend"
[commands]
dev = "npm run dev"
build = "npm run build"

# web/backend/shopify.web.toml
type = "backend"
port = 3001
[commands]
dev = "npm run dev"
```

## Complete Example

```toml
name = "My Awesome App"
client_id = "abc123def456ghi789"
handle = "my-awesome-app"
application_url = "https://myapp.example.com"
embedded = true

[access_scopes]
scopes = "read_products,write_products,read_orders,read_customers"

[access.admin]
direct_api_mode = "online"
embedded_app_direct_api_access = true

[auth]
redirect_urls = ["https://myapp.example.com/auth/callback"]

[webhooks]
api_version = "2026-01"

[[webhooks.subscriptions]]
topics = ["app/uninstalled", "products/update", "orders/create"]
uri = "/webhooks"

[[webhooks.subscriptions]]
compliance_topics = ["customers/data_request", "customers/redact", "shop/redact"]
uri = "/webhooks"

[[webhooks.subscriptions]]
topics = ["app_subscriptions/update"]
uri = "/webhooks"

[app_proxy]
url = "https://myapp.example.com/api/proxy"
subpath = "apps"
prefix = "my-awesome-app"

[build]
automatically_update_urls_on_dev = true
include_config_on_deploy = true

[[extensions]]
type = "flow_trigger"
name = "Custom Alert"
handle = "custom-alert"

[extensions.settings]
description = "Fires when a custom alert condition is met"

[[extensions.settings.fields]]
key = "alert_type"
name = "Alert Type"
type = "string_field"
```
