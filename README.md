# n8n Workflow Automation

Build and deploy n8n workflows using natural language with Claude Code and n8n-MCP tools. Describe what you want, approve the architecture, and Claude builds, validates, and deploys it.

---

## Setup

### Prerequisites

- [Node.js](https://nodejs.org/) (v18+)
- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code) installed globally:
  ```bash
  npm install -g @anthropic-ai/claude-code
  ```

### 1. Clone and configure MCP

```bash
git clone <repo-url> && cd workflows
cp .mcp.json.example .mcp.json
```

Open `.mcp.json` and replace `<your-n8n-api-key>` with your API key. Ask Isaac for the key.

### 2. Install the n8n-MCP skills plugin

From inside the repo, run:

```bash
claude
/add-plugin n8n-mcp-skills
```

This adds the `n8n-mcp-skills` plugin which provides validation, expression syntax, workflow patterns, and code node expertise.

### 3. Verify

```bash
claude
> "Run n8n health check"
```

Claude should confirm the connection to `legalfly.app.n8n.cloud`. If it fails, double-check your API key in `.mcp.json`.

> **Note:** `.claude/settings.local.json` is already committed with the correct project permissions. You don't need to configure permissions manually.

---

## How it works

Workflows are created in two phases:

1. **Phase 1 — Architecture:** You describe what you want in plain English. Claude searches n8n's 2,700+ templates, designs the workflow, and presents an architecture document with a diagram for your approval.
2. **Phase 2 — Build:** Once approved, Claude builds the workflow, validates every node and connection, deploys it to the n8n instance, and saves all artifacts to this repo.

The full process reference is in [CLAUDE.md](CLAUDE.md).

---

## Project structure

```
.
├── .claude/
│   └── settings.local.json            # Claude Code project permissions (committed)
├── docs/
│   ├── QUICK_REFERENCE.md             # Command examples and tips
│   └── DEVELOPMENT.md                 # Detailed development guide
├── workflows/
│   ├── hubspot-industry-categorization/   # Reference implementation
│   ├── hubspot-country-enrichment/
│   ├── hubspot-employee-enrichment/
│   ├── line-items-workflow/
│   ├── n8n-error-handler/
│   ├── oneflow-signed-contract-to-gdrive/
│   └── billing/                           # Spec only — not yet built
├── .mcp.json.example                   # MCP config template (copy to .mcp.json)
├── .gitignore
├── CLAUDE.md                           # Claude Code project instructions
└── README.md
```

### Workflow directory convention

Each workflow lives in its own folder under `workflows/` and follows this structure:

```
workflows/[workflow-name]/
├── README.md                   # Overview, credentials, n8n workflow ID & URL
├── ARCHITECTURE-v{X.Y}.md     # Full technical reference (versioned)
├── architecture.mmd            # Mermaid diagram source
├── workflow-v{X.Y}.json       # n8n workflow export — source of truth (versioned)
├── CHANGELOG.md                # Version history
└── prompts/                    # Only if the workflow uses AI prompts
    └── prompt-{name}.md
```

See `workflows/hubspot-industry-categorization/` as the reference implementation.

---

## Active workflows

| Workflow | Description | Version |
|----------|-------------|---------|
| [hubspot-industry-categorization](workflows/hubspot-industry-categorization/) | Classifies new HubSpot companies into 16 industry categories using Gemini | v3.3 |
| [hubspot-country-enrichment](workflows/hubspot-country-enrichment/) | Enriches HubSpot companies with country data | v2.2 |
| [hubspot-employee-enrichment](workflows/hubspot-employee-enrichment/) | Enriches HubSpot companies with employee count data | v4.0 |
| [line-items-workflow](workflows/line-items-workflow/) | Processes HubSpot deal line items | v2.1 |
| [oneflow-signed-contract-to-gdrive](workflows/oneflow-signed-contract-to-gdrive/) | Saves signed Oneflow contracts to Google Drive | v2.1 |
| [n8n-error-handler](workflows/n8n-error-handler/) | Global error handler for n8n workflows | — |
| [billing](workflows/billing/) | Billing automation (spec phase, not yet built) | — |

---

## Quick start

```bash
claude
> "Create workflow: When a new deal is created in HubSpot,
   send a notification to Slack #sales with deal details"
```

Claude will design the architecture, ask for your approval, then build and deploy it. See [docs/QUICK_REFERENCE.md](docs/QUICK_REFERENCE.md) for more examples and commands.

---

## Documentation

| Document | What it covers |
|----------|---------------|
| [docs/QUICK_REFERENCE.md](docs/QUICK_REFERENCE.md) | Command examples, workflow creation tips, troubleshooting |
| [docs/DEVELOPMENT.md](docs/DEVELOPMENT.md) | Detailed development process, review checklists, patterns |
| [CLAUDE.md](CLAUDE.md) | Full n8n-MCP tool reference and project conventions |

---

## n8n instance

**URL:** [https://legalfly.app.n8n.cloud](https://legalfly.app.n8n.cloud)

All workflows deploy to this shared instance. Each workflow's README contains its specific workflow ID and direct link.
