# Shopify Flow Integration Reference

## Overview

Flow is available on all paid Shopify plans. Apps get "Works with Flow" badge. Four extension types: Triggers, Actions, Conditions (built-in only), Templates.

## Flow Triggers

Fire events from your app that start Flow workflows.

### TOML Configuration
```toml
[[extensions]]
name = "Order Risk Detected"
type = "flow_trigger"
handle = "order-risk-detected"    # Max 30 chars, immutable after deploy
description = "Fires when a risky order is detected"

[settings]
  [[settings.fields]]
  type = "order_reference"

  [[settings.fields]]
  type = "number_decimal"
  key = "risk_score"
  description = "Risk score from 0 to 100"
```

### Reference Field Types
- `customer_reference` -> payload key `customer_id` (requires `read_customers`)
- `order_reference` -> payload key `order_id`
- `product_reference` -> payload key `product_id`

### Custom Field Types
`boolean`, `email`, `single_line_text_field`, `number_decimal`, `url`, `schema.<type>`

### Firing a Trigger
```graphql
mutation {
  flowTriggerReceive(
    handle: "order-risk-detected",
    payload: { "risk_score": "85", "order_id": 12345 }
  ) {
    userErrors { field message }
  }
}
```

Payload must be under 50KB. All defined fields are required.

## Flow Actions

Execute tasks in your app when Flow conditions are met.

### TOML Configuration
```toml
[[extensions]]
name = "Send Alert Email"
type = "flow_action"
handle = "send-alert"
description = "Sends an alert email"
runtime_url = "https://your-server/flow/send-alert"
schema = "./schema.graphql"       # Optional for complex return types
return_type_ref = "AlertResult"   # Optional

[settings]
  [[settings.fields]]
  type = "customer_reference"
  required = true

  [[settings.fields]]
  type = "single_line_text_field"
  key = "message"
  name = "Alert Message"
  required = true
```

### Server Endpoint (runtime_url)
- Receives POST with: `shop_id`, `shopify_domain`, `action_run_id`, `handle`, `properties`, `step_reference`
- Responses: 200 (processed), 202 (accepted, async), 4XX (stop retries), 5XX (retry up to 36hrs)
- 10-second timeout
- Use `action_run_id` as idempotency key
- Verify HMAC via `x-shopify-hmac-sha256` header

### Complex Return Data
Define types in `schema.graphql`:
```graphql
type AlertResult {
  success: Boolean!
  alertId: String!
  sentAt: String
}
```

Include `return_value` in action response. Must be under 50KB.

## Flow Templates

Pre-built workflow examples merchants can duplicate.

### Creation Process
1. Build workflow in dev store Flow editor
2. Export as `.flow` file
3. Run `shopify app generate extension` (select Flow Template)
4. Replace generated `template.flow` with your export
5. Configure TOML
6. Deploy with `shopify app deploy`

### Template TOML
```toml
[[extensions]]
name = "Alert on High-Risk Orders"
type = "flow_template"
handle = "alert-high-risk"
categories = ["orders", "risk"]   # 1-2 categories
module = "./template.flow"
description = "Automatically alert team on high-risk orders"
discoverable = true
enabled = true
```

### Constraints
- Max 25 templates per app
- One template per extension
- Public apps must be listed before submitting templates
- 3 business day review window

### Categories
`buyer_experience`, `customers`, `inventory_and_merch`, `loyalty`, `orders`, `promotions`, `risk`, `fulfillment`, `b2b`, `payment_reminders`, `custom_data`, `error_monitoring`

## Common Flow Use Cases for Analytics Apps

1. **Trigger: Anomaly Detected** - Fire when metrics deviate significantly
2. **Trigger: Daily Report Ready** - Fire when morning briefing is generated
3. **Action: Generate Insight** - AI-analyze specific order/customer data
4. **Action: Send Alert** - Deliver alerts via email/Slack
5. **Template: Alert on Revenue Drop** - Pre-built workflow for revenue monitoring
6. **Template: Weekly Performance Digest** - Scheduled performance email

## Deployment

```bash
# Generate extension
shopify app generate extension  # Select Flow type

# Edit TOML and implement handler

# Test in development
shopify app dev  # Creates draft version

# Deploy to production
shopify app deploy
```
