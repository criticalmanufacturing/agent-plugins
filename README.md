# CM GPT

CM GPT is Critical Manufacturing's MES documentation assistant. This repo has a single plugin,
[`plugins/cm-gpt`](plugins/cm-gpt), that runs on **both** Claude Code/Cowork and ChatGPT/Codex from
one folder — no per-platform copies of the assistant's behavior.

This is possible because both ecosystems now read the open
[Agent Plugins](https://agent-plugins.org) manifest format:

| Path | Read by |
|---|---|
| `plugins/cm-gpt/skills/ask/SKILL.md` | Both — search strategy, scope, citation rules. One file, no duplication. |
| `plugins/cm-gpt/plugin.json`, `plugins/cm-gpt/mcp.json` | ChatGPT / Codex (portable Agent Plugins manifests) |
| `plugins/cm-gpt/.claude-plugin/plugin.json`, `plugins/cm-gpt/.mcp.json` | Claude Code / Cowork (Claude's own manifest format) |

Both manifest pairs point at the same MCP server (`https://criticalmanufacturing.ai/docs/mcp`), so
there's one backend and one skill to maintain regardless of which platform a user is on.

See [`plugins/cm-gpt/README.md`](plugins/cm-gpt/README.md) for setup on each platform — it has
separate "Setup — Claude Code / Cowork" and "Setup — ChatGPT / Codex" sections.

## Repo layout

```
.claude-plugin/marketplace.json   # Claude Code marketplace listing (git-subdir source, main branch)
.agents/plugins/marketplace.json  # ChatGPT/Codex marketplace listing (same plugin, same git source)
plugins/
  cm-gpt/
    plugin.json                   # portable manifest (OpenAI/Codex)
    mcp.json                      # portable MCP config (OpenAI/Codex)
    .claude-plugin/plugin.json    # Claude Code manifest
    .mcp.json                     # Claude Code MCP config (includes OAuth client config)
    assets/logo.svg               # shared CM logo, referenced by both manifests
    skills/ask/                   # shared skill — read by both platforms
    README.md
dist/
  cm-gpt.plugin                   # packaged archive of plugins/cm-gpt, for HTTPS-hosted distribution
```

## Changing CM GPT's behavior

Edit `plugins/cm-gpt/skills/ask/SKILL.md` — it's the single source both platforms read, so there's
nothing else to keep in sync. Bump `version` in both `plugin.json` and `.claude-plugin/plugin.json`
when you do.

## Installing

- **Claude Code:** point the `allowedPluginMarketplaces` managed-configuration key at this repo — see
  [plugins/cm-gpt/README.md#rollout-admin](plugins/cm-gpt/README.md#rollout-admin) for all rollout
  options.
- **ChatGPT / Codex:** `codex plugin marketplace add` this repo, or add the MCP server URL directly in
  ChatGPT developer mode — see
  [plugins/cm-gpt/README.md#setup--chatgpt--codex](plugins/cm-gpt/README.md#setup--chatgpt--codex).
  This path is unverified against a live install; test before rolling out.
