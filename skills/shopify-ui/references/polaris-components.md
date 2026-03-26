# Polaris Component Inventory

Complete reference of all Polaris React components organized by category.

## Actions

### Button
Primary interaction element.
```tsx
<Button variant="primary" onClick={handleClick}>Save</Button>
<Button variant="plain" onClick={handleCancel}>Cancel</Button>
<Button tone="critical" onClick={handleDelete}>Delete</Button>
<Button loading disabled={isSubmitting}>Processing</Button>
<Button url="/app/settings" target="_blank">Open settings</Button>
<Button icon={PlusIcon}>Add item</Button>
<Button submit>Submit form</Button>
<Button disclosure onClick={togglePopover}>Actions</Button>
<Button size="slim">Compact</Button>
<Button size="large">Large action</Button>
```

Props: `variant` (primary | plain | monochromePlain | tertiary), `tone` (critical | success), `size` (slim | medium | large), `loading`, `disabled`, `submit`, `url`, `external`, `icon`, `disclosure`, `fullWidth`, `textAlign`, `accessibilityLabel`, `onClick`

### ButtonGroup
Groups related buttons.
```tsx
<ButtonGroup>
  <Button>Cancel</Button>
  <Button variant="primary">Save</Button>
</ButtonGroup>
<ButtonGroup gap="tight">...</ButtonGroup>
<ButtonGroup connectedTop>...</ButtonGroup>
```

Props: `gap` (tight | loose), `connectedTop`, `fullWidth`, `noWrap`

### ActionList
List of actions in a popover or sheet.
```tsx
<ActionList
  items={[
    { content: "Import", icon: ImportIcon, onAction: handleImport },
    { content: "Export", icon: ExportIcon, onAction: handleExport },
    { content: "Delete", destructive: true, onAction: handleDelete },
  ]}
  sections={[
    { title: "File actions", items: [...] },
    { title: "Bulk actions", items: [...] },
  ]}
/>
```

Props: `items` (array of ActionListItemDescriptor), `sections`, `onActionAnyItem`

### PageActions
Bottom-of-page actions for resource pages.
```tsx
<PageActions
  primaryAction={{ content: "Save", onAction: handleSave, loading: isSaving }}
  secondaryActions={[{ content: "Delete", destructive: true, onAction: handleDelete }]}
/>
```

## Layout

### Page
Top-level page wrapper with title, actions, breadcrumbs.
```tsx
<Page
  title="Products"
  subtitle="Manage your product catalog"
  backAction={{ content: "Home", url: "/app" }}
  primaryAction={{ content: "Add product", url: "/app/products/new" }}
  secondaryActions={[
    { content: "Import", onAction: handleImport },
    { content: "Export", onAction: handleExport },
  ]}
  actionGroups={[{
    title: "More actions",
    actions: [{ content: "Archive" }, { content: "Duplicate" }],
  }]}
  pagination={{ hasPrevious: true, hasNext: true, onPrevious, onNext }}
  fullWidth
  compactTitle
>
  {children}
</Page>
```

Props: `title`, `subtitle`, `backAction`, `primaryAction`, `secondaryActions`, `actionGroups`, `pagination`, `fullWidth`, `narrowWidth`, `compactTitle`, `additionalMetadata`, `titleMetadata`

### Layout
Two-column and annotated layouts.
```tsx
<Layout>
  <Layout.Section>
    <Card>Main content</Card>
  </Layout.Section>
  <Layout.Section variant="oneThird">
    <Card>Sidebar</Card>
  </Layout.Section>
</Layout>

<Layout>
  <Layout.AnnotatedSection
    title="Account details"
    description="Update your account information"
  >
    <Card>Form content</Card>
  </Layout.AnnotatedSection>
</Layout>
```

Layout.Section props: `variant` (oneHalf | oneThird | fullWidth)

### Card
Container for grouped content.
```tsx
<Card>Content</Card>
<Card padding="0">No padding (for tables)</Card>
<Card roundedAbove="sm">Rounded on small screens and up</Card>
```

Props: `padding` (space token or "0"), `roundedAbove` (xs | sm | md | lg | xl), `background`

### Box
Generic container with design token props.
```tsx
<Box padding="400" background="bg-surface-secondary" borderRadius="200">
  Content
</Box>
<Box as="section" paddingInlineStart="400" paddingInlineEnd="400">
  Content
</Box>
```

Props: `as`, `padding`, `paddingBlock`, `paddingInline`, `paddingBlockStart/End`, `paddingInlineStart/End`, `background`, `borderColor`, `borderWidth`, `borderRadius`, `color`, `minHeight`, `minWidth`, `maxWidth`, `overflowX/Y`, `shadow`, `width`, `role`, `id`

### BlockStack
Vertical stack with gap.
```tsx
<BlockStack gap="400" align="center" inlineAlign="start">
  <Text>Item 1</Text>
  <Text>Item 2</Text>
</BlockStack>
```

Props: `gap` (space token), `align` (start | center | end | space-around | space-between | space-evenly), `inlineAlign` (start | center | end | baseline | stretch), `as`

### InlineStack
Horizontal stack with gap and wrapping.
```tsx
<InlineStack gap="400" align="center" blockAlign="center" wrap={true}>
  <Badge>Active</Badge>
  <Text>Product name</Text>
</InlineStack>
```

Props: `gap`, `align`, `blockAlign` (start | center | end | baseline | stretch), `wrap`, `as`

### InlineGrid
CSS grid for responsive layouts.
```tsx
<InlineGrid columns={{ xs: 1, sm: 2, md: 3, lg: 4 }} gap="400">
  <Card>1</Card>
  <Card>2</Card>
  <Card>3</Card>
  <Card>4</Card>
</InlineGrid>

{/* Settings layout pattern */}
<InlineGrid columns={{ xs: "1fr", md: "2fr 5fr" }} gap="400">
  <Box as="section">Label area</Box>
  <Card>Form area</Card>
</InlineGrid>
```

Props: `columns` (number | string | responsive object), `gap`, `alignItems`

### Divider
Horizontal separator.
```tsx
<Divider />
<Divider borderColor="border-brand" borderWidth="050" />
```

### Bleed
Content that bleeds outside parent padding.
```tsx
<Card>
  <Bleed marginInline="400">
    {/* Full-width content inside card */}
    <Box background="bg-surface-secondary" padding="400">
      Banner area
    </Box>
  </Bleed>
</Card>
```

Props: `marginInline`, `marginBlock`, `marginBlockStart/End`, `marginInlineStart/End`

## Typography

### Text
All text rendering.
```tsx
<Text as="h1" variant="headingXl">Page Title</Text>
<Text as="h2" variant="headingLg">Section Title</Text>
<Text as="h3" variant="headingMd">Card Title</Text>
<Text as="p" variant="bodyMd">Body text</Text>
<Text as="p" variant="bodySm" tone="subdued">Secondary text</Text>
<Text as="span" fontWeight="bold">Bold text</Text>
<Text as="p" variant="bodyMd" tone="critical">Error text</Text>
<Text as="p" variant="bodyMd" tone="success">Success text</Text>
<Text as="p" alignment="center">Centered text</Text>
<Text as="span" numeric>1,234.56</Text>
<Text as="p" breakWord>very-long-url-that-should-break.example.com</Text>
<Text as="p" truncate>This text will be truncated with ellipsis if too long</Text>
```

Props: `as` (required), `variant` (headingXl | headingLg | headingMd | headingSm | headingXs | bodyLg | bodyMd | bodySm), `tone` (subdued | disabled | success | critical | caution | magic | textInverse), `fontWeight` (regular | medium | semibold | bold), `alignment` (start | center | end | justify), `numeric`, `breakWord`, `truncate`, `visuallyHidden`, `textDecorationLine`

## Forms

### TextField
Text input.
```tsx
<TextField
  label="Product title"
  value={title}
  onChange={setTitle}
  error="Title is required"
  helpText="Enter a descriptive title"
  placeholder="e.g., Blue T-Shirt"
  autoComplete="off"
  multiline={4}
  maxLength={100}
  showCharacterCount
  prefix="$"
  suffix="USD"
  type="number"
  min={0}
  max={999}
  step={0.01}
  clearButton
  onClearButtonClick={() => setTitle("")}
  connectedRight={<Button>Search</Button>}
  disabled
  readOnly
  requiredIndicator
  monospaced
/>
```

Props: `label`, `value`, `onChange`, `error`, `helpText`, `placeholder`, `type` (text | email | number | password | search | tel | url | date | datetime-local | month | week | time | integer | currency), `multiline`, `maxLength`, `showCharacterCount`, `prefix`, `suffix`, `connectedLeft/Right`, `clearButton`, `disabled`, `readOnly`, `autoComplete`, `requiredIndicator`, `monospaced`, `min`, `max`, `step`, `autoSize`, `focused`, `selectTextOnFocus`

### Select
Dropdown select.
```tsx
<Select
  label="Country"
  options={[
    { label: "United States", value: "US" },
    { label: "Canada", value: "CA" },
    { label: "---", value: "", disabled: true },
    { label: "Other", value: "other" },
  ]}
  value={country}
  onChange={setCountry}
  error="Please select a country"
  helpText="Select your country"
  disabled
  requiredIndicator
/>
```

### Checkbox
```tsx
<Checkbox
  label="Enable notifications"
  checked={notify}
  onChange={setNotify}
  helpText="Receive email notifications"
  error="Must accept terms"
  disabled
/>
```

### RadioButton
```tsx
<RadioButton label="Standard" checked={plan === "standard"} id="standard" name="plan" onChange={() => setPlan("standard")} />
<RadioButton label="Premium" checked={plan === "premium"} id="premium" name="plan" onChange={() => setPlan("premium")} />
```

### ChoiceList
Group of radio buttons or checkboxes.
```tsx
<ChoiceList
  title="Notification preferences"
  choices={[
    { label: "Email", value: "email" },
    { label: "SMS", value: "sms" },
    { label: "Push", value: "push" },
  ]}
  selected={selectedPrefs}
  onChange={setSelectedPrefs}
  allowMultiple
/>
```

### RangeSlider
```tsx
<RangeSlider label="Budget" value={budget} onChange={setBudget} min={0} max={1000} step={50} prefix="$" suffix="/month" output />
```

### FormLayout
Arranges form fields with proper spacing.
```tsx
<FormLayout>
  <TextField label="First name" />
  <TextField label="Last name" />
  <FormLayout.Group>
    <TextField label="City" />
    <Select label="Province" options={provinces} />
    <TextField label="Postal code" />
  </FormLayout.Group>
  <FormLayout.Group condensed>
    <TextField label="Price" prefix="$" />
    <TextField label="Compare at" prefix="$" />
  </FormLayout.Group>
</FormLayout>
```

### Form
Form wrapper with submit handler.
```tsx
<Form onSubmit={handleSubmit} preventDefault>
  <FormLayout>...</FormLayout>
</Form>
```

### Tag
Removable tags for multi-select.
```tsx
<InlineStack gap="200">
  {tags.map(tag => (
    <Tag key={tag} onRemove={() => removeTag(tag)}>{tag}</Tag>
  ))}
</InlineStack>
```

### Autocomplete
```tsx
<Autocomplete
  options={options}
  selected={selected}
  onSelect={setSelected}
  textField={<Autocomplete.TextField label="Tags" value={inputValue} onChange={setInputValue} />}
/>
```

### DropZone
File upload area.
```tsx
<DropZone onDrop={handleDrop} accept="image/*" type="image" allowMultiple>
  <DropZone.FileUpload actionTitle="Add image" actionHint="or drop files to upload" />
</DropZone>
```

### DatePicker
```tsx
<DatePicker month={month} year={year} onChange={setSelectedDates} onMonthChange={handleMonthChange} selected={selectedDates} allowRange multiMonth />
```

### ColorPicker
```tsx
<ColorPicker onChange={setColor} color={color} allowAlpha />
```

## Data Display

### IndexTable
Resource list with selection, sorting, and bulk actions.
```tsx
<IndexTable
  resourceName={{ singular: "order", plural: "orders" }}
  itemCount={orders.length}
  selectedItemsCount={allSelected ? "All" : selectedResources.length}
  onSelectionChange={handleSelectionChange}
  headings={[
    { title: "Order" },
    { title: "Date" },
    { title: "Customer" },
    { title: "Total", alignment: "end" },
    { title: "Status" },
  ]}
  promotedBulkActions={[
    { content: "Fulfill", onAction: handleBulkFulfill },
  ]}
  bulkActions={[
    { content: "Archive", onAction: handleBulkArchive },
    { content: "Delete", destructive: true, onAction: handleBulkDelete },
  ]}
  sortable={[true, true, false, true, false]}
  sortDirection={sortDirection}
  sortColumnIndex={sortColumnIndex}
  onSort={handleSort}
  pagination={{ hasNext: true, hasPrevious: true, onNext, onPrevious }}
  lastColumnSticky
  hasZebraStriping
>
  {orders.map((order, index) => (
    <IndexTable.Row id={order.id} key={order.id} selected={selectedResources.includes(order.id)} position={index}>
      <IndexTable.Cell><Text fontWeight="bold">{order.name}</Text></IndexTable.Cell>
      <IndexTable.Cell>{order.date}</IndexTable.Cell>
      <IndexTable.Cell>{order.customer}</IndexTable.Cell>
      <IndexTable.Cell><Text alignment="end" numeric>{order.total}</Text></IndexTable.Cell>
      <IndexTable.Cell><Badge tone={order.statusTone}>{order.status}</Badge></IndexTable.Cell>
    </IndexTable.Row>
  ))}
</IndexTable>
```

### IndexFilters
Filtering and search bar for IndexTable.
```tsx
<IndexFilters
  queryValue={queryValue}
  queryPlaceholder="Search orders"
  onQueryChange={setQueryValue}
  onQueryClear={() => setQueryValue("")}
  tabs={[
    { id: "all", content: "All", accessibilityLabel: "All orders" },
    { id: "active", content: "Active" },
    { id: "archived", content: "Archived" },
  ]}
  selected={selectedTab}
  onSelect={setSelectedTab}
  canCreateNewView
  onCreateNewView={handleCreateView}
  filters={[
    {
      key: "status",
      label: "Status",
      filter: <ChoiceList title="Status" choices={statusOptions} selected={statusFilter} onChange={setStatusFilter} allowMultiple />,
      shortcut: true,
    },
    {
      key: "date",
      label: "Date range",
      filter: <DatePicker {...dateProps} />,
    },
  ]}
  appliedFilters={appliedFilters}
  onClearAll={handleClearAll}
  sortOptions={[
    { label: "Date", value: "date asc", directionLabel: "Oldest first" },
    { label: "Date", value: "date desc", directionLabel: "Newest first" },
    { label: "Total", value: "total asc", directionLabel: "Lowest first" },
    { label: "Total", value: "total desc", directionLabel: "Highest first" },
  ]}
  sortSelected={sortSelected}
  onSort={setSortSelected}
  mode={mode}
  setMode={setMode}
  loading={isLoading}
/>
```

### DataTable
Simple read-only data table.
```tsx
<DataTable
  columnContentTypes={["text", "text", "numeric", "numeric", "numeric"]}
  headings={["Product", "SKU", "Qty", "Price", "Total"]}
  rows={[
    ["Blue T-Shirt", "BTS-001", 150, "$25.00", "$3,750.00"],
    ["Red Hat", "RH-002", 75, "$15.00", "$1,125.00"],
  ]}
  totals={["", "", 225, "", "$4,875.00"]}
  sortable={[true, false, true, true, true]}
  defaultSortDirection="descending"
  initialSortColumnIndex={4}
  onSort={handleSort}
  footerContent="Showing 2 of 100 products"
  hasZebraStripingOnData
  increasedTableDensity
  truncate
/>
```

### ResourceList
Alternative to IndexTable (less common, use IndexTable for most cases).
```tsx
<ResourceList
  resourceName={{ singular: "customer", plural: "customers" }}
  items={customers}
  renderItem={(item) => (
    <ResourceItem id={item.id} url={`/app/customers/${item.id}`} media={<Avatar customer name={item.name} />}>
      <Text variant="bodyMd" fontWeight="bold">{item.name}</Text>
      <div>{item.email}</div>
    </ResourceItem>
  )}
/>
```

### DescriptionList
Key-value pairs.
```tsx
<DescriptionList
  items={[
    { term: "Order number", description: "#1001" },
    { term: "Date", description: "January 15, 2024" },
    { term: "Status", description: <Badge tone="success">Fulfilled</Badge> },
  ]}
/>
```

## Feedback

### Banner
Persistent feedback message.
```tsx
<Banner title="Setup complete" tone="success" onDismiss={() => setShowBanner(false)}>
  <p>Your store is now connected.</p>
</Banner>
<Banner title="Missing information" tone="warning">
  <List>
    <List.Item>Add a shipping address</List.Item>
    <List.Item>Configure tax settings</List.Item>
  </List>
</Banner>
<Banner title="Payment failed" tone="critical" action={{ content: "Retry", onAction: handleRetry }}>
  <p>We could not process the payment.</p>
</Banner>
<Banner tone="info">
  <p>A new feature is available.</p>
</Banner>
```

Props: `title`, `tone` (info | success | warning | critical), `onDismiss`, `action`, `secondaryAction`, `icon`, `hideIcon`, `stopAnnouncements`

### Badge
Status indicator.
```tsx
<Badge tone="success">Active</Badge>
<Badge tone="warning">Pending</Badge>
<Badge tone="critical">Error</Badge>
<Badge tone="info">Draft</Badge>
<Badge tone="attention">Action needed</Badge>
<Badge progress="complete">Fulfilled</Badge>
<Badge progress="partiallyComplete">Partially fulfilled</Badge>
<Badge progress="incomplete">Unfulfilled</Badge>
<Badge size="small">New</Badge>
```

Props: `tone` (info | success | warning | critical | attention | new | magic | enabled | read-only), `progress` (complete | partiallyComplete | incomplete), `size` (small | medium), `icon`

### Toast (App Bridge - NOT Polaris)
```typescript
shopify.toast.show("Changes saved");
shopify.toast.show("Error saving", { isError: true });
shopify.toast.show("Processing...", { duration: 5000 });
```

### ProgressBar
```tsx
<ProgressBar progress={75} tone="primary" size="small" />
<ProgressBar progress={100} tone="success" />
<ProgressBar animated />
```

### Spinner
```tsx
<Spinner size="small" />
<Spinner size="large" />
<Spinner accessibilityLabel="Loading products" />
```

## Navigation

### Link
```tsx
<Link url="/app/products" removeUnderline>Products</Link>
<Link url="https://help.shopify.com" external>Help center</Link>
<Link monochrome>Subtle link</Link>
```

### Tabs
```tsx
<Tabs tabs={[
  { id: "all", content: "All", panelID: "all-panel" },
  { id: "active", content: "Active", panelID: "active-panel" },
  { id: "draft", content: "Draft", panelID: "draft-panel", badge: "3" },
]} selected={selectedTab} onSelect={setSelectedTab}>
  <Card>{tabContent}</Card>
</Tabs>
```

### Pagination
```tsx
<Pagination
  hasPrevious={page > 1}
  hasNext={hasMore}
  onPrevious={() => setPage(p => p - 1)}
  onNext={() => setPage(p => p + 1)}
  label={`Page ${page} of ${totalPages}`}
/>
```

### Popover
```tsx
<Popover
  active={popoverActive}
  activator={<Button onClick={togglePopover} disclosure>More actions</Button>}
  onClose={togglePopover}
  autofocusTarget="first-node"
  preferredAlignment="right"
>
  <Popover.Pane>
    <ActionList items={[
      { content: "Import", icon: ImportIcon },
      { content: "Export", icon: ExportIcon },
    ]} />
  </Popover.Pane>
</Popover>
```

## Overlays

### Modal (App Bridge Web Component)
```html
<s-modal id="delete-confirm" variant="base">
  <s-text>Are you sure you want to delete this product?</s-text>
  <s-button slot="footer" onclick="handleDelete()" variant="primary" tone="critical">Delete</s-button>
  <s-button slot="footer" onclick="shopify.modal.hide('delete-confirm')">Cancel</s-button>
</s-modal>
```

Open programmatically:
```typescript
shopify.modal.show('delete-confirm');
```

### Tooltip
```tsx
<Tooltip content="Mark as fulfilled" dismissOnMouseOut>
  <Button icon={CheckIcon} />
</Tooltip>
```

## Images and Icons

### Avatar
```tsx
<Avatar customer name="John Doe" />
<Avatar size="lg" source="https://example.com/photo.jpg" />
<Avatar initials="JD" />
```

### Thumbnail
```tsx
<Thumbnail source="https://example.com/product.jpg" alt="Product image" size="small" />
```

Sizes: `extraSmall`, `small`, `medium`, `large`

### Icon
```tsx
import { SearchIcon, PlusIcon, DeleteIcon } from "@shopify/polaris-icons";
<Icon source={SearchIcon} tone="base" />
<Icon source={PlusIcon} tone="success" />
<Icon source={DeleteIcon} tone="critical" />
```

## Loading

### SkeletonPage
```tsx
<SkeletonPage primaryAction>
  <Layout>
    <Layout.Section>
      <Card>
        <SkeletonBodyText lines={3} />
      </Card>
    </Layout.Section>
    <Layout.Section variant="oneThird">
      <Card>
        <SkeletonDisplayText size="small" />
        <SkeletonBodyText lines={2} />
      </Card>
    </Layout.Section>
  </Layout>
</SkeletonPage>
```

### SkeletonBodyText
```tsx
<SkeletonBodyText lines={3} />
```

### SkeletonDisplayText
```tsx
<SkeletonDisplayText size="small" />
<SkeletonDisplayText size="medium" />
<SkeletonDisplayText size="large" />
```

### SkeletonThumbnail
```tsx
<SkeletonThumbnail size="small" />
```

## Utility

### EmptyState
```tsx
<EmptyState
  heading="No products yet"
  action={{ content: "Add product", url: "/app/products/new" }}
  secondaryAction={{ content: "Learn more", url: "https://help.shopify.com", external: true }}
  image="https://cdn.shopify.com/s/files/1/0262/4071/2726/files/emptystate-files.png"
  fullWidth
>
  <p>Add products to start managing your catalog.</p>
</EmptyState>
```

### AccountConnection
Show connection to external accounts.
```tsx
<AccountConnection
  connected={isConnected}
  title="Google Analytics"
  accountName={connectedAccount}
  action={{ content: isConnected ? "Disconnect" : "Connect", onAction: handleToggle }}
  details={isConnected ? "Connected as user@example.com" : "No account connected"}
  termsOfService={<p>By clicking Connect, you agree to the terms.</p>}
/>
```

### CalloutCard
Highlight a key feature or promotion.
```tsx
<CalloutCard
  title="Customize your checkout"
  illustration="https://cdn.shopify.com/..."
  primaryAction={{ content: "Customize", url: "/app/checkout" }}
  secondaryAction={{ content: "Learn more" }}
>
  <p>Upload your store logo and choose brand colors.</p>
</CalloutCard>
```

### MediaCard
```tsx
<MediaCard
  title="Getting started"
  primaryAction={{ content: "Watch video", onAction: handlePlay }}
  description="Learn how to set up your app in 5 minutes."
  size="small"
>
  <img src="thumbnail.jpg" alt="Tutorial video thumbnail" />
</MediaCard>
```

### SettingToggle
```tsx
<SettingToggle
  action={{ content: isEnabled ? "Turn off" : "Turn on", onAction: toggleSetting }}
  enabled={isEnabled}
>
  Automatic sync is <Text fontWeight="bold">{isEnabled ? "enabled" : "disabled"}</Text>.
</SettingToggle>
```

### List
```tsx
<List type="bullet">
  <List.Item>First item</List.Item>
  <List.Item>Second item</List.Item>
</List>
<List type="number">
  <List.Item>Step one</List.Item>
  <List.Item>Step two</List.Item>
</List>
```

### ExceptionList
```tsx
<ExceptionList
  items={[
    { icon: NoteIcon, description: "This product has no images" },
    { icon: AlertIcon, status: "critical", description: "Inventory is low" },
  ]}
/>
```

### KeyboardKey
```tsx
<KeyboardKey>Ctrl</KeyboardKey> + <KeyboardKey>S</KeyboardKey>
```

### Scrollable
```tsx
<Scrollable shadow style={{ height: "200px" }}>
  {longContent}
</Scrollable>
```

### Collapsible
```tsx
<Collapsible open={isOpen} id="details-collapsible" transition={{ duration: "500ms", timingFunction: "ease-in-out" }}>
  <TextContainer>
    <p>Additional details shown when expanded.</p>
  </TextContainer>
</Collapsible>
```

### Frame
Top-level app wrapper (usually set up once).
```tsx
<Frame
  logo={{ topBarSource: "/logo.svg", url: "/", accessibilityLabel: "My App" }}
  topBar={topBarMarkup}
  navigation={navigationMarkup}
  showMobileNavigation={showMobileNav}
  onNavigationDismiss={() => setShowMobileNav(false)}
>
  {pageContent}
</Frame>
```

Note: In embedded apps, Frame is less common since Shopify admin provides the outer frame. Use `<Page>` and `<NavMenu>` instead.
