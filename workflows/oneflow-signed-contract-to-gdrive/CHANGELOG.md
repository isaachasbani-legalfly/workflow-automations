# Changelog

## v1.2 - 2026-02-27

- **Breaking**: Removed IF filter node ("Filter: Is Contract Signed?") — event filtering now done at Oneflow webhook source
- **Feature**: Configured Oneflow webhook (ID: 20565) with `EVENT_TYPE = contract:sign` filter via Oneflow API, eliminating unnecessary workflow executions
- **Bugfix**: Fixed Oneflow webhook payload paths — real payload uses `body.contract.id` and `body.events[0].type`, not `body.data.subject.id` and `body.type`
- Simplified workflow from 6 nodes to 5 nodes (webhook → parties → files → download → upload)
- Updated sticky note documentation to reflect source-side filtering

## v1.1 - 2026-02-26

- **Bugfix**: Fixed Get Contract Files URL — was using `$json.data[0]._links.self.href` (wrong context) instead of Oneflow API endpoint with contract ID
- **Feature**: Added "Get Contract Parties" node to fetch counterparty company name from Oneflow API
- **Feature**: File naming changed to `{Company Name} - {YYYY-MM-DD}.pdf` using counterparty name (`my_party: false`) and current date
- Added IF filter node "Filter: Is Contract Signed?" to check `body.type == contract:sign` before processing
- Switched from inline API token headers to stored `httpHeaderAuth` credential ("Oneflow")
- Download node uses HAL-style `_links.self.href` from files API response with `?download=true`
- Get Contract Files now uses `$('Oneflow Webhook')` reference for contract ID since `$json` context changed after inserting parties node
- Added explicit `resource: file` and `operation: upload` to Google Drive node

## v1.0 - 2026-02-26

- Initial workflow creation
- Webhook trigger for Oneflow contract:sign events
- Oneflow API integration to fetch and download signed contract PDFs
- Google Drive upload to specified folder
- Deployed and activated on n8n cloud
