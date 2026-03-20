# MySearch Proxy

`MySearch Proxy` is a public-facing search stack for `Codex`, `Claude Code`, `OpenClaw`, and custom agent workflows.

It combines three things that are usually split across separate projects:

- an installable search `MCP`
- reusable search `Skill` packages
- a unified `Proxy Console` for routing and operations

## Project entry points

- GitHub: [skernelx/MySearch-Proxy](https://github.com/skernelx/MySearch-Proxy)
- Docker Hub: [skernelx/mysearch-proxy](https://hub.docker.com/r/skernelx/mysearch-proxy)
- OpenClaw Hub Skill: [clawhub.ai/skernelx/mysearch](https://clawhub.ai/skernelx/mysearch)
- Recommended Tavily / Firecrawl provider layer: [skernelx/tavily-key-generator](https://github.com/skernelx/tavily-key-generator)

![MySearch Console Hero](./docs/images/mysearch-console-hero.jpg)

This is not just another Tavily wrapper.

The goal is to make `Tavily`, `Firecrawl`, and optional `X / Social` search work as one reusable product that can be installed locally, shared publicly, and routed through the same runtime across multiple environments.

The public OpenClaw page is live here:

- [clawhub.ai/skernelx/mysearch](https://clawhub.ai/skernelx/mysearch)
- the screenshot below is a real capture from `2026-03-17`
- treat the live ClawHub page as the source of truth for the current scan state

![MySearch Skill Security Scan](./docs/images/mysearch-skill-security-scan.jpg)

## What is in this repository

- [`mysearch/`](./mysearch/README_EN.md)
  - the installable MySearch MCP
  - ships `search`, `extract_url`, `research`, and `mysearch_health`
- [`proxy/`](./proxy/README_EN.md)
  - the proxy layer and web console
  - manages Tavily / Firecrawl key pools, downstream tokens, quota sync, and `/social/search`
- [`skill/README_EN.md`](./skill/README_EN.md)
  - MySearch skill guide for `Codex` / `Claude Code`
  - includes the “tell the AI to install it for me” flow
- [`openclaw/README_EN.md`](./openclaw/README_EN.md)
  - bundled MySearch skill guide for OpenClaw / ClawHub
  - includes the “tell the AI to install the OpenClaw skill” flow
- [`docs/mysearch-architecture.md`](./docs/mysearch-architecture.md)
  - architecture and design boundaries

## What problem this project solves

Many search MCPs or search skills share the same limitations:

- they only do web search and cannot extract content reliably
- they work for news, but break down on docs, GitHub, PDFs, pricing, or changelogs
- they ship prompts, not a real MCP runtime
- they ship a key panel, not an agent-ready search workflow
- they assume official APIs only, which makes self-hosting awkward
- they lose most of their value if X / Social is unavailable

`MySearch Proxy` addresses those gaps by splitting the stack into clear layers:

```text
tavily-key-generator
  -> provider layer for Tavily / Firecrawl and optional aggregation APIs

MySearch Proxy
  -> MCP, Skills, OpenClaw Skill, Proxy Console, Social / X routing

Codex / Claude Code / OpenClaw / custom agents
  -> one shared search entry
```

The default recommendation is not “paste official keys everywhere”. The recommended public deployment is:

- `tavily-key-generator` for Tavily / Firecrawl provider delivery
- `MySearch Proxy` for routing, MCP, skills, and console operations

## Why it is better than typical alternatives

### 1. It is not a single-provider MCP

`MySearch` routes by task type:

- general web and news -> Tavily
- docs, GitHub, PDFs, pricing, changelogs, and extraction -> Firecrawl
- X / Social -> xAI or compatible `/social/search`

That makes it a search orchestrator, not a thin one-provider wrapper.

### 2. It is not “just a skill”

This repository ships:

- MCP
- Codex / Claude Code skill
- OpenClaw skill
- Proxy Console

So the same search logic can be reused across local agents, OpenClaw, and team gateways instead of being re-implemented per runtime.

### 3. It handles extraction and lightweight research

The value is not only `search`:

- `extract_url`
  - prefers Firecrawl and falls back to Tavily extract
- `research`
  - bundles search, extraction, and evidence into a lightweight research flow

That is far more useful for agent workflows than returning a few links.

### 4. Official-first, but compatible-friendly

You can:

- use official Tavily / Firecrawl / xAI APIs
- override `BASE_URL + PATH + AUTH_*`
- route Tavily / Firecrawl through your own aggregation gateway
- route X / Social through a compatible `/social/search`

That matters if you want a reusable public project instead of a one-off setup.

### 5. X / Social is optional, not the installation gate

Without `xAI` or `grok2api`, the stack still supports:

- `web`
- `news`
- `docs`
- `github`
- `pdf`
- `extract`
- `research`

Only explicit social routes degrade.
