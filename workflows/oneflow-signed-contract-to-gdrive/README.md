# Oneflow Signed Contract to Google Drive + HubSpot

Automatically downloads signed contract PDFs from Oneflow and:
1. Uploads them to a Google Drive folder
2. Attaches the PDF to the HubSpot deal's `signed_contract` file property
3. Sets the `contract_signed_date` on the deal

## Trigger

Oneflow webhook event: `contract:sign` -- fires when all parties have signed a contract in Oneflow. The webhook is configured with an EVENT_TYPE filter at the source (Oneflow webhook ID: 20565), so only `contract:sign` events reach this workflow.

## Workflow Steps

1. **Oneflow Webhook** -- Receives POST from Oneflow at `/webhook/oneflow-signed-contracts`
2. **Get Full Contract** -- Calls Oneflow API `GET /v1/contracts/{id}` to get the full contract including `data_fields` (with HubSpot deal name) and `parties`
3. **Extract Contract Data** -- Code node extracts `dealName`, `counterpartyName`, `contractId`, and `signingDate`
4. **Get Contract Files** -- Calls Oneflow API `GET /v1/contracts/{id}/files` to retrieve file list
5. **Download Contract PDF** -- Downloads the PDF binary using HAL `_links.self.href` with `?download=true`
6. **Upload to Google Drive** -- Uploads as `{Company Name} - {YYYY-MM-DD}.pdf` (parallel branch 1)
7. **Upload to HubSpot Files** -- Uploads PDF to HubSpot File Manager `/signed-contracts/` folder (parallel branch 2)
8. **Search HubSpot Deal** -- Finds the deal by exact name match using `hs_deal_dealname` from Oneflow data fields
9. **Update Deal** -- Sets `signed_contract` (file URL) and `contract_signed_date` (YYYY-MM-DD)

## Credentials Required

| Service | Credential Name | Type | Used For |
|---------|----------------|------|----------|
| Google Drive | Google Drive account | Google Drive OAuth2 API | Uploading PDF to Drive |
| Oneflow | Oneflow | httpHeaderAuth (x-oneflow-api-token) | API calls to Oneflow |
| HubSpot | hubspot | hubspotAppToken | File upload, deal search, deal update |

### HubSpot Scopes Required
- `files` -- upload to File Manager
- `crm.objects.deals.write` -- update deal properties
- `crm.objects.deals.read` -- search deals

## Configuration

- **Google Drive Folder ID**: `1Z4j_Y_8RURFbG_rVHn2inU2LOwIaCC9c`
- **HubSpot File Folder**: `/signed-contracts/`
- **HubSpot File Access**: `PUBLIC_NOT_INDEXABLE` (required for file property display)
- **Oneflow API Base URL**: `https://api.oneflow.com/v1`
- **Webhook Path**: `oneflow-signed-contracts`

## Key Behaviors

- **signed_contract**: Appends new files (semicolon-separated IDs). Never overwrites existing files on the deal.
- **contract_signed_date**: Always overwrites with the latest signing date.
- **File display**: Uses HubSpot file ID (not URL) so the property shows the filename, not a raw URL.

## Oneflow-to-HubSpot Linkage

The Oneflow contract's `data_fields` contain HubSpot deal data populated when the contract was created from HubSpot. The field `custom_id: "hs_deal_dealname"` provides the exact deal name used to search HubSpot.

## Oneflow Webhook Setup

In Oneflow, webhook ID 20565 is configured with:
- **URL**: `https://legalfly.app.n8n.cloud/webhook/oneflow-signed-contracts`
- **Event filter**: `EVENT_TYPE = contract:sign` (only sends contract:sign events)

## Error Handling

- **Error workflow**: `TA6Iq4wMW0KYsCiH` -- sends Slack notification on failure
- **Update Deal**: Retries 3x with 1s delay on transient failures

## n8n Instance

- **Workflow ID**: `00YFVcmBURJZ3cGU`
- **URL**: https://legalfly.app.n8n.cloud/workflow/00YFVcmBURJZ3cGU
- **Status**: Active
