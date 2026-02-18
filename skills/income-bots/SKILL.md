---
name: income-bots
description: Manage and monitor the income-generating bots system. Check bot status, revenue reports, approve pending tasks, view content pipeline, and trigger workflows. Connect to the orchestrator MCP server for full control of all 7 income bots.
---

# Income Bots Manager

Manage the AI-powered income generation system with 7 specialized bots and an orchestrator.

## System Overview

The income-bots system consists of:

| Bot | Purpose |
|-----|---------|
| Content Harvester | Finds trending topics and curates content |
| Social Publisher | Posts to Twitter/X, LinkedIn, Pinterest |
| Gumroad Store | Creates and sells digital products |
| Newsletter | Manages Substack/email newsletters |
| SEO Content | Generates search-optimized articles |
| Course Generator | Creates online courses |
| Affiliate Finder | Discovers and manages affiliate partnerships |

## Common Tasks

### Check System Status
To get a full overview of all bots, revenue, and pending tasks:
```
Show me the income bots dashboard
```

### Revenue Reporting
```
What's my revenue for the last 30 days?
Which bot is generating the most revenue?
```

### Task Approval
Tasks that require human review (content publishing, purchases, etc.) go into an approval queue:
```
Show me pending approvals
Approve task [taskId]
Reject task [taskId] with feedback "needs more detail"
```

### Content Pipeline
Content flows through stages: draft → awaiting_approval → approved → scheduled → published
```
Show the content pipeline
What content is ready to publish?
```

### Manual Task Creation
```
Create a task for the social publisher to post about [topic]
Tell the content harvester to find trending topics in [niche]
```

### Pipeline Operations
```
Run the content pipeline now
Show pipeline status
```

## MCP Server Connection

The orchestrator runs as an MCP server at:
- **Location**: `/Users/danielkent/income-bots/`
- **Entry**: `dist/orchestrator/index.js`
- **Transport**: stdio

### Available MCP Tools

| Tool | Description |
|------|-------------|
| `get_dashboard` | Full system overview |
| `get_bot_status` | Status of specific or all bots |
| `get_revenue_report` | Revenue data for any time period |
| `get_approval_queue` | Tasks waiting for approval |
| `approve_task` | Approve or reject a task |
| `create_bot_task` | Create a new task for any bot |
| `get_alerts` | System alerts and warnings |
| `resolve_alert` | Mark alerts as resolved |
| `get_content_pipeline` | Content in all pipeline stages |
| `get_pipeline_status` | Pipeline runner status |
| `run_pipeline_now` | Trigger a pipeline cycle |
| `generate_report` | Daily/weekly/monthly reports |

## Bot Types for Task Creation

When creating tasks, use these bot IDs:
- `content-harvester`
- `social-publisher`
- `gumroad-store`
- `newsletter`
- `seo-content`
- `course-generator`
- `affiliate-finder`
- `substack-publisher`
- `pinterest-publisher`

## Interpreting Data

- Revenue is tracked per-bot and per-source
- Tasks have statuses: pending, awaiting_approval, approved, in_progress, completed, failed, cancelled
- Alerts have severity levels: info, warning, error, critical
- Content pipeline shows counts at each stage

## Suggested Daily Workflow

1. Check the dashboard for overview
2. Review and approve pending tasks
3. Check for any critical alerts
4. Review revenue trends
5. Run the pipeline if content is backed up
