# n8n Billing Automation Workflow — Specification

**Owner:** Finance Team, LegalFly
**Version:** 2.0
**Date:** March 20, 2026

---

## Problem Statement

Finance currently has no automated way to know which clients need to be invoiced each month, whether the invoice amount has changed due to new deals, or whether a billing cycle coincides with a contract renewal. This leads to missed invoices, incorrect amounts, and manual cross-referencing between HubSpot and Exact Online.

## Goals

1. **Eliminate missed invoices** — every client due for billing in a given month is surfaced automatically.
2. **Surface deal changes** — any closed-won deal (expansion, upsell, downsell, churn) since the last invoice is listed so Finance can check whether the amount needs adjusting.
3. **Distinguish renewals from regular billing** — renewal invoices often require contract checks; regular billing cycles do not.
4. **Handle new clients automatically** — the first time a company becomes a Customer in HubSpot and gets invoiced, it enters the billing cycle with no manual setup.
5. **Surface missing data** — companies that should be invoiced but have no Last Invoice Date are flagged for manual backfill.

## Non-Goals

- Automatically generating or sending invoices in Exact Online (Finance retains manual control).
- Computing expected invoice amounts (Finance determines the correct amount based on the deal delta).
- Checking outstanding receivables or payment collection (separate process).
- Handling one-time pilot billing (pilots are invoiced ad hoc, not on a recurring schedule).
- Updating HubSpot deal properties (this workflow is read-only, except for the post-invoicing sync in Workflow B).

---

## Data Model (Prerequisites in HubSpot)

### Company-level properties

| Property | Internal Name | Type | Description |
|---|---|---|---|
| Lifecycle Stage | `lifecyclestage` | Enum | Must be `customer` to be included in the workflow |
| Contract - Payment Schedule | `contract___payment_schedule` | Enum | `Annual`, `Quarterly`, `Half Yearly`, `One Time (Pilot)` |
| Contract - Latest Renewal Date | `contract___renewal_date` | Date | The next contract renewal/anniversary date |
| Contract - Service Start Date | `contract___service_start_date` | Date | Original service start date |
| Last Invoice Date | `last_invoice_date` | Date | Date of the most recent invoice from Exact Online |
| Last Invoice Number | `last_invoice_number` | Text | Invoice number of the most recent invoice |
| Exact Online Customer Code | `exact_online_customer_code` | Text | Customer code used to match with Exact Online |
| ARR | `arr` | Number | Annual Recurring Revenue |

### Deal-level properties

| Property | Internal Name | Type | Values |
|---|---|---|---|
| Deal Stage | `dealstage` | Enum | `closedwon` (Sales), `4914500817` (Expansion) |
| Pipeline | `pipeline` | Enum | `default` (Sales), `3585124587` (Expansion) |
| Deal Type | `dealtype` | Enum | `newbusiness`, `existingbusiness`, `Pilot`, `Churn`, `Downsell`, `Referral Partnership`, `Reseller Partnership`, `Partner - Whitelabel Partnership` |
| Expansion Type | `expansion_type` | Enum | `Seat`, `Department`, `Entity`, `Module`, `Renewal` |
| Billed Date | `billed_date` | Date | When the deal was last billed |
| co-term | `coterm` | Text | Whether expansion was co-termed *(property exists but not yet reliably populated)* |

---

## Workflow A: Monthly Billing Prep Report

### Step 1: Trigger

**Node type:** Schedule Trigger
**Cron:** `0 8 1 * *` — 1st of each month, 8:00 AM CET

---

### Step 2: Define Billing Window

**Node type:** Code (JavaScript)

```javascript
const now = new Date();
const windowStart = new Date(now.getFullYear(), now.getMonth(), 1);
const windowEnd = new Date(now.getFullYear(), now.getMonth() + 1, 0);
```

---

### Step 3: Fetch All Active Customers from HubSpot

**Node type:** HubSpot → Search CRM Objects
**Object:** Companies
**Filters (AND):**
- `lifecyclestage` EQ `customer`
- `contract___payment_schedule` is known
- `contract___payment_schedule` NOT EQ `One Time (Pilot)`

**Properties to fetch:**
- `name`, `hs_object_id`, `contract___payment_schedule`, `contract___renewal_date`, `contract___service_start_date`, `last_invoice_date`, `last_invoice_number`, `exact_online_customer_code`, `arr`

**Pagination:** HubSpot returns max 100 per page. Loop until all results are fetched.

---

### Step 4: Calculate Next Invoice Date per Company

**Node type:** Code (JavaScript)

For each company, calculate when their next invoice is due based on `last_invoice_date` + payment schedule interval:

```javascript
function getNextInvoiceDate(lastInvoiceDate, schedule) {
    const last = new Date(lastInvoiceDate);
    switch (schedule) {
        case 'Annual':      last.setMonth(last.getMonth() + 12); break;
        case 'Half Yearly': last.setMonth(last.getMonth() + 6); break;
        case 'Quarterly':   last.setMonth(last.getMonth() + 3); break;
    }
    return last;
}
```

**Split into three groups:**

| Group | Condition | Action |
|---|---|---|
| **Due this month** | `nextInvoiceDate` falls within `windowStart` to `windowEnd` | Include in billing report |
| **No Last Invoice Date** | `last_invoice_date` is empty | Flag as `⚠️ MISSING INVOICE HISTORY` |
| **Not due** | `nextInvoiceDate` is after `windowEnd` | Exclude |

---

### Step 5: Flag Renewal vs. Regular Billing

**Node type:** Code (JavaScript)

For each company due this month, check if `contract___renewal_date` falls within the billing window:

```javascript
const isRenewal = (renewalDate >= windowStart && renewalDate <= windowEnd);
const billingType = isRenewal ? '🔄 RENEWAL' : '📋 Regular Billing';
```

Renewals are flagged because they may require a new contract, price renegotiation, or updated terms.

---

### Step 6: Fetch Closed-Won Deals Since Last Invoice

**Node type:** HubSpot → Search CRM Objects (loop per company)
**Object:** Deals
**Filters:**
- Associated with the company (use `associatedWith` filter)
- `dealstage` IN [`closedwon`, `4914500817`]
- `closedate` GT company's `last_invoice_date`
- `closedate` LTE `windowEnd`

**Properties to fetch:**
- `dealname`, `amount`, `pipeline`, `closedate`, `dealtype`, `expansion_type`, `coterm`

This captures every deal that closed between the last invoice and now — the "deal delta."

---

### Step 7: Classify Each Deal Using `dealtype`

**Node type:** Code (JavaScript)

Use the `dealtype` property to classify each deal. Do NOT rely on deal name parsing.

| `dealtype` value | Classification | Flag |
|---|---|---|
| `Churn` | Churn | 🔴 |
| `Downsell` | Downsell | 📉 |
| `existingbusiness` | Expansion / Upsell | 📈 |
| `newbusiness` | New Business | 🆕 |
| `Pilot` | Pilot | 🧪 |

Additionally, if `expansion_type` = `Renewal`, add a 🔄 flag — this is a renewal deal, not just an expansion.

**Co-term note:** The `coterm` deal property exists but is not yet reliably populated. When it becomes available, add a 🔗 flag for co-termed deals. Until then, skip this flag.

---

### Step 8: Build the Monthly Billing Report

**Node type:** Code (JavaScript)

Compile all data into a structured report. Each company entry:

```json
{
    "companyName": "AB InBev",
    "billingType": "📋 Regular Billing",
    "paymentSchedule": "Quarterly",
    "lastInvoiceDate": "2025-11-25",
    "lastInvoiceNumber": "202500305",
    "nextInvoiceDate": "2026-02-25",
    "arr": 24380,
    "dealsDelta": [
        {
            "dealName": "AB InBev - 2 extra seats",
            "amount": 4776,
            "dealType": "existingbusiness",
            "classification": "📈 Expansion (Upsell)",
            "expansionType": "Seat",
            "closeDate": "2026-01-15"
        }
    ]
}
```

---

### Step 9: Sort and Prioritize

**Node type:** Code (JavaScript)

Sort the billing list into sections by priority:

1. **⚠️ MISSING INVOICE HISTORY** — companies with no `last_invoice_date` (need manual backfill or first invoice)
2. **🔴 CHURN / DOWNSELL** — deal delta contains churn or downsell deals (amount likely decreased)
3. **🔄 RENEWALS** — billing coincides with contract renewal date
4. **📈 EXPANSIONS / NEW DEALS** — deal delta contains upsells or new business (amount likely increased)
5. **📋 REGULAR BILLING** — no deal changes since last invoice

---

### Step 10: Deliver the Report

**Node type:** Slack → Send Message (and/or Email, and/or Google Sheet)

Format as a structured message:

```
📅 Monthly Billing Prep — April 2026
23 clients to invoice this month

⚠️ MISSING INVOICE HISTORY (3)
• Conexus AS — Annual — ARR: €12,164 — no Last Invoice Date on file
• Fórmula Aclamada — Annual — ARR: €12,164 — no Last Invoice Date on file
• Sihold NV — Annual — ARR: €12,632 — no Last Invoice Date on file

🔴 CHURN / DOWNSELL (2)
• Acerta — dealtype: Churn — "Acerta - Credit Note 3 users" (€-7,164) closed Jan 15
• PIA Group — dealtype: Downsell — "PIA Group - Downsell" (€-26,508) closed Oct 10

🔄 RENEWALS (3)
• KPMG S.A. — Renewal Apr 1 — No deal changes since last invoice
• DAS Rechtsbijstand — Renewal Apr 10 — 📈 1 expansion: "DAS Expansion" (+€2,388)
• Club Brugge — Renewal Apr 3 — No deal changes

📈 EXPANSIONS / NEW DEALS (4)
• AB InBev — 1 deal since last invoice: "AB InBev - 2 extra seats" (+€4,776, Seat expansion)
• Securex — 1 deal: "Securex - expansion to 22 seats" (+€25,752, Seat expansion)
• ...

📋 REGULAR BILLING — NO CHANGES (11)
• Ackermans & van Haaren — Quarterly — Last invoice: Dec 22, 2025 (#202500340)
• Curaleaf — Quarterly — Last invoice: Jan 2, 2026 (#26700001)
• WDP — Quarterly — Last invoice: Jan 26, 2026 (#26700025)
• ...
```

---

## Workflow B: Post-Invoicing HubSpot Sync

A separate workflow that keeps `Last Invoice Date` and `Last Invoice Number` up to date.

### Step 1: Trigger

**Node type:** Schedule Trigger
**Cron:** `0 9 * * *` — runs daily at 9:00 AM CET

---

### Step 2: Query Exact Online for Recent Invoices

**Node type:** Exact Online → List Sales Invoices
**Filter:** `InvoiceDate ge datetime'[yesterday]'`
**Select:** `InvoiceNumber`, `InvoiceDate`, `InvoiceToName`, `AmountDC`, `StatusDescription`

Only process invoices with `StatusDescription` = `Processed` (skip drafts).

---

### Step 3: Match to HubSpot Companies

**Node type:** Code (JavaScript) + HubSpot → Search CRM Objects

For each new invoice:
1. Extract the customer name from `InvoiceToName`
2. Search HubSpot companies by `exact_online_customer_code` OR by company name fuzzy match
3. If matched, proceed to update. If not matched, add to an "unmatched" list for manual review.

**Known name mismatches** (build into the matching logic or maintain a lookup table):

| Exact Online Name | HubSpot Company Name |
|---|---|
| ACERTA BV | Acerta |
| VANDELANOTTE ++ BV | Vandelanotte |
| VANDELANOTTE ++ NV | Vandelanotte |
| N.V. BESIX S.A. | BESIX |
| InBev Belgium BV | AB InBev |
| KBC GLOBAL SERVICES NV | KBC Bank & Verzekering |
| ZIEKENHUIS AAN DE STROOM VZW | ZAS |
| *(full mapping of ~100 entries available — see import matching file)* |

**Preferred matching:** Use `Exact Online Customer Code` on the company record. Query EO accounts to get the code, then match to HubSpot. This is more reliable than name matching.

---

### Step 4: Update HubSpot Company

**Node type:** HubSpot → Update CRM Object
**Object:** Company

For each matched invoice:
- Set `last_invoice_date` = `InvoiceDate`
- Set `last_invoice_number` = `InvoiceNumber`

**Only update if** the new `InvoiceDate` is more recent than the existing `last_invoice_date` (don't overwrite with older data).

---

### Step 5: Report Unmatched Invoices

If any invoices couldn't be matched to a HubSpot company, send a notification:

```
⚠️ Unmatched Exact Online invoices (manual review needed):
• #26700075 — "New Customer GmbH" — €12,164 — Mar 21, 2026
```

This catches new clients that don't yet have an `Exact Online Customer Code` in HubSpot.

---

## New Client Handling

New clients enter the billing cycle automatically through this flow:

1. **Sales closes a deal** → company `lifecyclestage` is set to `customer` in HubSpot
2. **Finance creates the first invoice** in Exact Online
3. **Workflow B (daily sync)** detects the new invoice and populates `last_invoice_date` + `last_invoice_number` on the company
4. **Next month's Workflow A** picks up the company in the billing list based on `last_invoice_date` + `contract___payment_schedule`

No manual setup required. The only prerequisite is that the company has `contract___payment_schedule` set and `lifecyclestage` = `customer`.

**First month edge case:** If the company has no `Last Invoice Date` yet (deal just closed, invoice not yet generated), Workflow A flags it as `⚠️ MISSING INVOICE HISTORY` so Finance knows to create the first invoice.

---

## Handling the 38 Companies with Missing Invoice History

There are currently 38 active companies with no `Last Invoice Date` because their last invoice predates the Exact Online API window (before Sep 2025). These are predominantly annual customers.

**Recommended approach:** Export the full invoice history from Exact Online's web UI (not the API) as a CSV. Filter for these 38 customer codes and extract the most recent invoice per customer. Then import into HubSpot to backfill `last_invoice_date` and `last_invoice_number`.

**Customer codes for the 38 companies:**

| Company | EO Code | ARR |
|---|---|---|
| European Commission | 10826 | €137,868 |
| Vision Invest | 12539 | €88,880 |
| Deutsche Lufthansa AG | 12872 | €73,124 |
| Credendo | 12931 | €66,820 |
| SD Worx | 12553 | €59,760 |
| Wacker Chemie AG | 12002 | €55,000 |
| Emerald Clinical | 12486 | €43,656 |
| WEALINS S.A. | 11998 | €36,000 |
| Neon Eight Holdings | 12938 | €35,613 |
| Bolton Group | 12614 | €33,940 |
| Ferrero | 5902 | €30,890 |
| SSOE Group | *(missing)* | €26,230 |
| Delen Private Bank | 12470 | €25,026 |
| Clerprem | *(missing)* | €22,164 |
| ECE | 12592 | €21,716 |
| Franki Foundations | 12613 | €19,449 |
| Fisher & Paykel | 12564 | €19,328 |
| AXA Group Operations | 12612 | €15,000 |
| GEOxyz | 12279 | €14,552 |
| Agrati Group | 12562 | €14,552 |
| iO Digital | 12357 | €13,358 |
| Sihold NV | 12851 | €12,632 |
| Zehnder Group International AG | 12519 | €12,164 |
| Pierstone | 7668 | €12,164 |
| KOMOREBI Capital | 12472 | €12,164 |
| Kerkstoel NV | 12484 | €12,164 |
| Fórmula Aclamada | 12362 | €12,164 |
| ESET | 12569 | €12,164 |
| Conexus AS | 12358 | €12,164 |
| Proper | 12593 | €11,980 |
| Arbor Investments | 12476 | €11,434 |
| Public Investment Fund (PIF) | 1236 | €10,000 |
| Elsyca | 12478 | €9,776 |
| CentroMotion | 12815 | €8,488 |
| Younique Concepts | 13260 | €7,500 |
| INTINYA | 12607 | €7,388 |
| Impact Europe | 12360 | €4,999 |
| Kelly Wearstler | 12594 | €2,768 |

**Note:** SSOE Group and Clerprem have no EO customer code in HubSpot. SSOE Group needs one assigned. Clerprem does not exist in Exact Online at all — verify whether they are billed under a different entity name.

---

## Edge Cases

| Scenario | Handling |
|---|---|
| **Company has no Last Invoice Date** | Appears in `⚠️ MISSING INVOICE HISTORY` section of monthly report |
| **Multiple deals in billing period** | All deals listed individually with their `dealtype` classification |
| **Churn deal reduces ARR to €0** | Flagged as 🔴; Finance confirms whether to skip invoicing entirely |
| **Mid-month billing** | The window covers the full month (1st to last day), so mid-month dates are captured |
| **Company changes payment schedule** | Next invoice date recalculates automatically based on current schedule + last invoice date |
| **Credit note in billing period** | Deals with `dealtype` = `Churn` or `Downsell` with negative amounts are included in the delta |
| **New client, no invoice yet** | Flagged in monthly report; enters regular cycle after Workflow B syncs the first invoice |
| **Multiple EO accounts for one company** | *(Open question — see below)* |
| **Co-termed expansion** | *(Open question — `coterm` field not yet reliable; will be flagged once available)* |

---

## Reference

### Pipeline IDs

| Pipeline | Internal ID | Closed Won Stage |
|---|---|---|
| Sales Pipeline | `default` | `closedwon` |
| Expansion Pipeline | `3585124587` | `4914500817` |

### Deal Type Values (`dealtype`)

| Value | Label | Meaning |
|---|---|---|
| `newbusiness` | New Business | First deal with this company |
| `existingbusiness` | Existing Business | Expansion / upsell |
| `Pilot` | Pilot | Trial engagement |
| `Churn` | Churn | Customer leaving |
| `Downsell` | Downsell | Reducing scope |
| `Referral Partnership` | Partner - Referral | Partner deal |
| `Reseller Partnership` | Partner - Reselling | Partner deal |
| `Partner - Whitelabel Partnership` | Partner - Whitelabel | Partner deal |

### Expansion Type Values (`expansion_type`)

| Value | Meaning |
|---|---|
| `Seat` | Additional user seats |
| `Department` | New department onboarded |
| `Entity` | New legal entity |
| `Module` | New product module |
| `Renewal` | Contract renewal |

### Payment Schedule Intervals

| Schedule | Interval | Invoices/Year |
|---|---|---|
| Annual | 12 months | 1 |
| Half Yearly | 6 months | 2 |
| Quarterly | 3 months | 4 |
| One Time (Pilot) | N/A — excluded from workflow | 0 |

### Company Lifecycle Stages (`lifecyclestage`)

| Value | Label |
|---|---|
| `customer` | Customer *(used as filter for this workflow)* |
| `1052271849` | Ex - Customer |
| `opportunity` | Opportunity |
| `1052271848` | Pilot |

---

## Open Questions

1. **Delivery channel** — Slack, email, spreadsheet, or a combination? *(Decision: Isaac)*
2. **Co-term field** — once the `coterm` property is reliably populated on expansion deals, add a 🔗 flag to the report so Finance knows to consolidate into a single invoice. *(Owner: Isaac — in progress)*
3. **Multiple Exact Online entities** — some companies (e.g., Vandelanotte with BV + NV accounts) have multiple EO accounts. Should the workflow consolidate invoices across entities or list them separately? *(Decision: Isaac/Finance)*
