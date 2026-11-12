# Flopinio plugin

Connect Claude, Cursor, ChatGPT, or Codex to your Flopinio account to find
client-work context and carry requests, projects, tasks, notes, decisions, time,
and invoices forward. The shared `flopinio-workspace` skill explains the
workflow; tools run on the hosted Flopinio MCP server at `https://flopinio.com/mcp`.

This repository contains only the plugin manifests, skill, and branding assets.
It contains no Rails application or MCP server implementation, account bindings,
access tokens, or reviewer credentials. The MIT license covers these package
files. The hosted service remains subject to Flopinio's terms and account access.

## Package formats

- Claude: `.claude-plugin/plugin.json` and `.mcp.json` (`type: http`).
- Cursor: `.cursor-plugin/plugin.json`. The hosted server is declared inline
  as `mcpServers.flopinio.url`. Skills come from folder discovery under `skills/`.
- ChatGPT and Codex portable package: root `plugin.json` and `mcp.json`
  (`type: streamable-http`, Agent Plugins `$schema`).
- All three formats share `skills/flopinio-workspace/SKILL.md` and version `0.2.1`.

The OpenAI package name preserves the existing published plugin identity.
Changes to this repository do not update any directory listing automatically
until a release source and its publication process are configured.

### Manifest precedence

Cursor identifies a package by manifest location
([Plugins](https://cursor.com/docs/plugins),
[Plugins reference](https://cursor.com/docs/reference/plugins)):

- Root `plugin.json` is an Agent Plugin. This file keeps the published
  OpenAI/Codex name `app-69f781e661a48191aeec398bb44bb811`. Codex and ChatGPT
  read it. A spec-conformant Agent Plugin can also load in Cursor, and that
  format reads root `mcp.json`, including an explicit transport such as
  `streamable-http`.
- `.cursor-plugin/plugin.json` is a Cursor Plugin. Marketplace submission and
  local Cursor testing of this repository use this file. Its `name` is
  `flopinio`. Cursor-only fields (`displayName`, `logo`, inline `mcpServers`)
  live here.

The reference does not merge those two manifests, and it does not say both are
installed as separate plugins. In a marketplace, resolution looks for
`.cursor-plugin/plugin.json` inside the plugin directory first, and that
manifest's values take precedence over the marketplace entry. An installed
marketplace plugin with the same name takes precedence over a copy in
`~/.cursor/plugins/local`.

Cursor Plugins discover root `mcp.json` only when the manifest omits
`mcpServers`. Setting `mcpServers` replaces that discovery. A Cursor remote
server is a `url`; Cursor infers Streamable HTTP or SSE from it. The plugins
reference does not use the Agent Plugins literal `streamable-http` for that
format. In Cursor's own MCP config, `type` is documented for `stdio` servers.

This repository also ships Claude's `.mcp.json`. Current Cursor builds look up
plugin MCP config as `.mcp.json`, then `mcp.json`, so auto-discovery reads the
Claude file and never opens the portable `mcp.json` (Cursor staff,
[cursor/plugins#252](https://github.com/cursor/plugins/issues/252)). The
plugins reference only documents `mcp.json`. The Cursor manifest therefore
inlines `https://flopinio.com/mcp`. That override leaves `.mcp.json` and
`mcp.json` unchanged for the hosts that own them. The inline server has no
`${VAR}` placeholders, so the manifest declares no `variables` schema.

`skills/flopinio-workspace/SKILL.md` is picked up by default folder discovery.
Its frontmatter `name` is lowercase kebab-case and `description` is set, which
is what the skills format requires. The manifest does not set `skills`, because
an explicit path would replace folder discovery.

Tool descriptions, annotations, and input schemas come from the hosted MCP
server at connection time. Server contract fixes require an application
deployment; this package carries the shared workflow guidance and branding.

Both packaged PNGs are exact copies of the current application's 512-by-512
`public/icon.png`, derived from `app/assets/images/flopinio_logo_svg.svg`.
The Claude manifest references `assets/icon.png`; the portable manifest
references both PNGs. The Cursor `logo` field is `assets/logo.png`, relative
to the plugin root (this repository root), which is the path form in the
plugins reference and the Cursor plugin template. An icon uploaded separately
for another host's connector listing must be updated in that submission portal.

## Connect

You need a Flopinio account with access to the intended organization and its MCP
features. Connect through the host's OAuth flow, select the intended workspace,
and review the requested tool-family scopes. Each user authorizes their own
account. Never put a shared bearer token or client secret in an MCP manifest.

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

Hosted Claude uses automatic OAuth client registration; Claude Code uses a
loopback callback with S256 PKCE. Both connect to the same hosted endpoint.
See the [connection guide](https://flopinio.com/docs/mcp) for setup,
reconnection, scopes, approvals, and revoking access. Package validation checks
structure; authenticated reads, staged writes, and token lifecycle behavior
still need testing in the intended Claude client.

## Local Cursor testing

Copy this folder to `~/.cursor/plugins/local/flopinio`. Cursor loads a symlink
there only when the target resolves to a directory inside
`~/.cursor/plugins/local`. Restart Cursor, or run Developer: Reload Window. Local plugin imports must be allowed. On Enterprise, that control is
Allow Local Plugin Imports under Dashboard > Settings > Security & Identity >
Marketplace and Plugins, and it is off by default.

Open Customize and confirm the `flopinio-workspace` skill and the Flopinio MCP
server. Plugin servers are shown as `plugin-` plus the plugin name and server
name, so this server is `plugin-flopinio-flopinio`. Sign in with OAuth, select
the intended workspace, and ask Cursor to read a project note you can access
or to call another scoped tool. If a marketplace plugin named `flopinio` is
already installed, that install takes precedence over the local copy.

Flopinio's OAuth server must allow Cursor's MCP OAuth redirect URLs before
sign-in can finish. The [MCP reference](https://cursor.com/docs/mcp) lists
these fixed callbacks:

- Web and Cursor Agents: `https://www.cursor.com/agents/mcp/oauth/callback`
- Desktop app: `http://localhost:8787/callback`

Dynamic client registration also sends
`cursor://anysphere.cursor-mcp/oauth/callback`, and some desktop installs still
authorize with that URI (Cursor staff on the
[redirect mismatch thread](https://forum.cursor.com/t/mcp-oauth-redirect-uri-mismatch-in-cursor-desktop/171867)
and the
[localhost callback thread](https://forum.cursor.com/t/mcp-oauth-callback-changed-to-http-localhost-8787-callback-and-authentication-still-fails/165752)).
Allow all three. These are manual test instructions, not evidence that they
have passed.

Marketplace reviewers need their own Flopinio test account. Contact
support@flopinio.com. Do not commit reviewer credentials to this repository.

## Data handling

The package runs no local MCP server, install script, or hook. Tool requests
and their arguments go to Flopinio's hosted service; permitted results return
to the connected AI client. The package declares no other data destination
and embeds no credentials. Flopinio retains service and approval records under
its [privacy policy](https://flopinio.com/privacy). Data received by Claude is
subject to [Anthropic's privacy policy](https://www.anthropic.com/legal/privacy)
and the user's applicable account terms and settings. Data received by Cursor
is subject to [Cursor's privacy policy](https://cursor.com/privacy) and the
user's applicable account terms and settings.

To revoke access, open Flopinio Settings > Integrations and revoke the relevant
connection or token. Removing the plugin alone may leave credentials active;
revoking access does not delete information already shared with the AI provider.

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
