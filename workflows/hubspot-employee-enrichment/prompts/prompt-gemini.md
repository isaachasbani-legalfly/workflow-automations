# Gemini Prompt: Branch/Subsidiary Detection

**Model**: gemini-2.5-flash
**Temperature**: 0.1
**Node**: Gemini Branch Check (`emp-gemini`)
**Purpose**: Detect if a company is a local branch/subsidiary and estimate the parent's global employee count

## Prompt Template

```
You are analyzing a company to determine its correct employee count.

Company: {companyName}
Domain: {domain}
Amplemarket reported employee count: {count}

Tasks:
1. Is this company a local branch, subsidiary, or regional office of a larger parent?
   - "KPMG Spain" is a branch of KPMG
   - "Google UK" is a branch of Alphabet/Google
   - Look for country names, city names, or region indicators in the company name
2. If it IS a branch: estimate the GLOBAL employee count of the parent organization
3. If it is NOT a branch: return the Amplemarket count as-is

Return ONLY valid JSON (no markdown, no explanation):
{"employeeCount": <number>, "isSubsidiary": true/false, "parentCompany": "<name or null>"}
```

## Variables

| Variable | Source | Description |
|----------|--------|-------------|
| `{companyName}` | `$json.companyName` | Company name from HubSpot |
| `{domain}` | `$json.domain` | Company domain from HubSpot |
| `{count}` | `$json.employeeCount` | Employee count from Amplemarket |

## Expected Output

```json
{"employeeCount": 265000, "isSubsidiary": true, "parentCompany": "KPMG"}
```

or

```json
{"employeeCount": 150, "isSubsidiary": false, "parentCompany": null}
```

## Enrichment Source Mapping

| `isSubsidiary` | `parentCompany` | Enrichment Source |
|----------------|-----------------|-------------------|
| `false` | `null` | `Amplemarket` |
| `true` | `"KPMG"` | `Amplemarket (parent: KPMG)` |
