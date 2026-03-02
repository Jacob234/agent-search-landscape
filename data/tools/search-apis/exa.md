# Exa

<!--
COLLECTION METADATA
==================
Collected: 2026-03-01
Collector: claude-opus-4-6
Session: agent-search-landscape research
GitHub Issue: N/A
Collection Method: WebSearch (multiple queries), cross-referenced with official docs URLs, GitHub repos, PyPI, npm
Last Verified: 2026-03-01
Confidence: medium — WebFetch unavailable; data gathered via WebSearch summaries cross-referenced across multiple sources. No direct page reads of docs.exa.ai or exa.ai/pricing were possible.
-->

## Identity & Basics

| Field | Value | Source |
|-------|-------|--------|
| Organization | Exa Labs, Inc. | <!-- https://www.ycombinator.com/companies/exa --> |
| Website | https://exa.ai | <!-- https://exa.ai --> |
| Former Name | Metaphor (rebranded to Exa early 2024) | <!-- https://cerebralvalley.ai/blog/exa-aims-to-reshape-web-search-4AOjbOWK9sxxXCKdjQG1EK --> |
| License | Proprietary API service (SDKs are MIT-licensed) | <!-- https://pypi.org/project/exa-py/ ; core search engine is proprietary --> |
| Founded | 2021 | <!-- https://www.ycombinator.com/companies/exa --> |
| Y Combinator | S21 batch | <!-- https://www.ycombinator.com/companies/exa --> |
| Founders | Will Bryk, Jeff Wang (met as Harvard freshmen) | <!-- https://cerebralvalley.ai/blog/exa-aims-to-reshape-web-search-4AOjbOWK9sxxXCKdjQG1EK --> |
| HQ | San Francisco, CA | <!-- https://www.cbinsights.com/company/metaphor-1 --> |
| Funding | $85M Series B (Sept 2025) at $700M valuation, led by Benchmark; Lightspeed, Y Combinator, NVentures (NVIDIA) participating. Seed $5M via YC S21. | <!-- https://exa.ai/blog/announcing-series-b ; https://siliconangle.com/2025/09/03/nvidia-backs-85m-round-ai-search-startup-exa/ --> |
| GitHub Org | https://github.com/exa-labs (73 repos) | <!-- https://github.com/exa-labs --> |
| GitHub Stars (exa-py) | ~170 | <!-- https://github.com/exa-labs/exa-py as of 2026-03-01 --> |
| GitHub Stars (exa-js) | ~97 | <!-- https://github.com/exa-labs/exa-js as of 2026-03-01 --> |
| GitHub Stars (exa-mcp-server) | ~3,700 | <!-- https://github.com/exa-labs/exa-mcp-server as of 2026-03-01 --> |
| Community Size | Not independently measured; LinkedIn page: "Exa (prev. Metaphor)" | <!-- https://www.linkedin.com/company/exa-ai --> |

## Category Placement

- **Primary**: AI-Native Web Search API
- **Secondary**: Web Crawler / Content Extraction, Entity Sourcing (Websets)

## Core Capability

> Exa is a web search engine built from scratch for AI agents and LLMs, using neural embeddings rather than keyword matching to retrieve semantically relevant web content, with its own proprietary web index of billions of pages.

## API Surface

<!-- Sources: https://docs.exa.ai/reference/search ; https://docs.exa.ai/llms.txt ; https://data4ai.com/blog/vendor-spotlights/exa-ai-review/ ; https://docs.exa.ai/reference/how-exa-search-works -->

### Endpoints / Methods

| Endpoint | Method | Purpose | Input | Output |
|----------|--------|---------|-------|--------|
| `/search` | POST | Semantic or keyword web search | JSON: `query`, `type`, `numResults`, `contents`, `includeDomains`, `excludeDomains`, `startPublishedDate`, `endPublishedDate`, `startCrawlDate`, `endCrawlDate`, `category`, `useAutoprompt` | JSON: `requestId`, `results[]` with `title`, `url`, `publishedDate`, `author`, `id`, `image`, `favicon`, `score`, `text`, `highlights`, `highlightScores`, `summary` |
| `/findSimilar` | POST | Find pages semantically similar to a given URL | JSON: `url`, `numResults`, `contents`, `includeDomains`, `excludeDomains` | Same result schema as `/search` |
| `/contents` (getContents) | POST | Retrieve clean parsed content for a list of URLs | JSON: `ids` (list of URLs or Exa IDs), `text`, `highlights`, `summary` | JSON: `results[]` with content fields populated |
| `/answer` | POST | Direct answers to questions with citations | JSON: `query`, `text` | JSON: answer text with source citations |
| `/research` | POST | Agentic multi-step research with structured output | JSON: `query`, `outputSchema` (supports text or JSON object) | JSON: structured research results with citations; supports `output_schema` for structured JSON (max nesting depth 2, max 10 properties) |
| Websets API (`/websets`) | POST/GET | Create, list, preview entity-sourcing jobs | JSON: search criteria, enrichment columns | JSON: webset items with enriched entity data |

### Search Types

| Type | Description | Latency (P50) |
|------|-------------|---------------|
| `auto` (default) | Intelligently combines neural and other search methods | Varies |
| `neural` | Pure embeddings-based semantic search | ~350ms |
| `keyword` | Google-style keyword/BM25 search | Fast |
| `fast` | Streamlined neural + reranker models | <350ms |
| `deep` | Light deep search with query expansion | ~3.5s |
| `deep-reasoning` | Base deep search | Slower |
| `deep-max` | Maximum-effort deep search | Slowest |
| `instant` | Lowest-latency neural search (launched Feb 2026) | 100-200ms (sub-200ms) |

### Content Retrieval Options

| Option | Description |
|--------|-------------|
| `text` | Full page text, configurable via `maxCharacters` |
| `highlights` | AI-extracted relevant excerpts/snippets, configurable via `maxCharacters` |
| `summary` | AI-generated summary of each result |
| `livecrawl` | Options: `fallback` (default, use cache first), `preferred` (try fresh, fallback to cache), `always` (always crawl fresh) |

### Authentication

- **Method**: API key via `x-api-key` HTTP header
- **Key management**: https://dashboard.exa.ai/api-keys
- **Key creation**: Supports named keys with optional per-key rate limits
- **Environment variable**: `EXA_API_KEY`

### Rate Limits

- Rate limits are defined as QPS (queries per second) per endpoint
- Research API uses concurrent task limits (not QPS) due to long-running operations
- Default rate limits apply to all users; specific values per tier are not publicly documented in detail
- Higher rate limits available via Enterprise plan (contact hello@exa.ai)
- Websets concurrency: Starter = 2 concurrent searches, Pro = 10 concurrent searches

### Input/Output Formats

- **Protocol**: REST API over HTTPS
- **Base URL**: `https://api.exa.ai`
- **Content-Type**: `application/json`
- **Date format**: ISO 8601 for temporal filtering
- **OpenAPI spec**: https://raw.githubusercontent.com/exa-labs/openapi-spec/refs/heads/master/exa-openapi-spec.yaml
- **Filtering**: `includeDomains`, `excludeDomains`, `category` (e.g., people, company, code), date ranges

## Pricing

<!-- Sources: https://exa.ai/pricing ; https://findmyaitool.io/tool/exa/ ; https://dev.to/ritza/best-serp-api-comparison-2025-serpapi-vs-exa-vs-tavily-vs-scrapingdog-vs-scrapingbee-2jci -->
<!-- Pricing as of: 2026-03-01 -->

### Search API (Pay-As-You-Go)

| Tier | Price | Includes |
|------|-------|----------|
| Free | $0 (one-time $10 credit, no expiration, no CC required) | ~2,000 searches at standard rate |
| Pay-as-you-go | $5 per 1,000 searches (1-25 results per query) | No monthly commitment |
| Higher result counts (26-100 results) | ~$0.025 per search | Premium rate for larger result sets |

### Websets Product (Subscription)

| Tier | Price | Credits/mo | Seats | Results per Webset | Enrichment Columns | Concurrent Searches |
|------|-------|-----------|-------|-------------------|--------------------|---------------------|
| Starter | $49/mo | 8,000 | 1 | Up to 100 | 10 | 2 |
| Pro | $449/mo | 100,000 | 10 | Up to 1,000 | 50 | 10 |
| Enterprise | Custom | Custom | Unlimited | Unlimited | Unlimited | Custom |

Websets credit economy: 10 credits = 1 all-green (fully validated) result in a generated webset.

### Per-Unit Economics at Scale

- Standard search: $0.005 per search (1-25 results)
- At scale, pricing approaches $0.025 per search for larger result sets
- Content retrieval (text/highlights/summary) costs are included in the search credit
- Enterprise volume discounts available upon request
- AWS Marketplace listing also available for procurement convenience

## Technical Architecture

<!-- Sources: https://exa.ai/blog/how-to-build-nextgen-search ; https://docs.exa.ai/reference/how-exa-search-works ; https://exa.ai/blog/meet-the-exacluster ; https://exa.ai/blog/bm25-optimization ; https://www.marktechpost.com/2026/02/13/exa-ai-introduces-exa-instant-a-sub-200ms-neural-search-engine-designed-to-eliminate-bottlenecks-for-real-time-agentic-workflows/ -->

### How It Works

Exa is fundamentally different from traditional search engines. Instead of preprocessing documents into keyword indices (inverted index / BM25), Exa trains neural transformer models to convert each document into dense vector embeddings. Search queries are also converted into embeddings, and retrieval is performed via nearest-neighbor similarity in embedding space.

**Key architectural concepts:**

1. **Next-Link Prediction**: Exa's core model is trained on a "next-link prediction" task -- given a description of a webpage, predict which URL would follow. This teaches the model to understand the semantic content of web pages.

2. **Hybrid Retrieval**: The production system uses a hybrid approach combining both embedding-based neural search and optimized BM25 keyword search, selecting the best method or combining results depending on the query.

3. **Own Web Index**: Exa crawls and indexes billions of web pages independently (not relying on Google or Bing). The index is updated every minute for real-time freshness.

4. **Multi-Stage Pipeline**: Query -> embedding generation -> approximate nearest neighbor search in vector DB -> neural reranking -> content extraction -> response formatting.

### Models / Infrastructure

- **Embedding models**: Proprietary transformer models trained on web-scale data; uses matryoshka embeddings (multi-resolution) and binary quantization for efficient storage and retrieval
- **Reranker**: Separate neural reranker model that re-scores candidate results for final ranking
- **Vector database**: Custom-built in Rust; uses clustering, matryoshka embeddings, binary quantization, and SIMD operations for performance
- **GPU cluster ("Exacluster")**: 144 H200 GPUs (18 nodes of 8 each) for embedding billions of pages, training rerankers, and serving inference. Plans to expand 5x with Series B funds.
- **Specialized models**: Fine-tuned retrieval models for specific categories (people search, company search, code search)
- **BM25 optimization**: Custom-optimized BM25 implementation for the keyword search component

### Self-Host Options

No self-hosting available. Exa is a proprietary cloud API service. The search engine, web index, and models are all hosted by Exa.

## Integration Surface

<!-- Sources: https://docs.exa.ai/reference/exa-mcp ; https://github.com/exa-labs/exa-mcp-server ; https://github.com/exa-labs/exa-py ; https://github.com/exa-labs/exa-js ; https://docs.exa.ai/reference/langchain ; https://docs.exa.ai/reference/llamaindex ; https://docs.exa.ai/reference/crewai ; https://docs.exa.ai/reference/vercel ; https://github.com/exa-labs/ai-sdk -->

| Integration | Status | Package / Install | Notes |
|------------|--------|-------------------|-------|
| MCP Server | Official, open-source | `npx exa-mcp-server` or hosted at `https://mcp.exa.ai/mcp` | ~3,700 GitHub stars; supports Claude Desktop, Cursor, VS Code, Zed; tools: web_search, code_search, crawl. Also has `websets-mcp-server` and `exa-knowledge-mcp`. |
| Python SDK | Official, MIT license | `pip install exa-py` (import as `exa_py`) | Sync + async support; Python >=3.9; ~170 GitHub stars |
| Node/TS SDK | Official | `npm install exa-js` (import as `exa-js`) | Full TypeScript types; ~97 GitHub stars |
| LangChain (Python) | Official partner integration | `pip install langchain-exa` | Provides `ExaSearchRetriever` and `ExaSearchResults` tool; supports neural/keyword/auto search types; lives in `langchain-ai/langchain` monorepo under `libs/partners/exa/` |
| LangChain (JS) | Official | `npm install @langchain/exa` | JavaScript/TypeScript counterpart |
| LlamaIndex | Official integration | `pip install llama-index-tools-exa` | Provides `ExaToolSpec` with tools: `search`, `retrieve_documents`, `search_and_retrieve_documents`, `find_similar`, `current_date` |
| CrewAI | Documented integration | Use `exa-py` with `@tool` decorator | Custom tool pattern using Exa Python SDK within CrewAI agents |
| Vercel AI SDK | Official | `npm install @exalabs/ai-sdk` | `webSearch` tool for Vercel AI SDK; defaults: type=auto, 10 results, 3000 chars |
| Composio | Third-party | Via Composio platform | Bridges Exa to LangChain, LlamaIndex, CrewAI via Composio toolkits |
| IBM WatsonX | Documented | Via MCP or direct API | Listed in Exa's integration docs |
| OpenRouter | Documented | Via MCP or direct API | Listed in Exa's integration docs |

## Performance Data

<!-- Sources: https://exa.ai/versus/tavily ; https://data4ai.com/blog/tool-comparisons/exa-ai-vs-tavily/ ; https://www.humai.blog/ai-search-apis-compared-tavily-vs-exa-vs-perplexity/ ; https://www.marktechpost.com/2026/02/13/exa-ai-introduces-exa-instant-a-sub-200ms-neural-search-engine-designed-to-eliminate-bottlenecks-for-real-time-agentic-workflows/ ; https://exa.ai/blog/people-search-benchmark ; https://exa.ai/blog/company-search-benchmarks -->

### Published Benchmarks

| Benchmark | Exa Score | Tavily Score | Source |
|-----------|-----------|-------------|--------|
| WebWalker (complex multi-hop retrieval) | 81% | 71% | Fortune 100 enterprise evaluation (cited on exa.ai/versus/tavily) |
| MKQA (multilingual queries) | 70% | 63% | Same enterprise evaluation |
| 500-query real customer workload | 0.648 | 0.343 | Same enterprise evaluation |

**Note**: The 81% vs 71% benchmark comes from Exa's own comparison page (exa.ai/versus/tavily), citing an independent Fortune 100 enterprise evaluation. This is Exa-published data; independent third-party verification of the exact methodology is not publicly available.

### Speed Claims

| Metric | Value | Source |
|--------|-------|--------|
| Exa Fast P50 latency | <350ms | docs.exa.ai |
| Exa Instant P50 latency | 100-200ms (sub-200ms) | exa.ai/blog/exa-instant (Feb 2026) |
| Exa Deep P50 latency | ~3.5s | docs.exa.ai |
| Exa p95 latency (enterprise eval) | 1.4-1.7s | exa.ai/versus/tavily |
| Tavily p95 latency (enterprise eval) | 3.8-4.5s | exa.ai/versus/tavily |
| Exa Instant claim | "Fastest web search engine in the world, faster than Google search" | exa.ai/blog/exa-instant |
| Exa Fast claim | "30% faster than the next fastest API" | docs.exa.ai |
| Exa Instant vs competitors | "Up to 15x faster" | marktechpost.com coverage |

### Accuracy / Coverage Claims

- Web index: "billions of pages" indexed
- Index freshness: Updated every minute
- People search: 1B+ public profiles indexed
- Open benchmarks published for: people search (1,400 queries), company search (800 queries), code search hallucination evaluation
- Content extraction: Exa 69.2% coverage vs Firecrawl 77.2% in one third-party evaluation (Source: firecrawl.dev/blog/exa-alternatives)

## Limitations

<!-- Sources: https://skywork.ai/skypage/en/Exa-AI-The-Ultimate-Guide-for-Developers-AI-Builders/1972878623855276032 ; https://10web.io/ai-tools/exa/ ; https://data4ai.com/blog/vendor-spotlights/exa-ai-review/ ; https://www.firecrawl.dev/blog/exa-alternatives ; https://docs.exa.ai/websets/faq -->

### What It Can't Do

- **No generation/synthesis**: Exa is retrieval-only. It does not generate text, summarize across multiple documents, or perform reasoning. (The `/answer` and `/research` endpoints provide some synthesis but are separate from core search.)
- **No self-hosting**: Entirely cloud-hosted; no on-premise or self-hosted option.
- **No image/video search**: Limited multimodal capabilities; primarily text-based retrieval.
- **No real-time streaming results**: Results are returned as complete JSON responses.

### Known Failure Modes

- **Niche/low-traffic domains**: Quality-based indexing means obscure or low-traffic sites may not be indexed. Less effective for ultra-niche or low-volume domains.
- **Overly restrictive Websets queries**: Very narrow searches with too many restrictive criteria may return zero matches.
- **Raw HTML quality**: Sometimes requires additional processing for clean data extraction from crawled content.
- **Content extraction coverage gap**: Third-party benchmarks show Firecrawl outperforms Exa on raw extraction coverage (77.2% vs 69.2%).

### Documented Issues

- **Pricing complexity**: Credit-based pricing can be difficult to predict; costs vary based on endpoint used, search type, number of results, and content options.
- **Documentation maturity**: Documentation described as "still minimal compared to older platforms" by some third-party reviewers (though this may be improving).
- **Scalability cost**: Massive data volumes can incur significant cost increases.
- **Websets is not general Q&A**: Websets is designed for entity sourcing, not open-ended question answering.

## Differentiation

> Exa is unique among AI search APIs because it built its own web index and neural search engine from scratch, rather than wrapping Google/Bing APIs. Its "next-link prediction" training approach gives it genuine semantic understanding of queries, not just keyword matching with an LLM layer on top. This architectural ownership -- from crawler to custom Rust vector DB to GPU inference cluster -- enables latency and relevance advantages that wrapper-based APIs cannot replicate.

**Key differentiators:**

1. **Own web index**: Unlike Tavily, SerpAPI, and others that wrap existing search engines, Exa maintains its own index of billions of pages updated every minute.

2. **Neural-native architecture**: Embeddings and transformers are core to the system, not bolted on. The "next-link prediction" training paradigm is architecturally distinct.

3. **Specialized entity search**: Fine-tuned models for people search (1B+ profiles), company search, and code search -- with published open benchmarks for each.

4. **Exa Instant (Feb 2026)**: Sub-200ms neural search, claimed faster than Google search, enabling real-time agentic and voice workflows.

5. **Full-stack ownership**: Custom Rust vector database, 144 H200 GPU cluster, proprietary crawler, embedding models, and reranker -- all built in-house. Enables optimization across the entire stack.

6. **Research API**: Agentic search that autonomously performs multi-step research, reads pages, clusters, and summarizes with structured JSON output schemas.

7. **Websets**: A distinct product for entity sourcing and enrichment (people, companies) that goes beyond search into structured data collection.

## Raw Notes

- Exa was initially building a consumer search engine but pivoted to API-for-AI after ChatGPT launched, realizing companies/agents were the primary users of redesigned search.
- Peter Fenton (Benchmark) joined the board after Series B.
- The company has an OpenAPI spec published on GitHub for API consumers.
- Exa's MCP server is one of the most starred MCP servers in the ecosystem (~3,700 stars), suggesting strong adoption in the Claude/Cursor/AI-coding-assistant ecosystem.
- `exa-knowledge-mcp` is a separate MCP server that provides access to Exa's own API documentation and best practices within development environments.
- AWS Marketplace listing available for enterprise procurement.
- Exa blog posts on technical topics: BM25 optimization, scaling highlights server, Exacluster infrastructure, next-gen search architecture.

## Source Log

<!-- Complete list of all sources consulted for this profile -->

| # | Source | Type | URL | Accessed |
|---|--------|------|-----|----------|
| 1 | Exa homepage | official-docs | https://exa.ai | 2026-03-01 |
| 2 | Exa API docs - Search | api-reference | https://docs.exa.ai/reference/search | 2026-03-01 |
| 3 | Exa docs - How Search Works | official-docs | https://docs.exa.ai/reference/how-exa-search-works | 2026-03-01 |
| 4 | Exa docs - Rate Limits | official-docs | https://exa.ai/docs/reference/rate-limits | 2026-03-01 |
| 5 | Exa Pricing page | official-docs | https://exa.ai/pricing | 2026-03-01 |
| 6 | Exa llms.txt | official-docs | https://docs.exa.ai/llms.txt | 2026-03-01 |
| 7 | Exa vs Tavily comparison | official-docs | https://exa.ai/versus/tavily | 2026-03-01 |
| 8 | Exa blog - API 2.0 | blog-official | https://exa.ai/blog/exa-api-2-0 | 2026-03-01 |
| 9 | Exa blog - How to build next-gen search | blog-official | https://exa.ai/blog/how-to-build-nextgen-search | 2026-03-01 |
| 10 | Exa blog - BM25 optimization | blog-official | https://exa.ai/blog/bm25-optimization | 2026-03-01 |
| 11 | Exa blog - Exacluster | blog-official | https://exa.ai/blog/meet-the-exacluster | 2026-03-01 |
| 12 | Exa blog - Exa Instant | blog-official | https://exa.ai/blog/exa-instant | 2026-03-01 |
| 13 | Exa blog - Series B announcement | blog-official | https://exa.ai/blog/announcing-series-b | 2026-03-01 |
| 14 | Exa blog - People Search Benchmarks | blog-official | https://exa.ai/blog/people-search-benchmark | 2026-03-01 |
| 15 | Exa blog - Company Search Benchmarks | blog-official | https://exa.ai/blog/company-search-benchmarks | 2026-03-01 |
| 16 | Exa docs - LangChain integration | official-docs | https://docs.exa.ai/reference/langchain | 2026-03-01 |
| 17 | Exa docs - LlamaIndex integration | official-docs | https://docs.exa.ai/reference/llamaindex | 2026-03-01 |
| 18 | Exa docs - CrewAI integration | official-docs | https://docs.exa.ai/reference/crewai | 2026-03-01 |
| 19 | Exa docs - Vercel AI SDK integration | official-docs | https://docs.exa.ai/reference/vercel | 2026-03-01 |
| 20 | Exa docs - MCP | official-docs | https://docs.exa.ai/reference/exa-mcp | 2026-03-01 |
| 21 | Exa docs - Websets FAQ | official-docs | https://docs.exa.ai/websets/faq | 2026-03-01 |
| 22 | Exa About page | official-docs | https://exa.ai/about | 2026-03-01 |
| 23 | GitHub - exa-labs org | github-repo | https://github.com/exa-labs | 2026-03-01 |
| 24 | GitHub - exa-py | github-repo | https://github.com/exa-labs/exa-py | 2026-03-01 |
| 25 | GitHub - exa-js | github-repo | https://github.com/exa-labs/exa-js | 2026-03-01 |
| 26 | GitHub - exa-mcp-server | github-repo | https://github.com/exa-labs/exa-mcp-server | 2026-03-01 |
| 27 | GitHub - exa-labs/ai-sdk | github-repo | https://github.com/exa-labs/ai-sdk | 2026-03-01 |
| 28 | GitHub - exa OpenAPI spec | github-repo | https://raw.githubusercontent.com/exa-labs/openapi-spec/refs/heads/master/exa-openapi-spec.yaml | 2026-03-01 |
| 29 | PyPI - exa-py | api-reference | https://pypi.org/project/exa-py/ | 2026-03-01 |
| 30 | PyPI - langchain-exa | api-reference | https://pypi.org/project/langchain-exa/0.1.0/ | 2026-03-01 |
| 31 | PyPI - llama-index-tools-exa | api-reference | https://pypi.org/project/llama-index-tools-exa/ | 2026-03-01 |
| 32 | npm - exa-js | api-reference | https://www.npmjs.com/package/exa-js | 2026-03-01 |
| 33 | npm - @exalabs/ai-sdk | api-reference | https://www.npmjs.com/package/@exalabs/ai-sdk | 2026-03-01 |
| 34 | npm - @langchain/exa | api-reference | https://www.npmjs.com/package/@langchain/exa | 2026-03-01 |
| 35 | LangChain API ref - ExaSearchRetriever | api-reference | https://python.langchain.com/api_reference/exa/retrievers/langchain_exa.retrievers.ExaSearchRetriever.html | 2026-03-01 |
| 36 | LlamaIndex - Exa tools | api-reference | https://docs.llamaindex.ai/en/stable/api_reference/tools/exa/ | 2026-03-01 |
| 37 | Y Combinator - Exa profile | official-docs | https://www.ycombinator.com/companies/exa | 2026-03-01 |
| 38 | CB Insights - Metaphor/Exa | news | https://www.cbinsights.com/company/metaphor-1 | 2026-03-01 |
| 39 | Cerebral Valley - Exa profile | blog-third-party | https://cerebralvalley.ai/blog/exa-aims-to-reshape-web-search-4AOjbOWK9sxxXCKdjQG1EK | 2026-03-01 |
| 40 | SiliconANGLE - Series B coverage | news | https://siliconangle.com/2025/09/03/nvidia-backs-85m-round-ai-search-startup-exa/ | 2026-03-01 |
| 41 | TechFundingNews - Series B coverage | news | https://techfundingnews.com/san-franciscos-exa-raises-85m-at-700m-valuation-to-build-the-search-engine-for-ai/ | 2026-03-01 |
| 42 | MarkTechPost - Exa Instant coverage | news | https://www.marktechpost.com/2026/02/13/exa-ai-introduces-exa-instant-a-sub-200ms-neural-search-engine-designed-to-eliminate-bottlenecks-for-real-time-agentic-workflows/ | 2026-03-01 |
| 43 | Data4AI - Exa review | blog-third-party | https://data4ai.com/blog/vendor-spotlights/exa-ai-review/ | 2026-03-01 |
| 44 | Data4AI - Exa vs Tavily comparison | blog-third-party | https://data4ai.com/blog/tool-comparisons/exa-ai-vs-tavily/ | 2026-03-01 |
| 45 | Skywork - Exa ultimate guide | blog-third-party | https://skywork.ai/skypage/en/Exa-AI-The-Ultimate-Guide-for-Developers-AI-Builders/1972878623855276032 | 2026-03-01 |
| 46 | HumAI blog - AI search APIs compared | blog-third-party | https://www.humai.blog/ai-search-apis-compared-tavily-vs-exa-vs-perplexity/ | 2026-03-01 |
| 47 | DEV Community - SERP API comparison 2025 | blog-third-party | https://dev.to/ritza/best-serp-api-comparison-2025-serpapi-vs-exa-vs-tavily-vs-scrapingdog-vs-scrapingbee-2jci | 2026-03-01 |
| 48 | Firecrawl - Exa alternatives | blog-third-party | https://www.firecrawl.dev/blog/exa-alternatives | 2026-03-01 |
| 49 | 10Web - Exa review | blog-third-party | https://10web.io/ai-tools/exa/ | 2026-03-01 |
| 50 | FindMyAITool - Exa review 2026 | blog-third-party | https://findmyaitool.io/tool/exa/ | 2026-03-01 |
| 51 | Evolv Agency - Exa Explained | blog-third-party | https://evolvagency.io/learn/generative-search/what-is-exa | 2026-03-01 |
| 52 | Hacker News - Exa pricing discussion | community | https://news.ycombinator.com/item?id=43910228 | 2026-03-01 |
| 53 | LangChain GitHub - Exa tools source | github-repo | https://github.com/langchain-ai/langchain/blob/master/libs/partners/exa/langchain_exa/tools.py | 2026-03-01 |
| 54 | LlamaIndex GitHub - Exa tools | github-repo | https://github.com/run-llama/llama_index/tree/main/llama-index-integrations/tools/llama-index-tools-exa | 2026-03-01 |
| 55 | AWS Marketplace - Exa | official-docs | https://aws.amazon.com/marketplace/pp/prodview-d5kxpjxyrsdkw | 2026-03-01 |

<!--
Source Types:
  official-docs  - Official documentation / API reference
  api-reference  - Detailed API/SDK documentation
  github-repo    - GitHub repository README, code, issues
  blog-official  - Official blog post from the tool's org
  blog-third-party - Third-party blog, comparison article
  benchmark      - Independent benchmark or test results
  news           - News article or press release
  community      - Forum post, Discord, Stack Overflow
  inference      - Derived/inferred from other data (flag for verification)
-->
