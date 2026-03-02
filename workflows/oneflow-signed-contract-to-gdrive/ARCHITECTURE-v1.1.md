# Oneflow Signed Contract to Google Drive -- Architecture v1.1

## Overview

When a contract is signed in Oneflow (all parties have signed), Oneflow sends a webhook event to this workflow. The workflow filters for `contract:sign` events, fetches the counterparty company name and signed contract PDF from the Oneflow API, and uploads it to a designated Google Drive folder named `{Company Name} - {YYYY-MM-DD}.pdf`.

## Workflow Diagram

```mermaid
flowchart LR
    A[Oneflow Webhook\nPOST /oneflow-signed-contracts] --> B{Filter: Is Contract Signed?\nbody.type == contract:sign}
    B -->|TRUE| C[Get Contract Parties\nGET /v1/contracts/id/parties]
    B -->|FALSE| X[Ignored]
    C --> D[Get Contract Files\nGET /v1/contracts/id/files]
    D --> E[Download Contract PDF\nGET _links.self.href?download=true]
    E --> F[Upload to Google Drive\nCompany - date.pdf]
```

## Node Reference

### Oneflow Webhook (`a1b2c3d4-webhook`)
- **Type**: n8n-nodes-base.webhook v2
- **Purpose**: Receives POST requests from Oneflow for all webhook events
- **Key config**: Path `oneflow-signed-contracts`, HTTP Method POST, responds immediately with 200
- **Output**: Full webhook payload including `body.type`, `body.data.subject.id`

### Filter: Is Contract Signed? (`a1b2c3d4-ifnode`)
- **Type**: n8n-nodes-base.if v2
- **Purpose**: Filters events to only process `contract:sign` webhooks
- **Key config**: Checks `$json.body.type` equals `contract:sign` (case-sensitive, strict type validation)
- **Output**: TRUE branch continues; FALSE branch is ignored

### Get Contract Parties (`a1b2c3d4-getparties`)
- **Type**: n8n-nodes-base.httpRequest v4.2
- **Purpose**: Fetches the list of parties (companies/individuals) on the contract to get the counterparty name
- **Key config**: GET `https://api.oneflow.com/v1/contracts/{{ $json.body.data.subject.id }}/parties` with httpHeaderAuth credential
- **Output**: JSON with `data[]` array containing party objects with `name`, `my_party` (boolean), `type`, `participants[]`

### Get Contract Files (`a1b2c3d4-getfiles`)
- **Type**: n8n-nodes-base.httpRequest v4.2
- **Purpose**: Fetches the list of files associated with the signed contract
- **Key config**: GET `https://api.oneflow.com/v1/contracts/{{ $('Oneflow Webhook').item.json.body.data.subject.id }}/files` — references webhook directly since `$json` now contains parties response
- **Output**: JSON with `data[]` array containing file objects with `_links.self.href`, `name`, `extension`

### Download Contract PDF (`a1b2c3d4-download`)
- **Type**: n8n-nodes-base.httpRequest v4.2
- **Purpose**: Downloads the actual PDF binary from the Oneflow file download URL
- **Key config**: GET `{{ $json.data[0]._links.self.href }}?download=true` with httpHeaderAuth credential, response format `file`, follow redirects enabled
- **Output**: Binary PDF data (Oneflow redirects to S3 presigned URL)

### Upload to Google Drive (`a1b2c3d4-upload`)
- **Type**: n8n-nodes-base.googleDrive v3
- **Purpose**: Uploads the downloaded PDF to Google Drive with a recognisable company name and date
- **Key config**: Upload operation, folder ID `1Z4j_Y_8RURFbG_rVHn2inU2LOwIaCC9c`
- **File name**: `={{ $('Get Contract Parties').item.json.data.filter(p => !p.my_party)[0].name }} - {{ $now.format('yyyy-MM-dd') }}.pdf` — filters parties to find the counterparty (`my_party: false`), uses their company name + today's date
- **Credential**: Google Drive account (googleDriveOAuth2Api, ID: AuwcL65OqohMjmOj)

### Workflow Info (`a1b2c3d4-sticky`)
- **Type**: n8n-nodes-base.stickyNote v1
- **Purpose**: In-workflow documentation

## Routing Logic

1. Webhook receives all Oneflow events -> passes to IF filter
2. IF filter checks `body.type == "contract:sign"` -> TRUE continues, FALSE stops
3. Get Contract Parties -> calls Oneflow API with contract ID -> passes party data downstream
4. Get Contract Files -> references webhook node directly for contract ID (since `$json` is now parties data) -> passes file list
5. Download Contract PDF -> uses HAL link from files response -> passes binary
6. Upload to Google Drive -> names file using counterparty name from parties response + current date

## Error Handling

- Default n8n error handling (workflow stops on error)
- HTTP Request nodes will fail if Oneflow API returns non-2xx status
- Google Drive upload will fail if credentials are invalid or folder doesn't exist
- IF filter silently drops non-`contract:sign` events

## Design Decisions

- **IF filter node**: Oneflow sends ALL webhook event types to the same URL. The filter ensures only `contract:sign` events are processed.
- **Stored httpHeaderAuth credential**: Uses n8n's credential store ("Oneflow") for security.
- **Counterparty name via parties API**: Calls `GET /contracts/{id}/parties` and filters for `my_party: false` to get the counterparty company name. This is used in the file name for easy identification.
- **Explicit webhook reference in Get Contract Files**: After inserting the Get Contract Parties node, `$json` in downstream nodes refers to the parties response, not the webhook. Get Contract Files uses `$('Oneflow Webhook').item.json...` to reach back to the original webhook payload for the contract ID.
- **Date in file name**: Uses `$now.format('yyyy-MM-dd')` for the current date when the workflow runs, providing a clear chronological reference.

## Credentials Required

| Service | Credential name | Used for |
|---------|----------------|---------|
| Google Drive | Google Drive account | Uploading signed PDF to Drive folder |
| Oneflow | Oneflow (httpHeaderAuth) | API calls to fetch parties, files, and download PDF |

## n8n Instance
- **Workflow ID**: `00YFVcmBURJZ3cGU`
- **URL**: https://legalfly.app.n8n.cloud/workflow/00YFVcmBURJZ3cGU
