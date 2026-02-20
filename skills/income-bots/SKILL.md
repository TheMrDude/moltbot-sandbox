---
name: income-bots
description: Manage and monitor the income-generating bots system via HTTP API. Check bot status, revenue reports, approve pending tasks, view content pipeline, and trigger workflows.
---

# Income Bots Manager

Manage the AI-powered income generation system with 12 specialized bots and an orchestrator.

## API Connection

The income-bots API is accessible via HTTP. Use the `fetch` tool or equivalent to make requests.

**Base URL**: Set via the `INCOME_BOTS_API_URL` environment variable.
**Auth**: Include `Authorization: Bearer {INCOME_BOTS_API_TOKEN}` header if a token is configured.

## Bots in the System

| Bot ID | Name | Purpose |
|--------|------|---------|
| orchestrator | Orchestrator | Master coordinator |
| content-harvester | Content Harvester | Finds trending topics from Reddit, HackerNews |
| social-publisher | Social Publisher | Posts to Twitter/X, LinkedIn |
| gumroad-store | Gumroad Store | Creates and sells digital products |
| newsletter | Newsletter | Manages email newsletters |
| seo-content | SEO Content | Generates search-optimized articles for Medium, Dev.to |
| course-generator | Course Generator | Creates online courses and eBooks |
| affiliate-finder | Affiliate Finder | Discovers affiliate partnerships |
| crypto-harvester | Crypto Harvester | Crypto-specific content harvesting |
| habitquest-content | HabitQuest Blog | Habit formation content for habitquest.dev |
| substack-publisher | Substack Publisher | Publishes to Substack |
| pinterest-publisher | Pinterest Publisher | Creates Pinterest pins |
| tiktok-publisher | TikTok Publisher | Generates TikTok photo carousel slideshows with Cloudflare AI images |

## API Endpoints

### GET /dashboard
Full system overview: bot count, revenue, pending tasks, alerts.

### GET /revenue?days=30
Revenue report. Optional `days` query param (default 30).

### GET /bots
Status of all bots with task counts and revenue.

### GET /bots/:botId
Detailed status of a specific bot.

### GET /approvals
All tasks awaiting approval with details.

### POST /approve
Approve or reject a task. Body: `{"taskId": "...", "approved": true/false, "feedback": "optional"}`

### GET /pipeline
Content pipeline overview (draft, awaiting_approval, approved, scheduled, published counts).

### GET /pipeline/status
Pipeline runner status (last run, active state).

### POST /pipeline/run
Trigger a pipeline cycle manually.

### GET /alerts?botId=...&severity=...
System alerts. Optional filters.

### GET /images/:filename
Serves generated slideshow images (PNG). Images are created by the TikTok publisher's `generate_images` tool using Cloudflare Workers AI. Returns binary PNG with `Cache-Control: public, max-age=86400`.

### GET /health
Health check. Returns `{"ok": true}`.

### GET /personas
List all bot personas with identity summaries.

### GET /personas/:botId
Get the full detailed persona/soul file for a specific bot. Use this when you need to understand a bot's expertise, decision-making framework, or communication style in depth.

## How to Use

When the user asks about their income bots, make HTTP requests to the API:

1. **"Show dashboard"** → GET /dashboard
2. **"Revenue report"** → GET /revenue
3. **"Bot status"** → GET /bots
4. **"Pending approvals"** → GET /approvals
5. **"Approve task X"** → POST /approve with `{"taskId": "X", "approved": true}`
6. **"Reject task X"** → POST /approve with `{"taskId": "X", "approved": false}`
7. **"Run pipeline"** → POST /pipeline/run
8. **"Any alerts?"** → GET /alerts

Format responses in a clean, readable way for Telegram. Use short summaries, not raw JSON dumps.

## Response Formatting for Telegram

When presenting data, format it cleanly:
- Use bullet points for lists
- Show revenue as dollar amounts
- Summarize bot statuses as active/idle/error
- For approvals, show task type, bot name, and a brief description
- Keep messages concise — Telegram messages should be scannable

## Bot Personas

Each bot has a detailed expert persona that defines its identity, decision-making framework, quality standards, and communication style. These are based on research showing that detailed expert identities dramatically outperform generic role labels for AI-driven content quality.

When generating content or making decisions on behalf of a bot, fetch its persona first:
- **GET /personas/:botId** — returns the full persona document
- Use the persona's guidelines for tone, structure, and quality standards
- When reporting bot activity, use the persona's communication style

## TikTok Slideshow Workflow

The TikTok publisher generates photo carousels using Cloudflare Workers AI for image generation (free tier). Flow:

1. **generate_slideshow** — Creates 6-slide spec with image prompts from harvested content
2. **generate_images** — Generates actual PNG images via Cloudflare Workers AI (Flux Schnell or SDXL)
3. **publish_slideshow** — Posts to TikTok via Postiz API using the generated image URLs

Images are served at `{API_URL}/images/{contentId}-slide-{n}.png`. The tunnel URL must be accessible for Postiz to download images during posting.

Models available: `flux-schnell` (default, fast+quality), `sdxl`, `sdxl-lightning` (fastest).

## Suggested Daily Workflow

1. Check the dashboard for overview
2. Review and approve pending tasks
3. Check for any critical alerts
4. Review revenue trends
5. Run the pipeline if content is backed up
