# MeroFoundry for Claude Code

Build and publish full-stack applications from Claude Code: data models, pages, forms,
business rules, consumer authentication and per-record access, all through the hosted
MeroFoundry MCP server.

## Install

```
/plugin install merofoundry
```

On first use Claude will prompt you to authorize with MeroFoundry. The plugin points at the
hosted server at `https://platform.merofoundry.com/mcp`, so there is nothing to run locally
and no API key to paste.

You need a MeroFoundry account. Sign in or sign up at
[platform.merofoundry.com](https://platform.merofoundry.com).

## What you get

**The connector, pre-wired.** No `claude mcp add`, no local process. The plugin ships the
server configuration and Claude handles the OAuth consent.

**A builder skill.** `build-merofoundry-app` teaches Claude the order of operations and the
platform's real constraints, so it does not have to discover them by trial. It covers
creating and **publishing** a schema, building pages from the published block catalog,
creating model-bound forms with an explicit access setting, per-record access through the
server-bound identity parameter, and verifying a published page in a browser rather than
trusting an exit code.

## Quickstart

Ask Claude for what you want. For example:

> Build me a bike shop catalog on MeroFoundry: a Bike model with name, price and category,
> a public page listing them, and a form to add one. Then publish it and show me the page.

Claude will create the application, define and publish the model, add pages and a form, seed
a few records, publish, and show you the rendered result.

## Documentation

- Docs and quickstart: [docs.merofoundry.app](https://docs.merofoundry.app)
- Block catalog: [docs.merofoundry.app/page-builder-blocks](https://docs.merofoundry.app/page-builder-blocks)
- Recipes, including per-record access:
  [docs.merofoundry.app/recipes/row-level-security](https://docs.merofoundry.app/recipes/row-level-security)
- MCP tool reference: [docs.merofoundry.app/mcp-tools](https://docs.merofoundry.app/mcp-tools)

## Support

[merofoundry.com](https://merofoundry.com)
