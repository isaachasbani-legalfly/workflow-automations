# Changelog

## v1.0 (2026-04-09)

Initial implementation.

- HubSpot webhook trigger on `dealstage` property change
- Filter for 06_Sales Closed stage (`2406692058`) in Sales Pipeline
- Fetch deal details and check for SE assignment
- Resolve SE name via HubSpot Owners API
- Search Linear for existing ticket linked to deal (attachment-based dedup, same as SE-Linear workflow)
- Filter to Solutions Engineering team tickets only
- Build and post Slack message with deal link, SE name, Linear ticket link, and CTA
- Silent skip on all negative paths (no SE, no ticket, wrong stage)
- Connected to global error handler workflow
