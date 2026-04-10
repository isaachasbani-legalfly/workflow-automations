# HubSpot SE Assignment → Linear Ticket

Automatically creates a Linear ticket in the Solutions Engineering backlog when a Solutions Engineer is assigned to a HubSpot deal. If the SE changes on a deal that already has a ticket, updates the assignee and logs the change with a comment.

## Trigger

Webhook: `POST /hubspot-se-assignment`

HubSpot Private App webhook subscription configured manually to send `deal.propertyChange` events for the `solutions_engineer` property to the n8n webhook URL.

## How It Works

1. **HubSpot Webhook** receives the POST from HubSpot when `solutions_engineer` changes on any deal
2. **Extract Event** parses the HubSpot event array (takes first event), extracts objectId, propertyValue, portalId
3. **SE Empty Guard** skips processing if the SE was removed (value cleared)
4. **Fetch Deal Details** retrieves deal properties (name, stage, amount, close date, AE owner)
5. **Resolve SE/AE Names** looks up HubSpot owner names for the SE and AE (AE resolved via `Fetch Deal Details` node reference for `hubspot_owner_id`)
6. **Map SE → Linear User** matches the SE to a Linear user by HubSpot ID
7. **Search Linear** queries Linear GraphQL `attachments` API to find issues with an attachment URL containing the deal ID
8. **Find Existing Ticket** filters attachment results to the Solutions Engineering team
9. **Branch**:
   - **No ticket** → Creates Linear issue in Backlog + adds HubSpot link as attachment + sends Slack notification
   - **Ticket exists** → Updates assignee + adds reassignment comment (no Slack notification)

### Slack Notification (new ticket only)

After a new Linear ticket is created and the HubSpot link is attached, the workflow sends a Slack notification to channel `C09QT2RFSBW`:

10. **Lookup SE in Slack** resolves the SE's email to a Slack user ID via `users.lookupByEmail`
11. **Build Slack Message** composes a message tagging the SE and @dm (`U043SB6G5C5`) with the deal link and Linear ticket link
12. **Post to Slack** sends the message to the channel

This notification only fires on new ticket creation, not on reassignment of existing tickets.

## SE → Linear User Mappings

| HubSpot SE | HubSpot ID | Linear User ID |
|---|---|---|
| Harry Day | 1891381453 | `127d293c-86a5-439d-be0e-31e59022e840` |
| Anastasia Screve | 29995860 | `633f0c0d-c45d-4fa1-b75d-0cced134efe3` |
| Stephanie Adriaens | 32163999 | `1f66b0f0-3476-43dc-8c99-1899dcf4f79a` |

## Credentials Required

| Service | Credential Type | Used By |
|---|---|---|
| HubSpot | hubspotAppToken (`5ww8XNGf4HTQu4UI` / "hubspot") | HTTP Request nodes (deal + owner API calls) |
| Linear | linearApi (`Hy0y7IGsd1kE4waU` / "Linear account") | Linear nodes + GraphQL search HTTP Request |
| Slack | slackApi (`lYs0WHzWk4c7z9Kk` / "Slack") | SE user lookup + post notification to channel |

**Required HubSpot scope**: `crm.objects.owners.read` (for Resolve SE Name / Resolve AE Name API calls)

**Required Slack scopes**: `chat:write` (post messages to channel), `users:read.email` (look up Slack user by email)

## Error Handling

Error handler workflow connected: `TA6Iq4wMW0KYsCiH`. All unhandled errors in this workflow are forwarded to the error handler for alerting.

## n8n Instance

- **Workflow ID**: `UNU5IniUPrnckW91`
- **URL**: https://legalfly.app.n8n.cloud/workflow/UNU5IniUPrnckW91
- **Portal ID**: 142047914
- **Linear Team**: Solutions Engineering (`fa68a3c7-fcbe-407e-8d66-94b572c31522`)
- **Linear Backlog State**: `3a018a5f-a622-41f6-b989-f63dfc3a9d99`
