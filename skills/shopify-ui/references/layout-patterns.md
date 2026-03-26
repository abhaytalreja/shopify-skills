# Layout Patterns Reference

Complete code examples for every common Shopify app layout pattern.

## Resource Index Page (List)

The most common page type. Shows a filterable, sortable, selectable list of resources.

```tsx
import { useState, useCallback } from "react";
import { Page, Card, IndexTable, IndexFilters, Text, Badge,
  useSetIndexFiltersMode, useIndexResourceState, InlineStack } from "@shopify/polaris";
import type { TabProps } from "@shopify/polaris";

interface Order {
  id: string;
  name: string;
  date: string;
  customer: string;
  total: string;
  status: string;
  statusTone: "success" | "warning" | "critical" | "info";
}

export default function OrdersIndex() {
  const [queryValue, setQueryValue] = useState("");
  const [selectedTab, setSelectedTab] = useState(0);
  const [sortSelected, setSortSelected] = useState(["date desc"]);
  const { mode, setMode } = useSetIndexFiltersMode();

  const orders: Order[] = [/* loaded from loader */];

  const resourceName = { singular: "order", plural: "orders" };
  const { selectedResources, allResourcesSelected, handleSelectionChange } =
    useIndexResourceState(orders);

  const tabs: TabProps[] = [
    { id: "all", content: "All", accessibilityLabel: "All orders" },
    { id: "unfulfilled", content: "Unfulfilled" },
    { id: "fulfilled", content: "Fulfilled" },
  ];

  const sortOptions = [
    { label: "Date", value: "date asc", directionLabel: "Oldest first" },
    { label: "Date", value: "date desc", directionLabel: "Newest first" },
    { label: "Total", value: "total asc", directionLabel: "Lowest first" },
    { label: "Total", value: "total desc", directionLabel: "Highest first" },
  ];

  const rowMarkup = orders.map((order, index) => (
    <IndexTable.Row
      id={order.id}
      key={order.id}
      selected={selectedResources.includes(order.id)}
      position={index}
    >
      <IndexTable.Cell>
        <Text variant="bodyMd" fontWeight="bold" as="span">{order.name}</Text>
      </IndexTable.Cell>
      <IndexTable.Cell>{order.date}</IndexTable.Cell>
      <IndexTable.Cell>{order.customer}</IndexTable.Cell>
      <IndexTable.Cell>
        <Text as="span" alignment="end" numeric>{order.total}</Text>
      </IndexTable.Cell>
      <IndexTable.Cell>
        <Badge tone={order.statusTone}>{order.status}</Badge>
      </IndexTable.Cell>
    </IndexTable.Row>
  ));

  return (
    <Page
      title="Orders"
      primaryAction={{ content: "Create order", url: "/app/orders/new" }}
    >
      <Card padding="0">
        <IndexFilters
          queryValue={queryValue}
          queryPlaceholder="Search orders"
          onQueryChange={setQueryValue}
          onQueryClear={() => setQueryValue("")}
          tabs={tabs}
          selected={selectedTab}
          onSelect={setSelectedTab}
          sortOptions={sortOptions}
          sortSelected={sortSelected}
          onSort={setSortSelected}
          mode={mode}
          setMode={setMode}
          filters={[]}
          appliedFilters={[]}
          onClearAll={() => {}}
        />
        <IndexTable
          resourceName={resourceName}
          itemCount={orders.length}
          selectedItemsCount={allResourcesSelected ? "All" : selectedResources.length}
          onSelectionChange={handleSelectionChange}
          headings={[
            { title: "Order" },
            { title: "Date" },
            { title: "Customer" },
            { title: "Total", alignment: "end" },
            { title: "Status" },
          ]}
          promotedBulkActions={[
            { content: "Fulfill selected", onAction: () => console.log("fulfill") },
          ]}
          bulkActions={[
            { content: "Archive", onAction: () => console.log("archive") },
          ]}
          pagination={{
            hasNext: true,
            hasPrevious: false,
            onNext: () => {},
          }}
        >
          {rowMarkup}
        </IndexTable>
      </Card>
    </Page>
  );
}
```

## Resource Detail Page

Shows a single resource with editable fields.

```tsx
import { useState, useCallback } from "react";
import { Page, Layout, Card, FormLayout, TextField, Select,
  BlockStack, InlineStack, Text, Badge, Thumbnail, Box,
  PageActions, Banner } from "@shopify/polaris";
import { TitleBar } from "@shopify/app-bridge-react";

export default function ProductDetail() {
  const [title, setTitle] = useState("Blue T-Shirt");
  const [description, setDescription] = useState("");
  const [status, setStatus] = useState("active");
  const [price, setPrice] = useState("29.99");
  const [compareAt, setCompareAt] = useState("");
  const [sku, setSku] = useState("BTS-001");
  const [isDirty, setIsDirty] = useState(false);

  return (
    <Page
      backAction={{ content: "Products", url: "/app/products" }}
      title="Blue T-Shirt"
      titleMetadata={<Badge tone="success">Active</Badge>}
      secondaryActions={[
        { content: "Duplicate" },
        { content: "Archive" },
      ]}
      pagination={{ hasPrevious: true, hasNext: true }}
    >
      <Layout>
        {/* Main content */}
        <Layout.Section>
          <Card>
            <BlockStack gap="400">
              <TextField
                label="Title"
                value={title}
                onChange={(v) => { setTitle(v); setIsDirty(true); }}
                autoComplete="off"
              />
              <TextField
                label="Description"
                value={description}
                onChange={(v) => { setDescription(v); setIsDirty(true); }}
                multiline={4}
                autoComplete="off"
              />
            </BlockStack>
          </Card>

          <Card>
            <BlockStack gap="400">
              <Text as="h2" variant="headingSm">Pricing</Text>
              <FormLayout>
                <FormLayout.Group>
                  <TextField
                    label="Price"
                    type="currency"
                    value={price}
                    onChange={(v) => { setPrice(v); setIsDirty(true); }}
                    prefix="$"
                    autoComplete="off"
                  />
                  <TextField
                    label="Compare at price"
                    type="currency"
                    value={compareAt}
                    onChange={(v) => { setCompareAt(v); setIsDirty(true); }}
                    prefix="$"
                    autoComplete="off"
                  />
                </FormLayout.Group>
              </FormLayout>
            </BlockStack>
          </Card>

          <Card>
            <BlockStack gap="400">
              <Text as="h2" variant="headingSm">Inventory</Text>
              <TextField
                label="SKU"
                value={sku}
                onChange={(v) => { setSku(v); setIsDirty(true); }}
                autoComplete="off"
              />
            </BlockStack>
          </Card>
        </Layout.Section>

        {/* Sidebar */}
        <Layout.Section variant="oneThird">
          <Card>
            <BlockStack gap="400">
              <Text as="h2" variant="headingSm">Status</Text>
              <Select
                label="Status"
                labelHidden
                options={[
                  { label: "Active", value: "active" },
                  { label: "Draft", value: "draft" },
                  { label: "Archived", value: "archived" },
                ]}
                value={status}
                onChange={(v) => { setStatus(v); setIsDirty(true); }}
              />
            </BlockStack>
          </Card>

          <Card>
            <BlockStack gap="400">
              <Text as="h2" variant="headingSm">Product image</Text>
              <Thumbnail
                source="https://example.com/product.jpg"
                alt="Product image"
                size="large"
              />
            </BlockStack>
          </Card>
        </Layout.Section>
      </Layout>

      <PageActions
        primaryAction={{
          content: "Save",
          disabled: !isDirty,
          loading: false,
          onAction: () => {},
        }}
        secondaryActions={[
          { content: "Delete", destructive: true, onAction: () => {} },
        ]}
      />
    </Page>
  );
}
```

## Settings Page (Annotated Layout)

Two-column layout with description on the left, fields on the right.

```tsx
import { useState } from "react";
import { Page, InlineGrid, Card, FormLayout, TextField, Select,
  Checkbox, BlockStack, Box, Text, Divider, PageActions } from "@shopify/polaris";

export default function Settings() {
  const [appName, setAppName] = useState("");
  const [email, setEmail] = useState("");
  const [frequency, setFrequency] = useState("daily");
  const [notifyEmail, setNotifyEmail] = useState(true);
  const [notifyPush, setNotifyPush] = useState(false);
  const [apiKey, setApiKey] = useState("");

  return (
    <Page
      title="Settings"
      backAction={{ content: "Home", url: "/app" }}
    >
      <BlockStack gap="800">
        {/* General Settings */}
        <InlineGrid columns={{ xs: "1fr", md: "2fr 5fr" }} gap="400">
          <Box as="section" paddingInlineStart={{ xs: "400", md: "0" }}>
            <BlockStack gap="400">
              <Text as="h3" variant="headingMd">General</Text>
              <Text as="p" variant="bodyMd" tone="subdued">
                Basic configuration for your app.
              </Text>
            </BlockStack>
          </Box>
          <Card>
            <FormLayout>
              <TextField
                label="App display name"
                value={appName}
                onChange={setAppName}
                autoComplete="off"
                helpText="Shown to merchants in the dashboard"
              />
              <TextField
                label="Contact email"
                type="email"
                value={email}
                onChange={setEmail}
                autoComplete="email"
              />
            </FormLayout>
          </Card>
        </InlineGrid>

        <Divider />

        {/* Notification Settings */}
        <InlineGrid columns={{ xs: "1fr", md: "2fr 5fr" }} gap="400">
          <Box as="section" paddingInlineStart={{ xs: "400", md: "0" }}>
            <BlockStack gap="400">
              <Text as="h3" variant="headingMd">Notifications</Text>
              <Text as="p" variant="bodyMd" tone="subdued">
                Configure how and when you receive alerts.
              </Text>
            </BlockStack>
          </Box>
          <Card>
            <FormLayout>
              <Select
                label="Frequency"
                options={[
                  { label: "Real-time", value: "realtime" },
                  { label: "Daily digest", value: "daily" },
                  { label: "Weekly digest", value: "weekly" },
                ]}
                value={frequency}
                onChange={setFrequency}
              />
              <Checkbox
                label="Email notifications"
                checked={notifyEmail}
                onChange={setNotifyEmail}
              />
              <Checkbox
                label="Push notifications"
                checked={notifyPush}
                onChange={setNotifyPush}
              />
            </FormLayout>
          </Card>
        </InlineGrid>

        <Divider />

        {/* API Settings */}
        <InlineGrid columns={{ xs: "1fr", md: "2fr 5fr" }} gap="400">
          <Box as="section" paddingInlineStart={{ xs: "400", md: "0" }}>
            <BlockStack gap="400">
              <Text as="h3" variant="headingMd">API Configuration</Text>
              <Text as="p" variant="bodyMd" tone="subdued">
                Connect to external services.
              </Text>
            </BlockStack>
          </Box>
          <Card>
            <FormLayout>
              <TextField
                label="API Key"
                value={apiKey}
                onChange={setApiKey}
                autoComplete="off"
                helpText="Find this in your service dashboard"
              />
            </FormLayout>
          </Card>
        </InlineGrid>
      </BlockStack>

      <Box paddingBlockStart="800">
        <PageActions
          primaryAction={{ content: "Save", onAction: () => {} }}
          secondaryActions={[{ content: "Discard", onAction: () => {} }]}
        />
      </Box>
    </Page>
  );
}
```

## Dashboard Page

Overview page with metric cards and activity feed.

```tsx
import { Page, BlockStack, InlineGrid, Card, Text, DataTable,
  InlineStack, Badge, Box, Divider } from "@shopify/polaris";

export default function Dashboard() {
  const metrics = [
    { label: "Total Revenue", value: "$12,430", change: "+14%", positive: true },
    { label: "Orders", value: "284", change: "+8%", positive: true },
    { label: "Avg. Order Value", value: "$43.77", change: "-2%", positive: false },
    { label: "Active Customers", value: "1,203", change: "+22%", positive: true },
  ];

  const recentActivity = [
    ["Jan 15", "Order #1001 fulfilled", "$84.50"],
    ["Jan 15", "New customer registered", "-"],
    ["Jan 14", "Product inventory low", "Widget Pro"],
    ["Jan 14", "Order #999 created", "$129.00"],
  ];

  return (
    <Page title="Dashboard" subtitle="Performance overview - Last 30 days">
      <BlockStack gap="400">
        {/* Metric Cards */}
        <InlineGrid columns={{ xs: 1, sm: 2, md: 4 }} gap="400">
          {metrics.map((metric) => (
            <Card key={metric.label}>
              <BlockStack gap="200">
                <Text variant="bodyMd" tone="subdued" as="p">{metric.label}</Text>
                <InlineStack gap="200" blockAlign="baseline">
                  <Text variant="headingXl" as="p">{metric.value}</Text>
                  <Badge tone={metric.positive ? "success" : "critical"}>
                    {metric.change}
                  </Badge>
                </InlineStack>
              </BlockStack>
            </Card>
          ))}
        </InlineGrid>

        {/* Two column layout */}
        <InlineGrid columns={{ xs: 1, md: "2fr 1fr" }} gap="400">
          {/* Activity table */}
          <Card>
            <BlockStack gap="400">
              <Text as="h2" variant="headingMd">Recent Activity</Text>
              <DataTable
                columnContentTypes={["text", "text", "numeric"]}
                headings={["Date", "Event", "Value"]}
                rows={recentActivity}
                footerContent="Showing 4 of 50 events"
              />
            </BlockStack>
          </Card>

          {/* Quick stats sidebar */}
          <Card>
            <BlockStack gap="400">
              <Text as="h2" variant="headingMd">Quick Stats</Text>
              <Divider />
              <BlockStack gap="300">
                <InlineStack align="space-between">
                  <Text tone="subdued" as="span">Products</Text>
                  <Text fontWeight="bold" as="span">142</Text>
                </InlineStack>
                <InlineStack align="space-between">
                  <Text tone="subdued" as="span">Collections</Text>
                  <Text fontWeight="bold" as="span">12</Text>
                </InlineStack>
                <InlineStack align="space-between">
                  <Text tone="subdued" as="span">Pending orders</Text>
                  <Text fontWeight="bold" as="span">8</Text>
                </InlineStack>
                <InlineStack align="space-between">
                  <Text tone="subdued" as="span">Low stock items</Text>
                  <Badge tone="warning">3</Badge>
                </InlineStack>
              </BlockStack>
            </BlockStack>
          </Card>
        </InlineGrid>
      </BlockStack>
    </Page>
  );
}
```

## Onboarding / Setup Page

Guide new users through initial setup steps.

```tsx
import { Page, Card, BlockStack, InlineStack, Text, Badge,
  Button, Box, Divider, Banner, Icon } from "@shopify/polaris";
import { CheckIcon, ClockIcon } from "@shopify/polaris-icons";

interface SetupStep {
  title: string;
  description: string;
  completed: boolean;
  action: { content: string; url: string };
}

export default function Onboarding() {
  const steps: SetupStep[] = [
    {
      title: "Connect your store",
      description: "Authorize access to your Shopify store data.",
      completed: true,
      action: { content: "Connected", url: "#" },
    },
    {
      title: "Configure settings",
      description: "Set up your preferences and notification rules.",
      completed: false,
      action: { content: "Configure", url: "/app/settings" },
    },
    {
      title: "Import products",
      description: "Sync your product catalog for analysis.",
      completed: false,
      action: { content: "Import", url: "/app/import" },
    },
  ];

  const completedCount = steps.filter(s => s.completed).length;

  return (
    <Page title="Get started" narrowWidth>
      <BlockStack gap="400">
        <Banner tone="info">
          <p>Complete these steps to start using the app. {completedCount} of {steps.length} complete.</p>
        </Banner>

        <Card>
          <BlockStack gap="0">
            {steps.map((step, index) => (
              <Box key={step.title}>
                {index > 0 && <Divider />}
                <Box padding="400">
                  <InlineStack align="space-between" blockAlign="center">
                    <InlineStack gap="400" blockAlign="center">
                      <Box>
                        {step.completed ? (
                          <Icon source={CheckIcon} tone="success" />
                        ) : (
                          <Icon source={ClockIcon} tone="subdued" />
                        )}
                      </Box>
                      <BlockStack gap="100">
                        <Text as="h3" variant="headingSm">{step.title}</Text>
                        <Text as="p" variant="bodySm" tone="subdued">{step.description}</Text>
                      </BlockStack>
                    </InlineStack>
                    <Button
                      variant={step.completed ? "plain" : "primary"}
                      disabled={step.completed}
                      url={step.action.url}
                    >
                      {step.action.content}
                    </Button>
                  </InlineStack>
                </Box>
              </Box>
            ))}
          </BlockStack>
        </Card>
      </BlockStack>
    </Page>
  );
}
```

## Empty State Page

Shown when a resource has no data yet.

```tsx
import { Page, Card, EmptyState } from "@shopify/polaris";

export default function ProductsEmpty() {
  return (
    <Page title="Products">
      <Card>
        <EmptyState
          heading="Add your first product"
          action={{ content: "Add product", url: "/app/products/new" }}
          secondaryAction={{ content: "Import from CSV", onAction: handleImport }}
          image="https://cdn.shopify.com/s/files/1/0262/4071/2726/files/emptystate-files.png"
        >
          <p>Track inventory, manage pricing, and organize your catalog.</p>
        </EmptyState>
      </Card>
    </Page>
  );
}
```

## Skeleton Loading Page

Show while data is loading.

```tsx
import { SkeletonPage, Layout, Card, SkeletonBodyText,
  SkeletonDisplayText, BlockStack, InlineGrid } from "@shopify/polaris";

export default function LoadingState() {
  return (
    <SkeletonPage primaryAction>
      <Layout>
        <Layout.Section>
          <Card>
            <BlockStack gap="400">
              <SkeletonDisplayText size="small" />
              <SkeletonBodyText lines={3} />
            </BlockStack>
          </Card>
          <Card>
            <BlockStack gap="400">
              <SkeletonDisplayText size="small" />
              <SkeletonBodyText lines={2} />
            </BlockStack>
          </Card>
        </Layout.Section>
        <Layout.Section variant="oneThird">
          <Card>
            <BlockStack gap="400">
              <SkeletonDisplayText size="small" />
              <SkeletonBodyText lines={2} />
            </BlockStack>
          </Card>
        </Layout.Section>
      </Layout>
    </SkeletonPage>
  );
}
```

## Full Page with Navigation (App Layout Route)

The layout route that wraps all authenticated pages.

```tsx
// app/routes/app.tsx
import type { LoaderFunctionArgs } from "react-router";
import { Outlet } from "react-router";
import { NavMenu } from "@shopify/app-bridge-react";
import { AppProvider } from "@shopify/polaris";
import "@shopify/polaris/build/esm/styles.css";
import { authenticate } from "../shopify.server";

export const loader = async ({ request }: LoaderFunctionArgs) => {
  await authenticate.admin(request);
  return null;
};

export default function AppLayout() {
  return (
    <AppProvider i18n={{}}>
      <NavMenu>
        <a href="/app" rel="home">Dashboard</a>
        <a href="/app/products">Products</a>
        <a href="/app/orders">Orders</a>
        <a href="/app/analytics">Analytics</a>
        <a href="/app/settings">Settings</a>
      </NavMenu>
      <Outlet />
    </AppProvider>
  );
}
```

## Modal Confirmation Pattern

Destructive action confirmation using App Bridge modal.

```tsx
import { Page, Card, Button, BlockStack, Text } from "@shopify/polaris";

export default function ConfirmationExample() {
  const handleDelete = async () => {
    // Perform delete action
    shopify.toast.show("Product deleted");
    shopify.modal.hide("delete-confirm");
  };

  return (
    <Page title="Product">
      <Card>
        <BlockStack gap="400">
          <Text as="p">Product details here...</Text>
          <Button
            tone="critical"
            onClick={() => shopify.modal.show("delete-confirm")}
          >
            Delete product
          </Button>
        </BlockStack>
      </Card>

      {/* App Bridge Modal (web component) */}
      <s-modal id="delete-confirm">
        <s-box padding="base">
          <s-text>Are you sure you want to delete this product? This action cannot be undone.</s-text>
        </s-box>
        <s-button slot="footer" variant="primary" tone="critical" onclick="handleDelete()">
          Delete
        </s-button>
        <s-button slot="footer" onclick="shopify.modal.hide('delete-confirm')">
          Cancel
        </s-button>
      </s-modal>
    </Page>
  );
}
```

## Form with Save Bar Pattern

Automatic unsaved changes detection.

```tsx
import { Page, Card, FormLayout, TextField, BlockStack } from "@shopify/polaris";
import { useFetcher } from "react-router";

export default function SettingsWithSaveBar() {
  const fetcher = useFetcher();

  return (
    <Page title="Settings" backAction={{ content: "Home", url: "/app" }}>
      {/* data-save-bar auto-detects form changes and shows save/discard bar */}
      <fetcher.Form method="post" data-save-bar data-discard-confirmation>
        <Card>
          <FormLayout>
            <TextField label="Store name" name="storeName" autoComplete="off" />
            <TextField label="Email" name="email" type="email" autoComplete="email" />
          </FormLayout>
        </Card>
        {/* No explicit save button needed - save bar appears on change */}
      </fetcher.Form>
    </Page>
  );
}
```
