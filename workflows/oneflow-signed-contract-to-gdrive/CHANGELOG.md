# Changelog

## v2.0 - 2026-03-06

- **Feature**: Added HubSpot integration -- uploads signed PDF to HubSpot File Manager and attaches to deal's `signed_contract` file property
- **Feature**: Added `contract_signed_date` deal property update with the signing date from Oneflow webhook event
- **Feature**: File append logic -- new files are appended (semicolon-separated IDs) to `signed_contract`, never overwriting existing files
- **Feature**: Uses HubSpot file ID (not URL) so the property displays as a clickable filename in the deal record
- **Feature**: HubSpot file access set to `PUBLIC_NOT_INDEXABLE` for proper display in deal records
- **Breaking**: Replaced "Get Contract Parties" node with "Get Full Contract" (`GET /contracts/{id}`) to access `data_fields` containing HubSpot deal name
- **Feature**: Added "Extract Contract Data" code node to extract dealName, counterpartyName, contractId, signingDate
- **Feature**: After PDF download, workflow splits into two parallel branches: Google Drive upload (top) and HubSpot upload + deal update (bottom)
- **Feature**: Deal lookup uses `hs_deal_dealname` from Oneflow's `data_fields` to search HubSpot deals by exact name
- **Feature**: Error handler workflow (`TA6Iq4wMW0KYsCiH`) linked -- sends Slack notification on failure
- Workflow renamed from "Oneflow Signed Contract to Google Drive" to "Oneflow Signed Contract to Google Drive + HubSpot"
- Node count increased from 5 to 9
- New credentials required: HubSpot (hubspotAppToken) with `files` and `crm.objects.deals.write` scopes

## v1.2 - 2026-02-27

- **Breaking**: Removed IF filter node ("Filter: Is Contract Signed?") -- event filtering now done at Oneflow webhook source
- **Feature**: Configured Oneflow webhook (ID: 20565) with `EVENT_TYPE = contract:sign` filter via Oneflow API, eliminating unnecessary workflow executions
- **Bugfix**: Fixed Oneflow webhook payload paths -- real payload uses `body.contract.id` and `body.events[0].type`, not `body.data.subject.id` and `body.type`
- Simplified workflow from 6 nodes to 5 nodes (webhook -> parties -> files -> download -> upload)
- Updated sticky note documentation to reflect source-side filtering

## v1.1 - 2026-02-26

- **Bugfix**: Fixed Get Contract Files URL -- was using `$json.data[0]._links.self.href` (wrong context) instead of Oneflow API endpoint with contract ID
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
