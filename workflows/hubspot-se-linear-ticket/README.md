# HubSpot SE Assignment → Linear Ticket

Automatically creates a Linear ticket in the Solutions Engineering backlog when a Solutions Engineer is assigned to a HubSpot deal. If the SE changes on a deal that already has a ticket, updates the assignee and logs the change with a comment.

## Trigger

HubSpot webhook: `deal.propertyChange` on `solutions_engineer`

## How It Works

1. **HubSpot Trigger** fires when `solutions_engineer` property changes on any deal
2. **SE Empty Guard** skips processing if the SE was removed (value cleared)
3. **Fetch Deal Details** retrieves deal properties (name, stage, amount, close date, AE owner)
4. **Resolve SE/AE Names** looks up HubSpot owner names for the SE and AE
5. **Map SE → Linear User** matches the SE to a Linear user by HubSpot ID
6. **Search Linear** checks if a ticket already exists for this deal (`hs-deal-id:` tag)
7. **Branch**:
   - **No ticket** → Creates Linear issue in Backlog + adds HubSpot link
   - **Ticket exists** → Updates assignee + adds reassignment comment

## SE → Linear User Mappings

| HubSpot SE | HubSpot ID | Linear User ID |
|---|---|---|
| Harry Day | 1891381453 | `127d293c-86a5-439d-be0e-31e59022e840` |
| Anastasia Screve | 29995860 | `633f0c0d-c45d-4fa1-b75d-0cced134efe3` |
| Stephanie Adriaens | 32163999 | `1f66b0f0-3476-43dc-8c99-1899dcf4f79a` |

## Credentials Required

| Service | Credential Type | Used By |
|---|---|---|
| HubSpot | hubspotDeveloperApi | Trigger node |
| HubSpot | hubspotAppToken | HTTP Request nodes (deal + owner API calls) |
| Linear | linearApi (API token) | Create/Update/Comment nodes |
| Linear | linearApi (predefined) | GraphQL search HTTP Request |

## n8n Instance

- **Workflow ID**: `UNU5IniUPrnckW91`
- **URL**: https://legalfly.app.n8n.cloud/workflow/UNU5IniUPrnckW91
- **Portal ID**: 142047914
- **Linear Team**: Solutions Engineering (`fa68a3c7-fcbe-407e-8d66-94b572c31522`)
- **Linear Backlog State**: `3a018a5f-a622-41f6-b989-f63dfc3a9d99`
