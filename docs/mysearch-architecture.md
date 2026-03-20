# MySearch Architecture

The goal of `MySearch` is not to cram every search behavior into one “universal endpoint”, but to separate capabilities into clear layers:

```text
┌──────────────────────────────────────────────────────┐
│ Layer 1: Skill / Decision Layer                     │
│ Decides when to search, which mode to use, what     │
│ evidence standard to apply, and how to present      │
│ the result.                                         │
├──────────────────────────────────────────────────────┤
│ Layer 2: MCP / Orchestration Layer                  │
│ search / extract_url / research / route / normalize │
├──────────────────────────────────────────────────────┤
│ Layer 3: Provider Layer                             │
│ Tavily | Firecrawl | official xAI | custom social   │
└──────────────────────────────────────────────────────┘
```

## Design Principles

- `search` is the unified entry point, so callers do not have to reason about provider differences directly
- `intent` and `strategy` separately describe **what** to search for and **how** to search
- `extract_url` is responsible only for content extraction and does not overload the search result itself
- `research` is a lightweight research workflow, not a heavy pipeline for every query
- `official xAI` and `grok2api-compatible` are treated as distinct modes so model gateways are not confused with true search backends

## Four Capability Tracks

### 1. Web discovery

- Default provider: Tavily
- Best for: general web pages, news, fast answers
- Main outputs: `answer`, base `results`, `citations`
- Under `verify` / `deep`: Tavily and Firecrawl are used for lightweight cross-provider verification and deduplication

### 2. Docs / content extraction

- Default provider: Firecrawl
- Best for: docs sites, GitHub, pricing, changelogs, PDFs, and page extraction
- Main outputs: Markdown content and structured metadata

### 3. Official X / social search

- Default provider: official xAI Responses API
- Requirement: the backend truly supports `x_search` / `web_search` server-side tools
- Main outputs: social `results`, `citations`, `tool_usage`

### 4. Custom social search

- Best for: `grok2api`-compatible gateways plus your own aggregated search API
- Mode: `MYSEARCH_XAI_SEARCH_MODE=compatible`
- Recommendation: set only `MYSEARCH_XAI_SOCIAL_BASE_URL`; `/social/search` is appended automatically by default
- Principle: keep the model gateway separate from the search backend
- Built-in reference implementation: `mysearch.social_gateway`

## Why It Is Split This Way

This structure borrows ideas from several public projects, but does not copy any of them directly:

- `blessonism/openclaw-search-skills`
  - borrowed: separation between the search layer and content-extraction layer
  - not copied: script-style multi-source scoring and thread pulling
- `ckckck/UltimateSearchSkill`
  - borrowed: skill-first decision flow and evidence standards
  - not copied: the Shell + Docker infrastructure model
- `skernelx/xai-mcp-server`
  - borrowed: separation between MCP server, API client, and tool handler
  - not copied: the assumption that official xAI server-side search semantics apply to every compatible gateway

## Capabilities Already Absorbed

- intent-aware routing
  - `comparison`, `exploratory`, `resource`, and `tutorial` no longer follow exactly the same path as generic web Q&A
- strategy-aware execution
  - `fast`, `balanced`, `verify`, and `deep` let callers control evidence density explicitly
- lightweight cross-provider verification
  - web-oriented flows can call Tavily + Firecrawl together when needed and return `evidence`
- research evidence metadata
  - `research` returns provider coverage, citation counts, verification state, and related metadata

## Current Boundaries

`MySearch` now explicitly supports:

- official and compatible Tavily interfaces
- official and compatible Firecrawl interfaces
- official xAI search mode
- `grok2api-compatible + custom social gateway`

It does **not** currently build in:

- automatic multi-query bundle expansion
- heavy cross-source global scoring and ranking
- thread pulling / citation tracing
- `web_map`

If these capabilities are added later, they should live in `research` or in a new advanced tool, not turn the base `search` path into an uncontrolled everything-bucket.
