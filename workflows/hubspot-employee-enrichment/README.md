# HubSpot Employee Count Enrichment

Automatically enriches employee count data for newly created HubSpot companies using a two-track system. Track A fills in missing employee counts using Amplemarket. Track B identifies subsidiaries and writes their parent company's employee count to a separate property — so the sales team can see the real scale of a company even when HubSpot only has data for the local entity.

---

## Why it exists

When a new company is created in HubSpot, the `numberofemployees` field is often empty. For subsidiaries like "KPMG Spain" or "Cigna Healthcare International Health", even when the field is populated, the number reflects the local office rather than the parent organization. The sales team needs both: the company's own headcount for sizing, and the parent group's headcount for understanding the real opportunity.

A previous version (v3.0) tried to solve this by overwriting `numberofemployees` with the parent count for subsidiaries — which destroyed the company's real data. v4.0 fixes this by keeping the two concerns completely separate.

---

## When it runs

Every day at **02:01 AM** (Europe/London), it picks up all companies created **the previous day** and processes them.

---

## What it does, step by step

### 1. Fetch yesterday's companies (with pagination)

Queries HubSpot for all companies created in the previous 24-hour window. Uses cursor-based pagination (200 per page) to handle any daily volume. Fetches `name`, `domain`, `numberofemployees`, and `parent_company_number_of_employees` for each company.

### 2. Classify subsidiaries with Gemini

All company names are sent to **Google Gemini 2.5 Pro** in a single batch API call. Gemini classifies each company as either **independent** or **subsidiary**, and assigns a confidence level:

- **HIGH** — Strong evidence like well-known parent-subsidiary relationships (Instagram -> Meta, KPMG Spain -> KPMG)
- **MEDIUM** — Probable but not certain (Keepmoat Regeneration -> Keepmoat)

A self-referencing block prevents a company from being classified as subsidiary of itself (e.g., "Kotak Mahindra" -> "Kotak Mahindra").

### 3. Look up parent companies in HubSpot

For each subsidiary, the workflow searches HubSpot for the parent company by domain. If found:
- The `parent_company_name` field is set to a **clickable HubSpot URL** linking to the parent record
- The parent's `numberofemployees` is captured for later use (avoids an unnecessary Amplemarket API call)

If not found, the parent name is stored as plain text.

### 4. Route into Track A or Track B

Each company goes into exactly one track — they are **mutually exclusive**:

| Track | Condition | Purpose |
|-------|-----------|---------|
| **A** | No employee count | Get the company's own headcount |
| **B** | Has employee count AND is subsidiary | Get the parent group's headcount |
| Skip | Has employee count AND is independent | Nothing to do |

Companies without employee counts are assumed to be small and unlikely to be meaningful subsidiaries, so they only get Track A treatment.

### 5. Batch enrichment via Amplemarket

All domains that need enrichment are collected into a **single Amplemarket batch request**:
- Track A domains: the company's own domain (to get its employee count)
- Track B domains: the parent company's domain (to get the group employee count) — but only if the parent wasn't already found in HubSpot with an employee count

The workflow polls the Amplemarket API every 15 seconds until results are ready.

### 6. Smart merge

Results are mapped back to each company:

- **Track A**: If Amplemarket returns a count, use it. Otherwise, default to **5** (tagged as "Estimated" so it can be identified later). The default of 5 is a non-zero value chosen for CRM segmentation purposes.
- **Track B**: Priority is **HubSpot parent count > Amplemarket parent count**. The group count is only written if it's **greater than the company's own count** — this prevents writing misleadingly small values (e.g., Amplemarket returning 35 for a company that actually has 80,000 employees).

### 7. Conditional HubSpot update

The write rules depend on confidence:

| Confidence | Parent in HubSpot | Action |
|---|---|---|
| HIGH | Yes or No | Auto-write to HubSpot |
| MEDIUM | Yes (with employee count) | Auto-write — HubSpot validates the relationship |
| MEDIUM | No | Skip entirely, flag in Slack for manual review |

Track A always writes (no confidence check needed — it's just the company's own count). Track B follows the confidence rules above.

**What gets written:**

| Track | Properties written |
|-------|-------------------|
| A | `numberofemployees`, `numberemployeesernichmentsource` |
| B | `is_subsidiary`, `parent_company_name`, `parent_company_number_of_employees` |

`numberofemployees` is **never** touched by Track B.

### 8. Slack summary

A single Slack message is posted to the production channel with:
- **Track A section** — companies that got employee counts, showing source (Amplemarket or default)
- **Track B section** — subsidiaries that got group-level counts, showing data source (HubSpot or Amplemarket) and parent company link
- **Needs review section** — medium-confidence subsidiaries whose parent wasn't found in HubSpot, listed for manual verification

---

## HubSpot Properties

| Property | Internal Name | Type | Track | Notes |
|----------|--------------|------|-------|-------|
| Employee Count | `numberofemployees` | Standard | A | Only written when no existing value |
| Enrichment Source | `numberemployeesernichmentsource` | Custom | A | `Amplemarket` or `Estimated` |
| Is Subsidiary | `is_subsidiary` | Custom | B | Checkbox |
| Parent Company | `parent_company_name` | Custom | B | Clickable HubSpot URL if parent found in CRM |
| Parent Company Employee Count | `parent_company_number_of_employees` | Custom | B | Parent/group-level count (from HubSpot or Amplemarket) |

---

## Credentials Required

| Service | What it's used for |
|---------|-------------------|
| HubSpot | Read company data, write enrichment results, look up parent companies |
| Amplemarket | Batch employee count enrichment by domain |
| Google Gemini | AI classification of independent vs subsidiary |
| Slack | Daily summary message |

---

## Error Handling

Workflow errors send a Slack alert via the shared **Error Handler - Slack Notification** workflow (`TA6Iq4wMW0KYsCiH`).

| Node | Strategy |
|------|----------|
| Gemini Classify | Retry 3x/3s, 180s timeout for large batches |
| Submit Batch (Amplemarket) | Retry 3x/2s |
| Poll Status (Amplemarket) | Retry 2x/3s, continue on fail |
| Parse Classification | Try/catch — falls back to all-independent on parse failure |
| PATCH HubSpot | Continue on fail — errors don't block other companies |

---

## Files in this folder

| File | Purpose |
|------|---------|
| `README.md` | This file — overview, setup, credentials |
| `ARCHITECTURE-v4.0.md` | Full technical reference: all nodes, routing logic, design decisions |
| `architecture.mmd` | Mermaid diagram source |
| `workflow-v4.0.json` | n8n workflow export — source of truth |
| `CHANGELOG.md` | Version history (v1.0 through v4.0) |
| `prompts/prompt-classify-batch.md` | Gemini classification prompt with confidence tiers |

---

## n8n Instance

- **Workflow ID**: `TxZMblqjvC86tHAu`
- **URL**: https://legalfly.app.n8n.cloud/workflow/TxZMblqjvC86tHAu
- **Error Workflow**: `TA6Iq4wMW0KYsCiH` (Error Handler - Slack Notification)
- **Status**: Active (production)
- **v1.0 (rollback)**: `u9IcVLMFzBO6Idkw`
