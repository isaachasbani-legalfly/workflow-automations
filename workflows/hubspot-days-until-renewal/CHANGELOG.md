# Changelog

## v1.0 — 2026-03-30

Initial release.

- Daily schedule trigger at 12:05 AM UTC
- Paginated fetch of all customer companies with `contract___renewal_date` set
- Date calculation: exact days (string) or "Overdue" for past dates
- Batch update via HubSpot `/companies/batch/update` API (batches of 100)
- Error workflow linked to shared error handler (`TA6Iq4wMW0KYsCiH`)
