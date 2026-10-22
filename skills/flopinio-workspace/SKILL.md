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

## Writes And Approvals

- Only create or update records when the user asks for the change or the write is an explicit step in the requested workflow.
- Preserve useful existing descriptions and context when updating records.
- Flopinio write tools normally stage approval-backed actions. Report an action as pending when the tool returns an approval or review URL; do not claim the record changed until execution is confirmed.
- Keep invoice writes, approval resolution, and other financially significant or destructive operations explicit. Do not broaden their scope or bypass Flopinio approval controls.

## OAuth Permissions

Flopinio requests permissions per tool family, including separate read and write scopes for requests, tasks, notes, decisions, time entries, invoices, approvals, and runtime context. If a tool reports `insufficient_scope`, explain which permission is needed and let the user reconnect or grant that scope.

## Response Style

Lead with the workspace outcome. Include stable record names and links returned by Flopinio, distinguish confirmed changes from staged approvals, and keep operational detail concise.
