# CM Agent Plugins

Critical Manufacturing's Agent Plugins. Each plugin under [`plugins/`](plugins) runs on **both**
Claude Code/Cowork and ChatGPT/Codex from one folder, with no per-platform copies of its behavior.

| Plugin | What it does |
|---|---|
| [`plugins/cm-gpt`](plugins/cm-gpt) | CM GPT, Critical Manufacturing's MES documentation assistant. |
| [`plugins/cm-customerportal`](plugins/cm-customerportal) | Customer Portal: tickets, ticket and reply drafts, licenses, environment deployment logs, known issues and enhancement requests (`https://portal.criticalmanufacturing.com/mcp`). Uses cm-gpt's documentation search when that plugin is also installed. |

Both plugins follow the same layout, shown here for cm-gpt.

The OpenAI manifest explicitly loads the same OAuth-capable MCP configuration used by Claude:

| Path | Read by |
|---|---|
| `plugins/cm-gpt/skills/ask/SKILL.md` | Both — search strategy, scope, citation rules. One file, no duplication. |
| `plugins/cm-gpt/.codex-plugin/plugin.json`, `plugins/cm-gpt/.mcp.json` | ChatGPT / Codex (explicit MCP/OAuth configuration) |
| `plugins/cm-gpt/.claude-plugin/plugin.json`, `plugins/cm-gpt/.mcp.json` | Claude Code / Cowork (Claude's own manifest format) |

Both manifests use the same MCP configuration (`https://criticalmanufacturing.ai/protected/docs/mcp`), so
there's one backend and one skill to maintain regardless of which platform a user is on.

See [`plugins/cm-gpt/README.md`](plugins/cm-gpt/README.md) for setup on each platform — it has
separate "Setup — Claude Code / Cowork" and "Setup — ChatGPT / Codex" sections.

## Repo layout

```
.claude-plugin/marketplace.json   # Claude Code marketplace listing (git-subdir source, main branch)
.agents/plugins/marketplace.json  # ChatGPT/Codex marketplace listing (same plugin, same git source)
plugins/
  cm-gpt/
    .codex-plugin/plugin.json     # OpenAI manifest; explicitly declares the shared .mcp.json
    .claude-plugin/plugin.json    # Claude Code manifest
    .mcp.json                     # Claude Code MCP config (includes OAuth client config)
    assets/logo.svg               # shared CM logo, referenced by both manifests
    skills/ask/                   # shared skill — read by both platforms
    README.md
  cm-customerportal/              # same layout; skills: check-tickets, open-ticket, reply-to-ticket,
                                  # licenses, environment-logs, known-issues, enhancement-requests
dist/
  cm-gpt.plugin                   # packaged archive of plugins/cm-gpt, for HTTPS-hosted distribution
```

## Changing CM GPT's behavior

Edit `plugins/cm-gpt/skills/ask/SKILL.md` — it's the single source both platforms read, so there's
nothing else to keep in sync. Bump `version` in both `.codex-plugin/plugin.json` and `.claude-plugin/plugin.json`
when you do.

## Installing

- **Claude Code:** point the `allowedPluginMarketplaces` managed-configuration key at this repo — see
  [plugins/cm-gpt/README.md#rollout-admin](plugins/cm-gpt/README.md#rollout-admin) for all rollout
  options.
- **ChatGPT / Codex:** `codex plugin marketplace add` this repo, or add the MCP server URL directly in
  ChatGPT developer mode — see
  [plugins/cm-gpt/README.md#setup--chatgpt--codex](plugins/cm-gpt/README.md#setup--chatgpt--codex).
  This path is unverified against a live install; test before rolling out.
