# App Bridge API Reference

## Overview

App Bridge is the communication layer between your embedded app and the Shopify admin. It provides:
- UI components (modal, toast, save bar, loading bar)
- Navigation control
- Resource selection
- Session token management

App Bridge is loaded automatically via CDN script tag in embedded apps. No npm install needed for the core APIs. The `shopify` global object is available in all embedded app contexts.

## Setup

App Bridge is automatically configured when using the Shopify app template. The script tag is injected by the framework.

For React components (NavMenu, TitleBar), install:
```bash
npm install @shopify/app-bridge-react
```

## The `shopify` Global Object

Available in any embedded app page. Provides direct access to App Bridge APIs.

### shopify.toast

Show transient notifications.

```typescript
// Success toast (default)
shopify.toast.show("Changes saved");

// Error toast
shopify.toast.show("Failed to save changes", { isError: true });

// Custom duration (milliseconds, default: 5000)
shopify.toast.show("Processing your request...", { duration: 10000 });

// Action toast
shopify.toast.show("Product archived", {
  action: {
    content: "Undo",
    onAction: () => handleUndo(),
  },
});
```

Toast rules:
- Max duration: 10,000ms
- Only one toast visible at a time (new toasts replace existing)
- Toasts appear at the bottom of the Shopify admin frame
- Use for transient, non-critical feedback only
- For persistent messages, use `<Banner>` component instead

### shopify.modal

Show modal dialogs. Uses web components (`<s-modal>`).

```html
<!-- Define modal in your page HTML -->
<s-modal id="my-modal" variant="base">
  <s-box padding="base">
    <s-text>Modal content here</s-text>
  </s-box>
  <s-button slot="footer" variant="primary" onclick="handleConfirm()">Confirm</s-button>
  <s-button slot="footer" onclick="shopify.modal.hide('my-modal')">Cancel</s-button>
</s-modal>

<!-- Large modal -->
<s-modal id="large-modal" variant="large">
  <s-box padding="base">
    <s-text>Extended content with more space</s-text>
  </s-box>
</s-modal>

<!-- Max modal (full screen on mobile) -->
<s-modal id="max-modal" variant="max">
  <s-box padding="base">
    <s-text>Full-width content</s-text>
  </s-box>
</s-modal>
```

Programmatic control:
```typescript
// Show modal
shopify.modal.show("my-modal");

// Hide modal
shopify.modal.hide("my-modal");
```

Modal variants:
- `base` - Standard size (default)
- `large` - Wider modal
- `max` - Maximum width, full screen on mobile

Modal events:
```typescript
document.getElementById("my-modal").addEventListener("show", () => {
  console.log("Modal opened");
});

document.getElementById("my-modal").addEventListener("hide", () => {
  console.log("Modal closed");
});
```

### shopify.saveBar

Show the contextual save bar when form data changes.

Automatic (recommended):
```html
<!-- Add data-save-bar to any form -->
<form data-save-bar data-discard-confirmation>
  <input type="text" name="title" value="Original value" />
  <!-- Save bar appears automatically when form values change -->
  <!-- data-discard-confirmation shows a confirmation dialog on discard -->
</form>
```

Programmatic:
```typescript
// Show save bar
shopify.saveBar.show("my-save-bar");

// Hide save bar
shopify.saveBar.hide("my-save-bar");
```

With web component:
```html
<s-save-bar id="my-save-bar">
  <button variant="primary" onclick="handleSave()">Save</button>
  <button onclick="handleDiscard()">Discard</button>
</s-save-bar>
```

Save bar events:
```typescript
document.getElementById("my-save-bar").addEventListener("save", async () => {
  await saveData();
  shopify.saveBar.hide("my-save-bar");
  shopify.toast.show("Saved");
});

document.getElementById("my-save-bar").addEventListener("discard", () => {
  resetForm();
  shopify.saveBar.hide("my-save-bar");
});
```

### shopify.loading

Show/hide the loading bar at the top of the Shopify admin.

```typescript
// Show loading bar
shopify.loading.show();

// Hide loading bar
shopify.loading.hide();
```

Use for async operations that take more than 500ms. Hide when complete.

### shopify.resourcePicker

Open the native Shopify resource picker dialog.

```typescript
// Product picker
const products = await shopify.resourcePicker({
  type: "product",
  multiple: true,           // Allow selecting multiple (default: false)
  action: "select",         // "select" or "add"
  selectionIds: [],         // Pre-selected resource IDs
  filter: {
    query: "",              // Search query
    variants: true,         // Show variant selection
    draft: false,           // Include draft products
    archived: false,        // Include archived products
  },
});

// Collection picker
const collections = await shopify.resourcePicker({
  type: "collection",
  multiple: false,
});

// Variant picker
const variants = await shopify.resourcePicker({
  type: "variant",
  multiple: true,
});
```

Return value (when user selects):
```typescript
// products is an array of selected resources:
[{
  id: "gid://shopify/Product/123",
  title: "Blue T-Shirt",
  handle: "blue-t-shirt",
  images: [{ originalSrc: "https://..." }],
  variants: [{
    id: "gid://shopify/ProductVariant/456",
    title: "Small",
    price: "29.99",
  }],
}]
```

Return value (when user cancels): `undefined`

### shopify.idToken

Get the current session token.

```typescript
const token = await shopify.idToken();
// JWT token string for server-side verification
```

Use this when making direct API calls outside the authenticate flow.

### shopify.config

Access app configuration.

```typescript
const config = shopify.config;
// { apiKey: "...", host: "...", shop: "store.myshopify.com" }
```

## React Components (@shopify/app-bridge-react)

### NavMenu

Sidebar navigation for your app.

```tsx
import { NavMenu } from "@shopify/app-bridge-react";

function AppLayout() {
  return (
    <>
      <NavMenu>
        <a href="/app" rel="home">Dashboard</a>
        <a href="/app/products">Products</a>
        <a href="/app/orders">Orders</a>
        <a href="/app/analytics">Analytics</a>
        <a href="/app/settings">Settings</a>
      </NavMenu>
      <Outlet />
    </>
  );
}
```

Rules:
- Use plain `<a>` tags, not React Router `<Link>`
- First item should have `rel="home"` attribute
- Maximum 6-8 items recommended
- Items appear in the left sidebar of Shopify admin
- NavMenu should be in the layout route (e.g., `app.tsx`)

### TitleBar

Override the page title bar.

```tsx
import { TitleBar } from "@shopify/app-bridge-react";

function ProductPage() {
  return (
    <>
      <TitleBar title="Product Details" />
      {/* Page content */}
    </>
  );
}
```

Note: In most cases, use the `<Page>` Polaris component instead. TitleBar is for cases where you need to set the title outside the Page component context.

## Web Components (s-prefix)

Used in embedded app contexts. Available without React.

### s-page

```html
<s-page heading="Page Title" back-action="navigateBack()">
  <s-section heading="Section Title">
    <s-stack direction="block" gap="base">
      <s-text>Content here</s-text>
    </s-stack>
  </s-section>
</s-page>
```

### s-text-field

```html
<s-text-field
  label="Product title"
  name="title"
  value="Blue T-Shirt"
  placeholder="Enter title"
  help-text="This is shown to customers"
  error="Title is required"
  required
  disabled
  multiline
  max-length="100"
  show-character-count
></s-text-field>
```

### s-button

```html
<s-button variant="primary">Save</s-button>
<s-button variant="plain">Cancel</s-button>
<s-button tone="critical">Delete</s-button>
<s-button loading>Processing</s-button>
<s-button disabled>Unavailable</s-button>
```

Variants: `primary`, `secondary` (default), `plain`, `tertiary`
Tones: `critical`

### s-select

```html
<s-select label="Country" name="country">
  <option value="US">United States</option>
  <option value="CA">Canada</option>
  <option value="UK">United Kingdom</option>
</s-select>
```

### s-checkbox

```html
<s-checkbox label="Enable notifications" name="notify" checked></s-checkbox>
```

### s-banner

```html
<s-banner tone="success" title="Setup complete">
  Your store is now connected.
</s-banner>
<s-banner tone="warning">Check your settings.</s-banner>
<s-banner tone="critical">Payment failed.</s-banner>
```

### s-badge

```html
<s-badge tone="success">Active</s-badge>
<s-badge tone="warning">Pending</s-badge>
<s-badge tone="critical">Error</s-badge>
```

### s-card

```html
<s-card>
  <s-box padding="base">
    <s-text>Card content</s-text>
  </s-box>
</s-card>
```

### s-stack

```html
<!-- Vertical stack -->
<s-stack direction="block" gap="base">
  <s-text>Item 1</s-text>
  <s-text>Item 2</s-text>
</s-stack>

<!-- Horizontal stack -->
<s-stack direction="inline" gap="tight">
  <s-badge>Tag 1</s-badge>
  <s-badge>Tag 2</s-badge>
</s-stack>
```

Directions: `block` (vertical), `inline` (horizontal)
Gaps: `none`, `extra-tight`, `tight`, `base`, `loose`, `extra-loose`

### s-box

```html
<s-box padding="base" background="subdued">
  Content with padding and background
</s-box>
```

### s-text

```html
<s-text>Regular text</s-text>
<s-text variant="headingLg">Large heading</s-text>
<s-text variant="bodySm" tone="subdued">Small subdued text</s-text>
```

### s-divider

```html
<s-divider></s-divider>
```

### s-thumbnail

```html
<s-thumbnail source="https://example.com/image.jpg" alt="Product" size="small"></s-thumbnail>
```

## Direct API Access

Make GraphQL calls directly from the frontend:

```typescript
// Requires embedded_app_direct_api_access = true in shopify.app.toml
const response = await fetch("shopify:admin/api/graphql.json", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    query: `
      query GetProducts($first: Int!) {
        products(first: $first) {
          nodes { id title }
        }
      }
    `,
    variables: { first: 10 },
  }),
});

const { data } = await response.json();
```

This uses the session token automatically. No need to pass authentication headers.

Note: Direct API access uses the merchant's online session, so it respects the merchant's permissions. For background tasks, use server-side calls with offline tokens.

## Navigation

Navigate within the embedded app:

```typescript
// Using React Router (recommended in React apps)
import { useNavigate } from "react-router";
const navigate = useNavigate();
navigate("/app/products");

// Using window.location (works in non-React contexts)
window.location.href = "/app/products";

// External links open in new tab automatically when target="_blank"
```

Navigate to Shopify admin pages:
```typescript
// Open a specific Shopify admin page
open("shopify://admin/products", "_top");
open("shopify://admin/orders", "_top");
open("shopify://admin/customers", "_top");
```

## Full Page Redirect

For OAuth or external redirects that need to break out of the iframe:

```typescript
// Server-side (in loader/action)
const { redirect } = await authenticate.admin(request);
return redirect("/app/destination"); // Handles iframe-safe redirect

// For external URLs
return redirect("https://external-site.com", { target: "_top" });
```

## Authentication Flow

Token exchange happens automatically when using `authenticate.admin(request)`. The flow:

1. Shopify admin loads your app in an iframe
2. App Bridge provides a session token via `shopify.idToken()`
3. Your server exchanges this for an access token
4. Access token is stored in session storage
5. Subsequent requests use the stored token

You rarely need to interact with this directly. The `authenticate` middleware handles everything.

## App Bridge Version

App Bridge auto-updates via CDN. No version pinning needed. The embedded app loads the latest compatible version automatically.

Check current version:
```typescript
console.log(shopify.version);
```
