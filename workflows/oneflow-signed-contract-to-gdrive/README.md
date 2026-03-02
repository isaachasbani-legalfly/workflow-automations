# Oneflow Signed Contract to Google Drive

Automatically downloads signed contract PDFs from Oneflow and uploads them to a Google Drive folder when a contract is fully signed by all parties.

## Trigger

Oneflow webhook event: `contract:sign` -- fires when all parties have signed a contract in Oneflow. The webhook is configured with an EVENT_TYPE filter at the source (Oneflow webhook ID: 20565), so only `contract:sign` events reach this workflow.

## Workflow Steps

1. **Oneflow Webhook** -- Receives POST from Oneflow at `/webhook/oneflow-signed-contracts`
2. **Get Contract Parties** -- Calls Oneflow API `GET /v1/contracts/{id}/parties` to get the counterparty company name
3. **Get Contract Files** -- Calls Oneflow API `GET /v1/contracts/{id}/files` to retrieve the list of files for the signed contract
4. **Download Contract PDF** -- Downloads the actual PDF binary using the HAL `_links.self.href` from the files response with `?download=true`
5. **Upload to Google Drive** -- Uploads the PDF as `{Company Name} - {YYYY-MM-DD}.pdf` to the configured Google Drive folder

## Credentials Required

| Service | Credential Name | Type | Used For |
|---------|----------------|------|----------|
| Google Drive | Google Drive account | Google Drive OAuth2 API | Uploading PDF to Drive |
| Oneflow | Oneflow | httpHeaderAuth (x-oneflow-api-token) | API calls to Oneflow |

## Configuration

- **Google Drive Folder ID**: `1Z4j_Y_8RURFbG_rVHn2inU2LOwIaCC9c`
- **Oneflow API Base URL**: `https://api.oneflow.com/v1`
- **Webhook Path**: `oneflow-signed-contracts`

## Oneflow Webhook Setup

In Oneflow, webhook ID 20565 is configured with:
- **URL**: `https://legalfly.app.n8n.cloud/webhook/oneflow-signed-contracts`
- **Event filter**: `EVENT_TYPE = contract:sign` (only sends contract:sign events)

## n8n Instance

- **Workflow ID**: `00YFVcmBURJZ3cGU`
- **URL**: https://legalfly.app.n8n.cloud/workflow/00YFVcmBURJZ3cGU
- **Status**: Active
