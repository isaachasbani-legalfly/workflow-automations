# n8n Workflows

This directory contains all n8n automation workflows for the project.

## Directory Structure

```
workflows/
├── _templates/                      # Reusable templates
│   ├── ARCHITECTURE.md.template    # Template for architecture documents
│   ├── CHANGELOG.md.template       # Template for version history
│   ├── deployment.json.template    # Template for deployment metadata
│   └── webhook-to-slack.json       # Example reusable workflow pattern
│
└── [workflow-name]/                # Individual workflow directory
    ├── ARCHITECTURE.md              # Phase 1: Architecture design
    ├── architecture.mmd             # Mermaid diagram
    ├── workflow.json                # Phase 2: n8n workflow export
    ├── deployment.json              # Deployment metadata
    ├── CHANGELOG.md                 # Version history
    └── test-data/                   # Test fixtures
        ├── sample-request.json
        └── expected-response.json
```

## Creating a New Workflow

### Phase 1: Architecture (Design)

Tell Claude what you want to build:

```bash
# Example
claude
> "Create workflow: When a new deal is created in HubSpot,
   send a notification to Slack #sales with deal details"
```

Claude will:
1. Search n8n templates and project patterns
2. Design the workflow architecture
3. Create ARCHITECTURE.md and architecture.mmd
4. Present for your approval

**Review the architecture** and approve before proceeding to Phase 2.

### Phase 2: Implementation (Build)

After approving the architecture, Claude will:
1. Build the workflow from the approved architecture
2. Validate with multi-level checks
3. Deploy to n8n instance
4. Save all artifacts (workflow.json, deployment.json, CHANGELOG.md)
5. Commit to git

## Workflow Files Reference

### ARCHITECTURE.md
- **Purpose**: Document the workflow design before building
- **Phase**: Phase 1 (Pre-Build)
- **Creator**: Claude (based on your requirements)
- **Review**: You must approve before Phase 2
- **Content**: Overview, requirements, node breakdown, data flow, error handling, dependencies, testing strategy, deployment plan

### architecture.mmd
- **Purpose**: Visual diagram of the workflow using Mermaid syntax
- **Format**: Markdown with Mermaid flowchart
- **Shows**: All nodes, connections, decision points, error handling
- **Viewer**: Works in GitHub, n8n, and Markdown editors

### workflow.json
- **Purpose**: The actual n8n workflow definition
- **Phase**: Phase 2 (Post-Build)
- **Creator**: Claude (generated from ARCHITECTURE.md)
- **Format**: n8n workflow JSON export
- **Usage**: Deployed to n8n instance, tracked in git

### deployment.json
- **Purpose**: Metadata about the deployed workflow
- **Phase**: Phase 2 (Post-Deploy)
- **Creator**: Claude (auto-generated during deployment)
- **Content**: Workflow ID, URL, deployment date, credentials used, tags, execution stats
- **Usage**: Track which workflows are deployed where and when

### CHANGELOG.md
- **Purpose**: Version history and change tracking
- **Phase**: Created at v1.0.0, updated with each version
- **Creator**: Claude
- **Format**: Keep a Changelog format with Semantic Versioning
- **Usage**: Document what changed in each version

### test-data/
- **Purpose**: Sample data for testing the workflow
- **Files**:
  - `sample-request.json`: Example input data (webhook payload, etc.)
  - `expected-response.json`: Expected output from the workflow
  - Additional files for edge cases or error scenarios
- **Usage**: Used to test workflow locally and verify behavior

## Workflow Lifecycle

### 1. Design Phase
- User describes workflow requirements
- Claude generates ARCHITECTURE.md
- User reviews and approves

### 2. Build Phase
- Claude builds workflow.json
- Claude validates workflow
- Claude deploys to n8n

### 3. Active Phase
- Workflow runs in n8n
- Executions are tracked
- deployment.json records stats

### 4. Update Phase
- If changes needed: Update ARCHITECTURE.md
- User approves changes
- Claude rebuilds workflow
- Update CHANGELOG.md with new version

### 5. Archive Phase
- When workflow is no longer needed
- Move to archive directory (if keeping for reference)
- Or delete if no longer needed

## Workflow Naming Convention

Workflow directory names should be **kebab-case** (lowercase with hyphens):

- ✅ `slack-help-bot`
- ✅ `hubspot-deal-notification`
- ✅ `email-processor`
- ❌ `SlackHelpBot` (use kebab-case)
- ❌ `slack_help_bot` (use hyphens, not underscores)

## Example: Slack Help Bot

See `slack-help-bot/` for a reference workflow.

**Workflow Purpose**: Automatically respond to "help" messages in Slack DMs

**Files**:
- `ARCHITECTURE.md`: Design and planning document
- `architecture.mmd`: Visual diagram
- `workflow.json`: n8n workflow definition
- `deployment.json`: Deployment metadata
- `CHANGELOG.md`: Version history
- `test-data/`: Sample Slack messages for testing

## Useful Commands

### View all active workflows
```bash
claude
> "List all active workflows in my n8n instance"
```

### Test a workflow
```bash
claude
> "Test the slack-help-bot workflow with a sample message"
```

### Update a workflow
```bash
claude
> "Update slack-help-bot: Change response message to 'You got this!'"
```

### Export workflow from n8n
```bash
claude
> "Export slack-help-bot workflow from n8n and save to git"
```

## Git Integration

### What Gets Committed
- ✅ ARCHITECTURE.md
- ✅ architecture.mmd
- ✅ workflow.json
- ✅ deployment.json
- ✅ CHANGELOG.md
- ✅ test-data/ files
- ✅ README.md

### What Doesn't Get Committed
- ❌ Credentials (stored in n8n UI only)
- ❌ Execution logs
- ❌ Temporary test files
- ❌ .env files with secrets

### Commit Strategy

When creating a new workflow:
```
Commit 1: "feat(workflow): Add [workflow-name] architecture"
Commit 2: "feat(workflow): Implement [workflow-name] workflow"
```

When updating a workflow:
```
Commit: "feat(workflow): Add error handling to [workflow-name]"
```

## Best Practices

1. **Always Start with Architecture**
   - Design before building
   - Get approval before spending time on implementation
   - Document decisions for future reference

2. **Keep Workflows Focused**
   - One workflow = one main task
   - Break complex workflows into multiple smaller ones
   - Use sub-workflow calls for reuse

3. **Test Thoroughly**
   - Use test-data/ for sample inputs
   - Test error scenarios
   - Monitor first execution in production

4. **Document Everything**
   - Keep ARCHITECTURE.md updated
   - Update CHANGELOG.md with changes
   - Add inline comments in complex workflows (sticky notes in n8n)

5. **Version Semantically**
   - MAJOR: Breaking changes
   - MINOR: New features
   - PATCH: Bug fixes

## Troubleshooting

### Workflow won't deploy
- Check error message in Claude's response
- Verify all credentials are configured
- Run validation checks
- Check n8n instance health

### Workflow fails at runtime
- Check execution logs in n8n
- Verify test-data matches actual data format
- Check API credentials haven't expired
- Review error handling in ARCHITECTURE.md

### Need to rollback to previous version
- Check CHANGELOG.md for previous version
- Use n8n_workflow_versions to see history
- Ask Claude to revert to previous version

## Support

For help creating, updating, or troubleshooting workflows, just ask Claude naturally:

```
"Create a new workflow that..."
"Update the Slack workflow to..."
"Why isn't the HubSpot workflow working?"
"Show me all my active workflows"
```

Claude will handle all the heavy lifting with the n8n MCP tools.
