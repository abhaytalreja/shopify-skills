# Shopify Skills for Claude Code

Comprehensive Claude Code skills for building, designing, and launching Shopify apps.

## Skills Included

### `shopify-dev` - Shopify App Development
Complete development skill covering:
- React Router v7 app architecture (NOT Remix)
- GraphQL Admin API (queries, mutations, rate limits, bulk operations)
- Authentication (Token Exchange, Session Tokens)
- Webhooks (declarative configuration, HMAC verification)
- Billing API (subscriptions, usage-based, one-time)
- Shopify Flow integration (triggers, actions, templates)
- GDPR compliance (mandatory webhooks)
- App Store launch requirements and review process
- Built for Shopify certification
- Common mistakes and how to avoid them

### `shopify-ui` - Shopify App UI (Polaris + App Bridge)
Complete UI/design skill covering:
- Full Polaris component inventory with props
- App Bridge UI (modals, toasts, save bar, navigation, resource picker)
- Design tokens (colors, spacing, typography, shadows, breakpoints)
- Layout patterns (index, detail, settings, dashboard)
- Form patterns with validation
- Data table patterns (IndexTable + IndexFilters)
- Empty states, loading states, skeleton pages
- Accessibility requirements
- Responsive design with breakpoints

## Installation

### Option 1: Copy to your project
```bash
cp -r skills/shopify-dev /path/to/your/project/.claude/skills/shopify-dev
cp -r skills/shopify-ui /path/to/your/project/.claude/skills/shopify-ui
```

### Option 2: Symlink
```bash
ln -s /path/to/shopify-skills/skills/shopify-dev /path/to/your/project/.claude/skills/shopify-dev
ln -s /path/to/shopify-skills/skills/shopify-ui /path/to/your/project/.claude/skills/shopify-ui
```

## Structure

```
skills/
  shopify-dev/
    SKILL.md                        # Main skill (< 500 lines)
    references/
      build-reference.md            # Full API reference
      launch-reference.md           # App Store launch guide
      flow-reference.md             # Shopify Flow integration
  shopify-ui/
    SKILL.md                        # Main skill (< 500 lines)
    references/
      polaris-reference.md          # Complete component inventory
```

## Key Design Decisions

- **React Router v7** (not Remix) - Shopify's official recommendation as of 2025
- **GraphQL only** - REST Admin API is legacy since October 2024
- **Skills under 500 lines** - Following Claude Code best practices for progressive loading
- **Reference files on-demand** - Detailed docs loaded only when needed to save context

## Sources

All content sourced from official Shopify documentation:
- https://shopify.dev/docs/apps/build
- https://shopify.dev/docs/apps/design
- https://shopify.dev/docs/apps/launch
- https://polaris.shopify.com
