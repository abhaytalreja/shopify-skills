# Polaris Component Reference - Complete Inventory

## Actions
- **Button** - `variant`: primary/plain/monochrome. `tone`: critical. `size`, `loading`, `disabled`, `submit`, `url`, `icon`
- **ButtonGroup** - Groups buttons horizontally. `gap`, `connectedTop`
- **AccountConnection** - Connect/disconnect external accounts

## Layout & Structure
- **Page** - `title`, `subtitle`, `titleMetadata`, `primaryAction`, `secondaryActions`, `actionGroups`, `pagination`, `backAction`, `fullWidth`, `narrowWidth`, `compactTitle`
- **Layout** - `Layout.Section` (full/two-thirds), `Layout.Section variant="oneThird"`, `Layout.AnnotatedSection`
- **Card** - Groups content. Use `BlockStack` + `Box padding` inside (no `sectioned` prop)
- **Box** - Primitive for design tokens: `padding`, `background`, `borderColor`, `borderRadius`, `shadow`
- **BlockStack** - Vertical stack. `gap`: space-100 to space-800
- **InlineStack** - Horizontal row. `gap`, `align`, `blockAlign`, `wrap`
- **InlineGrid** - Columns: `columns={{ xs: '1fr', md: '2fr 5fr' }}`, `gap`
- **Grid** - CSS Grid layouts
- **Bleed** - Negative margin for edge-to-edge content
- **Divider** - Visual separator
- **FormLayout** - Form field layout. `FormLayout.Group` for horizontal, `condensed`
- **EmptyState** - `heading`, `image`, `action`, `secondaryAction`, `footerContent`
- **MediaCard** - Info card with media
- **CalloutCard** - Feature promotion card

## Selection & Input
- **TextField** - `label`, `type`, `value`, `error`, `helpText`, `prefix`, `suffix`, `multiline`, `connectedLeft/Right`, `showCharacterCount`, `maxLength`, `clearButton`, `autoComplete`, `monospaced`, `loading`
- **Select** - Dropdown. `label`, `options`, `value`, `error`, `helpText`
- **Checkbox** - `label`, `checked`, `error`, `helpText`
- **RadioButton** - `label`, `checked`, `id`, `name`
- **ChoiceList** - Grouped radio/checkbox. `title`, `choices`, `selected`, `allowMultiple`
- **Autocomplete** - Input with suggestions
- **Combobox** - Accessible autocomplete with filtering
- **DatePicker** - Calendar picker. `month`, `year`, `selected`, `onMonthChange`
- **DropZone** - Drag-and-drop upload. `accept`, `allowMultiple`, `type`
- **ColorPicker** - Color selection
- **RangeSlider** - Numeric range
- **Tag** - Interactive keyword labels
- **Form** - `onSubmit`, `noValidate`, `implicitSubmit`
- **Filters** - Composite filtering
- **IndexFilters** - Search/sort/filter bar with saved views, mode management
- **InlineError** - Brief field-level errors

## Feedback
- **Banner** - `title`, `tone` (success/info/warning/critical), `action`, `secondaryAction`, `onDismiss`, `hideIcon`
- **Badge** - `tone`: success, info, warning, critical, attention, new. `progress`: incomplete, partiallyComplete, complete
- **ProgressBar** - `progress` (0-100), `tone`, `size`
- **Spinner** - `size`: small/large

## Typography
- **Text** - `variant`: headingXl/Lg/Md/Sm, bodyLg/Md/Sm. `as`: h1-h6/p/span. `fontWeight`: regular/medium/semibold/bold. `alignment`: start/center/end. `tone`: subdued/success/critical/caution. `truncate`, `breakWord`, `numeric`

## Tables
- **IndexTable** - `headings`, `itemCount`, `resourceName`, `selectedItemsCount`, `onSelectionChange`, `sortable`, `sortDirection`, `sortColumnIndex`, `onSort`, `pagination`, `loading`, `promotedBulkActions`, `bulkActions`, `hasZebraStriping`, `condensed`, `lastColumnSticky`. Hook: `useIndexResourceState`
- **DataTable** - `headings`, `rows`, `columnContentTypes`, `sortable`, `totals`, `showTotalsInFooter`

## Lists
- **ActionList** - Selectable options (in Popover). `items`, `sections`
- **ResourceList** / **ResourceItem** - Object collections
- **List** - Text-only. `type`: bullet/number
- **DescriptionList** - Term/definition pairs
- **Listbox** / **OptionList** - Interactive options

## Navigation
- **Link** - Inline navigation. `url`, `external`, `monochrome`
- **Pagination** - `hasPrevious`, `hasNext`, `onPrevious`, `onNext`
- **Tabs** - `tabs`, `selected`, `onSelect`, `fitted`
- **FooterHelp** - Reference links at page bottom

## Overlays
- **Popover** - `active`, `activator`, `onClose`, `preferredPosition`, `sectioned`, `fullWidth`. Children: `Popover.Pane`, `Popover.Section`
- **Tooltip** - `content`, `active`, `preferredPosition`

## Images & Icons
- **Avatar** - `source`, `initials`, `size`, `name`
- **Icon** - `source` (from @shopify/polaris-icons)
- **Thumbnail** - `source`, `alt`, `size`
- **VideoThumbnail** - `thumbnailUrl`, `onClick`

## Skeleton Loading
- **SkeletonPage** - `title`, `fullWidth`, `primaryAction`, `backAction`
- **SkeletonBodyText** - `lines` (default 3)
- **SkeletonDisplayText** - `size`: small/medium/large
- **SkeletonTabs** - Tab bar placeholder
- **SkeletonThumbnail** - Image placeholder

## Utilities
- **AppProvider** - REQUIRED root component
- **Collapsible** - Expandable content. `open`, `transition`
- **Scrollable** - Vertical overflow. `shadow`, `horizontal`

## Design Tokens - Complete

### Spacing
| Token | Px | Use Case |
|-------|-----|----------|
| space-0 | 0 | Reset |
| space-025 | 1 | Hairline |
| space-050 | 2 | Tight |
| space-100 | 4 | Minimum |
| space-150 | 6 | Compact |
| space-200 | 8 | Small gap |
| space-300 | 12 | Medium gap |
| space-400 | 16 | Card padding, standard gap |
| space-500 | 20 | Section gap |
| space-600 | 24 | Large gap |
| space-800 | 32 | Section spacing |
| space-1000 | 40 | Large section |
| space-1200 | 48 | Page section |
| space-1600 | 64 | Major section |
| space-2000 | 80 | Hero |

### Colors - Backgrounds
- `bg`: rgba(241,241,241,1) - Page background
- `bg-surface`: #FFFFFF - Card/surface
- `bg-surface-secondary`: rgba(247,247,247,1) - Secondary surface
- `bg-inverse`: rgba(26,26,26,1) - Dark backgrounds

### Colors - Fills
- `bg-fill-brand`: rgba(48,48,48,1) - Primary brand
- `bg-fill-success`: rgba(4,123,93,1) - Success
- `bg-fill-warning`: rgba(255,184,0,1) - Warning
- `bg-fill-critical`: rgba(199,10,36,1) - Error/critical
- `bg-fill-info`: rgba(145,208,255,1) - Information
- `bg-fill-emphasis`: rgba(0,91,211,1) - Emphasis/link
- `bg-fill-magic`: rgba(128,81,255,1) - AI/magic

### Colors - Text
- `text`: rgba(48,48,48,1) - Primary text
- `text-secondary`: rgba(97,97,97,1) - Secondary
- `text-disabled`: rgba(181,181,181,1) - Disabled
- `text-link`: rgba(0,91,211,1) - Links
- `text-success`: rgba(1,75,64,1) - Success text
- `text-critical`: rgba(142,11,33,1) - Error text
- `text-inverse`: rgba(227,227,227,1) - On dark backgrounds

### Colors - Icons
- `icon`: rgba(74,74,74,1) - Default
- `icon-secondary`: rgba(138,138,138,1) - Secondary
- `icon-success`: rgba(4,123,93,1) - Success
- `icon-critical`: rgba(226,44,56,1) - Error
- `icon-magic`: rgba(128,81,255,1) - AI

### Colors - Borders
- `border`: rgba(227,227,227,1) - Default
- `border-focus`: rgba(0,91,211,1) - Focus ring
- `border-success`: rgba(146,252,172,1) - Success
- `border-critical`: rgba(254,193,199,1) - Error

### Typography
- Font: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif
- Regular weight: 450, Bold: 700
- Sizes: 300=12px, 325=13px, 350=14px, 400=16px, 500=20px, 600=24px, 750=30px, 900=36px

### Shadows
- `shadow-100`: 0 1px 0 0 rgba(26,26,26,0.07) - Card
- `shadow-200`: 0 3px 1px -1px rgba(26,26,26,0.07) - Raised
- `shadow-300`: 0 4px 6px -2px rgba(26,26,26,0.2) - Popover
- `shadow-400`: 0 8px 16px -4px rgba(26,26,26,0.22) - Modal

### Border Radius
- `border-radius-100`: 4px
- `border-radius-200`: 8px
- `border-radius-300`: 12px
- `border-radius-400`: 16px
- `border-radius-full`: 9999px

### Motion
- `duration-100`: 100ms, `duration-150`: 150ms, `duration-200`: 200ms
- `duration-300`: 300ms, `duration-400`: 400ms, `duration-500`: 500ms
- `ease`: cubic-bezier(0.25, 0.1, 0.25, 1)

### Breakpoints
- `sm`: 490px, `md`: 768px, `lg`: 1040px, `xl`: 1440px

## App Bridge UI APIs

### Toast
```js
shopify.toast.show('Saved');
shopify.toast.show('Error', { isError: true });
shopify.toast.show('Done', { action: 'Undo', onAction: handleUndo, duration: 5000 });
```

### Modal
```html
<s-modal id="my-modal">
  <s-text>Content here</s-text>
  <s-button variant="primary" onclick="...">Confirm</s-button>
  <s-button onclick="shopify.modal.hide('my-modal')">Cancel</s-button>
</s-modal>
```

### Save Bar
```html
<form data-save-bar data-discard-confirmation>...</form>
```

### Resource Picker
```js
const selection = await shopify.resourcePicker({
  type: 'product', action: 'select', multiple: true,
  filter: { variants: false }
});
```

### Loading Bar
```js
shopify.loading.show();
shopify.loading.hide();
```

### TitleBar (React)
```tsx
import { TitleBar } from '@shopify/app-bridge-react';
<TitleBar title="Products">
  <button variant="primary" onClick={save}>Save</button>
</TitleBar>
```

### NavMenu (React)
```tsx
import { NavMenu } from '@shopify/app-bridge-react';
<NavMenu>
  <a href="/" rel="home">Home</a>
  <a href="/analytics">Analytics</a>
</NavMenu>
```

## Accessibility Requirements

- All interactive elements keyboard accessible
- Visible form labels (use `labelHidden` only when context is clear)
- Color is never the sole state indicator (add icons + text)
- Focus trapped in modals, returned to activator on close
- Semantic HTML (`as="h2"`, `as="p"`)
- AA contrast minimum
- Use `useBreakpoints()` for responsive rendering
- `condensed` prop on IndexTable for mobile
