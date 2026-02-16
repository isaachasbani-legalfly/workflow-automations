# HubSpot Company Industry Categorization - Deployment v3.3

**Date**: 2026-02-16
**Status**: ✅ Workflow Built - Ready for Deployment
**Workflow ID**: `8DM3CwXLxOT3G8B7`

---

## Deployment Summary

Built v3.3 workflow with **three independent LLM paths** for data-source-specific prompts.

**Workflow URL**: https://legalfly.app.n8n.cloud/workflow/8DM3CwXLxOT3G8B7

---

## Architecture Changes (v3.2.14 → v3.3.0)

### ✅ Three Independent LLM Paths

**OLD (v3.2.14)**: Single merged path
```
All enrichment sources → Merge → Prepare Gemini Input → Gemini → Parse
```

**NEW (v3.3.0)**: Three separate paths
```
Path 1: HubSpot Description → Prepare Gemini (HubSpot) → Gemini (HubSpot) → Parse (HubSpot)
Path 2: LinkedIn Enrichment → Prepare Gemini (LinkedIn) → Gemini (LinkedIn) → Parse (LinkedIn)
Path 3: Website Scraping → Prepare Gemini (Website) → Gemini (Website) → Parse (Website)
```

### Nodes Summary

| Category | Count | Change |
|----------|-------|--------|
| **Total Nodes** | 26 | +8 nodes |
| Trigger | 1 | No change |
| HubSpot | 3 | No change |
| IF Nodes | 6 | +2 (Check Has URL, Check Website Data) |
| Code Nodes | 7 | +4 (3× Prepare, 3× Parse, -1 Split) |
| HTTP Request | 7 | +3 (Amplemarket, 2× Gemini) |
| Set | 1 | No change |
| Slack | 2 | No change |

### Added Nodes (9)

1. **Check Has URL/Domain** - IF node to verify domain exists
2. **Amplemarket LinkedIn API** - HTTP Request for LinkedIn enrichment
3. **Check LinkedIn Data Retrieved** - IF node to verify LinkedIn response
4. **Prepare Gemini Input - LinkedIn** - LinkedIn-optimized prompt
5. **Gemini Categorization - LinkedIn** - Gemini API call
6. **Parse Response - LinkedIn** - Extract category from Gemini
7. **Check Website Data Retrieved** - IF node to verify website scraping
8. **Prepare Gemini Input - Website** - Website-optimized prompt
9. **Gemini Categorization - Website** - Gemini API call
10. **Parse Response - Website** - Extract category from Gemini

### Removed Nodes (1)

- **Merge Enrichment Branches** - No longer needed with independent paths

### Renamed Nodes (3)

- `Prepare Gemini Input` → `Prepare Gemini Input - HubSpot`
- `Gemini Categorization` → `Gemini Categorization - HubSpot`
- `Parse Gemini Response` → `Parse Gemini Response - HubSpot`

---

## Benefits

1. **Data-Source-Specific Prompts**: Each path optimized for its data format
2. **Independent Processing**: No data format conflicts between sources
3. **Preserved Working Logic**: HubSpot description path unchanged (proven working)
4. **Graceful Fallback**: LinkedIn → Website → Online Research cascade maintained
5. **Unified Convergence**: All paths converge at "Check Confidence" for consistent updates

---

## Credentials Required

### Existing (Configured) ✅
- HubSpot Private App Token (`5ww8XNGf4HTQu4UI`)
- Slack API (`lYs0WHzWk4c7z9Kk`)
- Google Gemini API (`QnUjlFnc4gypgplL`)

### New (Required) ⚠️
- **Amplemarket API Key** - HTTP Header Authentication
  - Node: "Amplemarket LinkedIn API"
  - Type: `genericCredentialType` → `httpHeaderAuth`
  - Header: `Authorization: Bearer YOUR_API_KEY`

---

## Next Steps

### 1. Configure Amplemarket Credentials

In n8n UI:
1. Go to https://legalfly.app.n8n.cloud/credentials
2. Create new credential: "HTTP Header Auth"
3. Name: "Amplemarket API"
4. Header Name: `Authorization`
5. Header Value: `Bearer YOUR_AMPLEMARKET_API_KEY`
6. Save credential

### 2. Deploy Workflow

**Option A: Manual Import (Recommended for first deployment)**
1. Open https://legalfly.app.n8n.cloud/workflow/8DM3CwXLxOT3G8B7
2. Click "..." → "Settings" → "Import Workflow"
3. Upload `workflow-v3.3.json`
4. Review all nodes
5. Link "Amplemarket LinkedIn API" node to Amplemarket credential
6. Save workflow

**Option B: n8n MCP Tool**
```bash
n8n_update_full_workflow({
  id: "8DM3CwXLxOT3G8B7",
  name: "HubSpot Company Industry Categorization v3.3",
  nodes: [...],  # from workflow-v3.3.json
  connections: {...}
})
```

### 3. Validate Deployment

```bash
# Validate workflow structure
n8n_validate_workflow({id: "8DM3CwXLxOT3G8B7"})

# Auto-fix common issues
n8n_autofix_workflow({id: "8DM3CwXLxOT3G8B7"})
```

### 4. Test Workflow

**Test Path 1: HubSpot Description**
- Find a company WITH description in HubSpot
- Trigger workflow manually
- Verify: Uses HubSpot path, no Amplemarket API call
- Check: Slack notification shows `Source: hubspot`

**Test Path 2: LinkedIn Enrichment**
- Find a company WITHOUT description, WITH domain
- Trigger workflow manually
- Verify: Calls Amplemarket API
- Check: Slack notification shows `Source: linkedin`

**Test Path 3: Website Scraping**
- Find a company WITHOUT description, simulate Amplemarket failure
- Trigger workflow manually
- Verify: Falls back to website scraping
- Check: Slack notification shows `Source: website`

### 5. Monitor First Run

- [ ] Wait for 6 PM daily trigger
- [ ] Monitor Slack #n8n-helper-test-bot
- [ ] Check n8n execution logs
- [ ] Verify HubSpot updates
- [ ] Review path distribution (which % use each path)

---

## Testing Checklist

- [ ] HubSpot description path works (existing logic preserved)
- [ ] LinkedIn enrichment path triggered correctly
- [ ] Website scraping path triggered as fallback
- [ ] Amplemarket API integration successful
- [ ] All three paths converge at Check Confidence
- [ ] HubSpot updates successful
- [ ] Slack notifications correct (show enrichment source)
- [ ] Manual review flagging works
- [ ] Error handling works (APIs fail gracefully)

---

## Rollback Plan

If issues arise:

**Option 1: Restore v3.2.14**
1. Open workflow in n8n UI
2. Click "..." → "Workflow History"
3. Find v3.2.14 (last working version)
4. Click "Restore"

**Option 2: Git Revert**
```bash
git checkout main
git revert HEAD
git push origin main
```

---

## Files Updated

- ✅ `workflow-v3.3.json` - New workflow definition
- ✅ `ARCHITECTURE-v3.3.md` - Complete architecture design
- ✅ `CHANGELOG.md` - Added v3.3.0 entry
- ✅ `DEPLOYMENT-v3.3.md` - This file

---

## Performance Expectations

### Execution Time (per company)
- Path 1 (HubSpot): ~2-3 seconds
- Path 2 (LinkedIn): ~5-8 seconds
- Path 3 (Website): ~8-12 seconds

### Daily Volume
- Expected: 10-100 companies/day
- Capacity: 1000+ companies/day

---

**Deployed By**: Claude (n8n-MCP)
**Version**: v3.3.0
**Status**: Ready for deployment
