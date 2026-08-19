---
name: Generate on-brand landing pages from a template
description: Use Flint's Agent Tasks API to generate many on-brand landing pages from a template page and a list of items, then poll for completion.
api: openapi/flint-agent-tasks-api-openapi.yml
operations: [createAgentTask, getAgentTask]
---

# Generate landing pages with Flint

Flint generates on-brand landing pages programmatically. This skill covers the
`generate_pages` flow.

## Prerequisites
- A Flint **API key** (create in Team Settings, https://app.tryflint.com/app/team; requires member role). Send it as `Authorization: Bearer ak_your_api_key_here`.
- A **siteId** and a **templatePageSlug** for a template page on that site.
- Base URL: `https://app.tryflint.com/api/v1`.

## Steps
1. **Create the generation task** — call `createAgentTask` (`POST /agent/tasks`) with:
   - `siteId` (required)
   - `command: "generate_pages"` (required)
   - `templatePageSlug` (required)
   - `items` (required array — one entry per page to generate)
   - optional `callbackUrl` (HTTPS) to be notified on completion, and `publish: true` to publish after generation.
   The response returns `taskId` and `status: "running"`.
2. **Wait for completion** — either receive the `task.completed` webhook at your `callbackUrl`, or poll `getAgentTask` (`GET /agent/tasks/{taskId}`) until `status` is `completed` or `failed`.
3. **Read the result** — on `completed`, the `output` object lists `pagesCreated`, `pagesModified`, and `pagesDeleted`. On `failed`, read `errorMessage`.

## Rules
- Tasks are **asynchronous** — never assume the pages exist until the task reaches `completed`.
- There is **no idempotency key**; each POST starts a new task, so do not blindly retry a create on a network timeout — poll first.
- Back off on `429 Too Many Requests`.
- `callbackUrl` must be HTTPS.
