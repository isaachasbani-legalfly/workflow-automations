# Changelog

## v1.0 — 2026-03-30

Initial release.

- Monthly schedule trigger: 1st of each month at 8:00 AM UTC
- Fetches customer companies with `contract___renewal_date` in the current month
- Formats Slack digest sorted by renewal date with HubSpot links
- Posts to Slack channel `C0APNNQAN5B`
- Silent when no renewals found
- Error workflow linked to shared error handler (`TA6Iq4wMW0KYsCiH`)
