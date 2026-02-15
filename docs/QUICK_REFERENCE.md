# Quick Reference: Natural Language Workflow Creation

**Goal**: Build n8n workflows by describing them in natural language.

## Two-Phase Process

### Phase 1: Architecture (Design) - ~5 minutes
```bash
claude
> "Create workflow: [describe what you want]"
```
Claude generates:
- ARCHITECTURE.md (full design document)
- architecture.mmd (visual diagram)
- test-data/ templates

**Your job**: Review and approve (or request changes)

### Phase 2: Build (Implementation) - ~5 minutes
After approval, Claude:
- Builds workflow.json
- Validates everything
- Deploys to n8n
- Saves all artifacts
- Commits to git

**Total time**: ~10 minutes from idea to live workflow

---

## Workflow Examples

### Example 1: Simple Notification

```bash
> "Create workflow: Send a Slack message to #alerts
   whenever a HubSpot deal is closed"
```

**Claude's architecture**:
- Trigger: HubSpot event (deal.closed)
- Process: Extract deal info
- Action: Send Slack message
- Est. time: 10 min

### Example 2: Data Sync

```bash
> "Create workflow: Every morning at 9am,
   fetch yesterday's deals from HubSpot
   and send a summary email"
```

**Claude's architecture**:
- Trigger: Scheduled (daily at 9am)
- Process: Query HubSpot, format data
- Action: Send email
- Est. time: 15 min

### Example 3: Webhook Integration

```bash
> "Create workflow: Receive webhook from Stripe,
   extract customer and amount,
   create order in our system,
   and notify fulfillment team"
```

**Claude's architecture**:
- Trigger: Webhook (POST /webhook/stripe)
- Process: Validate payment, transform data
- Action: Create order + send notification
- Est. time: 20 min

---

## Commands

### Create Workflow
```bash
> "Create workflow: [description]"
```

### Approve Architecture
```bash
> "Looks good, build it!"
> "Yes, proceed"
> "Go ahead and build"
```

### Request Changes
```bash
> "Can you change [detail]?"
> "Use [service] instead of [other service]"
> "Add [feature]"
```

### Update Workflow
```bash
> "Update [workflow name]: [change]"
```

### Test Workflow
```bash
> "Test the [workflow name] workflow"
> "Test with this data: [sample data]"
```

### View Workflows
```bash
> "List all active workflows"
> "Show me my [workflow name] workflow"
```

### Debug
```bash
> "Why did the [workflow] fail?"
> "Show me recent executions"
```

### Export/Backup
```bash
> "Export the [workflow] workflow"
> "Backup all workflows"
```

---

## Approval Guidelines

### Approve When:
- [ ] Architecture makes sense
- [ ] All required services are included
- [ ] Trigger and output are correct
- [ ] Error handling is appropriate

```bash
> "Looks good, build it!"
```

### Request Changes When:
- [ ] Wrong channel/destination
- [ ] Missing a step
- [ ] Need different error handling
- [ ] Want to try different approach

```bash
> "Can you use #sales-alerts instead?"
> "Add email notification too"
> "Retry on failures instead of stopping"
```

### Restart When:
- [ ] Fundamentally different approach needed
- [ ] Way too complex
- [ ] Major concerns

```bash
> "This is too complicated. Let's try a simpler approach."
```

---

## File Structure

After workflow is created:

```
workflows/
└── [workflow-name]/
    ├── ARCHITECTURE.md          ← Design document
    ├── architecture.mmd         ← Visual diagram
    ├── workflow.json            ← n8n workflow (deployed)
    ├── deployment.json          ← Metadata
    ├── CHANGELOG.md             ← Version history
    └── test-data/
        ├── sample-request.json
        └── expected-response.json
```

---

## Common Requests

### Change Channel
```bash
> "Update [workflow]: Use #new-channel instead of #old-channel"
```

### Add Error Handling
```bash
> "Update [workflow]: Retry 3 times on API failures"
```

### Add Second Action
```bash
> "Update [workflow]: Also send an email when done"
```

### Different Trigger
```bash
> "Update [workflow]: Trigger on schedule instead of webhook"
```

### Filter Data
```bash
> "Update [workflow]: Only process deals > $10,000"
```

---

## Tips

1. **Be Specific**
   - ✅ "Send to #deals-closed"
   - ❌ "Send a message"

2. **Start Simple**
   - Create basic workflow first
   - Add features later
   - Build incrementally

3. **Test Thoroughly**
   - Always test after deployment
   - Try different data types
   - Test error scenarios

4. **Document Decisions**
   - Review Claude's reasoning
   - Understand trade-offs
   - Update CHANGELOG when you change things

5. **Version Your Workflows**
   - Semantic versioning (1.0.0)
   - Update CHANGELOG.md
   - Keep git history clean

---

## Status Indicators

### In Architecture Phase
- [ ] ARCHITECTURE.md created
- [ ] architecture.mmd created
- [ ] Waiting for approval

### In Build Phase
- [ ] Workflow being built
- [ ] Validation in progress
- [ ] Deployment starting

### Deployed
- ✅ workflow.json saved
- ✅ deployment.json created
- ✅ Committed to git
- ✅ Live in n8n

---

## Troubleshooting

### "Architecture doesn't match what I want"
Request changes in the approval phase. Claude will iterate.

### "Workflow failed after deployment"
Ask Claude to debug:
```bash
> "Why did the workflow fail?"
> "Show me the error logs"
```

### "Need to change something"
Update the workflow:
```bash
> "Update [workflow]: [change]"
```

### "Want to revert to previous version"
Roll back:
```bash
> "Revert [workflow] to v1.0.0"
```

---

## Performance Baseline

| Task | Time |
|------|------|
| Brainstorm architecture | 5 min |
| Review + approve | 2 min |
| Build + deploy | 5 min |
| Test + iterate | 5 min |
| **Total** | **~15 min** |

Subsequent iterations (updates) are typically 5-10 minutes.

---

## Next Steps

1. **Create your first workflow**:
   ```bash
   claude
   > "Create workflow: [your idea]"
   ```

2. **Review the architecture**

3. **Approve and let Claude build it**

4. **Test with real data**

5. **Iterate as needed**

Happy workflow building! 🚀
