---
description: "Use when building UI for Shopify apps. Covers Polaris design system components, App Bridge UI (modals, toasts, save bar, navigation), design tokens (colors, spacing, typography), layout patterns, form patterns, data tables, empty/loading states, and accessibility. Trigger when code imports @shopify/polaris or user asks about Shopify app UI/design."
---

# Shopify App UI Skill (Polaris + App Bridge)

## Quick Reference

**Design System:** Polaris (`@shopify/polaris`)
**Icons:** `@shopify/polaris-icons` (400+ commerce icons)
**App Bridge React:** `@shopify/app-bridge-react` (TitleBar, NavMenu)
**Toast:** `shopify.toast.show()` (App Bridge, NOT Polaris)
**Modal:** `<s-modal>` web component (App Bridge, NOT Polaris)
**Save Bar:** `data-save-bar` attribute on forms or `shopify.saveBar.show()`

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

### Resource Detail (Settings/Edit Page)
```tsx
<Page backAction={{ content: "Back", url: "/app" }} title="Settings"
  primaryAction={{ content: "Save" }} secondaryActions={[{ content: "Discard" }]}>
  <InlineGrid columns={{ xs: "1fr", md: "2fr 5fr" }} gap="400">
    <Box as="section">
      <BlockStack gap="400">
        <Text as="h3" variant="headingMd">General</Text>
        <Text as="p" variant="bodyMd" tone="subdued">Configure your settings</Text>
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

### Dashboard (Single Column)
```tsx
<Page title="Dashboard" subtitle="Performance overview">
  <BlockStack gap="400">
    <InlineGrid columns={{ xs: 1, md: 3 }} gap="400">
      <Card><BlockStack gap="200">
        <Text variant="bodyMd" tone="subdued">Revenue</Text>
        <Text variant="headingXl">$4,230</Text>
      </BlockStack></Card>
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

## Navigation

```tsx
import { NavMenu, TitleBar } from "@shopify/app-bridge-react";

// Sidebar navigation
<NavMenu>
  <a href="/" rel="home">Home</a>
  <a href="/analytics">Analytics</a>
  <a href="/settings">Settings</a>
</NavMenu>

// Page title bar
<TitleBar title="Analytics" subtitle="Last 30 days" />
```

## Core Components Quick Reference

### Feedback
```tsx
// Banner (persistent messages)
<Banner title="Setup complete" tone="success" onDismiss={() => {}}>
  <p>Your store is connected.</p>
</Banner>

// Toast (transient, via App Bridge)
shopify.toast.show("Changes saved");
shopify.toast.show("Failed to save", { isError: true });

// Badge (status indicators)
<Badge tone="success">Active</Badge>
<Badge tone="warning">Pending</Badge>
<Badge tone="critical">Error</Badge>
```

### Forms
```tsx
<Form onSubmit={handleSubmit}>
  <FormLayout>
    <TextField label="Title" value={title} onChange={setTitle} error={errors.title} helpText="Enter a descriptive title" />
    <FormLayout.Group>
      <TextField label="Price" type="number" prefix="$" value={price} onChange={setPrice} />
      <Select label="Currency" options={currencies} value={currency} onChange={setCurrency} />
    </FormLayout.Group>
    <Checkbox label="Enable notifications" checked={notify} onChange={setNotify} />
    <Button submit variant="primary">Save</Button>
  </FormLayout>
</Form>
```

### Save Bar (App Bridge)
```html
<!-- Automatic detection -->
<form data-save-bar data-discard-confirmation>
  <!-- fields -->
</form>

<!-- Programmatic -->
<script>
  shopify.saveBar.show('my-save-bar');
  shopify.saveBar.hide('my-save-bar');
</script>
```

### Modal (App Bridge)
```html
<s-modal id="confirm-delete">
  <s-text>Are you sure?</s-text>
  <s-button onclick="handleDelete()" variant="primary" tone="critical">Delete</s-button>
  <s-button onclick="shopify.modal.hide('confirm-delete')">Cancel</s-button>
</s-modal>
```

### Popover
```tsx
<Popover active={active} activator={<Button onClick={toggle} disclosure>Actions</Button>} onClose={toggle}>
  <ActionList items={[{ content: "Import" }, { content: "Export" }]} />
</Popover>
```

## Empty & Loading States

```tsx
// Empty state
<Card>
  <EmptyState heading="No data yet" action={{ content: "Connect store" }}
    image="https://cdn.shopify.com/s/files/1/0262/4071/2726/files/emptystate-files.png">
    <p>Connect your store to start receiving insights.</p>
  </EmptyState>
</Card>

// Skeleton loading
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

// Inline spinner
<Spinner size="small" />

// Full page loading (App Bridge)
shopify.loading.show();
shopify.loading.hide();
```

## Design Tokens

### Spacing (4px base grid)
`space-100`=4px, `space-200`=8px, `space-300`=12px, `space-400`=16px, `space-500`=20px, `space-600`=24px, `space-800`=32px, `space-1200`=48px

Card padding: `space-400` (16px). Card gap: `space-400`. Button group gap: `space-200`.

### Typography
Font: Inter, system sans-serif. Weights: regular=450, bold=700.
- `headingXl`=36px, `headingLg`=24px, `headingMd`=20px, `headingSm`=14px
- `bodyLg`=16px, `bodyMd`=14px, `bodySm`=12px

### Colors (Semantic)
- Backgrounds: `bg`=#F1F1F1, `bg-surface`=#FFF, `bg-surface-secondary`=#F7F7F7
- Brand fill: `bg-fill-brand`=#303030
- Status: `bg-fill-success`=#047B5D, `bg-fill-warning`=#FFB800, `bg-fill-critical`=#C70A24
- Text: `text`=#303030, `text-secondary`=#616161, `text-link`=#005BD3
- Borders: `border`=#E3E3E3, `border-focus`=#005BD3

### Breakpoints
`sm`=490px, `md`=768px, `lg`=1040px, `xl`=1440px

### Shadows
`shadow-100`: subtle card, `shadow-200`: raised, `shadow-300`: popover, `shadow-400`: modal

For complete component inventory and design token reference, read:
- [references/polaris-reference.md](references/polaris-reference.md)

## Key Design Principles

1. **Merchant-centered** - Prioritize merchant needs over aesthetic differentiation
2. **Consistency** - Match Shopify admin appearance and behavior exactly
3. **Monochromatic base** - Black/white foundation; color for status/emphasis only
4. **Mobile-first** - All layouts stack on small screens
5. **Accessibility** - AA contrast, keyboard nav, screen reader support
6. **No custom styling** - Use Polaris components and tokens, not custom CSS

## Decision Guide: Which Component?

| Need | Use |
|------|-----|
| Transient success message | `shopify.toast.show()` |
| Persistent info/warning/error | `<Banner>` |
| Blocking confirmation | `<s-modal>` (App Bridge) |
| Status label | `<Badge>` |
| Data list with selection | `<IndexTable>` + `<IndexFilters>` |
| Simple data display | `<DataTable>` |
| Form fields | `<FormLayout>` + `<TextField>/<Select>/<Checkbox>` |
| Settings page | `<InlineGrid>` 2fr/5fr + `<Card>` |
| Save/discard | `data-save-bar` form attribute |
| Sidebar nav | `<NavMenu>` (App Bridge React) |
| Page title/actions | `<Page>` or `<TitleBar>` |
| Dropdown actions | `<Popover>` + `<ActionList>` |
| Loading placeholder | `<SkeletonPage>` + `<SkeletonBodyText>` |
| No data state | `<EmptyState>` |
| File upload | `<DropZone>` |
| Date selection | `<DatePicker>` |
