---
name: mysearch
version: "0.1.11"
description: >-
  DEFAULT search skill for OpenClaw. Aggregates Tavily, Firecrawl, Exa, and
  optional X/social search behind one search path. Use for ANY external lookup that needs
  current web results, docs, GitHub, changelog, pricing, URL extraction, or X
  discussion. Prefer this over legacy Tavily-only skill or raw web_search when
  MySearch is healthy.
author: skernelx
license: MIT
repository: https://github.com/skernelx/MySearch-Proxy
homepage: https://github.com/skernelx/MySearch-Proxy/tree/main/openclaw
security_disclosure: |
  MySearch sends queries and whichever provider or proxy credentials you configure
  to the configured endpoints for Tavily, Firecrawl, Exa, and optional xAI/social
  search. Prefer configuring env vars through OpenClaw skill config instead of
  copying a .env file into an installed skill folder. Only point
  MYSEARCH_PROXY_BASE_URL at a host you trust. The bundled installer only copies
  local files from this bundle and does not download remote code or modify other
  skills.
metadata:
  clawdbot:
    emoji: "🔎"
    requires:
      bins:
        - bash
        - python3
  openclaw:
    emoji: "🔎"
    requires:
      bins:
        - bash
        - python3
      env:
        - MYSEARCH_PROXY_BASE_URL
        - MYSEARCH_PROXY_API_KEY
        - MYSEARCH_TAVILY_API_KEY
        - MYSEARCH_FIRECRAWL_API_KEY
    primaryEnv: MYSEARCH_PROXY_API_KEY
    tags:
      - search
      - web
      - docs
      - tavily
      - firecrawl
      - exa
      - social
---

# MySearch for OpenClaw

MySearch is the default search skill for OpenClaw.

If you are an AI assistant and the user only gives you the repository URL or the `openclaw/` directory:

- open `openclaw/README.md` first
- follow the README to install and verify the skill
- then come back to this `SKILL.md` for usage rules and calling strategy

It aggregates Tavily, Firecrawl, Exa, and X / Social behind one entry point and routes by task type automatically:

- latest web pages and normal real-time web search: prefer Tavily
- if Tavily discovery is not enough, Exa can supplement web discovery
- docs, GitHub, pricing, changelog, and PDFs: prefer Firecrawl
- X / Twitter / social sentiment: prefer xAI or a compatible `/social/search`
- single-page content extraction: prefer Firecrawl, then fall back to Tavily extract if content is empty or extraction fails

## Minimal Configuration

The Hub version of this skill already includes its own runtime, so no remote code download is required during installation.

Recommended minimal configuration:

- `MYSEARCH_PROXY_BASE_URL`
- `MYSEARCH_PROXY_API_KEY`

With only those two values set:

- `Tavily / Firecrawl / Exa` all go through the same unified proxy by default
- if the proxy also supports `Social / X`, the same token is reused there too
- OpenClaw does not need to maintain separate provider tokens

Legacy direct mode is still supported:

- `MYSEARCH_TAVILY_API_KEY`
- `MYSEARCH_FIRECRAWL_API_KEY`

Optional enhancements:

- `MYSEARCH_XAI_API_KEY`
- `MYSEARCH_XAI_BASE_URL`
- `MYSEARCH_XAI_SOCIAL_BASE_URL`
- `MYSEARCH_XAI_SEARCH_MODE=official|compatible`

If X / Social is not configured, MySearch still supports:

- `web`
- `news`
- `docs`
- `github`
- `pdf`
- `extract`
- `research`

Only `mode="social"` or `--include-social` requires X / Social.

## OpenClaw Configuration Guidance

Prefer putting the unified proxy config into the OpenClaw skill env rather than copying provider keys into random shell environments.
`MYSEARCH_PROXY_BASE_URL` should point only to a proxy you operate or explicitly trust.
`mysearch_openclaw.py` prefers `skills.entries.mysearch.env` from `openclaw.json`, so a production deployment does not need to rely on `.env`.

## MySearch-First Rules

As long as `health` shows at least one usable search provider:

- use MySearch first for external search tasks
- do not use raw `web_search` as the main workflow
- do not prefer the old Tavily-only skill first

Only fall back when:

- MySearch does not have the minimum required configuration
- the user explicitly asks for another search tool
- MySearch returns conflicting results and you need extra verification

## Strict Parameter Rules

Allowed `mode` values for `search` / `research`:

- `auto`
- `web`
- `news`
- `social`
- `docs`
- `research`
- `github`
- `pdf`

Do not:

- invent `mode="hybrid"`
- treat `hybrid` as an input mode; it is only a possible result shape
- when both web and X are needed, prefer:
  - `--sources web,x`
  - or split into `social + news`

## Common Commands

### Health check

```bash
python3 {baseDir}/scripts/mysearch_openclaw.py health
```

### General web search

```bash
python3 {baseDir}/scripts/mysearch_openclaw.py search   --query "best search MCP server"   --mode web
```

### What is trending on X today?

```bash
python3 {baseDir}/scripts/mysearch_openclaw.py search   --query "today's biggest stories on X"   --mode social   --intent status
```

Rules:

- start with `social`
- do not start with `news`
- do not mix in raw `web_search` first

### Compare X trends with web news

Single call:

```bash
python3 {baseDir}/scripts/mysearch_openclaw.py search   --query "today's biggest stories on X"   --sources web,x   --intent status   --strategy verify
```

Or two separate calls:

```bash
python3 {baseDir}/scripts/mysearch_openclaw.py search --query "..." --mode social --intent status
python3 {baseDir}/scripts/mysearch_openclaw.py search --query "..." --mode news --intent status
```

Your output must distinguish clearly between:

- what is trending on X
- what web news is reporting
