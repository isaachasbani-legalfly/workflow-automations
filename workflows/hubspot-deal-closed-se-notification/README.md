# HubSpot Deal Sales Closed -> SE Linear Ticket Reminder

When a HubSpot deal moves to **06_Sales Closed** in the Sales Pipeline with a Solutions Engineer assigned, this workflow checks if the SE-Linear workflow created a Linear ticket for that deal. If found, it posts a Slack message reminding the SE to update the ticket and move it to Done.

## Trigger

HubSpot internal workflow triggers when a deal enters stage **06_Sales Closed** (`2406692058`). The HubSpot workflow sends a webhook with the deal ID to this n8n workflow.

**Webhook path**: `/webhook/hubspot-deal-closed-se`

## How It Works

1. HubSpot workflow sends deal ID when a deal enters 06_Sales Closed
2. **Normalize**: Extract deal ID from webhook payload
3. **Fetch deal**: Get deal properties including `solutions_engineer`
4. **Guard**: Stop if no SE is assigned
5. **Resolve SE name**: Look up the SE's first/last name via HubSpot Owners API
6. **Search Linear**: GraphQL query on Linear's attachments API to find a ticket linked to this deal (same dedup pattern as the SE-Linear workflow)
7. **Filter to SE team**: Only match tickets in the Solutions Engineering team
8. **Guard**: Stop if no Linear ticket found
9. **Post Slack message**: Send a message to the channel with the deal name, SE name, Linear ticket link, and a CTA to update the ticket and move it to Done

## SE Mapping

Uses the same SE HubSpot owner IDs as the SE-Linear workflow:

| SE Name | HubSpot Owner ID |
|---------|-----------------|
| Harry Day | `1891381453` |
| Anastasia Screve | `29995860` |
| Stephanie Adriaens | `32163999` |

## Credentials

| Service | Credential Name | ID | Used For |
|---------|----------------|-----|----------|
| HubSpot | hubspot | `5ww8XNGf4HTQu4UI` | Deal details, SE name resolution |
| Linear | Linear account | `Hy0y7IGsd1kE4waU` | Ticket search via GraphQL |
| Slack | Slack | `lYs0WHzWk4c7z9Kk` | Post message to channel |

## n8n Instance

- **Workflow ID**: `WGDJgIZwuuv15uf4`
- **URL**: https://legalfly.app.n8n.cloud/workflow/WGDJgIZwuuv15uf4
- **Error handler**: `TA6Iq4wMW0KYsCiH`
- **Slack channel**: `C0AFSAD1E5A`

## Related Workflows

- [hubspot-se-linear-ticket](../hubspot-se-linear-ticket/) -- Creates the Linear tickets this workflow checks for
