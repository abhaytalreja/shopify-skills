# Polaris Design Tokens Reference

## Spacing

Based on a 4px grid. Use as gap, padding, and margin values in Polaris components.

| Token | Value | Common Use |
|-------|-------|-----------|
| `space-0` | 0px | No spacing |
| `space-025` | 1px | Hairline borders |
| `space-050` | 2px | Tight inline spacing |
| `space-100` | 4px | Minimal gap |
| `space-150` | 6px | Tight spacing |
| `space-200` | 8px | Button group gap, tight padding |
| `space-300` | 12px | Medium-tight spacing |
| `space-400` | 16px | Card padding, standard gap |
| `space-500` | 20px | Section padding |
| `space-600` | 24px | Large gap |
| `space-800` | 32px | Section gap |
| `space-1000` | 40px | Large section gap |
| `space-1200` | 48px | Page section gap |
| `space-1600` | 64px | Extra large spacing |
| `space-2000` | 80px | Hero spacing |
| `space-2400` | 96px | Maximum spacing |
| `space-2800` | 112px | Layout spacing |
| `space-3200` | 128px | Maximum layout spacing |

### Common Spacing Patterns

- Card internal padding: `space-400` (16px)
- Gap between cards: `space-400` (16px)
- Gap between form fields: `space-400` (16px)
- Gap between page sections: `space-400` - `space-800`
- Button group gap: `space-200` (8px)
- Inline icon gap: `space-200` (8px)
- BlockStack default gap: `space-400` (16px)

## Typography

### Font Family
Primary: Inter, -apple-system, BlinkMacSystemFont, "San Francisco", "Segoe UI", Roboto, "Helvetica Neue", sans-serif

Monospace: ui-monospace, SFMono-Regular, "SF Mono", Menlo, Consolas, "Liberation Mono", monospace

### Font Weights

| Token | Value | Use |
|-------|-------|-----|
| `font-weight-regular` | 450 | Body text |
| `font-weight-medium` | 550 | Emphasized text |
| `font-weight-semibold` | 650 | Sub-headings |
| `font-weight-bold` | 700 | Headings, emphasis |

### Font Sizes (via Text variant prop)

| Variant | Size | Line Height | Weight | Use |
|---------|------|-------------|--------|-----|
| `headingXl` | 27px (mobile) / 36px (desktop) | 32px / 44px | Bold | Page titles |
| `headingLg` | 21px / 24px | 28px / 32px | Bold | Section titles |
| `headingMd` | 16px / 20px | 24px / 28px | Bold | Card titles |
| `headingSm` | 14px | 20px | Bold | Sub-section titles |
| `headingXs` | 13px | 20px | Bold | Minor headings |
| `bodyLg` | 16px | 24px | Regular | Large body text |
| `bodyMd` | 14px | 20px | Regular | Default body text |
| `bodySm` | 12px | 16px | Regular | Captions, help text |

### Usage in Components

```tsx
<Text as="h1" variant="headingXl">Page Title</Text>
<Text as="h2" variant="headingLg">Section Title</Text>
<Text as="h3" variant="headingMd">Card Title</Text>
<Text as="h4" variant="headingSm">Subsection</Text>
<Text as="p" variant="bodyLg">Large paragraph</Text>
<Text as="p" variant="bodyMd">Standard paragraph</Text>
<Text as="span" variant="bodySm" tone="subdued">Help text</Text>
```

## Colors

### Background Colors

| Token | Hex | Use |
|-------|-----|-----|
| `bg` | #F1F1F1 | Page background |
| `bg-surface` | #FFFFFF | Card/surface background |
| `bg-surface-secondary` | #F7F7F7 | Secondary surface |
| `bg-surface-tertiary` | #EFEFEF | Tertiary surface |
| `bg-surface-hover` | #F1F1F1 | Surface hover state |
| `bg-surface-active` | #E7E7E7 | Surface active/pressed state |
| `bg-surface-selected` | #F0F0FF | Selected row/item |
| `bg-surface-disabled` | #FAFAFA | Disabled surface |

### Fill Colors (Buttons, Badges, etc.)

| Token | Hex | Use |
|-------|-----|-----|
| `bg-fill-brand` | #303030 | Primary brand fill (buttons) |
| `bg-fill-brand-hover` | #1A1A1A | Primary button hover |
| `bg-fill-brand-active` | #000000 | Primary button active |
| `bg-fill-brand-disabled` | #D4D4D4 | Disabled primary button |
| `bg-fill-success` | #047B5D | Success actions |
| `bg-fill-success-secondary` | #AFEACD | Success badge background |
| `bg-fill-warning` | #FFB800 | Warning fill |
| `bg-fill-warning-secondary` | #FFF0B3 | Warning badge background |
| `bg-fill-critical` | #C70A24 | Critical/destructive fill |
| `bg-fill-critical-secondary` | #FED3D1 | Critical badge background |
| `bg-fill-info` | #005BD3 | Info fill |
| `bg-fill-info-secondary` | #D3E5FF | Info badge background |
| `bg-fill-transparent` | transparent | Transparent fill |
| `bg-fill-transparent-hover` | rgba(0,0,0,0.05) | Transparent hover |

### Text Colors

| Token | Hex | Use |
|-------|-----|-----|
| `text` | #303030 | Primary text |
| `text-secondary` | #616161 | Secondary/subdued text |
| `text-disabled` | #B5B5B5 | Disabled text |
| `text-brand` | #303030 | Brand text |
| `text-link` | #005BD3 | Link text |
| `text-link-hover` | #003FB5 | Link hover |
| `text-success` | #047B5D | Success text |
| `text-warning` | #8A6500 | Warning text |
| `text-critical` | #C70A24 | Error/critical text |
| `text-inverse` | #FFFFFF | Text on dark backgrounds |
| `text-magic` | #7B61FF | AI/magic feature text |

### Icon Colors

| Token | Hex | Use |
|-------|-----|-----|
| `icon` | #303030 | Default icon |
| `icon-secondary` | #616161 | Secondary icon |
| `icon-disabled` | #B5B5B5 | Disabled icon |
| `icon-brand` | #303030 | Brand icon |
| `icon-success` | #047B5D | Success icon |
| `icon-warning` | #8A6500 | Warning icon |
| `icon-critical` | #C70A24 | Critical icon |
| `icon-info` | #005BD3 | Info icon |
| `icon-interactive` | #005BD3 | Interactive/clickable icon |

### Border Colors

| Token | Hex | Use |
|-------|-----|-----|
| `border` | #E3E3E3 | Default border |
| `border-hover` | #C9C9C9 | Border on hover |
| `border-focus` | #005BD3 | Focus ring |
| `border-disabled` | #EBEBEB | Disabled border |
| `border-brand` | #303030 | Brand border |
| `border-success` | #047B5D | Success border |
| `border-warning` | #FFB800 | Warning border |
| `border-critical` | #C70A24 | Critical border |
| `border-info` | #005BD3 | Info border |
| `border-transparent` | transparent | Transparent border |

## Shadows

| Token | Value | Use |
|-------|-------|-----|
| `shadow-0` | none | No shadow |
| `shadow-100` | 0 1px 0 0 rgba(0,0,0,0.05) | Subtle card shadow |
| `shadow-200` | 0 3px 6px -3px rgba(0,0,0,0.1), 0 1px 0 0 rgba(0,0,0,0.05) | Raised card |
| `shadow-300` | 0 4px 12px -2px rgba(0,0,0,0.12), 0 1px 0 0 rgba(0,0,0,0.05) | Popover |
| `shadow-400` | 0 8px 24px -4px rgba(0,0,0,0.15), 0 1px 0 0 rgba(0,0,0,0.05) | Modal |

### Usage

```tsx
<Box shadow="200">Raised card</Box>
```

## Border Radius

| Token | Value | Use |
|-------|-------|-----|
| `border-radius-0` | 0px | No rounding |
| `border-radius-050` | 2px | Minimal rounding |
| `border-radius-100` | 4px | Badges, small elements |
| `border-radius-150` | 6px | Buttons |
| `border-radius-200` | 8px | Cards, inputs |
| `border-radius-300` | 12px | Larger cards |
| `border-radius-400` | 16px | Modals |
| `border-radius-500` | 20px | Large rounded elements |
| `border-radius-750` | 30px | Pills |
| `border-radius-full` | 9999px | Circles, fully rounded |

### Usage

```tsx
<Box borderRadius="200">Rounded card</Box>
```

## Border Width

| Token | Value |
|-------|-------|
| `border-width-0` | 0px |
| `border-width-025` | 1px |
| `border-width-050` | 2px |
| `border-width-100` | 4px |

## Breakpoints

| Token | Min Width | Typical Device |
|-------|-----------|---------------|
| `xs` | 0px | Small phones |
| `sm` | 490px | Large phones |
| `md` | 768px | Tablets |
| `lg` | 1040px | Small desktops |
| `xl` | 1440px | Large desktops |

### Usage in Components

```tsx
<InlineGrid columns={{ xs: 1, sm: 2, md: 3, lg: 4 }} gap="400">
  {/* Responsive grid */}
</InlineGrid>

<InlineGrid columns={{ xs: "1fr", md: "2fr 5fr" }} gap="400">
  {/* Settings layout: stacked on mobile, side-by-side on desktop */}
</InlineGrid>
```

## Motion / Animation

| Token | Value | Use |
|-------|-------|-----|
| `motion-duration-0` | 0ms | Instant |
| `motion-duration-50` | 50ms | Micro interactions |
| `motion-duration-100` | 100ms | Fast transitions |
| `motion-duration-150` | 150ms | Standard transitions |
| `motion-duration-200` | 200ms | Hover effects |
| `motion-duration-250` | 250ms | Opening/closing |
| `motion-duration-300` | 300ms | Complex transitions |
| `motion-duration-350` | 350ms | Page transitions |
| `motion-duration-400` | 400ms | Large element transitions |
| `motion-duration-500` | 500ms | Slow transitions |

### Easing

| Token | Value | Use |
|-------|-------|-----|
| `motion-ease` | cubic-bezier(0.25, 0.1, 0.25, 1) | Standard ease |
| `motion-ease-in` | cubic-bezier(0.42, 0, 1, 1) | Entering elements |
| `motion-ease-out` | cubic-bezier(0, 0, 0.58, 1) | Exiting elements |
| `motion-ease-in-out` | cubic-bezier(0.42, 0, 0.58, 1) | Moving elements |
| `motion-linear` | linear | Progress bars, loading |

## Z-Index

| Token | Value | Use |
|-------|-------|-----|
| `z-index-1` | 100 | Dropdowns |
| `z-index-2` | 400 | Navigation |
| `z-index-3` | 500 | Overlays |
| `z-index-4` | 510 | Modals |
| `z-index-5` | 512 | Toast |
| `z-index-6` | 513 | Highest (loading bar) |

## Applying Tokens in Components

Tokens are applied through component props, not CSS custom properties:

```tsx
// Spacing via gap/padding props
<BlockStack gap="400">...</BlockStack>
<Box padding="400">...</Box>
<Card padding="0">...</Card>

// Colors via tone/background props
<Text tone="subdued">Secondary text</Text>
<Text tone="critical">Error text</Text>
<Box background="bg-surface-secondary">...</Box>
<Banner tone="success">...</Banner>
<Badge tone="warning">Warning</Badge>

// Typography via variant prop
<Text variant="headingLg">Title</Text>
<Text variant="bodyMd">Body</Text>

// Shadows via shadow prop
<Box shadow="200">Elevated card</Box>

// Border radius via borderRadius prop
<Box borderRadius="200">Rounded</Box>
```

Do NOT use raw CSS values. Always use Polaris tokens through component props for consistency with the Shopify admin.
