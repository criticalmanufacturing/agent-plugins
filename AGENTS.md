# Repository contribution rules

This repository contains Agent Plugins that are installed by both Claude Code/Cowork and
ChatGPT/Codex. Read the root `README.md` and the touched plugin's `README.md` before making
structural or release-related changes.

## Plugin development

- Treat each directory under `plugins/` as an independently versioned plugin.
- Keep the portable files (`plugin.json` and `mcp.json`) and Claude files
  (`.claude-plugin/plugin.json` and `.mcp.json`) consistent. Platform-specific schema differences
  are allowed, but the plugin identity, server URL, OAuth client ID, and scopes must not drift.
- Keep shared behavior in the plugin's shared `skills/` files. Do not create platform-specific copies
  of a skill unless the platform genuinely requires one.
- Preserve the plugin's documented scope, source/citation rules, safety constraints, and tool-calling
  limits when editing a skill. If behavior changes, update the relevant plugin documentation too.
- Do not commit credentials, client secrets, tokens, or other private configuration. Public OAuth
  client IDs and required scopes may be committed when the server requires them.
- Keep marketplace entries in `.claude-plugin/marketplace.json` and `.agents/plugins/marketplace.json`
  pointing to the correct plugin path and repository source when adding or moving plugins.

## Versioning and release artifacts

Whenever a plugin is changed, bump its plugin version before handing off the work. Update every
version-bearing plugin manifest for the touched plugin, including both `plugin.json` and
`.claude-plugin/plugin.json` when both are present. Keep the version references in that plugin's
README synchronized as well.

If a changed skill has its own `metadata.version`, bump that version when the skill behavior or
instructions change. Do not change it for unrelated asset or manifest-only edits.

Do not commit generated distribution archives. The `dist/` directory is intentionally ignored; create
local packages only when a release or installation workflow specifically requires one.

## Validation checklist

Before handing off a plugin change:

1. Parse every changed JSON file, including hidden files, with a JSON parser.
2. Check that the portable and Claude manifests have the same plugin name and version, and that
   required MCP settings are present in both configurations.
3. Check that marketplace paths and plugin names still resolve to the intended plugin.
4. If a skill changed, inspect its front matter and verify referenced files and assets exist.
5. Run `git diff --check` and review `git diff` for accidental secrets, stale versions, or unrelated
   changes.
6. Report any live OAuth handshake, platform installation, or remote MCP behavior that could not be
   tested locally; do not claim it was verified.

Use repository-appropriate tooling for these checks. There is no assumption that a test suite or
build script exists unless the repository provides one.
