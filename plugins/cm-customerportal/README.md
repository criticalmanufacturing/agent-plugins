![Critical Manufacturing](assets/logo.svg)

# CM Customer Portal plugin

Brings the Critical Manufacturing Customer Portal into **both** Claude Code / Cowork and ChatGPT /
Codex, from the same plugin folder. It bundles the connector to the Customer Portal MCP server with
skills for the most common portal tasks. Every tool acts as the signed-in portal user, so customers
see their own sites' data, and CM staff see what their portal roles allow.

Layout follows the [cm-gpt plugin](../cm-gpt/README.md):

- `skills/` is read as-is by both platforms. There is one copy of each skill.
- `.codex-plugin/plugin.json` declares `skills: "./skills/"` and `mcpServers: "./.mcp.json"` for
  OpenAI.
- `.claude-plugin/plugin.json` is Claude's manifest. Claude uses the same `.mcp.json`.
- There is deliberately no root portable `plugin.json` or `mcp.json`. The portable MCP schema
  rejects `oauth`, and this server needs its pre-registered client ID.

## Components

| Component | Name | Purpose |
|---|---|---|
| MCP server | `cm-customerportal` | Remote streamable-HTTP connector to `https://portal.criticalmanufacturing.com/mcp`. |
| Skill | `check-tickets` | List, search and summarise support tickets and their conversations. |
| Skill | `open-ticket` | Collect details, check for duplicates and known issues, and draft a new ticket. |
| Skill | `reply-to-ticket` | Read a ticket's thread and draft a reply. |
| Skill | `licenses` | List licenses, classify expiry (30 / 60 / 90 days), and show modules and limitations. |
| Skill | `environment-logs` | Find DevOps Center customer environments and explain their last deployment log. |
| Skill | `known-issues` | Check whether a problem is a known product bug and which release fixes it. |
| Skill | `enhancement-requests` | Search feature requests, their state, votes, queue position and planned release. |

### Server tools (all read-only)

`search_tickets`, `get_ticket`, `get_ticket_conversation`, `get_ticket_article`,
`search_known_issues`, `search_enhancement_requests`, `get_thread`, `list_licenses`,
`get_license_modules`, `get_license_entity_limitations`, `get_license_property_limitations`,
`list_customer_environments`, `get_customer_environment`, `get_customer_environment_apps`,
`get_deployment_logs`, `get_current_date_time`.

### Ticket creation and replies are drafts

The MCP server has **no tool to create a ticket or post a reply**. `open-ticket` and
`reply-to-ticket` produce a reviewed, copy-ready draft with a link to the portal
(`/Support/Tickets` for a new ticket and `/Entity/Ticket/<id>` for a reply), and the user submits it
there. The skills are written never to claim that something was sent. When the server gains write
tools, update these two skills to submit directly after explicit user confirmation, and add `Write`
to `interface.capabilities` in `.codex-plugin/plugin.json`.

### Works with the cm-gpt plugin (optional)

Three skills add official documentation to their answers when the [cm-gpt plugin](../cm-gpt/README.md)
is also installed. They call cm-gpt's `search_documentation` tool and follow the search and citation
rules of its `ask` skill:

| Skill | What cm-gpt adds | Search limit |
|---|---|---|
| `environment-logs` | Documented guidance for the failing deployment step or error (DevOps Center, deployment targets, Infrastructure Agent), with citations. | 2 |
| `open-ticket` | For `Question` tickets, checks whether the answer is already documented before drafting. | 2 |
| `reply-to-ticket` | Backs how-to and configuration answers with cited documentation. | 3 |

This plugin doesn't bundle the cm-gpt server. That avoids a second sign-in configuration that could
drift from cm-gpt's. Without cm-gpt the three skills work as before and skip the step without
mentioning it. The other skills don't use cm-gpt, because the portal's own data is live and specific
to the user.

### Environment logs

The only logs the server exposes are the **last deployment log** of a customer environment (`Issues`
or `Full` mode, with secret-looking values masked). MES runtime or application logs are not available
through this connector.

## Setup — Claude Code / Cowork

Uses `.mcp.json`. The connector uses OAuth (authorization code with PKCE) against
`https://security.criticalmanufacturing.com/tenant/CustomerPortal`. The server does not support
dynamic client registration, so the plugin is pre-configured with:

- **Client ID:** `Applications`
- **Scopes:** `MES`
- **Transport:** streamable HTTP

No client secret or other credential is stored in the plugin. Users are prompted to sign in on first
use.

To install manually from this repo's marketplace:

```
/plugin install cm-customerportal --marketplace criticalmanufacturing/agent-plugins
```

## Setup — ChatGPT / Codex

Uses `.codex-plugin/plugin.json`, which explicitly loads the shared `.mcp.json`, so OpenAI gets the
same pre-registered OAuth client as Claude:

- **Client ID:** `Applications`
- **Scopes:** `MES`

OpenAI performs the OAuth authorization-code flow with PKCE (`S256`). No client secret is stored in
the plugin. The `CustomerPortal` tenant must accept the OpenAI connection's redirect URI and token
exchange for the `Applications` client. Public portal submissions containing MCP servers must use
**With MCP**, because **Skills only** strips the MCP configuration.

This repo's [`.agents/plugins/marketplace.json`](../../.agents/plugins/marketplace.json) already lists
the plugin. In the ChatGPT desktop app or Codex CLI:

```bash
codex plugin marketplace add https://github.com/criticalmanufacturing/agent-plugins.git
```

Then enable it in `.codex/config.toml`:

```toml
[plugins."cm-customerportal@cm-agent-plugins"]
enabled = true
```

**Unverified:** the plugin hasn't been installed end to end in ChatGPT / Codex, or in Claude Code
from this marketplace. The OAuth handshake with the `Applications` client hasn't been confirmed on
either platform. The authorization server's discovery document lists only `client_secret_post` as a
token endpoint auth method, so confirm that a public PKCE client works on the first sign-in, and that
the tenant accepts the Claude and OpenAI redirect URIs.

## Usage

Skills trigger on their own from the request, or can be invoked as `/cm-customerportal:<skill>`.
Examples:

- "Show my open Customer Portal tickets."
- "Summarise the conversation on ticket 12345 and draft a reply."
- "Open a ticket: our production MES throws a timeout on Track-In since this morning."
- "Which of our licenses expire in the next 90 days?"
- "Why did the last deployment of our staging environment fail?" (with cm-gpt installed, the answer
  also cites the relevant DevOps Center documentation)
- "Is the SPC chart export problem a known issue? Which version fixes it?"
- "Has anyone requested multi-language label printing?"

## Behaviour notes

- **Customers and CM staff.** CM staff can see many customers. The skills ask which customer or site
  is meant instead of listing everything, and they don't mix customers in one answer.
- **Dates.** Skills call `get_current_date_time` before any relative date or expiry calculation.
- **License tiers.** Critical is 0–30 days, Warning is 31–60 days and Upcoming is 61–90 days. The
  portal's own `expirationStatus` is shown alongside.
- **Secrets.** Drafts and log excerpts never reproduce passwords, tokens, connection strings or
  license keys.
- **Tool-call budgets.** Each skill states a small per-request limit and uses small pages, so large
  portals (thousands of tickets and licenses for CM staff) don't flood the conversation.

## Rollout (admin)

Same options as cm-gpt. See [cm-gpt rollout](../cm-gpt/README.md#rollout-admin). The plugin is
listed in the root [`.claude-plugin/marketplace.json`](../../.claude-plugin/marketplace.json) with a
`git-subdir` source pointing at `plugins/cm-customerportal`.

### Tool policy

Every tool is read-only, so they can be auto-approved through `orgPluginSettings`, keyed on the
server name `cm-customerportal`:

```json
{ "cm-customerportal": { "toolPolicy": {
  "search_tickets": "allow", "get_ticket": "allow", "get_ticket_conversation": "allow",
  "get_ticket_article": "allow", "search_known_issues": "allow",
  "search_enhancement_requests": "allow", "get_thread": "allow", "list_licenses": "allow",
  "get_license_modules": "allow", "get_license_entity_limitations": "allow",
  "get_license_property_limitations": "allow", "list_customer_environments": "allow",
  "get_customer_environment": "allow", "get_customer_environment_apps": "allow",
  "get_deployment_logs": "allow", "get_current_date_time": "allow"
} } }
```

cm-gpt's tools are configured under its own server name, `cm-gpt`. See the cm-gpt README.
Revisit this if write tools are added to the server. Those must keep asking for confirmation.

## Versioning

Current version: `0.1.0`. Bump the `version` field in **both** `.claude-plugin/plugin.json` and
`.codex-plugin/plugin.json` on every change to a skill or the connector config. Bump a skill's own
`metadata.version` when its instructions change.
