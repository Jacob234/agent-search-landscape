# Tavily

<!--
COLLECTION METADATA
==================
Collected: 2026-03-01
Collector: claude-opus-4-6
Session: profile/search-apis
GitHub Issue: #2
Collection Method: see docs/methodology.md
Last Verified: 2026-03-01
Confidence: high — Initial profile from training data, then comprehensively updated with live web verification on 2026-03-01. Pricing, endpoints, credit costs, GitHub stars, and parameters all verified against official docs and pricing page.
-->

## Identity & Basics

| Field | Value | Source |
|-------|-------|--------|
| Organization | Tavily AI, Inc. | <!-- https://tavily.com --> |
| Website | https://tavily.com | |
| License | Proprietary (API service); Python SDK is MIT-licensed, Node SDK is ISC-licensed | <!-- https://github.com/tavily-ai/tavily-python/blob/main/LICENSE, https://github.com/tavily-ai/tavily-node --> |
| Launch Date | ~mid-2023 (exact date not publicly documented) | <!-- https://tavily.com --> |
| GitHub (Python SDK) | https://github.com/tavily-ai/tavily-python | |
| GitHub (Node SDK) | https://github.com/tavily-ai/tavily-node | |
| GitHub Stars (Python SDK) | ~1,000 (as of 2026-03-01) | <!-- https://github.com/tavily-ai/tavily-python --> |
| Community Size | Active Discord community; exact member count not publicly listed | <!-- Discord invite not checked --> |

## Category Placement

- **Primary**: AI-Native Search API (Search-for-Agents)
- **Secondary**: RAG data source, LLM tool-use integration

## Core Capability

> Tavily is a search API purpose-built for AI agents and LLM applications, returning structured, pre-processed search results optimized for retrieval-augmented generation (RAG) pipelines.

## API Surface

<!-- Sources: https://docs.tavily.com/documentation/api-reference (verified via web search 2026-03-01); https://www.tavily.com/blog/what-tavily-shipped-in-january-26 -->

### Endpoints / Methods

| Endpoint | Purpose | Input | Output |
|----------|---------|-------|--------|
| `POST /search` | Core search — returns relevant results for a query | JSON: `query` (required), `search_depth`, `topic`, `include_domains`, `exclude_domains`, `include_answer`, `include_raw_content`, `include_images`, `max_results`, `days` | JSON: `results[]` with `title`, `url`, `content` (extracted text), `score` (relevance), `raw_content` (optional full text); optionally `answer` (AI-generated summary), `images[]` |
| `POST /extract` | Content extraction — extracts clean content from specific URLs | JSON: `urls` (required, list of URLs to extract from) | JSON: `results[]` with `url`, `raw_content` (full extracted text). Supports markdown or text output, optional images, advanced depth for tables/embedded data |
| `POST /map` | Site mapping — generates structured URL map of a site | JSON: URL + depth/breadth/limits, optional instructions | JSON: graph of URLs discovered via traversal |
| `POST /crawl` | Map + extract combined — site-scale content extraction | JSON: URL + crawl config | JSON: map of site + extracted content from discovered pages |
| `POST /research` | Multi-search research — synthesis into a report with sources | JSON: research query + config | JSON/SSE: synthesized report with citations. Supports streaming via SSE. [Launched Jan 2026](https://www.tavily.com/blog/what-tavily-shipped-in-january-26) |
| `GET /usage` | Account usage — programmatic visibility into usage/limits | API key auth | JSON: key, project ID, account usage details, per-endpoint breakdown. [Launched Jan 2026](https://www.tavily.com/blog/what-tavily-shipped-in-january-26) |

**Note**: The API has expanded significantly since launch. Originally only `/search`, it now includes 6 endpoints. `/map`, `/crawl`, and `/research` represent Tavily's expansion from pure search into site-scale extraction and autonomous research — directly competing with Firecrawl.

### Authentication

- **Method**: API key passed as `api_key` field in the JSON request body (REST API) or via `Authorization: Bearer <key>` header
- **Key management**: Generated at https://app.tavily.com
- **SDK authentication**: Pass API key to client constructor (`TavilyClient(api_key="...")` in Python; `tavily.Client({apiKey: "..."})` in Node)

### Rate Limits

| Tier | Rate Limit | Notes |
|------|-----------|-------|
| Free (Researcher) | 1,000 credits/month | Lower priority |
| Project | 4,000 credits/month + higher rate limits | Per pricing page |
| Enterprise | Custom rate limits | Per agreement |
| Per-second | Not publicly documented | API returns HTTP 429 on excessive requests, 432 on plan limits, 433 on PAYG limits |

### Input/Output Formats

- **Input**: JSON POST body (REST); native objects via SDKs
- **Output**: JSON response
- **Search depth options**: `basic` (faster, cheaper) and `advanced` (more thorough, uses more credits)
- **Topic filtering**: `general` (default) or `news` — switches the underlying search behavior
- **Domain filtering**: `include_domains` and `exclude_domains` arrays for scoping results
- **Time filtering**: `days` parameter to restrict results to recent content (e.g., last 7 days)
- **Max results**: Up to 20 results per query (configurable via `max_results`, default 5)

### Key Parameters Detail (POST /search)

<!-- Sources: https://docs.tavily.com/documentation/api-reference/endpoint/search (verified 2026-03-01) -->

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Search query (recommended max ~400 chars for best results) |
| `search_depth` | string | No | `"basic"` (default, 1 credit), `"advanced"` (2 credits), `"fast"`, `"ultra-fast"` (1 credit each) |
| `topic` | string | No | `"general"` (default) or `"news"` |
| `time_range` | string | No | `"day"`, `"week"`, `"month"`, `"year"` (or `"d"`, `"w"`, `"m"`, `"y"`) |
| `start_date` / `end_date` | string | No | Date range in `YYYY-MM-DD` format |
| `max_results` | integer | No | Max results to return, 0-20 (default 5) |
| `chunks_per_source` | integer | No | 1-3 chunks per source (default 3, advanced only) |
| `include_images` | boolean | No | Include image results |
| `include_image_descriptions` | boolean | No | Include AI-generated image descriptions |
| `include_favicon` | boolean | No | Include site favicon URLs |
| `include_answer` | boolean/string | No | `true`, `"basic"`, or `"advanced"` — AI-generated answer summary |
| `include_raw_content` | boolean/string | No | `true`, `"markdown"`, or `"text"` — full page content |
| `include_domains` | string[] | No | Only include results from these domains (max 300) |
| `exclude_domains` | string[] | No | Exclude results from these domains (max 150) |
| `country` | string | No | Boost results from a specific country |
| `auto_parameters` | boolean | No | Automatically optimize parameters (costs 2 credits) |
| `exact_match` | boolean | No | Require exact match of query terms |
| `include_usage` | boolean | No | Include credit usage info in response |

## Pricing

<!-- Sources: https://tavily.com/pricing, https://docs.tavily.com/documentation/api-credits (both verified 2026-03-01) -->
<!-- Pricing as of: 2026-03-01 -->

| Tier | Price | Credits/Month | Per-Credit Cost | Notes |
|------|-------|---------------|-----------------|-------|
| Researcher (Free) | $0/month | 1,000 | $0 | No credit card required, email support |
| Student | $0/month | Complimentary | $0 | For students pursuing knowledge |
| Pay As You Go | $0.008/credit | On-demand | $0.008 | Cancel anytime, email support |
| Project | $30/month | 4,000 | $0.0075 | Higher rate limits, email support |
| Enterprise | Custom | Custom | Custom | SLAs, enterprise security & privacy |

[Source: tavily.com/pricing](https://tavily.com/pricing)

### Credit Cost per Endpoint

| Endpoint | Basic/Standard | Advanced | Notes |
|----------|---------------|----------|-------|
| `/search` | 1 credit | 2 credits | `fast` and `ultra-fast` depths also 1 credit. `auto_parameters` costs 2 credits. |
| `/extract` | 1 credit per 5 URLs | 2 credits per 5 URLs | Failed extractions not charged |
| `/map` | 1 credit per 10 pages | 2 credits per 10 pages (with instructions) | Failed requests not charged |
| `/crawl` | Mapping cost + extraction cost | Combined | E.g., 10 pages basic = 3 credits (1 map + 2 extract) |
| `/research` (Pro) | 15 credits min | 250 credits max | Dynamic pricing within range |
| `/research` (Mini) | 4 credits min | 110 credits max | Dynamic pricing within range |

[Source: docs.tavily.com/documentation/api-credits](https://docs.tavily.com/documentation/api-credits)

### Per-Unit Economics at Scale

| Volume | Approximate Cost per 1K Basic Searches | Notes |
|--------|---------------------------------------|-------|
| Free tier | $0 (up to 1K/month) | Researcher plan |
| PAYG | $8.00 per 1K queries | $0.008/credit × 1 credit |
| Project ($30/mo) | $7.50 per 1K queries | $0.0075/credit effective rate |
| Enterprise | Custom | Volume discounts likely |

## Technical Architecture

<!-- Sources: training data, https://tavily.com, project landscape doc, various third-party comparisons cited in agent-search-landscape.md -->

### How It Works

1. **Query Processing**: Tavily receives a search query and processes it to optimize for the requested topic type (general vs news) and search depth
2. **Multi-Source Search**: Tavily does NOT maintain its own web index. It aggregates results from multiple underlying search providers/engines (the exact combination is proprietary and not publicly documented)
3. **Content Extraction**: For each result, Tavily fetches and extracts the page content, converting it to clean text suitable for LLM consumption
4. **AI Processing**: Results are ranked by relevance. When `include_answer` is true, Tavily uses an LLM to generate a synthesized answer from the retrieved content
5. **Structured Output**: Returns JSON with titles, URLs, relevance scores, extracted content snippets, and optionally full page content and AI-generated answers

**Key architectural decisions:**
- No proprietary index — acts as a meta-search + extraction + AI processing layer
- Optimized for LLM consumption — content is pre-cleaned and structured
- Multi-vertical — switches search behavior based on `topic` parameter (news uses news-specific sources)
- The "advanced" search depth appears to do deeper crawling/extraction, potentially fetching more pages and doing more thorough content analysis

### Models / Infrastructure

- **Search backend**: Aggregates from multiple search providers (specific providers not publicly disclosed; likely includes Bing/Google and news aggregators)
- **AI models**: Uses LLMs for answer generation (specific model not disclosed; likely a proprietary or fine-tuned model)
- **Extraction**: Proprietary content extraction pipeline that handles JavaScript-rendered pages and various page structures
- **Infrastructure**: Cloud-hosted API service; exact cloud provider not publicly disclosed

### Self-Host Options

- **No self-host option**: Tavily is a fully managed cloud API service. There is no self-hosted or on-premise deployment option
- The Python and Node SDKs are open-source client libraries that call the hosted API; they do not contain the search/extraction logic

## Integration Surface

<!-- Sources: GitHub repos (tavily-ai org), PyPI, npm, framework documentation, project landscape doc -->

| Integration | Status | Notes |
|------------|--------|-------|
| MCP Server | Available | Official Tavily MCP server exists; also included in mcp-omnisearch aggregator. See https://github.com/tavily-ai/tavily-mcp |
| Python SDK | Available | `pip install tavily-python` — package `tavily-python` on PyPI. Provides `TavilyClient` class with `search()` and `extract()` methods. Async support included. |
| Node/TS SDK | Available | `npm install @tavily/core` (v0.7.2). Also `@tavily/ai-sdk` for Vercel AI SDK integration. [GitHub: tavily-ai/tavily-js](https://github.com/tavily-ai/tavily-js) |
| LangChain | Available (first-class) | `TavilySearchResults` tool built into LangChain. `from langchain_community.tools.tavily_search import TavilySearchResults`. Tavily is the default/recommended search tool in LangChain documentation. |
| LlamaIndex | Available | Integration exists via LlamaIndex tools/data connectors. `TavilyToolSpec` in `llama-index-tools-tavily`. |
| CrewAI | Available | Supported as a tool in CrewAI agent definitions. |
| OpenAI Assistants | Available | Can be used as a function tool in OpenAI Assistants/function calling |
| Autogen | Available | Supported in Microsoft AutoGen framework |
| Haystack | Available | Integration available for deepset Haystack |

**Key ecosystem position**: Tavily's deepest integration is with **LangChain**, where it has been the recommended/default search tool since early in LangChain's development. This is a significant distribution advantage — many LangChain tutorials and examples use Tavily by default. [Source: project landscape doc, training data]

## Performance Data

<!-- Sources: project landscape doc (agent-search-landscape.md), third-party comparisons cited therein -->

### Published Benchmarks

| Metric | Tavily | Comparison | Source |
|--------|--------|------------|--------|
| Complex retrieval accuracy | 71% | Exa: 81% | [Exa vs Tavily comparison](https://exa.ai/versus/tavily) (cited in project landscape doc) |
| Speed (relative) | Baseline | Exa: 2-3x faster | Project landscape doc |

**Note**: The 71% complex retrieval accuracy figure comes from Exa's comparison page — this is a competitor-published benchmark. Independent verification recommended.

### Speed Claims

- Tavily aims for low-latency responses suitable for real-time agent use
- `basic` search depth is faster than `advanced`
- Typical response times: ~1-3 seconds for basic, ~3-8 seconds for advanced (approximate, from community reports — NEEDS VERIFICATION)

### Accuracy / Coverage Claims

- Tavily positions itself as optimized for relevance and accuracy in the context of RAG/agent use cases
- The `advanced` search depth is claimed to provide more thorough and accurate results at the cost of speed and credits
- No official accuracy benchmark published by Tavily (the 71% figure is from Exa's competing comparison)

## Limitations

<!-- Sources: training data, project landscape doc, general knowledge of the API -->

### What It Can't Do

- **No proprietary index**: Cannot search content that underlying search providers don't index
- **No interactive browsing**: Cannot click through pages, fill forms, or handle JavaScript-heavy interactive sites beyond initial page rendering for extraction
- **No structured data extraction with schemas**: Unlike Firecrawl's `/extract`, Tavily does not support user-defined output schemas for structured data extraction
- **No historical/cached access**: Cannot access pages that are currently down or retrieve historical versions
- **Limited multimedia**: Primarily text-focused; image support is secondary and limited
- **Crawl is invite-only**: The `/crawl` endpoint is currently available on an invite-only basis (as of 2026-03-01) [Source: GitHub README](https://github.com/tavily-ai/tavily-python)
- **No self-host**: Fully managed cloud service only

### Known Failure Modes

- **Paywalled content**: Results may include URLs to paywalled articles where content extraction is incomplete or unavailable
- **Dynamic/SPA content**: Some JavaScript-heavy single-page applications may not have their content fully extracted
- **Recency lag**: Newly published content may not appear immediately (depends on underlying search providers' indexing speed)
- **Domain bias**: Results may over-represent certain domains depending on the underlying search providers' ranking
- **Answer quality variance**: The `include_answer` feature quality varies significantly with query complexity and topic

### Documented Issues

- Historical issues have included: rate limiting inconsistencies, occasional timeout errors on advanced searches, content extraction failures on certain page types
- GitHub issues not reviewed in this collection session — check https://github.com/tavily-ai/tavily-python/issues for current state

## Differentiation

> Tavily's primary differentiator is its **LangChain-first ecosystem position** — it became the default search tool in the most popular LLM framework early on, giving it massive distribution advantage. Technically, it offers a clean **"search + extract + optional AI answer"** pipeline in a single API call, purpose-built for agent/RAG consumption rather than adapted from traditional web search APIs.

**vs Exa**: Exa uses semantic/neural search (finds conceptually related content); Tavily uses traditional keyword-augmented search with AI processing. Exa reportedly has better accuracy (81% vs 71% on complex retrieval) and is 2-3x faster, but Tavily has broader ecosystem integration. [Source: project landscape doc]

**vs SerpAPI**: SerpAPI is a SERP scraping tool (structured JSON from Google/Bing results pages); Tavily goes further by extracting actual page content and optionally generating AI answers. SerpAPI gives you what Google shows; Tavily gives you the content behind the results.

**vs Brave Search API**: Brave has its own independent index (privacy-focused, no query logging); Tavily aggregates from multiple sources. Brave is a raw search API; Tavily adds the content extraction and AI processing layers.

**vs Perplexity Sonar**: Perplexity Sonar returns fully synthesized LLM answers with citations; Tavily returns structured search results that your own LLM processes. Sonar is "outsource the reasoning"; Tavily is "give my agent good data to reason with."

## Raw Notes

### Distribution Advantage
Tavily's position as the default search tool in LangChain documentation and tutorials is arguably its strongest asset. Developers building with LangChain encounter Tavily first, creating a powerful default-choice network effect. This is more of a go-to-market advantage than a technical one.

### Credit System Complexity
The credit-based pricing model (where advanced searches cost more credits) adds complexity compared to simple per-query pricing. This makes cost estimation harder for users, especially when mixing basic and advanced searches.

### API Evolution (Verified 2026-03-01)
Tavily has expanded dramatically since its initial `/search`-only API. Now 6 endpoints: search, extract, map, crawl, research, usage. The `/research` endpoint (launched Jan 2026) positions Tavily in the "deep research" category alongside Perplexity Deep Research and OpenAI Deep Research. The `/crawl` endpoint is still invite-only. New search depths (`fast`, `ultra-fast`) and parameters (`time_range`, `country`, `auto_parameters`, `exact_match`) added.

### Competitive Pressure
The AI search API space is intensifying. Exa offers better benchmarks on accuracy, Perplexity Sonar offers synthesized answers, and the emergence of built-in LLM search (Claude Web Search, ChatGPT Browse) reduces the need for standalone search APIs in some use cases.

### Remaining Gaps
The following items still need verification:
1. **Discord community size** — exact member count unknown
2. **Exact launch date** — approximate "mid-2023" but no precise date found
3. **Per-second rate limits** — not publicly documented (only HTTP error codes known)
4. **Current open GitHub issues** — not checked in this session

## Source Log

<!-- Complete list of all sources consulted for this profile -->

| # | Source | Type | URL | Accessed |
|---|--------|------|-----|----------|
| 1 | Project landscape overview | inference | /agent-search-landscape.md (local project file) | 2026-03-01 |
| 2 | Tavily official website | official-docs | https://tavily.com | 2026-03-01 (via web search) |
| 3 | Tavily API documentation | api-reference | https://docs.tavily.com/documentation/api-reference | 2026-03-01 |
| 4 | Tavily Search endpoint docs | api-reference | https://docs.tavily.com/documentation/api-reference/endpoint/search | 2026-03-01 |
| 5 | Tavily API credits docs | api-reference | https://docs.tavily.com/documentation/api-credits | 2026-03-01 |
| 6 | Tavily Python SDK GitHub | github-repo | https://github.com/tavily-ai/tavily-python | 2026-03-01 |
| 7 | Tavily JS SDK GitHub | github-repo | https://github.com/tavily-ai/tavily-js | 2026-03-01 (via web search) |
| 8 | Tavily pricing page | official-docs | https://tavily.com/pricing | 2026-03-01 |
| 9 | Tavily Jan 2026 blog post | blog-official | https://www.tavily.com/blog/what-tavily-shipped-in-january-26 | 2026-03-01 (via web search) |
| 10 | npm @tavily/core | api-reference | https://www.npmjs.com/package/@tavily/core | 2026-03-01 (via web search) |
| 11 | Exa vs Tavily comparison | blog-third-party | https://exa.ai/versus/tavily | Referenced via project landscape doc |
| 12 | Firecrawl vs Tavily comparison | blog-third-party | https://blog.apify.com/firecrawl-vs-tavily/ | Referenced via project landscape doc |
| 13 | Beyond Tavily - AI Search APIs | blog-third-party | https://websearchapi.ai/blog/tavily-alternatives | Referenced via project landscape doc |
| 14 | Tavily vs Exa vs Perplexity comparison | blog-third-party | https://www.humai.blog/tavily-vs-exa-vs-perplexity-vs-you-com-the-complete-ai-search-api-comparison-2025/ | Referenced via project landscape doc |
| 15 | Firecrawl Tavily pricing breakdown | blog-third-party | https://www.firecrawl.dev/blog/tavily-pricing | 2026-03-01 (via web search) |
| 16 | Claude training data | inference | Training data through May 2025 | 2026-03-01 |

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
