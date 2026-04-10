# Changelog

## v1.3 — 2026-04-10

- Added Slack notification on new ticket creation (channel `C09QT2RFSBW`)
- Tags SE dynamically via email lookup + @dm (`U043SB6G5C5`, hardcoded)
- Changed Slack channel for both workflows from `C0AFSAD1E5A` to `C09QT2RFSBW`
- 19 nodes total (3 new: Lookup SE in Slack, Build Slack Message, Post to Slack)

## v1.2 — 2026-04-09

- Added SE Work Summary template with interactive checkboxes to Linear ticket descriptions (engagement type: Custom demo, Free trial support, Bootcamp, Meeting support; free-text fields: What you did, Notes for CS team)
- Removed deprecated `hs-deal-id` tag from ticket description (dedup now uses Linear attachments API)

## v1.1 — 2026-04-08

- Changed trigger from HubSpot Trigger node to Webhook node (Private App webhook subscription)
- Added Extract Event node to parse HubSpot event array
- Fixed AE name resolution (now references Fetch Deal Details node for hubspot_owner_id)
- Changed dedup from deprecated `issueSearch` to Linear `attachments` API (searches by HubSpot link URL)
- Added Find Existing Ticket node to filter attachment results to Solutions Engineering team
- Connected error handler workflow (TA6Iq4wMW0KYsCiH)
- Added `crm.objects.owners.read` scope requirement for HubSpot Private App

## v1.0 — 2026-04-08

- Initial implementation
- HubSpot trigger on `deal.propertyChange` for `solutions_engineer`
- Guard for empty SE values
- Deal + owner resolution via HubSpot REST API
- SE → Linear user mapping (Harry Day, Anastasia Screve, Stephanie Adriaens)
- Linear dedup via GraphQL `issueSearch` with `hs-deal-id:` tag
- Create issue path: Linear issue in Solutions Engineering backlog + HubSpot link
- Update path: Reassign + comment on existing ticket
- HTTP Request retry (3 attempts, 1s interval)
