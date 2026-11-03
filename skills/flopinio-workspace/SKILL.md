---
name: flopinio-workspace
description: Use the connected Flopinio workspace to find client context and carry requests, projects, tasks, notes, decisions, time, invoices, and approval-backed actions forward. Trigger when the user asks to inspect, summarize, plan, create, or update work in Flopinio.
---

# Flopinio Workspace

Use the Flopinio MCP server as the source of truth for workspace records.

## Workflow

1. Search or list before creating records so existing projects, tasks, notes, requests, and time coverage are reused when they already fit.
2. Fetch the exact record before making a consequential update.
3. Keep reads scoped to the connected user and workspace. Never infer record IDs, workspace membership, permissions, billing state, or private content.
4. Use the narrowest tool that matches the requested action.
5. Keep delivery work logs untimed: omit `duration_minutes`. Use time-entry tools for billable time; never use work logs as a tracked-time substitute.

## Read tools and result IDs

- Use `search` for workspace discovery and pass its opaque result `id` to `fetch`. Do not pass a record or knowledge-document UUID to `fetch`.
- Use `knowledge-search` or `record-similarity-search` for indexed context. Pass a returned `document_id` to `knowledge-fetch-document`, or a chunk ID to `knowledge-fetch-chunk`.
- Use `assistant-query` for structured record lists and summaries, and `metrics-query` for analytics. Build calls from the advertised schemas and allowed-value guidance; omit optional fields only when that schema permits it.
- Assistant-query filters are `field`/`operator`/`value` objects in an array. Metrics-query filters are an object mapping supported field names to values. Supported filters and date windows depend on the selected query or metric.

## Writes And Approvals

- Only create or update records when the user asks for the change or the write is an explicit step in the requested workflow.
- Preserve useful existing descriptions and context when updating records.
- Flopinio write tools normally stage approval-backed actions. When `approval_required` is true or the approval is pending, report the action as pending and link its review URL. Do not claim the record changed until execution is confirmed.
- Request `self_approve` only for an explicitly authorized write when the token permits it. A successful tool response alone does not establish that the write executed; check the returned approval and agent-run status.
- Keep invoice writes, approval resolution, and other financially significant or destructive operations explicit. Do not broaden their scope or bypass Flopinio approval controls.

## Triage and invoice drafts

- `request-run-triage` persists a triage run without applying its recommendations. Use `request-apply-triage` only after a successful run and authorization to update the request; application can change summary, status, triage state, urgency, business value, response need, and project routing.
- `invoice-prepare-draft` analyzes a visible draft invoice and persists suggested line items and findings in an agent run. It does not apply the line items. Inspect the returned run and require successful preparation before the next step.
- `invoice-apply-prepared-draft` applies the latest successful preparation to a draft invoice through the normal approval controls. Neither invoice tool issues, finalizes, or sends an invoice.

## OAuth Permissions

Flopinio requests permissions per tool family, including separate read and write scopes for requests, tasks, notes, decisions, time entries, invoices, approvals, and runtime context. If a tool reports `insufficient_scope`, explain which permission is needed and let the user reconnect or grant that scope.

## Response Style

Lead with the workspace outcome. Include stable record names and links returned by Flopinio, distinguish confirmed changes from staged approvals, and keep operational detail concise.
