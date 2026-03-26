# App Store Launch Checklist

## Pre-Submission Requirements

### Technical Requirements (Mandatory)

- [ ] **GraphQL Admin API only** - No REST API calls. REST has been legacy since Oct 2024.
- [ ] **Latest App Bridge** - Use CDN script tag (auto-updates). No npm `@shopify/app-bridge` package.
- [ ] **Embedded in Shopify Admin** - `embedded = true` in shopify.app.toml. Non-embedded apps are rejected except for specific use cases.
- [ ] **Session Token Auth** - Token Exchange flow. No third-party cookie reliance.
- [ ] **HTTPS/TLS on all endpoints** - Including webhook receivers, OAuth callbacks, and app URLs.
- [ ] **OAuth initiates immediately** - On first install, the auth flow must start without requiring merchant action.
- [ ] **Responsive layout** - Must work on mobile viewports (test at 375px width minimum).

### GDPR Compliance (Mandatory - Instant Rejection if Missing)

- [ ] **customers/data_request webhook** - Handle data export requests within 30 days
- [ ] **customers/redact webhook** - Delete customer personal data within 30 days
- [ ] **shop/redact webhook** - Delete all shop data (sent 48hrs after uninstall)
- [ ] **Privacy policy URL** - Linked in app listing and accessible

### Billing (If Applicable)

- [ ] **Billing API for all paid features** - No external payment (Stripe, PayPal) for in-admin features
- [ ] **Clear pricing on listing** - Match Billing API amounts with listing prices
- [ ] **Free trial or free tier** - Recommended but not required
- [ ] **Test mode disabled** - Remove `isTest: true` before submission

### Data Handling

- [ ] **Minimal scopes** - Request only the access scopes your app actually needs
- [ ] **No sensitive data in metafields** - Metafields are readable via Storefront API
- [ ] **Data deletion on uninstall** - Handle `app/uninstalled` webhook to clean up sessions
- [ ] **Data retention policy** - Document how long you store merchant data

## App Listing Requirements

### Basic Information

- [ ] **App name** - Unique, descriptive, not misleading. Cannot contain "Shopify" or imply Shopify endorsement.
- [ ] **Tagline** - Under 80 characters, describes core value
- [ ] **Description** - Clear explanation of features and benefits (min 200 characters)
- [ ] **Category** - Select the most relevant app category
- [ ] **Pricing** - Accurate pricing details matching Billing API configuration

### Media

- [ ] **App icon** - 1200x1200px PNG, no transparency, no text overlays
- [ ] **Screenshots** - Minimum 3 screenshots. 1600x900px recommended. Show actual app UI in context.
- [ ] **Demo video** - English screencast with subtitles. Under 3 minutes. Show core workflow.
  - Must show the app running embedded in Shopify admin
  - Must demonstrate the primary use case end-to-end
  - Must include audio narration with closed captions/subtitles
- [ ] **Key benefits images** - Up to 6 feature highlight images

### Support

- [ ] **Support email** - Valid, monitored email for merchant support
- [ ] **Support URL** - Link to help center, docs, or FAQ
- [ ] **FAQ** - At least 3-5 frequently asked questions on listing

## Built for Shopify Requirements

"Built for Shopify" is a premium badge that signals quality. Requirements beyond standard:

### Performance
- [ ] Page load under 3 seconds (measured in admin)
- [ ] No blocking JavaScript in critical path
- [ ] Efficient API usage (batch queries, use bulk operations for large datasets)
- [ ] Handle rate limiting gracefully with exponential backoff

### Quality
- [ ] No console errors in production
- [ ] All Polaris components used correctly (no deprecated components)
- [ ] Follows Polaris design patterns for layout, navigation, and feedback
- [ ] Proper error handling with user-friendly messages
- [ ] Skeleton loading states on all pages
- [ ] Empty states for pages with no data

### Reliability
- [ ] 99.9% uptime SLA
- [ ] Webhook processing within 5 seconds
- [ ] Graceful degradation when Shopify API is slow
- [ ] Idempotent webhook handling (deduplicate with X-Shopify-Event-Id)

### Merchant Experience
- [ ] Onboarding flow guides new users
- [ ] In-app help or documentation
- [ ] Clear error messages with actionable steps
- [ ] Support response within 24 hours (business days)
- [ ] 4.0+ average rating maintained

### Technical Excellence
- [ ] All compliance webhooks implemented and tested
- [ ] Webhook subscriptions automatically registered
- [ ] Session management with proper token refresh
- [ ] No deprecated API endpoints
- [ ] Latest stable API version

## Review Process

### Timeline
- Initial review: 5-10 business days
- Revisions: 3-5 business days per round
- Total: Plan for 2-4 weeks from submission to approval

### What Reviewers Check
1. Install and OAuth flow
2. Core functionality works as described
3. Compliance webhooks respond correctly
4. Embedded app renders properly
5. Billing flow (if applicable)
6. Uninstall and data cleanup
7. Mobile responsiveness
8. Error handling
9. App listing accuracy

### Common Rejection Reasons

1. **Missing compliance webhooks** - All 3 GDPR webhooks must be implemented and return 200
2. **REST API usage** - Any REST calls will be flagged. Migrate to GraphQL.
3. **Broken OAuth flow** - Install must be seamless, no manual steps required
4. **Non-embedded** - App must render inside Shopify admin iframe
5. **Excessive scopes** - Requesting scopes your app does not use
6. **External billing** - Charging merchants outside Shopify Billing API
7. **Missing demo video** - Must include English screencast with subtitles
8. **Poor mobile experience** - Must be responsive at minimum 375px
9. **Misleading listing** - Screenshots or description do not match actual functionality
10. **Broken uninstall** - Sessions not cleaned up, or errors during uninstall
11. **No error handling** - Unhandled exceptions or unclear error messages
12. **Stale API version** - Using deprecated API versions

### Resubmission Tips

- Address ALL feedback items before resubmitting
- Include a changelog of what you fixed
- Test the complete flow on a fresh development store
- Have someone else (not the developer) test the install-to-use flow

## Post-Launch

### Monitoring
- Set up uptime monitoring for app URL and webhook endpoints
- Monitor Shopify Partner dashboard for reviews and support tickets
- Track API error rates and response times
- Monitor webhook delivery success rates

### Maintaining Compliance
- Update API version within 12 months of each release (Shopify deprecates old versions)
- Respond to compliance webhook changes
- Keep privacy policy updated
- Maintain support response SLA

### App Updates
- Test updates on development store before deploying
- Use `shopify app deploy` for config and extension changes
- Monitor for breaking changes in Shopify API changelog
- Announce changes to merchants via in-app banners or email

## Development Store Setup

For testing and review submission:

```bash
# Create a development store via Shopify Partners dashboard
# https://partners.shopify.com

# Install your app on the dev store
shopify app dev --reset  # Reset dev store connection

# Generate test data
# Use Shopify admin > Settings > Test data to populate products/orders
```

Development stores have:
- All Shopify Plus features enabled
- No transaction fees
- Cannot process real payments
- Do not count toward app install limits
