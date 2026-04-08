# Changelog

## v1.0 — 2026-04-08

- Initial implementation
- HubSpot trigger on `deal.propertyChange` for `solutions_engineer`
- Guard for empty SE values
- Deal + owner resolution via HubSpot REST API
- SE → Linear user mapping (Harry Day, Anastasia Screve, Stephanie Adriaens)
- Linear dedup via GraphQL `issueSearch` with `hs-deal-id:` tag
- Create issue path: Linear issue in Solutions Engineering backlog + HubSpot link
- Update path: Reassign + comment on existing ticket
- HTTP Request retry (3 attempts, 1s interval)
