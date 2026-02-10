# n8n Workflow Builder Project

This project exists to build n8n workflows via Claude. Use the MCP tools to interact with the n8n instance and consult the available skills for guidance on patterns, node configuration, and best practices.

## Available MCP Tools

These tools come from the `n8n-mcp` server. Refer to the `n8n-mcp-tools-expert` skill when unsure which tool to use.

### Node Discovery & Validation

- **`search_nodes`** - Search for available nodes. Use the `nodes-base.*` prefix (e.g., `nodes-base.slack`).
- **`get_node`** - Get node details. Use `detail: "standard"` (not `"full"`) to save tokens. Only use `"full"` when debugging complex issues.
- **`validate_node`** - Validate a single node configuration. Use `profile: "runtime"`.
- **`validate_workflow`** - Validate an entire workflow. Use `profile: "runtime"`. Run after every significant change.

### Workflow Management

- **`n8n_create_workflow`** - Create a new workflow. Node types must use `n8n-nodes-base.*` prefix (e.g., `n8n-nodes-base.slack`).
- **`n8n_update_partial_workflow`** - Update an existing workflow incrementally. Always include the `intent` parameter describing what the update does. Build iteratively, not in one shot.
- **`activateWorkflow`** - Activate a workflow when it is ready for production use.

### Templates

- **`search_templates`** - Search the n8n template library for existing workflow patterns.
- **`n8n_deploy_template`** - Deploy a template as a starting point.

### Critical: nodeType Format Difference

| Context | Prefix | Example |
|---|---|---|
| `search_nodes`, `get_node`, `validate_node`, `validate_workflow` | `nodes-base.*` | `nodes-base.slack` |
| `n8n_create_workflow`, `n8n_update_partial_workflow` | `n8n-nodes-base.*` | `n8n-nodes-base.slack` |

Getting this wrong will cause silent failures or validation errors. Always use the correct prefix for the tool you are calling.

## Available Skills

Invoke these skills for domain-specific guidance. Each skill is triggered by consulting it (via the Skill tool) when the situation matches.

| Skill | When to Use |
|---|---|
| `n8n-workflow-patterns` | **Consult FIRST** when designing any new workflow. Covers the 5 core patterns. |
| `n8n-node-configuration` | When configuring specific nodes, understanding required fields, or choosing detail levels. |
| `n8n-expression-syntax` | When writing `{{ }}` expressions or accessing `$json`/`$node` variables. |
| `n8n-code-javascript` | When writing JavaScript in Code nodes (`$input`, `$json`, `$helpers`). |
| `n8n-code-python` | When writing Python in Code nodes (`_input`, `_json`, standard library). |
| `n8n-validation-expert` | When encountering validation errors or understanding validation results. |
| `n8n-mcp-tools-expert` | When unsure which MCP tool to use or how to structure parameters. |

## Workflow Building Procedure

Follow this step-by-step process for every workflow build:

1. **Understand requirements** - Ask clarifying questions about what the workflow should do, what triggers it, what services are involved, and what the expected output is.

2. **Identify the pattern** - Determine which core pattern applies:
   - Webhook Processing
   - HTTP API Integration
   - Database Operations
   - AI Agent Workflow
   - Scheduled Tasks

3. **Consult the `n8n-workflow-patterns` skill** for architectural guidance on the identified pattern.

4. **Search for existing templates** using `search_templates` to see if a ready-made solution or starting point exists.

5. **Discover nodes** - Use `search_nodes` to find relevant nodes, then `get_node(detail="standard")` to understand their configuration.

6. **Create a Mermaid diagram** - Draft a Mermaid flowchart showing the workflow structure (nodes and connections) and present it for review before building. Wait for approval before proceeding.

7. **Create the workflow** - Use `n8n_create_workflow` with a descriptive name and the initial set of nodes.

8. **Iterate** - Use `n8n_update_partial_workflow` with the `intent` parameter to add nodes and connections incrementally. Build in small steps, not all at once.

9. **Validate** - Run `validate_workflow` with `profile: "runtime"` after every significant change. Consult the `n8n-validation-expert` skill if errors arise.

10. **Activate** - Use `activateWorkflow` only when the workflow is validated and ready.

## Quality Guidelines

### Do

- Start with the simplest pattern that solves the problem.
- Create a Mermaid diagram of the workflow structure and get approval before building.
- Use error handling on all workflows (Error Trigger nodes, try/catch in Code nodes).
- Use descriptive node names that explain what the node does.
- Validate after every significant change.
- Use `get_node(detail="standard")` to save tokens; reserve `"full"` for debugging.
- Use smart parameters (`branch`, `case`) for connections.
- Include `intent` in every `n8n_update_partial_workflow` call.
- Prefer built-in nodes over Code nodes when a built-in node exists.
- Document workflow purpose in the workflow description field.
- Test with sample data before activation.

### Don't

- Build workflows in one shot. Iterate with incremental updates.
- Skip validation before activation.
- Use `detail: "full"` unless debugging complex issues.
- Forget the nodeType prefix differences between search/validate and workflow tools.
- Hardcode credentials in node parameters. Use the Credentials section.
- Ignore error scenarios or empty data cases.
- Skip validation profiles. Use `profile: "runtime"`.

## Common Gotchas

- **Webhook data location**: Webhook data is under `$json.body`, not `$json` directly.
- **Expression syntax**: Always wrap expressions in `{{ }}`. Do not use literal text where an expression is expected.
- **Execution order**: Use v1 connection-based execution order (recommended).
- **Credentials**: Configure credentials via the Credentials section in the n8n UI, not as hardcoded parameters in nodes.
- **Auto-sanitization**: Runs on ALL nodes during any workflow update, not just the nodes you changed. Be aware of unintended side effects.

## Credentials

- Credentials (API keys, OAuth tokens, database passwords) **must** be configured directly in the n8n UI (Settings > Credentials), never through this chat.
- The MCP tools do not include credential management — this is intentional for security.
- Before building a workflow that connects to external services, list all required credentials and ask the user to set them up in n8n first.
- Reference credentials by name/ID in node configurations, never hardcode secrets.
- Never ask the user to paste API keys, passwords, or tokens into the chat.
