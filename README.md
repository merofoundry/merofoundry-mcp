# MeroFoundry MCP server

MeroFoundry's authoring MCP server lets a coding agent (Claude Code, Claude Desktop,
Cursor, Codex, or any MCP client) build and publish a complete application: data models,
records, business rules, pages and forms, authentication, custom domains, and a consumer
MCP server for the application's own end users.

- **Endpoint:** `https://platform.merofoundry.com/mcp`
- **Transport:** Streamable HTTP (JSON-RPC 2.0 over HTTPS)
- **Auth:** OAuth 2.1 with PKCE, dynamic client registration supported. The consent screen
  binds the grant to one application in your workspace.
- **Free tier:** Hobby, 1 workspace, 1 published app, full MCP access, no card.

## Connect

```bash
claude mcp add --transport http merofoundry https://platform.merofoundry.com/mcp
```

Register first at https://platform.merofoundry.com/register, then authorize when your
client opens the consent screen.

## What is here

This repository holds the registry metadata (`server.json`) for the hosted server. The
server itself is not open source. For a complete application built through it, see
[merofoundry/juniper-workshop](https://github.com/merofoundry/juniper-workshop).

- Site: https://merofoundry.com
- Architecture and specs: https://merofoundry.com/specs
- Docs: https://docs.merofoundry.app/guide
