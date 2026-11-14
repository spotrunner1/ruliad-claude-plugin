# Ruliad for Claude

Ruliad's TV and CTV ad production connection for Claude users. It packages the production MCP server (`https://api.theruliad.ai/mcp`, OAuth 2.1 with PKCE and dynamic client registration) together with three skills: `ruliad-tv-production` (the MCP route, including Creative IQ), `ruliad-media-planning` (media plans and the customer's own Vibe account) and `ruliad-browser-operator` (operating the Ruliad app through the Claude Code desktop Browser pane or the Claude in Chrome extension, for the parts of the product the MCP catalog does not expose). The public procedure both skills follow is https://theruliad.ai/agent-guides/claude-v1.html. The same backend and account model serve the ChatGPT plugin; this package adds Claude-specific onboarding only.

## Surfaces

| Claude surface | How it connects | Distribution |
| --- | --- | --- |
| Claude Code (CLI, desktop app, IDE) | Install this plugin. It registers the `ruliad` MCP server and the skill. Sign in once with `/mcp` → `ruliad`. | Marketplace in this repository; a public marketplace repository is required for self-serve installs. |
| Claude web and desktop (claude.ai) | Add a custom connector: Customize → Connectors → **+ Add custom connector**, name `Ruliad`, URL `https://api.theruliad.ai/mcp`, sign in when prompted. | Self-serve for Pro, Max, Team and Enterprise; Team/Enterprise owners add it under Organization settings → Connectors. |
| Anthropic directory (claude.ai/directory) | Not submitted. | Separate review; see "Public distribution" below. |

## Install in Claude Code

From a checkout of this repository:

```bash
claude plugin marketplace add /path/to/Ruliad/plugins/ruliad-claude
claude plugin install ruliad@ruliad
```

Then start Claude Code, run `/mcp`, choose `ruliad` and complete the Ruliad sign-in and consent in the browser. Tools appear as `mcp__plugin_ruliad_ruliad__<tool>`; the skills are `/ruliad:ruliad-tv-production`, `/ruliad:ruliad-media-planning` and `/ruliad:ruliad-browser-operator`.

To try it without installing:

```bash
claude --plugin-dir /path/to/Ruliad/plugins/ruliad-claude/ruliad
```

Validate after edits:

```bash
claude plugin validate /path/to/Ruliad/plugins/ruliad-claude --strict
```

## Sign-in and permissions

Claude registers itself with the Ruliad authorization server, opens the Ruliad consent page in your browser, and receives a short-lived access token bound to your Ruliad account, your workspaces and the scopes you approve: `connection:read`, `projects:read`, `projects:write`, `production:write`, `exports:write`. There is no payment or billing scope. Revoking the connection in Claude (`/mcp` → `ruliad` → clear authentication, or removing the connector) invalidates the tokens; signing out of Ruliad or changing your password revokes every connection.

Claude Code uses a loopback callback (`http://localhost:<port>/callback`); Claude web and desktop use `https://claude.ai/api/mcp/auth_callback`. Both are accepted only as exact, PKCE-bound callbacks.

## What works, and what does not

- Reads of workspaces, projects, references, production jobs and export status.
- Briefs, scene plans and conflict-checked timeline edits (trim, move, volume, mute, text, titles, clip removal).
- Production and targeted revisions when `get_production_readiness` reports `productionAvailable: true`. Creation is free; the server enforces Ruliad's own execution limits.
- Creating, editing and the watermarked preview are free. Exports render a private full-quality master; the customer unlocks the clean master per ad in Ruliad (per-ad pack or subscription), and Claude gives them the unlock link. Claude never charges a card.
- Creative IQ (free): analyze a Ruliad export or a public video URL, read the report, run clearly labelled synthetic audiences, and get a confirm-first Fix with Ruliad plan (`analyze_ad_creative`, `get_ad_creative_report`, `run_ad_synthetic_audience`, `prepare_ad_from_creative_report`).
- QC holds: a take that fails voice, continuity, product-fidelity, physics or direction QC is retaken once automatically; if it still fails, only that shot is held with a named reason, and the customer chooses to retake it (`revise_ad_shot`) or keep it (`accept_ad_held_take`).
- Chat attachments are not forwarded to Ruliad. Import an image or a product page by URL (`import_ad_reference`, `import_product_from_url`), send a small local file as `data_base64` in Claude Code, or add it in the Ruliad project and register it. See the skill for details.
- Shared detail for every assistant (the live tool list, Creative IQ, QC holds, review links, Vibe): `docs/ruliad-mcp-assistant-reference.md` in this repository.

## Public distribution

- **Claude Code**: publish this `plugins/ruliad-claude` directory (or a copy) as a public GitHub repository and users can run `claude plugin marketplace add <owner>/<repo>` followed by `claude plugin install ruliad@ruliad`.
- **Anthropic directory**: submissions go through claude.ai/directory/manage (plugin bundle and MCP connector). Review the current connector review criteria before submitting; at the time of writing they exclude connectors whose purpose is generating video, so expect the connector listing to be declined or scoped.
- Nothing in this directory changes or republishes the ChatGPT plugin package in `plugins/ruliad`.
