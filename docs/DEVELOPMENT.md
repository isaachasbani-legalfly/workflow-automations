# Development Workflow Guide

This guide explains how to develop n8n workflows using the natural language approach.

## Quick Start

### 1. Create a New Workflow (5 minutes)

```bash
claude
> "Create workflow: [describe what you want]"
```

Claude will generate an architecture document for your approval.

### 2. Review Architecture (2 minutes)

Claude presents:
- Workflow overview and diagram
- Node-by-node breakdown
- Configuration requirements
- Testing strategy

**Approve or request changes** before proceeding to build.

### 3. Build and Deploy (3 minutes)

Once approved, Claude will:
- Build the workflow
- Validate all nodes and connections
- Deploy to n8n instance
- Save all artifacts (workflow.json, CHANGELOG.md, etc.)
- Commit to git

**Total time: ~10 minutes from idea to production**

---

## Detailed Workflow Development Process

### Phase 1: Brainstorm & Design

This is where you describe what you want to build.

**Do**:
- Be specific about requirements
- Mention data sources and destinations
- Describe the trigger (webhook, schedule, event, manual)
- List any error handling needs

**Example**:
```
"Create workflow: When a HubSpot deal is updated,
 check if the deal stage is 'Closed Won',
 and if so, create an invoice in our billing system
 and send a thank you email to the contact.
 Retry on errors."
```

**Don't**:
- Over-specify implementation details
- Worry about node names or types (Claude handles that)
- Try to include everything at once (iterate instead)

**Claude's Response (Architecture)**:
- ARCHITECTURE.md with full design
- architecture.mmd with visual diagram
- Highlights requirements and dependencies
- Asks for approval before building

### Phase 2: Validate & Approve Architecture

Review the architecture Claude generated.

**Check For**:
- [ ] Purpose and overview make sense
- [ ] Trigger is correct (webhook, schedule, etc.)
- [ ] All required nodes are listed
- [ ] Data transformations look reasonable
- [ ] Error handling strategy is appropriate
- [ ] Dependencies/credentials are clear

**Feedback Options**:
```bash
# Approve and proceed to build
> "Looks good, build it!"

# Request a specific change
> "Change the Slack channel from #sales to #deals"

# Ask a question
> "Why do we need a Set node? Can't we just use the data directly?"

# Suggest a different approach
> "Instead of email, send a Slack message instead"
```

Claude will iterate the architecture until you're satisfied.

### Phase 3: Build (Automated)

Once you approve, Claude will:

1. **Validate Each Node**
   - Check required parameters
   - Verify data types
   - Confirm credentials available

2. **Build Workflow**
   - Create workflow.json
   - Set all parameters explicitly
   - Add error handling nodes
   - Connect all nodes properly

3. **Validate Complete Workflow**
   - Check all connections
   - Validate n8n expressions
   - Verify data flow

4. **Deploy to n8n**
   - Create workflow in n8n instance
   - Capture workflow ID
   - Save deployment metadata

5. **Save Artifacts**
   - workflow.json (n8n export)
   - deployment.json (metadata)
   - CHANGELOG.md (version history)
   - Commit to git

**Claude's Response**:
```
✅ Workflow Deployed

[Workflow Name]
- Workflow ID: xyz123
- URL: https://legalfly.app.n8n.cloud/workflow/xyz123
- Status: Active

Files Saved:
✓ workflows/[name]/ARCHITECTURE.md
✓ workflows/[name]/workflow.json
✓ workflows/[name]/deployment.json
✓ workflows/[name]/CHANGELOG.md

Ready to test with real data.
```

### Phase 4: Test & Monitor

Once deployed, you can test the workflow:

```bash
# Manual test with sample data
> "Test the HubSpot workflow with a sample deal update"

# Check recent executions
> "Show me the last 10 executions of the HubSpot workflow"

# Debug a failed execution
> "Why did the HubSpot workflow fail on execution exec-123?"
```

### Phase 5: Iterate & Update

If you need to change the workflow:

```bash
# Simple change
> "Update HubSpot workflow: Add a Slack notification when done"

# Significant change
> "Rewrite HubSpot workflow to use database instead of email"

# Bug fix
> "Fix the HubSpot workflow: The contact name isn't being extracted correctly"
```

Claude will:
1. Update ARCHITECTURE.md
2. Rebuild the workflow
3. Update CHANGELOG.md with new version
4. Commit changes to git

---

## Architecture Review Checklist

Before approving an architecture, verify:

### Trigger & Input
- [ ] Trigger type is correct (webhook, schedule, event, etc.)
- [ ] Expected input data format is clear
- [ ] Any required parameters or headers are documented

### Processing Nodes
- [ ] Each node has a clear purpose
- [ ] Data transformations are reasonable
- [ ] Node configurations look complete
- [ ] No obvious missing steps

### Output & Actions
- [ ] Final action/output is what you want
- [ ] All required integrations are included
- [ ] Success and failure paths are defined

### Error Handling
- [ ] Error scenarios are identified
- [ ] Retry logic is appropriate
- [ ] Failure notifications are configured
- [ ] No data loss on errors

### Dependencies
- [ ] All required credentials are listed
- [ ] External services are available
- [ ] Rate limits are considered
- [ ] API key rotation is planned

### Testing
- [ ] Test plan is realistic
- [ ] Edge cases are considered
- [ ] Error scenarios are testable

---

## Common Workflow Patterns

### Pattern 1: Event-Triggered Action

**Trigger**: Event from external service (Slack, HubSpot, etc.)
**Process**: Transform data
**Output**: Action in another service

**Example**: "When a Slack message is sent, post it to a database"

### Pattern 2: Scheduled Task

**Trigger**: Time-based schedule (daily, hourly, etc.)
**Process**: Fetch data from service
**Output**: Generate report or send notification

**Example**: "Every morning at 9am, get yesterday's deals and email summary"

### Pattern 3: Webhook-Based Integration

**Trigger**: HTTP webhook from external service
**Process**: Validate and transform data
**Output**: Create/update records in target system

**Example**: "Receive webhook from payment provider, create invoice in billing system"

### Pattern 4: Data Sync

**Trigger**: Scheduled or manual
**Process**: Read from source, compare with target
**Output**: Create/update records in target

**Example**: "Sync all HubSpot contacts to mailing list service"

### Pattern 5: Notification Workflow

**Trigger**: Event or condition
**Process**: Check conditions, format message
**Output**: Send notification (email, Slack, SMS, etc.)

**Example**: "Send Slack alert when deal value exceeds threshold"

---

## Best Practices for Development

### 1. Start Simple, Add Complexity Later

**Good**:
```
Phase 1: "Send Slack message when deal closes"
Phase 2: "Extract deal details and include in message"
Phase 3: "Add email notification to sales team"
```

**Bad**:
```
"Do everything: extract deal details, send Slack, email, update CRM,
generate invoice, post to accounting system, notify warehouse"
```

### 2. Be Specific About Data

Instead of: "Get the deal information"
Better: "Get deal name, amount, owner, and close date from HubSpot"

### 3. Test Error Scenarios

When testing, include:
- ✅ Happy path (successful execution)
- ⚠️ Missing data
- ❌ Invalid data format
- ⏱️ Timeouts
- 🔐 Invalid credentials

### 4. Document Architectural Decisions

When Claude shows you alternatives, understand:
- Why one option was chosen
- What the trade-offs are
- When you might want to reconsider

### 5. Use Descriptive Workflow Names

**Good**: `hubspot-deal-closed-notification`
**Bad**: `workflow1`, `automation`

### 6. Tag Workflows Logically

Common tags:
- By source: `hubspot`, `slack`, `stripe`
- By purpose: `notification`, `sync`, `backup`
- By frequency: `scheduled`, `event-driven`, `manual`

---

## Handling Approvals

### When to Approve Immediately

The architecture looks good and you:
- [ ] Understand the workflow
- [ ] Trust the design
- [ ] Are ready to test

```
> "Looks good, build it!"
```

### When to Request Changes

Something doesn't seem right:
- [ ] Different channel or destination
- [ ] Missing a step
- [ ] Different error handling needed
- [ ] Want to try a different approach

```
> "Can we use #sales-alerts instead of #general?"
> "Add a second path for when deal amount > $10k"
> "Retry with exponential backoff instead of immediate retry"
```

### When to Reject and Restart

The approach doesn't feel right:
- [ ] Fundamentally different problem than expected
- [ ] Major complexity concerns
- [ ] Technical concerns

```
> "This is more complex than I expected. Let's try a simpler approach.
   Instead of real-time, can we do a daily summary email?"
```

Claude will redesign and present a new architecture.

---

## Monitoring Active Workflows

### Check Status
```bash
> "Are all my workflows running successfully?"
```

### View Execution History
```bash
> "Show recent executions of the HubSpot workflow"
```

### Debug Failures
```bash
> "The Slack workflow failed. What went wrong?"
```

### View Logs
```bash
> "Show me detailed logs for execution exec-123"
```

---

## Performance Optimization

### Monitor Execution Time
Keep an eye on how long workflows take to execute.

### Identify Bottlenecks
If a workflow is slow:
- Identify which node is slowest
- Consider parallel processing
- Optimize data transformations

### Request Changes
```bash
> "The HubSpot workflow is running too slow. Can you optimize it?"
```

Claude can suggest:
- Parallel processing instead of sequential
- Filtering data earlier
- Batch processing
- Caching strategies

---

## Disaster Recovery

### Rollback to Previous Version
```bash
> "Revert the HubSpot workflow to v1.0.0"
```

### Export Current Version
```bash
> "Export the current HubSpot workflow"
```

### Backup All Workflows
```bash
> "Backup all workflows to git"
```

---

## Tips for Effective Communication

### Be Specific
❌ "The workflow isn't working"
✅ "The Slack notification isn't including the deal amount"

### Provide Context
❌ "Why did it fail?"
✅ "The workflow failed at execution exec-123. Can you check why?"

### Use Examples
❌ "Change the message format"
✅ "Instead of 'Deal: $5000', format it as 'Deal Value: $5,000.00'"

### Describe the Outcome
❌ "Add error handling"
✅ "If the API fails, retry 3 times then send me a Slack alert"

---

## Workflow Development Workflow

```
1. Brainstorm Requirements
   ↓
2. Create Architecture
   ↓
3. Review & Approve
   ↓
4. Build & Deploy
   ↓
5. Test with Real Data
   ↓
6. Monitor & Iterate
   ↓
7. Version & Document Changes
```

This cycle continues as you refine and improve workflows over time.
