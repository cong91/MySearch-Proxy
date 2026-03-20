# MySearch Proxy Console

[Back to repo](../README_EN.md)

`proxy/` is the console and gateway layer inside `MySearch Proxy`.

It is not just a key panel. It brings Tavily, Firecrawl, and Social / X into one operational workspace so you can manage:

- upstream provider keys
- downstream access tokens
- official quota sync
- compatible gateway wiring
- the final connection pattern used by MySearch MCP and skills

![MySearch Console Hero](../docs/images/mysearch-console-hero.jpg)

## What this console solves

Most proxy panels solve only one slice of the problem:

- they store keys but do not issue downstream tokens
- they issue tokens but do not sync upstream quotas
- they support Tavily but not Firecrawl
- they assume official APIs only and ignore compatible social gateways
- they are usable by humans, but not designed as a backend for MCP / Skills

`MySearch Proxy Console` pulls those pieces back together:

- a dedicated Tavily workspace
- a dedicated Firecrawl workspace
- a dedicated Social / X workspace
- one place to inspect key pools, token pools, real quota state, proxy stats, and MySearch wiring instructions

## Why it is better than a generic key panel

### 1. It separates services instead of mixing them

This is not one shared bucket for every credential.

It treats these as distinct operational surfaces:

- Tavily
- Firecrawl
- Social / X

That gives you:

- cleaner quota accounting
- isolated token usage
- clearer statistics
- more obvious downstream endpoints

### 2. It is built for a real MySearch runtime

This console is not a standalone dashboard experiment.

It is designed to feed:

- the `mysearch/` MCP
- the `skill/` bundle for Codex / Claude Code
- the `openclaw/` skill bundle

So it is a real backend for search agents, not only a UI for manual operations.
