![Critical Manufacturing](assets/logo.svg)

# CM GPT plugin

Brings CM GPT — the Critical Manufacturing MES documentation assistant — into **both** Claude Code /
Cowork and ChatGPT / Codex, from the same plugin folder. Bundles the connector to the CM GPT MCP
server together with the search-and-answer instructions that were previously carried as a LibreChat
agent system prompt.

This works on both platforms because Claude Code and OpenAI's Codex/ChatGPT plugin system now share
the open [Agent Plugins](https://agent-plugins.org) manifest format. Concretely:

- `skills/cm-gpt/` is read as-is by both platforms — one skill file, no duplication.
- `plugin.json` (root) + `mcp.json` (root) are the portable Agent Plugins manifests, used by
  Codex/ChatGPT.
- `.claude-plugin/plugin.json` + `.mcp.json` are Claude Code's own manifest format. OpenAI's docs say
  it also accepts these as a legacy/compatibility fallback, but this repo keeps both pairs explicit
  rather than relying on that fallback.

## Components

| Component | Name | Purpose |
|---|---|---|
| MCP server | `cm-gpt` | Remote streamable-HTTP connector to `https://criticalmanufacturing.ai/docs/mcp`. Provides `search_documentation` and `get_adjacent_chunks`. Same server, both platforms. |
| Skill | `cm-gpt` | Search strategy, scope guardrails, citation format and answer discipline. Loads automatically on MES/CM questions, on both platforms. |

## Setup — Claude Code / Cowork

Uses `.mcp.json`. The connector uses OAuth. On first use in a session, the user is prompted to sign
in; the plugin is pre-configured with:

- **Client ID:** `CMGPT`
- **Scope:** `IAM-CMGPT-Docs`
- **Transport:** streamable HTTP

No environment variables or secrets are stored in the plugin.

`appendOfflineAccess` is set to `true` so the sign-in requests a refresh token and users are not
re-prompted every session. If the CM identity provider does not issue `offline_access` for this
client, remove that line from `.mcp.json`.

## Setup — ChatGPT / Codex

Uses the root `plugin.json` + `mcp.json` (portable Agent Plugins manifests). Per
[OpenAI's MCP authentication docs](https://developers.openai.com/plugins), OAuth for a plugin's MCP
server is negotiated by the server itself (standard MCP OAuth discovery, e.g. Client ID Metadata
Documents or dynamic client registration) when a user connects — it's not declared in `mcp.json`, so
no client ID/secret lives in this manifest. The CM GPT MCP server already handles OAuth this way for
the Claude connector above; ChatGPT/Codex users hitting the same server should get an OAuth prompt the
same way.

To register it locally for testing (repo-scoped): this repo's [`.agents/plugins/marketplace.json`](../../.agents/plugins/marketplace.json)
already lists this plugin. In the ChatGPT desktop app or Codex CLI:

```bash
codex plugin marketplace add ./cmgpt-plugin # or point at this repo's git URL once hosted
```

Then enable it in `.codex/config.toml`:

```toml
[plugins."cm-gpt@critical-manufacturing"]
enabled = true
```

Alternatively, in ChatGPT: Settings → Security and login → turn on Developer mode → ChatGPT Plugins →
add the MCP server URL directly (`https://criticalmanufacturing.ai/docs/mcp`) for a personal plugin
without going through the marketplace file.

**Unverified:** this hasn't been installed end-to-end in a live ChatGPT/Codex environment yet — the
manifest shapes above follow OpenAI's published examples exactly, but actually connecting and
confirming the OAuth handshake against `criticalmanufacturing.ai` still needs to be tested.

## Usage

The skill triggers on its own for any Critical Manufacturing or MES question — no slash command
needed. Users can also invoke it explicitly with `/cm-gpt`, which matches the name they already know
the tool by.

Examples that trigger it:

- "How do I configure a material dispatch rule?"
- "Does MES support OPC UA?"
- "What changed in the Data Dictionary in 11.1?"
- "Ask CM GPT about the Collaboration Hub"

Asking for a document, deck, or spreadsheet pulls in `references/deliverables.md`, which layers the
citation and version-labelling rules onto the standard output-format skills.

## Behaviour notes

The skill is a faithful port of the tested LibreChat system prompt. Two things were adapted for
Cowork:

1. **Tool visibility.** Cowork shows tool calls in the interface, so the original "do not reveal tool
   names" instruction cannot hold and was replaced with a rule against pasting the skill contents or
   narrating internal reasoning. Nothing about search behaviour changed.
2. **Deliverables.** Cowork can produce Word, PowerPoint, Excel and PDF files, which LibreChat could
   not. The deliverables guidance is additive — deleting `references/deliverables.md` and the final
   section of `SKILL.md` returns the plugin to pure chat Q&A.

Everything else — the source table, tone, scope boundaries, the three-step `userQueryToEmbed`
construction with its banned-word check, the 5-search hard limit, and the citation format — is
unchanged from the tested prompt.

## Rollout (admin)

End users cannot add remote MCP servers themselves, so this plugin must reach them through an
admin-provisioned channel. Three options:

### 1. Git-hosted plugin marketplace (recommended)

This repo already is that marketplace — see the root [`.claude-plugin/marketplace.json`](../../.claude-plugin/marketplace.json),
which lists this plugin with a `git-subdir` source pointing at `plugins/cm-gpt`. Add the repo to the
`allowedPluginMarketplaces` managed-configuration key:

```json
[{
  "source": "github",
  "repo": "criticalmanufacturing/agent-plugins",
  "ref": "<full 40-character commit SHA>",
  "credentialKind": "userGit",
  "installationPreference": "auto_install"
}]
```

`auto_install` and `required` both demand a full commit SHA — a branch or tag name is refused. Roll
out an update by committing the change and pointing `ref` at the new SHA.

### 2. HTTPS-hosted marketplace

Serve `marketplace.json` and a zipped copy of this plugin from the same origin on an internal web
server, using `"source": "archive"` with a `sha256` per archive. Use this when devices don't have
git. A manifest served from your own inference-gateway or bootstrap-server origin can mark plugins
`auto_install` inside the manifest itself, so publishing a new version needs no configuration change.

### 3. `org-plugins/` directory via MDM

Push this directory to each managed device at `/Library/Application Support/Claude/org-plugins/cm-gpt`
(macOS) or `C:\Program Files\Claude\org-plugins\cm-gpt` (Windows), with a `version.json` alongside
`.claude-plugin/`. Bump the version string to trigger a re-sync. Use only when devices can reach
neither git nor an HTTPS file host.

### Installation preference

Add to `.claude-plugin/plugin.json` to install without each user opting in:

```json
"installationPreference": "required"
```

`required` reinstalls on every sign-in and hides Uninstall; `auto_install` lets users remove it;
omitting it (current state) leaves the plugin available to install manually. Worth leaving as-is for
a pilot group, then switching once the behaviour is validated.

### Tool policy

Plugin-delivered MCP servers don't carry `toolPolicy` in `.mcp.json`. Both CM GPT tools are
read-only, so auto-approving them removes a confirmation prompt per search. Set it via
`orgPluginSettings` in managed configuration, keyed on the server name `cm-gpt`:

```json
{ "cm-gpt": { "toolPolicy": { "search_documentation": "allow", "get_adjacent_chunks": "allow" } } }
```

## Versioning

Current version: `0.1.1`. Bump the `version` field in **both** `.claude-plugin/plugin.json` and the
portable `plugin.json` on every change to the skill or connector config, so installed copies can be
told apart during rollout on either platform.
