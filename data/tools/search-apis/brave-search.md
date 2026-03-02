# Brave Search API

<!--
COLLECTION METADATA
==================
Collected: 2026-03-01
Collector: claude-opus-4-6
Session: agent-search-landscape data collection
GitHub Issue: N/A
Collection Method: WebSearch with multi-source cross-referencing
Last Verified: 2026-03-01
Confidence: high — multiple official sources cross-referenced, recent pricing change verified across sources
-->

## Identity & Basics

| Field | Value | Source |
|-------|-------|--------|
| Organization | Brave Software, Inc. (founded 2015 by Brendan Eich & Brian Bondy) | <!-- https://en.wikipedia.org/wiki/Brave_(web_browser) --> |
| Website | https://brave.com/search/api/ | <!-- https://brave.com/search/api/ --> |
| License | Proprietary commercial API; terms at api-dashboard.search.brave.com/terms-of-service. Limited, non-exclusive, non-transferable license. Storage/caching of results prohibited without "Data w/ Storage Rights" plan. AI inference rights included only on Search plan ($5/1k tier). | <!-- https://api-dashboard.search.brave.com/terms-of-service --> |
| Launch Date (Search API) | May 2023 (API specifically). Brave Search itself launched June 2021 (beta), fully released June 2022. | <!-- https://brave.com/blog/search-api-launch/, https://en.wikipedia.org/wiki/Brave_Search --> |
| GitHub (Official MCP Server) | https://github.com/brave/brave-search-mcp-server | <!-- https://github.com/brave/brave-search-mcp-server --> |
| GitHub Stars (MCP Server) | ~663 stars | <!-- as of 2026-03-01 --> |
| Community Size | 100M+ Brave browser users worldwide; 32M monthly active Brave Search users; 200K+ developers signed up for the API (as cited in growth blog); 8B+ annualized searches. | <!-- https://brave.com/blog/search-api-growth/ --> |

## Category Placement

- **Primary**: Web Search API
- **Secondary**: AI Grounding / LLM Context API, Summarization API, Local/Place Search API

## Core Capability

> A commercial REST API providing web, image, news, video, local/place, and LLM-context search results from Brave's independently-crawled index of 40+ billion pages, with privacy-first design (no query logging, optional Zero Data Retention) and AI summarization capabilities.

## API Surface

<!-- Sources: https://api-dashboard.search.brave.com/app/documentation/web-search/get-started, https://brave.com/blog/most-powerful-search-api-for-ai/, https://brave.com/blog/place-search-api/, https://api-dashboard.search.brave.com/documentation/services/spellcheck, https://api-dashboard.search.brave.com/documentation/services/summarizer, https://api-dashboard.search.brave.com/documentation/services/llm-context -->

### Endpoints / Methods

| Endpoint | Purpose | Input | Output |
|----------|---------|-------|--------|
| `GET /res/v1/web/search` | Web search — returns ranked web results with snippets, FAQ, infoboxes | Query string `q`, optional params: `country`, `search_lang`, `count`, `offset`, `safesearch`, `freshness`, `goggles_id`, `units`, `extra_snippets` | JSON: `web.results[]` (url, title, description, age, extra_snippets), plus `mixed`, `query`, `discussions`, `faq`, `infobox`, `locations`, `news`, `videos` |
| `GET /res/v1/llm/context` | LLM Context — pre-extracted page content optimized for RAG/LLM grounding | Query string `q`, similar params to web search | JSON: smart chunks with clean text extraction, markdown conversion, JSON-LD schemas, tables with row-level granularity |
| `GET /res/v1/images/search` | Image search | Query string `q`, optional `count`, `safesearch`, `country`, `search_lang` | JSON: image results with thumbnail URLs, source page, dimensions |
| `GET /res/v1/news/search` | News search | Query string `q`, optional `count`, `freshness`, `country` | JSON: news results with title, URL, source, age, thumbnail |
| `GET /res/v1/videos/search` | Video search | Query string `q`, optional `count`, `country`, `safesearch` | JSON: video results with title, URL, thumbnail, source, duration |
| `GET /res/v1/places/search` | Place/Local search (announced Feb 2026) — 200M+ POI index | Query string `q` (optional), `lat`/`lng` center point, `radius` (up to 20km / 13 miles) | JSON: places with name, address, coordinates, category, ratings |
| `GET /res/v1/summarizer/search` | Summarizer search — returns summary key if query is eligible | Uses key from web search response `summarizer.key` | JSON: summarizer metadata and key |
| `GET /res/v1/summarizer/summary` | Summarizer summary — AI-generated summary from search results | Summarizer key from prior step | JSON: AI summary text with citations |
| `GET /res/v1/summarizer/summary_streaming` | Streaming summarizer — same as above but SSE streaming | Summarizer key | SSE stream of summary chunks |
| `GET /res/v1/spellcheck/search` | Spell checking for queries | Query string `q`, `country` | JSON: corrected query with `altered` flag |
| `GET /res/v1/suggest/search` | Autocomplete/suggest — query completions as user types | Query string `q`, optional `count`, `country`, `rich` | JSON: array of suggested completions |

Note: The Summarizer operates as a two-step workflow. First call web search, which returns a `summarizer.key` if the query is eligible. Then use that key to call the summarizer endpoint. Summarizer requests are not billed separately — only the initial web search request counts.

### Authentication

- **Method**: API key passed via HTTP header `X-Subscription-Token`
- **Key management**: Via the Brave Search API dashboard at api-dashboard.search.brave.com
- **Separate keys per plan**: Each subscription (Search, Answers, Spellcheck, Autosuggest) requires its own API key
- **Example**: `curl -H "X-Subscription-Token: YOUR_API_KEY" "https://api.search.brave.com/res/v1/web/search?q=query"`

### Rate Limits

Per-plan rate limits (as of Feb 2026 restructuring):

| Plan | Burst Limit (req/sec) | Monthly Quota | Notes |
|------|-----------------------|---------------|-------|
| Free (legacy, grandfathered) | 1 req/sec | 2,000 queries/month | No longer available for new signups |
| Search ($5/1k) | Up to 100 req/sec (plan-dependent) | Pay-per-use after $5 credit | $5 free credit/month = ~1,000 queries |
| Answers ($4/1k + $5/M tokens) | Plan-dependent | Pay-per-use after $5 credit | Same $5 monthly credit |
| Spellcheck ($5/10k) | Plan-dependent | Pay-per-use after $5 credit | |
| Autosuggest ($5/10k) | Plan-dependent | Pay-per-use after $5 credit | |
| Enterprise | Custom (up to 100 req/sec) | Custom | Contact sales |

Rate limit response headers:
- `X-RateLimit-Limit` — max requests allowed
- `X-RateLimit-Remaining` — requests remaining in window
- `X-RateLimit-Reset` — time when limit resets
- `X-RateLimit-Policy` — rate limit policy details

Only successful (non-error) responses are counted against quota and billed.

### Input/Output Formats

- **Input**: HTTP GET with URL query parameters. Query string `q` is required for all search endpoints.
- **Output**: JSON responses. Key fields include `type`, `query`, `web.results[]`, `news.results[]`, `videos.results[]`, `locations.results[]`, `mixed.main[]` (for blended result ordering).
- **Result fields** (web): `url`, `title`, `description`, `page_age`, `age`, `extra_snippets[]` (up to 5 per result), `deep_results`, `thumbnail`
- **Summarizer output**: AI-generated text with source citations
- **LLM Context output**: Pre-extracted page content as structured smart chunks with markdown, JSON-LD, tables

## Pricing

<!-- Sources: https://brave.com/blog/most-powerful-search-api-for-ai/, https://www.implicator.ai/brave-drops-free-search-api-tier-puts-all-developers-on-metered-billing/, https://alternativeto.net/news/2026/2/brave-updates-its-search-api-with-a-new-llm-context-endpoint-for-ai-and-new-pricing/, https://api-dashboard.search.brave.com/app/plans -->
<!-- Pricing as of: 2026-02-13 (Feb 2026 restructuring) -->

| Tier | Price | Includes |
|------|-------|----------|
| Search | $5 per 1,000 requests | Web search, LLM Context, Images, News, Videos, Place Search. AI inference rights included. $5 free credit renews monthly (~1,000 free queries). |
| Answers | $4 per 1,000 web searches + $5 per 1M tokens (input+output) | AI-researched answers grounded in web results. $5 free credit renews monthly. |
| Spellcheck | $5 per 10,000 requests | Spelling correction for queries. $5 free credit renews monthly (~10,000 free checks). |
| Autosuggest | $5 per 10,000 requests | Query autocompletion. $5 free credit renews monthly (~10,000 free suggestions). |
| Enterprise | Custom pricing | Custom rate limits (up to 100 req/sec), Zero Data Retention (ZDR) option, SOC 2 Type II compliant, dedicated support. |
| Free (legacy, grandfathered) | $0 | 2,000 queries/month, 1 req/sec. No longer available for new signups as of Feb 2026. |

**Historical context**: From May 2023 to Feb 2026, a free tier offered 2,000 queries/month. In Aug 2025, a "Free AI" tier briefly offered 5,000 queries/month. As of Feb 2026, all plans are metered with $5 monthly credit.

### Per-Unit Economics at Scale

| Volume | Cost per Query | Notes |
|--------|---------------|-------|
| Up to ~1,000/month | $0.00 (covered by $5 free credit) | Search plan |
| 1,001 - 100,000/month | $0.005 per query | $5 per 1,000 requests |
| 100,000+/month | $0.005 per query (volume discounts via Enterprise) | Contact sales for custom pricing |

- Summarizer requests are free (only the initial web search is billed)
- Spellcheck/Autosuggest are 10x cheaper at $0.0005 per request
- Answers plan: $0.004 per search + $0.005 per 1M tokens for AI generation
- AWS Marketplace: Also available via AWS Marketplace for consolidated billing

## Technical Architecture

<!-- Sources: https://brave.com/blog/search-independence/, https://search.brave.com/help/independence, https://support.brave.app/hc/en-us/articles/4409406835469-What-is-the-Web-Discovery-Project, https://brave.com/blog/ai-summarizer/, https://brave.com/blog/search-api-growth/ -->

### How It Works

**Independent Web Index**: Brave Search operates its own independently crawled web index, unlike most search API alternatives that depend on Google or Bing. As of 2026, the index contains **40+ billion pages** and is refreshed with **100+ million page updates per day**.

**Crawling**: Brave uses a traditional web crawler plus the opt-in Web Discovery Project (WDP) to build and maintain its index.

**Web Discovery Project (WDP)**:
- Opt-in feature in the Brave browser
- Consenting users anonymously contribute: visited URLs, search queries, clicks, time-on-page, and page metadata
- Privacy safeguards: code is open-source for auditing; queries containing emails, phone numbers, or PII are automatically discarded
- Purpose: guides crawler prioritization, surfaces long-tail content that traditional crawlers might miss
- Users cannot be identified — cryptographic protocols prevent linking contributions to individuals

**Independence timeline**:
- June 2021: Launch with ~87% independent results (13% from Bing fallback)
- Within one year: rose to 93% independent
- April 2023: achieved **100% independence** — all Bing API calls removed
- August 2023: Image and video search also cut ties with Bing

**How this differs from Google/Bing-dependent APIs**: Most alternative search APIs (SerpApi, Serper, etc.) scrape or proxy Google/Bing results. Brave controls the entire stack — crawling, indexing, ranking, and serving. This means: (a) no dependency on third-party index availability (critical after Bing API shutdown Aug 2025), (b) ability to enforce Zero Data Retention since queries never leave Brave's infrastructure, (c) independent ranking algorithms not subject to Google/Bing changes.

### Models / Infrastructure

**Ranking**: Machine learning models for result ranking with advanced relevance scoring. Brave has stated they use "advanced machine learning models" for search relevance, with ongoing improvements to natural language query support.

**Summarizer AI**: Composed of **three different LLMs** trained on different tasks:
1. **QA (Question Answering)** — extracts concrete answers from text snippets
2. **Summarization** — synthesizes information from multiple web sources
3. **Language generation** — produces coherent, natural-language output

The Summarizer was fully developed by Brave's Search team, trained to process multiple web sources rather than generating from parametric knowledge alone. Unlike purely generative models, it grounds all output in retrieved web content.

**LLM Context API**: Real-time content extraction that produces "smart chunks" — clean text with markdown conversion, preserved JSON-LD schemas, and tables with row-level granularity. Designed specifically for RAG pipelines and LLM grounding.

**SOC 2 Type II**: Achieved in October 2025 via independent audit by Prescient Security.

**Goggles** (custom re-ranking):
- Domain-specific language for creating custom result ranking rules
- Actions: boost, downrank, or filter results based on URL patterns, domains, and criteria
- Can be provided as: hosted Goggles URL (GitHub/GitLab/Gist), inline specification in request, or mixed
- Work with both Web Search and News Search endpoints
- Community-created Goggles can be published and shared

**Rerank** (announced January 2025):
- Consumer-facing feature built on Goggles technology
- Users upvote/downvote domains to personalize rankings
- Changes are per-user, per-device only — no impact on other users
- Also affects "Answer with AI" feature sourcing

### Self-Host Options

Not available. Brave Search API is a cloud-hosted service only. The index, ranking infrastructure, and Summarizer models are all proprietary and run on Brave's infrastructure.

Available via:
- Direct API (api.search.brave.com)
- AWS Marketplace (since July 2025)
- Amazon Bedrock AgentCore container
- Snowflake integration (public preview, announced at BUILD London)

## Integration Surface

<!-- Sources: https://brave.com/search/api/tools/, https://github.com/brave/brave-search-mcp-server, https://python.langchain.com/docs/integrations/tools/brave_search/, https://docs.llamaindex.ai/en/stable/api_reference/tools/brave_search/, https://www.npmjs.com/package/brave-search -->

| Integration | Status | Notes |
|------------|--------|-------|
| MCP Server (Official) | Available | https://github.com/brave/brave-search-mcp-server — ~663 stars. Part of official modelcontextprotocol/servers repository. Provides web search, local search, image, video, news, and summarization tools. Endorsed by Anthropic. |
| Python SDK (community) | Available | `brave-search-python-client` — https://pypi.org/project/brave-search-python-client/ — supports Web, Image, News, Video. ~12 GitHub stars. Not official. Install: `pip install brave-search-python-client` |
| Node/TS SDK (community) | Available | `brave-search` — https://www.npmjs.com/package/brave-search — fully typed Brave Search API wrapper. Third-party, not official. |
| LangChain | Available (built-in) | `BraveSearchWrapper` in `langchain_community.utilities.brave_search`. Also `BraveSearch` tool in `langchain_community.tools.brave_search.tool`. Env var: `BRAVE_SEARCH_API_KEY`. Methods: `run()` (JSON string), `download_documents()` (list of Documents). |
| LlamaIndex | Available | `brave_search` tool in LlamaIndex. Documented at docs.llamaindex.ai. Returns list of search results. |
| CrewAI | Indirect | No dedicated Brave Search tool in CrewAI, but works through MCP server integration or LangChain tools (CrewAI supports LangChain tools). |
| Anthropic Claude | Primary provider | Brave powers Claude's web search feature. MCP integration highlighted in Claude Desktop and Claude Code demos. |
| Snowflake | Available (preview) | Integration for Cortex Code and Cortex Agents — agentic web search for enterprise. |
| AWS Marketplace | Available | Direct subscription and billing via AWS account. Also available as Bedrock AgentCore container. |
| Go Client (community) | Available | `github.com/cnosuke/go-brave-search` |
| Pipedream | Available | Pre-built integration for workflow automation. |

**No official SDK**: Brave does not publish an official Python or Node SDK. The API is designed as a simple REST API with HTTP GET requests. Community wrappers exist but are third-party.

## Performance Data

<!-- Sources: https://brave.com/blog/search-api-growth/, https://brave.com/blog/most-powerful-search-api-for-ai/, https://brave.com/blog/soc2/, https://aimultiple.com/agentic-search -->

### Published Benchmarks

| Metric | Value | Source | Notes |
|--------|-------|--------|-------|
| SimpleQA F1-score (multi-search + reasoning) | 94.1% | Brave blog (search-api-growth) | Measures factual accuracy and answer completeness |
| SimpleQA F1-score (single search) | 92.1% | Brave blog (search-api-growth) | Single-query factual accuracy |

### Speed Claims

| Metric | Value | Source |
|--------|-------|--------|
| LLM Context API p90 latency | Under 600ms total | Brave blog (most-powerful-search-api-for-ai) |
| LLM Context overhead (p90) | ~130ms on top of normal search | Brave blog (most-powerful-search-api-for-ai) |
| Average latency (independent benchmark) | 669ms | AIMultiple agentic search benchmark (2026) |
| Max throughput | Up to 100 req/sec (Enterprise) | Brave API docs |

### Accuracy / Coverage Claims

| Metric | Value | Source |
|--------|-------|--------|
| Index size | 40+ billion pages | Brave official (also stated as 30+ billion in some earlier references; 40B is latest) |
| Daily index updates | 100+ million page updates/day | Brave official blog |
| Snippets per result | Up to 5 (with `extra_snippets` param) | Brave API docs |
| Points of interest (Place Search) | 200+ million | Brave blog (place-search-api) |
| Search independence | 100% (since April 2023) | Brave blog (search-independence) |
| Annualized queries | 8+ billion | Brave blog |
| SOC 2 Type II | Certified (October 2025) | Audited by Prescient Security |

### Privacy Claims

| Claim | Details | Source |
|-------|---------|--------|
| No query logging | System-wide policies prevent logging/retention of end-user search queries for unintended purposes | Brave blog (search-api-zero-data-retention) |
| No user identification | No identifiers collected that can link queries to individuals or devices | Brave privacy notice |
| Standard retention | Search queries retained max 90 days for billing/troubleshooting | Brave API privacy policy |
| Zero Data Retention (ZDR) | Enterprise option — no queries stored, logged, or linked. True ZDR possible because Brave controls entire stack. | Brave blog (search-api-zero-data-retention) |
| Full stack control | Unlike scraper-based APIs, queries never leave Brave infrastructure — no third-party data exposure | Brave blog |

## Limitations

<!-- Sources: https://www.firecrawl.dev/blog/brave-search-api-alternatives, https://community.brave.app/, https://github.com/modelcontextprotocol/servers/issues/ -->

### What It Can't Do

- **No full-page content retrieval**: The Search API returns snippets and metadata, not full page content. The LLM Context API extracts smart chunks but does not return raw HTML/full page. For full page scraping, a separate tool is needed.
- **No multi-step navigation**: Cannot follow links, fill forms, or perform multi-step web interactions. It is a search API, not a browser automation tool.
- **No caching/storage rights by default**: Standard terms prohibit storing, caching, or creating databases of search results. Requires "Data w/ Storage Rights" plan tier.
- **No custom crawl requests**: Cannot request Brave to crawl specific URLs or prioritize specific domains in the index.
- **No real-time streaming for web search**: Streaming is only available for the Summarizer endpoint, not for standard web/image/news search.
- **Place Search radius limit**: Maximum 20km (13 miles) from center point.

### Known Failure Modes

- **Long-tail/niche query coverage gaps**: Brave's index is smaller than Google's. For very niche, academic, or obscure queries, results may be thinner or missing entirely.
- **Non-English coverage**: While improving, coverage for non-English content and regional results may lag behind Google.
- **Rate limit errors under load**: MCP server users have reported rate limit issues, especially with the legacy free tier's 1 req/sec limit (GitHub issue #596 on modelcontextprotocol/servers).
- **Summarizer not always available**: The Summarizer only works for eligible queries — not all queries return a summarizer key. Eligibility is determined by Brave's internal logic.

### Documented Issues

- **Feb 2026 pricing change disruption**: Removal of free tier surprised many developers. Legacy free-tier users grandfathered but new users must provide credit card for $5/month credit.
- **Separate API keys per plan**: Each subscription type (Search, Spellcheck, Autosuggest) requires its own API key, which can be confusing.
- **MCP server rate limit configurability**: GitHub issue #3127 on modelcontextprotocol/servers requests making rate limits configurable via environment variables, as defaults don't match all tiers.
- **TOS ambiguity for open-source**: GitHub issue #522 on modelcontextprotocol/servers raised concerns about potential TOS violations when using Brave Search API results in open-source MCP contexts.

## Differentiation

> **Independent index** (not Google/Bing dependent): Brave is one of only three web search indexes at scale in the West (alongside Google and Yandex), and the only one commercially available as a reliable, independent API — especially critical after Bing's API shutdown in August 2025. **Privacy-first**: No query logging, optional Zero Data Retention, full-stack control means queries never leave Brave infrastructure. **AI-native endpoints**: LLM Context API delivers pre-extracted smart chunks optimized for RAG/LLM grounding (not just links), plus a built-in Summarizer powered by three custom LLMs. **Goggles**: Unique custom re-ranking system allowing programmatic control over result ordering via a domain-specific language. **Cost structure**: $5/month free credit, $0.005 per query at scale, with included AI inference rights. **SOC 2 Type II certified** (Oct 2025). **Supplies most top-10 LLMs** with real-time search data, including Claude.

## Raw Notes

- Brave Search was built on technology acquired from Tailcat (open-source search engine by Cliqz GmbH, Germany), acquired in 2021.
- The Brave browser's Web Discovery Project is a key differentiator for index quality — it crowdsources URL discovery and engagement signals from opt-in browser users, providing long-tail coverage that traditional crawlers miss.
- Post-Bing API shutdown (Aug 2025), Brave has positioned itself as the de facto alternative for developers who previously used Bing Search API. Google took legal action against SerpApi around the same time, further consolidating Brave's position.
- The Feb 2026 pricing restructure aligned with the launch of the LLM Context API — Brave is clearly pivoting toward being the search infrastructure layer for AI applications.
- No official SDK exists — this is by design as the API is intentionally simple (HTTP GET + JSON). Community SDKs exist for Python, Node/TS, and Go.
- The API Assistant in the Developer Portal is trained to answer API questions and generate code examples.
- Brave Search MCP server is in the official modelcontextprotocol/servers repo and was highlighted in early Claude Code demos.

## Source Log

<!-- Complete list of all sources consulted for this profile -->

| # | Source | Type | URL | Accessed |
|---|--------|------|-----|----------|
| 1 | Brave Search API — official landing page | official-docs | https://brave.com/search/api/ | 2026-03-01 |
| 2 | Brave Search API Documentation — Get Started | api-reference | https://api-dashboard.search.brave.com/app/documentation/web-search/get-started | 2026-03-01 |
| 3 | Brave Blog — "Most powerful search API for AI to date" (Feb 2026) | blog-official | https://brave.com/blog/most-powerful-search-api-for-ai/ | 2026-03-01 |
| 4 | Brave Blog — "Search API shows exponential growth" | blog-official | https://brave.com/blog/search-api-growth/ | 2026-03-01 |
| 5 | Brave Blog — "Place Search API" (Feb 2026) | blog-official | https://brave.com/blog/place-search-api/ | 2026-03-01 |
| 6 | Brave Blog — "Search independence" (April 2023) | blog-official | https://brave.com/blog/search-independence/ | 2026-03-01 |
| 7 | Brave Blog — "Zero Data Retention" | blog-official | https://brave.com/blog/search-api-zero-data-retention/ | 2026-03-01 |
| 8 | Brave Blog — "SOC 2 Type II attestation" (Oct 2025) | blog-official | https://brave.com/blog/soc2/ | 2026-03-01 |
| 9 | Brave Blog — "AI Summarizer" | blog-official | https://brave.com/blog/ai-summarizer/ | 2026-03-01 |
| 10 | Brave Blog — "Search API launch" (May 2023) | blog-official | https://brave.com/blog/search-api-launch/ | 2026-03-01 |
| 11 | Brave Blog — "Introducing Rerank" (Jan 2025) | blog-official | https://brave.com/blog/search-rerank/ | 2026-03-01 |
| 12 | Brave Blog — "Snowflake integration" | blog-official | https://brave.com/blog/snowflake/ | 2026-03-01 |
| 13 | Brave Blog — "AWS Marketplace" (July 2025) | blog-official | https://brave.com/blog/search-api-aws-marketplace/ | 2026-03-01 |
| 14 | Brave Help Center — Web Discovery Project | official-docs | https://support.brave.app/hc/en-us/articles/4409406835469-What-is-the-Web-Discovery-Project | 2026-03-01 |
| 15 | Brave Search API — Rate Limiting docs | api-reference | https://api-dashboard.search.brave.com/documentation/guides/rate-limiting | 2026-03-01 |
| 16 | Brave Search API — Authentication docs | api-reference | https://api-dashboard.search.brave.com/documentation/guides/authentication | 2026-03-01 |
| 17 | Brave Search API — Goggles docs | api-reference | https://api-dashboard.search.brave.com/documentation/resources/goggles | 2026-03-01 |
| 18 | Brave Search API — Spellcheck docs | api-reference | https://api-dashboard.search.brave.com/documentation/services/spellcheck | 2026-03-01 |
| 19 | Brave Search API — Suggest docs | api-reference | https://api-dashboard.search.brave.com/documentation/services/suggest | 2026-03-01 |
| 20 | Brave Search API — LLM Context docs | api-reference | https://api-dashboard.search.brave.com/documentation/services/llm-context | 2026-03-01 |
| 21 | Brave Search API — Summarizer docs | api-reference | https://api-dashboard.search.brave.com/documentation/services/summarizer | 2026-03-01 |
| 22 | Brave Search API — Privacy notice | official-docs | https://api-dashboard.search.brave.com/documentation/resources/privacy-notice | 2026-03-01 |
| 23 | Brave Search API — Terms of Service | official-docs | https://api-dashboard.search.brave.com/terms-of-service | 2026-03-01 |
| 24 | Brave Search API — Plans page | official-docs | https://api-dashboard.search.brave.com/app/plans | 2026-03-01 |
| 25 | Brave Search API — "What sets it apart" guide | official-docs | https://brave.com/search/api/guides/what-sets-brave-search-api-apart/ | 2026-03-01 |
| 26 | GitHub — brave/brave-search-mcp-server | github-repo | https://github.com/brave/brave-search-mcp-server | 2026-03-01 |
| 27 | GitHub — brave-search-python-client | github-repo | https://github.com/helmut-hoffer-von-ankershoffen/brave-search-python-client | 2026-03-01 |
| 28 | npm — brave-search | github-repo | https://www.npmjs.com/package/brave-search | 2026-03-01 |
| 29 | LangChain — Brave Search integration docs | official-docs | https://python.langchain.com/docs/integrations/tools/brave_search/ | 2026-03-01 |
| 30 | LlamaIndex — Brave Search tool reference | official-docs | https://docs.llamaindex.ai/en/stable/api_reference/tools/brave_search/ | 2026-03-01 |
| 31 | Wikipedia — Brave Search | community | https://en.wikipedia.org/wiki/Brave_Search | 2026-03-01 |
| 32 | Implicator.ai — "Brave Drops Free Search API Tier" | blog-third-party | https://www.implicator.ai/brave-drops-free-search-api-tier-puts-all-developers-on-metered-billing/ | 2026-03-01 |
| 33 | AlternativeTo — "Brave updates Search API with LLM context endpoint" (Feb 2026) | blog-third-party | https://alternativeto.net/news/2026/2/brave-updates-its-search-api-with-a-new-llm-context-endpoint-for-ai-and-new-pricing/ | 2026-03-01 |
| 34 | AIMultiple — "Agentic Search 2026: Benchmark 8 Search APIs" | benchmark | https://aimultiple.com/agentic-search | 2026-03-01 |
| 35 | TechCrunch — "Anthropic using Brave for Claude web search" (March 2025) | news | https://techcrunch.com/2025/03/21/anthropic-appears-to-be-using-brave-to-power-web-searches-for-its-claude-chatbot/ | 2026-03-01 |
| 36 | GitHub — modelcontextprotocol/servers issue #596 (rate limits) | community | https://github.com/modelcontextprotocol/servers/issues/596 | 2026-03-01 |
| 37 | GitHub — modelcontextprotocol/servers issue #3127 (configurable rate limits) | community | https://github.com/modelcontextprotocol/servers/issues/3127 | 2026-03-01 |
| 38 | GitHub — modelcontextprotocol/servers issue #522 (TOS concerns) | community | https://github.com/modelcontextprotocol/servers/issues/522 | 2026-03-01 |
| 39 | Firecrawl Blog — "Top 5 Brave Search API Alternatives in 2026" | blog-third-party | https://www.firecrawl.dev/blog/brave-search-api-alternatives | 2026-03-01 |
| 40 | AWS Marketplace — Brave Search API listing | official-docs | https://aws.amazon.com/marketplace/pp/prodview-qjlabherxghtq | 2026-03-01 |
| 41 | Microsoft Learn — Bing Search API retirement announcement | official-docs | https://learn.microsoft.com/en-us/lifecycle/announcements/bing-search-api-retirement | 2026-03-01 |

<!--
Source Types:
  official-docs  -- Official documentation / API reference
  api-reference  -- Detailed API/SDK documentation
  github-repo    -- GitHub repository README, code, issues
  blog-official  -- Official blog post from the tool's org
  blog-third-party -- Third-party blog, comparison article
  benchmark      -- Independent benchmark or test results
  news           -- News article or press release
  community      -- Forum post, Discord, Stack Overflow
  inference      -- Derived/inferred from other data (flag for verification)
-->
