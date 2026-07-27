---
name: Modify a Flint site with a prompt agent
description: Start a background AI agent that edits a Flint site from a free-form prompt, then poll the task to completion and optionally publish.
api: openapi/flint-agent-tasks-openapi.yml
operations: [createAgentTask, getAgentTask]
---

# Modify a site with a prompt agent

Run Flint's background AI agent against a site using a natural-language prompt.

## Prerequisites
- Flint **API key** as `Authorization: Bearer ak_your_api_key_here`.
- A **siteId**.
- Base URL: `https://app.tryflint.com/api/v1`.

## Steps
1. **Start the agent** — call `createAgentTask` (`POST /agent/tasks`) with:
   - `siteId` (required)
   - `prompt` (required — the free-form instruction, e.g. "update the hero copy to emphasize enterprise security")
   - optional `command: "prompt"` (default), `callbackUrl` (HTTPS), `publish: true`.
   Capture the returned `taskId`.
2. **Track it** — poll `getAgentTask` (`GET /agent/tasks/{taskId}`) or handle the `task.completed` webhook. Terminal states are `completed` and `failed`.
3. **Inspect output** — on `completed`, read `output.pagesModified` / `pagesCreated` / `pagesDeleted`; on `failed`, read `errorMessage`.

## Rules
- Asynchronous; poll to a terminal state before acting on results.
- No idempotency key — poll before retrying a create.
- Honor `429` with backoff. Errors surface as HTTP `400/404/500` on create and as `errorMessage` on a failed task.

## MCP alternative
The same flow is available via Flint's hosted MCP server (`https://mcp.tryflint.com/mcp`) with tools `run_background_agent`, `check_background_agent_status`, `list_sites`, and `publish_site`.
