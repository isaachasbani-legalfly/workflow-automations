# Changelog

## v1.0 - 2026-02-25

Initial release.

- 17-node workflow: Schedule Trigger -> HubSpot fetch -> Amplemarket enrichment -> Gemini branch detection -> HubSpot update -> Slack summary
- Amplemarket lookup cascade: LinkedIn URL first, domain fallback
- Gemini 2.5 Flash for branch/subsidiary detection (temperature 0.1)
- Writes `numberofemployees` + `number-employees-enrichment-source` to HubSpot
- Daily Slack summary with enrichment statistics
- Error workflow: `TA6Iq4wMW0KYsCiH` (Error Handler - Slack Notification)
- Retry config (3x/2s) on Amplemarket and Gemini API calls
- `continueRegularOutput` on Amplemarket nodes for graceful failure handling
