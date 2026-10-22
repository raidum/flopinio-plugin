# Flopinio plugin

Connect Claude, ChatGPT, or Codex to your Flopinio account to find client-work
context and carry requests, projects, tasks, notes, decisions, time, and invoices
forward. The shared `flopinio-workspace` skill explains the workflow; tools run
on the hosted Flopinio MCP server at `https://flopinio.com/mcp`.

This repository contains only the plugin manifests, skill, and branding assets.
It contains no Rails application or MCP server implementation, account bindings,
access tokens, or reviewer credentials. The MIT license covers these package
files. The hosted service remains subject to Flopinio's terms and account access.

## Package formats

- Claude: `.claude-plugin/plugin.json` and `.mcp.json`.
- ChatGPT and Codex portable package: `plugin.json` and `mcp.json`.
- Both formats share `skills/flopinio-workspace/SKILL.md` and version `0.2.0`.

The OpenAI package name preserves the existing published plugin identity.
Changes to this repository do not update any directory listing automatically
until a release source and its publication process are configured.

## Connect

You need a Flopinio account with access to the intended organization and its MCP
features. Connect through the host's OAuth flow, select the intended workspace,
and review the requested tool-family scopes. Each user authorizes their own
account. Never put a shared bearer token or client secret in either MCP manifest.

Reads follow the connected user's permissions and organization scope. Writes use
Flopinio's runtime and approval controls. An action that returns an approval is
pending until execution is confirmed; eligible token configurations may permit
self-approved writes. Keep delivery work logs untimed by omitting `duration_minutes`; use time-entry
tools for billable time.

## Local Claude testing

```sh
claude plugin validate --strict .
claude --plugin-dir .
```

In Claude Code, check `/mcp`, complete OAuth, then ask Claude to read the latest
note in a project you can access. For Claude web or Cowork, upload a ZIP containing
this folder through Customize > Plugins > Add > Upload plugin, then connect
Flopinio from the plugin's Connectors tab.

Verify a scoped read, a harmless staged write, rejection of ungranted operations,
and reconnect/refresh behavior using a dedicated demo workspace. These are manual
test instructions, not evidence that they have passed.

Claude compatibility work is being prepared separately in the private Rails
repository. It is not deployed by this package. Do not treat schema validation
as proof that the deployed endpoint supports Claude's complete OAuth flow.

## Directory submission

Claude requires separate MCP connector and plugin-bundle submissions from the
same Claude organization. Submit the hosted endpoint as the connector and this
repository as the bundle, referencing the same URL. The repository must be public
before the bundle listing goes live. Connect GitHub with push access, validate
the selected commit in the portal, supply reviewer credentials privately, and
complete review before requesting publication. Keep automatic publication off
unless separately authorized.

- [Documentation](https://flopinio.com/docs/mcp)
- [Privacy](https://flopinio.com/privacy)
- [Terms](https://flopinio.com/terms)
- [Support](https://flopinio.com/support), support@flopinio.com
- [Claude submission guide](https://claude.com/docs/plugins/submit)
