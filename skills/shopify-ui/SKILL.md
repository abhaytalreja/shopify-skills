---
name: shopify-ui
description: Shopify app UI development guide covering Polaris components, App Bridge APIs, design tokens, layout patterns, and accessibility. Use when building or styling Shopify embedded app interfaces.
---

# Shopify App UI Skill (Polaris + App Bridge)

## Quick Reference

- Design System: Polaris (`@shopify/polaris`)
- Icons: `@shopify/polaris-icons` (400+ commerce icons)
- App Bridge React: `@shopify/app-bridge-react` (TitleBar, NavMenu)
- Toast: `shopify.toast.show()` (App Bridge global, NOT Polaris)
- Modal: `<s-modal>` web component (App Bridge, NOT Polaris)
- Save Bar: `data-save-bar` form attribute or `shopify.saveBar.show()`
- Polaris Web Components: `<s-*>` prefix for embedded app contexts

## Root Setup

```tsx
import { AppProvider } from "@shopify/polaris";
import "@shopify/polaris/build/esm/styles.css";

function App() {
  return (
    <AppProvider i18n={{}}>
      {/* Your app */}
    </AppProvider>
  );
}
```

## Component Decision Guide

| Need | Component |
|------|-----------|
| Transient success/error message | `shopify.toast.show()` |
| Persistent info/warning/error | `<Banner>` |
| Blocking confirmation dialog | `<s-modal>` (App Bridge web component) |
| Status label on a resource | `<Badge>` |
| Data list with bulk selection | `<IndexTable>` + `<IndexFilters>` |
| Simple read-only data grid | `<DataTable>` |
| Form with field layout | `<FormLayout>` + field components |
| Settings page layout | `<InlineGrid columns={{ xs: "1fr", md: "2fr 5fr" }}>` |
| Unsaved changes prompt | `data-save-bar` attribute on `<form>` |
| Sidebar navigation | `<NavMenu>` (App Bridge React) |
| Page title + actions | `<Page>` component |
| Dropdown menu | `<Popover>` + `<ActionList>` |
| Loading skeleton | `<SkeletonPage>` + `<SkeletonBodyText>` |
| Empty data state | `<EmptyState>` inside `<Card>` |
| File upload | `<DropZone>` |
| Date selection | `<DatePicker>` |
| Color selection | `<ColorPicker>` |
| Resource selection from store | `shopify.resourcePicker()` (App Bridge) |
| Inline code/keyboard shortcut | `<KeyboardKey>` |
| Progress indication | `<ProgressBar>` or `<Spinner>` |
| Contextual help | `<Tooltip>` |

## Layout Patterns

### Resource Index (List Page)

```tsx
<Page title="Orders" primaryAction={{ content: "Create order" }}>
  <Card padding="0">
    <IndexFilters
      queryValue={query} onQueryChange={setQuery} onQueryClear={() => setQuery("")}
      tabs={tabs} selected={selectedTab} onSelect={setSelectedTab}
      filters={filters} appliedFilters={appliedFilters}
      sortOptions={sortOptions} sortSelected={sortSelected} onSort={setSortSelected}
      mode={mode} setMode={setMode}
    />
    <IndexTable
      resourceName={{ singular: "order", plural: "orders" }}
      itemCount={orders.length}
      headings={[{ title: "Order" }, { title: "Date" }, { title: "Total", alignment: "end" }]}
      selectedItemsCount={allSelected ? "All" : selectedResources.length}
      onSelectionChange={handleSelectionChange}
      promotedBulkActions={[{ content: "Fulfill" }]}
      pagination={{ hasNext: true, onNext: () => {} }}
    >
      {rowMarkup}
    </IndexTable>
  </Card>
</Page>
```

### Resource Detail / Settings Page

```tsx
<Page backAction={{ content: "Back", url: "/app" }} title="Settings"
  primaryAction={{ content: "Save" }} secondaryActions={[{ content: "Discard" }]}>
  <InlineGrid columns={{ xs: "1fr", md: "2fr 5fr" }} gap="400">
    <Box as="section">
      <BlockStack gap="400">
        <Text as="h3" variant="headingMd">General</Text>
        <Text as="p" variant="bodyMd" tone="subdued">Configure settings</Text>
      </BlockStack>
    </Box>
    <Card>
      <FormLayout>
        <TextField label="App Name" value={name} onChange={setName} autoComplete="off" />
        <Select label="Frequency" options={options} value={freq} onChange={setFreq} />
      </FormLayout>
    </Card>
  </InlineGrid>
</Page>
```

### Dashboard

```tsx
<Page title="Dashboard" subtitle="Performance overview">
  <BlockStack gap="400">
    <InlineGrid columns={{ xs: 1, md: 3 }} gap="400">
      <Card>
        <BlockStack gap="200">
          <Text variant="bodyMd" tone="subdued">Revenue</Text>
          <Text variant="headingXl">$4,230</Text>
        </BlockStack>
      </Card>
      {/* More metric cards */}
    </InlineGrid>
    <Card title="Recent Activity">
      <DataTable
        columnContentTypes={["text", "text", "numeric"]}
        headings={["Date", "Event", "Impact"]}
        rows={rows}
      />
    </Card>
  </BlockStack>
</Page>
```

For complete code examples of all layout patterns:
- [references/layout-patterns.md](references/layout-patterns.md)

## Navigation

```tsx
import { NavMenu, TitleBar } from "@shopify/app-bridge-react";

// Sidebar navigation (in app layout route)
<NavMenu>
  <a href="/" rel="home">Home</a>
  <a href="/analytics">Analytics</a>
  <a href="/settings">Settings</a>
</NavMenu>

// Page title bar with actions
<TitleBar title="Analytics" subtitle="Last 30 days" />
```

Page-level back navigation:
```tsx
<Page backAction={{ content: "Back", url: "/app" }} title="Product Details">
```

## App Bridge APIs

Toast (transient feedback):
```typescript
shopify.toast.show("Changes saved");
shopify.toast.show("Failed to save", { isError: true });
shopify.toast.show("Processing...", { duration: 5000 });
```

Modal (blocking dialog - web component):
```html
<s-modal id="confirm-delete">
  <s-text>Are you sure you want to delete this?</s-text>
  <s-button onclick="handleDelete()" variant="primary" tone="critical">Delete</s-button>
  <s-button onclick="shopify.modal.hide('confirm-delete')">Cancel</s-button>
</s-modal>
```

Save Bar:
```html
<!-- Automatic: detects form changes -->
<form data-save-bar data-discard-confirmation>
  <!-- form fields -->
</form>
```

```typescript
// Programmatic
shopify.saveBar.show('my-save-bar');
shopify.saveBar.hide('my-save-bar');
```

Loading indicator:
```typescript
shopify.loading.show();
shopify.loading.hide();
```

Resource Picker:
```typescript
const selected = await shopify.resourcePicker({ type: "product", multiple: true });
```

For the complete App Bridge API reference:
- [references/app-bridge-reference.md](references/app-bridge-reference.md)

## Form Patterns

```tsx
<Form onSubmit={handleSubmit}>
  <FormLayout>
    <TextField label="Title" value={title} onChange={setTitle}
      error={errors.title} helpText="Enter a descriptive title" autoComplete="off" />
    <FormLayout.Group>
      <TextField label="Price" type="number" prefix="$" value={price} onChange={setPrice} />
      <Select label="Currency" options={currencies} value={currency} onChange={setCurrency} />
    </FormLayout.Group>
    <Checkbox label="Enable notifications" checked={notify} onChange={setNotify} />
    <Button submit variant="primary">Save</Button>
  </FormLayout>
</Form>
```

Validation pattern with useFetcher:
```tsx
const fetcher = useFetcher();
const isSubmitting = fetcher.state === "submitting";

<fetcher.Form method="post">
  <FormLayout>
    <TextField label="Name" name="name" error={fetcher.data?.errors?.name} />
    <Button submit variant="primary" loading={isSubmitting}>Save</Button>
  </FormLayout>
</fetcher.Form>
```

## Feedback Decision Tree

```
User completed an action successfully?
  -> Toast: shopify.toast.show("Saved")

Need to show persistent status/info/warning?
  -> Banner: <Banner tone="warning" title="...">

Need user confirmation before destructive action?
  -> Modal: <s-modal> with critical tone button

Need inline status on a resource row?
  -> Badge: <Badge tone="success">Active</Badge>

Need blocking error that prevents progress?
  -> Banner tone="critical" at top of page
```

## Loading States

Full page skeleton:
```tsx
<SkeletonPage primaryAction>
  <Layout>
    <Layout.Section>
      <Card><SkeletonBodyText /></Card>
    </Layout.Section>
    <Layout.Section variant="oneThird">
      <Card><SkeletonBodyText lines={2} /></Card>
    </Layout.Section>
  </Layout>
</SkeletonPage>
```

Inline spinner:
```tsx
<Spinner size="small" />
```

App Bridge loading bar:
```typescript
shopify.loading.show();
// ... async operation
shopify.loading.hide();
```

## Empty States

```tsx
<Card>
  <EmptyState
    heading="No products yet"
    action={{ content: "Add product", url: "/app/products/new" }}
    secondaryAction={{ content: "Learn more", url: "https://help.shopify.com" }}
    image="https://cdn.shopify.com/s/files/1/0262/4071/2726/files/emptystate-files.png"
  >
    <p>Add products to start managing your inventory.</p>
  </EmptyState>
</Card>
```

## Polaris Web Components (s-prefix)

For embedded app contexts where React is not available, use web components:

```html
<s-page heading="Title">
  <s-section heading="Section">
    <s-stack direction="block" gap="base">
      <s-text-field label="Name" name="name" />
      <s-button variant="primary">Save</s-button>
    </s-stack>
  </s-section>
</s-page>
```

## Design Tokens (Summary)

Spacing (4px grid): `space-100`=4px, `space-200`=8px, `space-300`=12px, `space-400`=16px, `space-600`=24px, `space-800`=32px, `space-1200`=48px

Typography: Inter font, weights 450 (regular) / 700 (bold)
- `headingXl`=36px, `headingLg`=24px, `headingMd`=20px, `headingSm`=14px
- `bodyLg`=16px, `bodyMd`=14px, `bodySm`=12px

Colors: `bg`=#F1F1F1, `bg-surface`=#FFF, `bg-fill-brand`=#303030, `text`=#303030, `text-secondary`=#616161

Breakpoints: `sm`=490px, `md`=768px, `lg`=1040px, `xl`=1440px

For the complete token reference:
- [references/design-tokens.md](references/design-tokens.md)

For the complete component inventory:
- [references/polaris-components.md](references/polaris-components.md)

## Key Design Principles

1. Merchant-centered - Prioritize merchant workflows over aesthetic differentiation
2. Consistency - Match Shopify admin appearance and behavior exactly
3. Monochromatic base - Black/white foundation; use color only for status and emphasis
4. Mobile-first - All layouts must stack gracefully on small screens
5. Accessibility - WCAG AA contrast, full keyboard navigation, screen reader support
6. No custom styling - Use Polaris components and design tokens exclusively, not custom CSS
7. Content first - Labels and messages should be clear without explanation
8. Progressive disclosure - Show essential info first, details on demand
