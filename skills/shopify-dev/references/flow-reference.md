# Shopify Flow Integration Reference

## Overview

Shopify Flow is a visual automation platform for merchants. Apps can integrate with Flow by providing:
- **Triggers** - Events from your app that start a Flow workflow
- **Actions** - Tasks your app performs when a Flow workflow reaches your step
- **Templates** - Pre-built workflows merchants can install with one click

## Flow Triggers

Triggers notify Flow when something happens in your app. Flow then runs the merchant's configured workflow.

### TOML Configuration

```toml
[[extensions]]
type = "flow_trigger"
name = "Order Risk Detected"
handle = "order-risk-detected"

[extensions.settings]
description = "Triggers when a high-risk order is detected by the app."

[[extensions.settings.fields]]
key = "risk_level"
name = "Risk Level"
description = "The risk level of the order"
type = "string_field"

[[extensions.settings.fields]]
key = "order_id"
name = "Order"
description = "The order associated with the risk"
type = "shopify_resource"
resource = "order"

[[extensions.settings.fields]]
key = "risk_score"
name = "Risk Score"
description = "Numerical risk score from 0-100"
type = "number_field"
```

### Sending a Trigger

```typescript
import { authenticate } from "../shopify.server";

async function sendFlowTrigger(request: Request, data: {
  riskLevel: string;
  orderId: string;
  riskScore: number;
}) {
  const { admin } = await authenticate.admin(request);

  const response = await admin.graphql(
    `mutation FlowTriggerReceive($body: JSON!) {
      flowTriggerReceive(body: $body) {
        userErrors { field message }
      }
    }`,
    {
      variables: {
        body: JSON.stringify({
          trigger_handle: "order-risk-detected",
          risk_level: data.riskLevel,
          order_id: data.orderId,
          risk_score: data.riskScore,
        }),
      },
    }
  );

  return response.json();
}
```

### Trigger Field Types

| Type | Description |
|------|-------------|
| `string_field` | Text value |
| `number_field` | Numeric value (integer or decimal) |
| `boolean_field` | True/false |
| `email_field` | Email address |
| `url_field` | URL |
| `date_field` | ISO 8601 date |
| `shopify_resource` | Reference to Shopify resource (product, order, customer, etc.) |

### Shopify Resource Types for Triggers

Use with `type = "shopify_resource"` and `resource = "..."`:
- `product`, `product_variant`, `order`, `customer`, `collection`
- `draft_order`, `fulfillment`, `inventory_item`, `location`

## Flow Actions

Actions let Flow call your app to perform a task as part of a workflow.

### TOML Configuration

```toml
[[extensions]]
type = "flow_action"
name = "Send Custom Notification"
handle = "send-notification"

[extensions.settings]
description = "Sends a custom notification via the app."
url = "https://myapp.example.com/api/flow/send-notification"

[[extensions.settings.fields]]
key = "recipient_email"
name = "Recipient Email"
description = "Email address to send the notification to"
type = "email_field"
required = true

[[extensions.settings.fields]]
key = "message"
name = "Message"
description = "The notification message body"
type = "string_field"
required = true

[[extensions.settings.fields]]
key = "priority"
name = "Priority"
description = "Notification priority level"
type = "single_line_text_field"
choices = ["low", "medium", "high"]
```

### Action Endpoint Handler

```typescript
// app/routes/api.flow.send-notification.tsx
import type { ActionFunctionArgs } from "react-router";
import { authenticate } from "../shopify.server";

export const action = async ({ request }: ActionFunctionArgs) => {
  // Verify the request is from Shopify Flow
  const { admin, payload } = await authenticate.flow(request);

  const { recipient_email, message, priority } = payload.properties;
  const shopDomain = payload.shopify_domain;

  try {
    await sendNotification({
      to: recipient_email,
      body: message,
      priority: priority || "medium",
      shop: shopDomain,
    });

    // Return success
    return new Response(JSON.stringify({ success: true }), {
      status: 200,
      headers: { "Content-Type": "application/json" },
    });
  } catch (error) {
    // Return error (Flow will retry)
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { "Content-Type": "application/json" },
    });
  }
};
```

### Action Payload Structure

Flow sends a POST request with:

```json
{
  "shopify_domain": "shop.myshopify.com",
  "handle": "send-notification",
  "properties": {
    "recipient_email": "customer@example.com",
    "message": "Your order is ready",
    "priority": "high"
  }
}
```

### Action Response

- Return HTTP 200 for success
- Return HTTP 4xx/5xx for failure (Flow retries on 5xx)
- Return a JSON body with results that can be used in subsequent Flow steps

## Flow Templates

Pre-built workflows that merchants can install from your app listing.

### TOML Configuration

```toml
[[extensions]]
type = "flow_template"
name = "Auto-tag high-risk orders"
handle = "auto-tag-high-risk"

[extensions.settings]
description = "Automatically tags orders when a high risk is detected."
categories = ["risk", "orders"]

# Define the workflow steps
[[extensions.settings.triggers]]
type = "app_trigger"
handle = "order-risk-detected"

[[extensions.settings.conditions]]
type = "condition"
field = "risk_score"
operator = "greater_than"
value = "80"

[[extensions.settings.actions]]
type = "shopify_action"
handle = "add_order_tags"
properties = { tags = "high-risk,review-needed" }
```

## Flow Configuration Fields

### Field Type Reference

| TOML Type | Description | Example Value |
|-----------|-------------|---------------|
| `string_field` | Free text | `"Hello world"` |
| `single_line_text_field` | Single line text with optional choices | `"option_a"` |
| `multi_line_text_field` | Multi-line text | `"Line 1\nLine 2"` |
| `number_field` | Integer or decimal | `42` or `3.14` |
| `boolean_field` | True/false | `true` |
| `email_field` | Email address | `"user@example.com"` |
| `url_field` | URL | `"https://example.com"` |
| `date_field` | ISO date | `"2024-01-15T10:30:00Z"` |
| `shopify_resource` | Shopify GID | `"gid://shopify/Order/123"` |
| `json_field` | Raw JSON | `{"key": "value"}` |

### Field Properties

```toml
[[extensions.settings.fields]]
key = "field_key"          # Unique identifier
name = "Display Name"      # Shown in Flow UI
description = "Help text"  # Shown below the field
type = "string_field"      # Field type
required = true            # Whether field is required (default: false)
```

For `single_line_text_field` with choices:
```toml
[[extensions.settings.fields]]
key = "status"
name = "Status"
type = "single_line_text_field"
choices = ["pending", "approved", "rejected"]
```

## Built-in Shopify Flow Actions

These are actions provided by Shopify that you can reference in templates:

- `add_order_tags` / `remove_order_tags`
- `add_customer_tags` / `remove_customer_tags`
- `add_product_tags` / `remove_product_tags`
- `send_email` - Send email to customer
- `send_http_request` - Make HTTP request
- `create_draft_order`
- `cancel_order`
- `archive_order`
- `add_order_note`

## Testing Flow Integrations

1. Install the app on a development store
2. Open Shopify admin > Apps > Flow
3. Create a new workflow using your trigger/action
4. Test triggers by sending events from your app
5. Monitor Flow execution logs in the Shopify admin

```bash
# During development, use shopify app dev
# Flow actions will be called at your tunnel URL
shopify app dev
```

## Best Practices

1. Always validate action payloads - do not trust field values blindly
2. Return meaningful error messages from action endpoints
3. Use idempotency keys when actions create resources
4. Keep action execution under 10 seconds (use async processing for long tasks)
5. Provide useful field descriptions - merchants configure these in a visual builder
6. Include at least one Flow template with your app to showcase the integration
7. Test with various Shopify plan types - Flow availability varies by plan
