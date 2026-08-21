# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 1.x     | ✅                 |

## What This Is

This repository is a **collection manifest** — it links to and documents the individual MCP server projects that make up the Minecraft MCP Toolchain. No code lives here directly.

## Security Considerations

Each individual MCP server in this toolchain follows its own security model:

- **stdio servers** (`mc-pack-builder-mcp`, `mc-changelog-mcp`, `mc-modpack-mcp`, `mc-mod-config-mcp`, `mc-rp-assistant-mcp`) — operate on local filesystem paths only. Network access is limited to public Modrinth API calls (HTTPS).
- **HTTP server** (`minecraft-mcp-server`) — deployed on Cloudflare Workers; no direct filesystem access.

### What this collection does NOT do
- ❌ Execute arbitrary shell commands
- ❌ Read or write files outside user-provided paths
- ❌ Install or modify system packages
- ❌ Expose private keys or credentials

### Safe usage recommendations
- Always provide absolute paths to modpack directories you control
- Do not pass untrusted user input as `pack_dir` or `project_id` without basic validation
- Modrinth API calls use public HTTPS endpoints — no authentication required
- Refer to each sub-project's [SECURITY.md](https://github.com/dcd887/mc-pack-builder-mcp/blob/main/SECURITY.md) for detailed security notes

## Reporting a Vulnerability

Please report security issues via the individual repository's [Security Advisories](https://github.com/dcd887/mc-pack-builder-mcp/security/advisories/new) page, or open a private issue against this repo.
