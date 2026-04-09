# Changelog

## v1.0 (2026-04-09)

Initial implementation.

- HubSpot internal workflow triggers webhook when deal enters 06_Sales Closed (sends deal ID directly, no event parsing needed)
- Normalize webhook payload to extract deal ID (same pattern as line-items workflow)
- Fetch deal details and check for SE assignment
- Resolve SE name and email via HubSpot Owners API
- Lookup SE in Slack via `users.lookupByEmail` API for dynamic `<@U...>` tagging (falls back to plain-text name)
- Search Linear for existing ticket linked to deal (attachment-based dedup, same as SE-Linear workflow)
- Filter to Solutions Engineering team tickets only
- Build and post "Deal Closed!" Slack message with deal link, SE Slack tag, Linear ticket link, and CTA
- Silent skip on all negative paths (no SE, no ticket)
- Connected to global error handler workflow
- 13 nodes total (webhook, normalize, fetch deal, SE guard, stop, resolve SE, Slack lookup, search Linear, find ticket, ticket guard, stop, build message, post Slack)
- Requires Slack scope `users:read.email` for SE lookup
