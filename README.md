# Ruliad for Claude

Ruliad's TV and CTV ad production connection for Claude users. It packages the production MCP server (`https://api.theruliad.ai/mcp`, OAuth 2.1 with PKCE and dynamic client registration) together with two skills: `ruliad-tv-production` (the MCP route) and `ruliad-browser-operator` (operating the Ruliad app through the Claude Code desktop Browser pane or the Claude in Chrome extension, for the parts of the product the MCP catalog does not expose). The public procedure both skills follow is https://theruliad.ai/agent-guides/claude-v1.html. The same backend and account model serve the ChatGPT plugin; this package adds Claude-specific onboarding only.

## Surfaces

| Claude surface | How it connects | Distribution |
| --- | --- | --- |
| Claude Code (CLI, desktop app, IDE) | Install this plugin. It registers the `ruliad` MCP server and the skill. Sign in once with `/mcp` → `ruliad`. | Marketplace in this repository; a public marketplace repository is required for self-serve installs. |
| Claude web and desktop (claude.ai) | Add a custom connector: Customize → Connectors → **+ Add custom connector**, name `Ruliad`, URL `https://api.theruliad.ai/mcp`, sign in when prompted. | Self-serve for Pro, Max, Team and Enterprise; Team/Enterprise owners add it under Organization settings → Connectors. |
| Anthropic directory (claude.ai/directory) | Not submitted. | Separate review; see "Public distribution" below. |

## Install in Claude Code

```bash
claude plugin marketplace add spotrunner1/ruliad-claude-plugin
claude plugin install ruliad@ruliad
```

Or from a local checkout of this repository:

```bash
claude plugin marketplace add /path/to/ruliad-claude-plugin
claude plugin install ruliad@ruliad
```

Then start Claude Code, run `/mcp`, choose `ruliad` and complete the Ruliad sign-in and consent in the browser. Tools appear as `mcp__plugin_ruliad_ruliad__<tool>`; the skills are `/ruliad:ruliad-tv-production` and `/ruliad:ruliad-browser-operator`.

To try it without installing:

```bash
claude --plugin-dir /path/to/ruliad-claude-plugin/ruliad
```

Validate after edits:

```bash
claude plugin validate /path/to/ruliad-claude-plugin --strict
```

## Sign-in and permissions

Claude registers itself with the Ruliad authorization server, opens the Ruliad consent page in your browser, and receives a short-lived access token bound to your SpotRunner account, your workspaces and the scopes you approve: `connection:read`, `projects:read`, `projects:write`, `production:write`, `exports:write`. There is no payment or billing scope. Revoking the connection in Claude (`/mcp` → `ruliad` → clear authentication, or removing the connector) invalidates the tokens; signing out of Ruliad or changing your password revokes every connection.

Claude Code uses a loopback callback (`http://localhost:<port>/callback`); Claude web and desktop use `https://claude.ai/api/mcp/auth_callback`. Both are accepted only as exact, PKCE-bound callbacks.

## What works, and what does not

- Reads of workspaces, projects, references, production jobs and export status.
- Briefs, scene plans and conflict-checked timeline edits (trim, move, volume, mute, text, titles, clip removal).
- Production and targeted revisions when `get_production_readiness` reports `productionAvailable: true`. Creation is free; the server enforces Ruliad's own execution limits.
- Exports render a private master; final playback or download requires the project's delivery access. Claude never purchases an unlock or charges a card.
- Chat attachments are not forwarded to Ruliad. Add images in the Ruliad project, then register them. See the skill for details.

## Public distribution

- **Claude Code**: this repository is the public marketplace. Users run `claude plugin marketplace add spotrunner1/ruliad-claude-plugin` followed by `claude plugin install ruliad@ruliad`. The source of truth is `plugins/ruliad-claude` in the private Ruliad repository; copy it here on each release and tag it.
- **Anthropic directory**: submissions go through claude.ai/directory/manage (plugin bundle and MCP connector). Review the current connector review criteria before submitting; at the time of writing they exclude connectors whose purpose is generating video, so expect the connector listing to be declined or scoped.
- Nothing in this repository changes or republishes the ChatGPT plugin package.
