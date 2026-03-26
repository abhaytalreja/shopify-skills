# Shopify App Launch Reference

## Distribution Methods (Permanent Choice)

- **Public**: App Store listing, requires review, supports Billing API
- **Custom**: Single store or Plus org, no review, CANNOT use Billing API
- **Shopify Admin (Deprecated)**: Single store, no App Bridge/extensions

## App Store Submission Checklist

### Technical Requirements
- [ ] GraphQL Admin API only (no REST - mandatory since April 1, 2025)
- [ ] Latest App Bridge via CDN script tag (mandatory since Oct 15, 2025)
- [ ] Embedded in Shopify Admin
- [ ] Session token auth (no third-party cookies)
- [ ] TLS/SSL on all data exchange
- [ ] OAuth initiates immediately on install (before any other steps)
- [ ] Redirect to app UI after OAuth acceptance
- [ ] Handle reinstallation for returning merchants

### Mandatory Compliance Webhooks (3 required)
```toml
[[webhooks.subscriptions]]
compliance_topics = ["customers/data_request", "customers/redact", "shop/redact"]
uri = "/webhooks"
```

1. `customers/data_request` - Respond to customer data access requests
2. `customers/redact` - Delete customer data on request
3. `shop/redact` - Delete ALL shop data 48 hours after uninstall

All must: respond with 200, complete within 30 days, validate HMAC.

### Billing Requirements
- Use Shopify Billing API or Managed Pricing (no external payment)
- Merchants must upgrade/downgrade without contacting support
- Revenue share: 0% on first $1M USD annually

### Privacy Requirements
- Privacy policy required, linked from listing
- Disclose: data collected, purposes, retention, storage locations
- Handle data access, correction, erasure, restriction requests

### Restricted Access Scopes (must justify)
- `read_all_orders`
- `write_payment_mandate`
- `write_checkout_extensions_apis`
- `read_advanced_dom_pixel_events`

## App Listing Optimization

| Element | Spec | Best Practice |
|---------|------|---------------|
| App Name | Max 30 chars | Brand name first (e.g., "QTeck - Announcement Bar") |
| App Icon | 1200x1200px JPEG/PNG | Bold colors, simple, no text/screenshots |
| Screenshots | 1600x900px (16:9), 3-6 min | Actual UI, no browser chrome, no pricing |
| Video | 1600x900px, 2-3 min max | Screencasts max 25% of video |
| Introduction | 100 chars | Highlight merchant benefits |
| Details | 500 chars | Functionality + uniqueness |
| Features | Up to 80 chars each | Merchant-facing benefits |
| Search Terms | Up to 5 terms | Complete words, one idea per term |

Pricing shown lowest-to-highest automatically. Translated listings convert up to 4x better.

## App Review Process

**Timeline:** 5-10 business days
**Stages:** Draft -> Submitted -> Reviewed -> Published (or Paused)

### Common Rejection Reasons
1. **Billing** - Inaccurate pricing, no upgrade/downgrade, missing Billing API
2. **Installation** - No immediate OAuth, fatal errors post-install, reinstall failures
3. **Embedding** - Switching between embedded/non-embedded
4. **UI** - Broken interface, HTTP 404/500 errors
5. **Testing** - Missing credentials, incomplete submissions, beta quality
6. **Online Store** - Not using theme app extensions, broken widgets

### Submission Requirements
- English screencast with subtitles
- Test credentials (full access, kept current)
- Emergency contact (email + phone)
- Whitelist `app-submissions@shopify.com` and `noreply@shopify.com`

## Built for Shopify (BFS) Certification

### Prerequisites
- All App Store requirements met
- Minimum 50 net installs from paid-plan merchants
- At least 5 app store reviews
- Meet minimum recent rating threshold

### Performance Standards (75th percentile, 28-day window)
- LCP: 2.5 seconds or less
- CLS: 0.1 or less
- INP: 200 milliseconds or less
- Storefront: Cannot reduce Lighthouse score by more than 10 points
- Checkout (if applicable): p95 response 500ms or less, max 0.1% failure

### Design Standards
- Mirror Shopify admin look/feel
- Card-like containers, matching button styles, sans-serif fonts
- Clear onboarding with dismiss mechanism
- Red for errors only
- No countdown timers, no guilt language
- No Shopify logo impersonation
- All ads/promotions must be dismissible
- Disable and label plan-gated features

### Analytics Apps Specifically
- Must use Web Pixel extensions for event tracking (no script tags)

### Benefits
- Badge on app cards
- Higher search ranking
- Dedicated search filter
- Priority review
- Featured placement eligibility

## Web Pixel Extensions (for Analytics Apps)

Required scopes: `write_pixels`, `read_customer_events`
Runtime context: `strict`

```javascript
// Subscribe to events
analytics.subscribe('all_events', (event) => {
  // Handle event
});

// Or specific events
analytics.subscribe('checkout_completed', (event) => {
  // Track conversion
});
```

Activate with `webPixelCreate` GraphQL mutation.
Deploy with `shopify app deploy`.
