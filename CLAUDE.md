# CLAUDE.md - AI Assistant Guide

This document provides guidance for AI assistants working on the moltbot-sandbox codebase.

## Project Overview

**moltbot-sandbox** is a Cloudflare Worker that runs [OpenClaw](https://github.com/openclaw/openclaw) (formerly Moltbot) personal AI assistant in a Cloudflare Sandbox container. It provides:

- **Proxying** - Routes HTTP/WebSocket requests to the Moltbot gateway (port 18789)
- **Admin UI** - React-based device management at `/_admin/`
- **API endpoints** - Device pairing and gateway control at `/api/*`
- **Debug endpoints** - Container inspection at `/debug/*`
- **CDP shim** - Browser automation via Puppeteer at `/cdp/*`
- **R2 persistence** - Optional backup/restore of config and chat history

**Important naming note:** The CLI tool is still named `clawdbot` (upstream hasn't renamed yet), so CLI commands and internal config paths use that name.

## Quick Reference

### Essential Commands

```bash
npm test              # Run tests (vitest)
npm run test:watch    # Watch mode
npm run build         # Build worker + client
npm run deploy        # Build and deploy to Cloudflare
npm run start         # wrangler dev (local worker)
npm run typecheck     # TypeScript check
```

### Key Files

| File | Purpose |
|------|---------|
| `src/index.ts` | Main Hono app, middleware stack, proxy logic |
| `src/types.ts` | TypeScript interfaces (`MoltbotEnv`, `AppEnv`) |
| `src/config.ts` | Constants (ports, timeouts, paths) |
| `wrangler.jsonc` | Cloudflare Worker + Container config |
| `Dockerfile` | Container image (Node 22 + clawdbot CLI) |
| `start-moltbot.sh` | Container startup script |

## Architecture

```
Browser/Client
       │
       ▼
┌─────────────────────────────────────┐
│     Cloudflare Worker (index.ts)    │
│  - Validates auth (CF Access JWT)   │
│  - Starts sandbox if needed         │
│  - Proxies HTTP/WebSocket to 18789  │
│  - Transforms error messages        │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│     Cloudflare Sandbox Container    │
│  ┌───────────────────────────────┐  │
│  │     Moltbot Gateway           │  │
│  │  - Control UI (web chat)      │  │
│  │  - WebSocket RPC protocol     │  │
│  │  - Agent runtime              │  │
│  └───────────────────────────────┘  │
│  (R2 mounted at /data/moltbot)      │
└─────────────────────────────────────┘
```

## Project Structure

```
src/
├── index.ts              # Main Hono app, route mounting, proxy logic
├── types.ts              # TypeScript type definitions
├── config.ts             # Constants (MOLTBOT_PORT, timeouts)
├── test-utils.ts         # Test mocks and helpers
├── auth/                 # Cloudflare Access authentication
│   ├── jwt.ts            # JWT verification with JWKS
│   ├── middleware.ts     # Hono middleware for CF Access
│   └── *.test.ts         # Tests
├── gateway/              # Moltbot gateway management
│   ├── process.ts        # Process lifecycle (find, start, wait)
│   ├── env.ts            # Environment variable building
│   ├── r2.ts             # R2 bucket mounting
│   ├── sync.ts           # R2 backup sync logic
│   ├── utils.ts          # Shared utilities
│   └── *.test.ts         # Tests
├── routes/               # API route handlers
│   ├── public.ts         # No-auth routes (/sandbox-health, /api/status)
│   ├── api.ts            # /api/admin/* endpoints (devices, gateway)
│   ├── admin-ui.ts       # /_admin/* SPA serving
│   ├── debug.ts          # /debug/* inspection endpoints
│   └── cdp.ts            # /cdp/* browser automation
├── assets/               # Static HTML (loading.html, config-error.html)
└── client/               # React admin UI (Vite)
    ├── main.tsx          # React entry point
    ├── App.tsx           # Root component
    ├── api.ts            # API client
    └── pages/            # Page components
```

## Code Conventions

### TypeScript

- **Strict mode** enabled - explicit types for function signatures
- **Naming**: kebab-case files, camelCase functions, SCREAMING_SNAKE_CASE constants
- **Types** in `src/types.ts`, colocated with implementations when small

### Hono Patterns

- Each route file exports a Hono app
- Use `c.json()`, `c.html()`, `c.text()` for responses
- Access environment via `c.env` (typed as `AppEnv`)
- Keep route handlers thin - extract logic to modules

### Testing

- Colocated test files: `*.test.ts`
- Use Vitest with mocking (`vi.fn()`, `vi.mock()`)
- Helpers in `src/test-utils.ts`: `createMockEnv()`, `createMockSandbox()`
- Always add tests when adding new functionality

## Key Patterns

### Environment Variables

Worker env vars are mapped to container env vars in `src/gateway/env.ts`:

| Worker Variable | Container Variable | Purpose |
|----------------|-------------------|---------|
| `MOLTBOT_GATEWAY_TOKEN` | `CLAWDBOT_GATEWAY_TOKEN` | Gateway auth token |
| `DEV_MODE` | `CLAWDBOT_DEV_MODE` | Skip auth + device pairing |
| `AI_GATEWAY_*` | Provider-specific | API routing via AI Gateway |
| `TELEGRAM_BOT_TOKEN` | Same | Telegram channel |
| `DISCORD_BOT_TOKEN` | Same | Discord channel |
| `SLACK_*` | Same | Slack channel |

### Route Authentication

```
Public (no auth):     /sandbox-health, /api/status, /logo.png, /_admin/assets/*
CDP (shared secret):  /cdp/* (query param ?secret=...)
Protected (CF JWT):   /_admin/*, /api/admin/*, /debug/*
```

Dev mode (`DEV_MODE=true`) bypasses all authentication.

### CLI Commands

When calling the moltbot CLI from the worker:

```typescript
// Always include --url flag
sandbox.startProcess('clawdbot devices list --json --url ws://localhost:18789')

// Commands take 10-15 seconds - use waitForProcess()
const { proc, output } = await waitForProcess(sandbox, processId, 30000)
```

### Process Identification

Distinguish gateway from CLI commands:

```typescript
const isGatewayProcess =
  proc.command.includes('start-moltbot.sh') ||
  proc.command.includes('clawdbot gateway')

// NOT CLI commands like:
// 'clawdbot devices list', 'clawdbot --version'
```

### WebSocket Interception

The worker intercepts WebSocket messages to transform error messages:

```typescript
// Example: "gateway token missing" → user-friendly hint with URL
```

## Common Tasks

### Adding a New API Endpoint

1. Add route handler in `src/routes/api.ts`
2. Add types in `src/types.ts` if needed
3. Update client API in `src/client/api.ts` if frontend needs it
4. Add tests

### Adding a New Environment Variable

1. Add to `MoltbotEnv` interface in `src/types.ts`
2. If passed to container, add to `buildEnvVars()` in `src/gateway/env.ts`
3. Update `.dev.vars.example`
4. Document in README.md secrets table
5. Add tests for the env var handling

### Adding a New Route File

1. Create new file in `src/routes/` (export a Hono app)
2. Mount in `src/index.ts` using `app.route('/path', newRoutes)`
3. Consider auth requirements (add to public list or use middleware)

## Gotchas and Pitfalls

### R2 Storage

- **Mount path**: `/data/moltbot` - this IS the R2 bucket
- **rsync**: Use `rsync -r --no-times` (s3fs doesn't support timestamps)
- **Mount detection**: Check `mount | grep s3fs`, don't rely on error messages
- **Never delete**: `rm -rf /data/moltbot/*` will DELETE your backup data

### Moltbot Config

- `agents.defaults.model` must be `{ "primary": "model/name" }` not a string
- `gateway.mode` must be `"local"` for headless operation
- No `webchat` channel - Control UI is served automatically
- `gateway.bind` is not config - use `--bind` CLI flag

### Process Status

The sandbox API's `proc.status` may not update immediately. Instead of checking `proc.status === 'completed'`, verify success by checking for expected output.

### Docker Cache

Bump the cache bust comment in Dockerfile when changing `moltbot.json.template` or `start-moltbot.sh`:

```dockerfile
# Build cache bust: 2026-01-26-v10
```

### Success Detection

CLI outputs "Approved" with capital A. Use case-insensitive checks:

```typescript
stdout.toLowerCase().includes('approved')
```

## Middleware Stack Order

The middleware in `src/index.ts` runs in this order:

1. Logging (all requests)
2. Initialize sandbox
3. Public routes (no auth) - `/sandbox-health`, `/api/status`
4. CDP routes (shared secret)
5. Required env vars check
6. Cloudflare Access JWT middleware
7. Protected routes - `/api/admin/*`, `/_admin/*`, `/debug/*`
8. Catch-all proxy to gateway

## Debugging

```bash
# View live logs
npx wrangler tail

# Check secrets
npx wrangler secret list

# Enable debug routes
DEBUG_ROUTES=true npm run start
# Then check /debug/processes, /debug/logs, /debug/env
```

## Documentation Reference

- **README.md** - User-facing setup and configuration guide
- **AGENTS.md** - Developer guide for AI agents (more detailed patterns)
- **skills/cloudflare-browser/SKILL.md** - Browser automation skill docs

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| hono | ^4.11.6 | Web framework |
| react | ^19.0.0 | Admin UI |
| @cloudflare/puppeteer | ^1.0.5 | Browser automation |
| jose | ^6.0.0 | JWT verification |
| vitest | ^4.0.18 | Testing |
| typescript | ^5.9.3 | Type checking |
