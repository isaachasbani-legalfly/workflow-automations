# HubSpot Deal Closed -> SE Linear Ticket Reminder

When a HubSpot deal moves to **06_Sales Closed** in the Sales Pipeline with a Solutions Engineer assigned, this workflow checks if the SE-Linear workflow created a Linear ticket for that deal. If found, it posts a Slack message reminding the SE to update the ticket and move it to Done.

## Trigger

HubSpot internal workflow triggers when a deal enters stage **06_Sales Closed** (`2406692058`). The HubSpot workflow sends a webhook with the deal ID to this n8n workflow.

**Webhook path**: `/webhook/hubspot-deal-closed-se`

## How It Works

1. HubSpot workflow sends deal ID when a deal enters 06_Sales Closed
2. **Normalize**: Extract deal ID from webhook payload
3. **Fetch deal**: Get deal properties including `solutions_engineer`
4. **Guard**: Stop if no SE is assigned
5. **Resolve SE name**: Look up the SE's first/last name and email via HubSpot Owners API
6. **Lookup SE in Slack**: Find the SE's Slack user ID via `users.lookupByEmail` API (enables dynamic `<@U...>` tagging)
7. **Search Linear**: GraphQL query on Linear's attachments API to find a ticket linked to this deal (same dedup pattern as the SE-Linear workflow)
8. **Filter to SE team**: Only match tickets in the Solutions Engineering team
9. **Guard**: Stop if no Linear ticket found
10. **Post Slack message**: Send a "Deal Closed!" message to the channel with the deal name, SE Slack tag, Linear ticket link, and a CTA to update the ticket and move it to Done

## Nodes (13)

| # | Node | Type |
|---|------|------|
| 1 | Webhook: Deal Sales Closed | n8n-nodes-base.webhook |
| 2 | Normalize Webhook Data | n8n-nodes-base.code |
| 3 | Fetch Deal Details | n8n-nodes-base.httpRequest |
| 4 | SE Assigned? | n8n-nodes-base.if |
| 5 | Stop (No SE) | n8n-nodes-base.noOp |
| 6 | Resolve SE Name | n8n-nodes-base.httpRequest |
| 7 | Lookup SE in Slack | n8n-nodes-base.httpRequest |
| 8 | Search Linear for Ticket | n8n-nodes-base.httpRequest |
| 9 | Find Existing Ticket | n8n-nodes-base.code |
| 10 | Linear Ticket Found? | n8n-nodes-base.if |
| 11 | Stop (No Ticket) | n8n-nodes-base.noOp |
| 12 | Build Slack Message | n8n-nodes-base.code |
| 13 | Post to Slack | n8n-nodes-base.slack |

## Credentials

| Service | Credential Name | ID | Used For |
|---------|----------------|-----|----------|
| HubSpot | hubspot | `5ww8XNGf4HTQu4UI` | Deal details, SE name resolution |
| Linear | Linear account | `Hy0y7IGsd1kE4waU` | Ticket search via GraphQL |
| Slack | Slack | `lYs0WHzWk4c7z9Kk` | Post message to channel, SE user lookup |

## Slack Scopes Required

- `chat:write` -- Post messages to channel
- `users:read.email` -- Look up Slack user by email (`users.lookupByEmail`)

## n8n Instance

- **Workflow ID**: `WGDJgIZwuuv15uf4`
- **URL**: https://legalfly.app.n8n.cloud/workflow/WGDJgIZwuuv15uf4
- **Error handler**: `TA6Iq4wMW0KYsCiH`
- **Slack channel**: `C0AFSAD1E5A`

## Related Workflows

- [hubspot-se-linear-ticket](../hubspot-se-linear-ticket/) -- Creates the Linear tickets this workflow checks for
