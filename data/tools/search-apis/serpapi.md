# SerpAPI

<!--
COLLECTION METADATA
==================
Collected: 2026-03-01
Collector: claude-opus-4-6
Session: agent-search-landscape research
GitHub Issue: N/A
Collection Method: WebSearch-based research with cross-referencing
Last Verified: 2026-03-01
Confidence: medium — WebFetch unavailable; pricing and engine counts cross-referenced across multiple third-party sources but official docs not directly scraped
-->

## Identity & Basics

| Field | Value | Source |
|-------|-------|--------|
| Organization | SerpApi (private company) | <!-- https://www.crunchbase.com/organization/serpapi --> |
| Founder/CEO | Julien Khaleghy | <!-- https://www.crunchbase.com/person/julien-khalegy --> |
| Website | https://serpapi.com | |
| License | Proprietary SaaS (SDKs and MCP server are MIT-licensed open source) | <!-- https://github.com/serpapi/serpapi-mcp --> |
| Founded | 2017 (some sources say 2016; 2017 is more commonly cited) | <!-- https://www.crunchbase.com/organization/serpapi, https://techcrunch.com/sponsor/serpapi/ --> |
| Headquarters | Austin, Texas, USA | <!-- https://www.crunchbase.com/organization/serpapi --> |
| GitHub Org | https://github.com/serpapi (98 repositories) | <!-- https://github.com/serpapi --> |
| GitHub Stars (legacy Python SDK) | ~731 stars (google-search-results-python) | <!-- https://github.com/serpapi/google-search-results-python, as of 2026-03-01 --> |
| GitHub Stars (new Python SDK) | ~121 stars (serpapi-python) | <!-- https://github.com/serpapi/serpapi-python, as of 2026-03-01 --> |
| GitHub Stars (JS SDK) | ~75 stars (serpapi-javascript) | <!-- https://github.com/serpapi, as of 2026-03-01 --> |
| Revenue | ~$3M ARR (as of 2025) | <!-- https://getlatka.com/companies/serpapi.com --> |
| Team Size | ~27 employees (as of 2025) | <!-- https://getlatka.com/companies/serpapi.com --> |
| npm Weekly Downloads | ~55,189 (serpapi package) | <!-- https://socket.dev/npm/package/serpapi, as of 2026-03-01 --> |

## Category Placement

- **Primary**: SERP Scraping API / Search Data Extraction
- **Secondary**: Web Search API for AI/LLM applications, SEO Data Provider

## Core Capability

> SerpAPI is a real-time SERP (Search Engine Results Page) scraping proxy that fetches, renders, and parses search engine results from 80+ search engines into structured JSON, handling all anti-bot detection, CAPTCHA solving, proxy rotation, and JavaScript rendering transparently behind a simple REST API.

## API Surface

<!-- Sources: https://serpapi.com/search-api, https://serpapi.com/search-engine-apis, https://deepwiki.com/serpapi/serpapi-python/4-supported-search-engines, https://www.smashingmagazine.com/2025/09/serpapi-complete-api-fetching-search-engine-data/, https://www.kdnuggets.com/2025/11/serpapi/automating-web-search-data-collection-for-ai-models-with-serpapi -->

### Endpoints / Methods

The API is accessed via a single primary endpoint with an `engine` parameter that selects the search engine:

**Primary endpoint:** `GET https://serpapi.com/search?engine={engine}&q={query}&api_key={key}`

| Engine Category | Specific Engines | Purpose |
|-----------------|-----------------|---------|
| **Web Search** | Google Search, Google Light, Bing, DuckDuckGo, Yahoo, Baidu, Yandex, Naver | General web search results |
| **Google Verticals** | Google Images, Google News, Google Videos, Google Shopping, Google Maps, Google Local, Google Flights, Google Finance, Google Trends, Google Jobs, Google Scholar, Google Patents, Google Play | Specialized Google result types |
| **E-commerce** | Amazon, Walmart, eBay, Home Depot | Product listings, pricing, reviews |
| **App Stores** | Apple App Store, Google Play Store | App search and metadata |
| **Other** | YouTube Search | Video search and metadata |

**Total count:** SerpAPI claims 80+ search engines / 100+ APIs. The distinction is that individual Google verticals (Images, News, Shopping, etc.) each count as separate APIs.

**Supplementary endpoints:**
| Endpoint | Purpose | Input | Output |
|----------|---------|-------|--------|
| `/search` | Primary search endpoint | `engine`, `q`, `api_key`, location/language params | JSON (structured results) or raw HTML |
| `/account` | Account info and usage | `api_key` | JSON (plan, usage, limits) |
| `/locations` | Location lookup | `q` (location string) | JSON (location IDs for geo-targeting) |
| Google Light API | Faster, leaner Google organic results | Same as `/search` with `engine=google_light` | JSON (organic results, knowledge graph, related questions only) |

### Authentication

- **Method:** API key passed as `api_key` query parameter on every request
- **Alternative:** `Authorization: Bearer {api_key}` header
- **MCP endpoint:** API key embedded in URL path: `https://mcp.serpapi.com/{API_KEY}/mcp`
- **Key obtained:** Free registration at serpapi.com; key visible in dashboard

### Rate Limits

Rate limits are tied to plan tier and calculated as a percentage of monthly allocation:

| Plan | Monthly Searches | Hourly Throughput Limit | Calculation |
|------|-----------------|------------------------|-------------|
| Free | 100 | ~20 | 20% of monthly |
| Developer | 5,000 | 1,000 | 20% of monthly |
| Production | 15,000 | 3,000 | 20% of monthly |
| Big Data | 30,000 | 6,000 | 20% of monthly |
| Enterprise (< 1M) | Custom | 20% of monthly volume | 20% of monthly |
| Enterprise (>= 1M) | Custom | 100,000 + 1% of monthly | Different formula |

**Key detail:** For plans under 1M searches/month, hourly throughput is capped at 20% of monthly volume. For plans at or above 1M, the formula changes to 100,000 + 1% of monthly volume.

### Input/Output Formats

- **Input:** HTTP GET request with URL query parameters (engine, q, location, gl, hl, num, start, tbm, etc.)
- **Output formats:**
  - `output=json` (default) -- Structured, parsed JSON with named fields for each SERP feature
  - `output=html` -- Raw HTML of the search results page
- **Structured JSON fields include:** organic_results, knowledge_graph, answer_box, local_results, shopping_results, related_questions (People Also Ask), related_searches, top_stories, ads, ai_overview, and many more depending on engine

## Pricing

<!-- Sources: https://serpapi.com/pricing, https://serpapi.com/enterprise, https://serpapi.com/ludicrous-speed, https://www.searchcans.com/blog/serpapi-pricing-alternatives-comparison-2026/, https://www.trustradius.com/products/serpapi/pricing, https://serpapi.com/blog/breakdown-of-serpapis-subscriptions/ -->
<!-- Pricing as of: 2026-03-01 -->

| Tier | Price/Month | Searches/Month | Hourly Throughput | Speed | Notes |
|------|-------------|----------------|-------------------|-------|-------|
| Free | $0 | 100 | ~20/hr | Best Effort | Auto-refund on failures |
| Developer | $75 | 5,000 | 1,000/hr | Best Effort | HTML + JSON output |
| Production | $150 | 15,000 | 3,000/hr | Best Effort | HTML + JSON output |
| Big Data | $275 | 30,000 | 6,000/hr | Best Effort (Medium Throughput); Legal US Shield | HTML + JSON output |
| Enterprise | $3,750+ | 100,000+ (base) | Custom | Best Effort or Ludicrous | Custom contracts, 99.97% SLA, ZeroTrace eligible |

**Note:** Some sources cite 6 pricing editions from $50-$2,500 (G2, GetApp). There may be additional tiers (e.g., a $50 Starter tier for 1,000 searches) or high-volume self-serve tiers between Big Data and Enterprise that are only visible after login. The four tiers above are confirmed across multiple sources.

### Speed Upgrades (Ludicrous Speed)

| Speed Tier | Multiplier vs. Best Effort | Cost Multiplier | Description |
|------------|---------------------------|-----------------|-------------|
| Best Effort | 1x (baseline) | 1x | Standard |
| Ludicrous Speed | 2.2x faster | 2x price (searches count as 2) | 2x server resources, parallel requests |
| Ludicrous Speed Max | ~2.4x faster (9.6% over Ludicrous) | 4x price (searches count as 4) | 4x server resources |

**Enterprise on-demand pricing:**
- Best Effort: $7.50 / 1,000 searches
- Ludicrous Speed: $15.00 / 1,000 searches
- Ludicrous Speed Max: $30.00 / 1,000 searches

### Per-Unit Economics at Scale

| Volume | Effective Cost/Search | Notes |
|--------|----------------------|-------|
| Free (100) | $0.00 | Limited to 100/month |
| Developer (5K) | $0.015 ($15/1K) | Unused searches expire monthly |
| Production (15K) | $0.010 ($10/1K) | Unused searches expire monthly |
| Big Data (30K) | ~$0.0092 ($9.17/1K) | Unused searches expire monthly |
| Enterprise (100K base) | ~$0.0375 ($37.50/1K base) or $7.50/1K on-demand | Volume discounts available |

**Critical note on economics:** Monthly subscriptions operate on "use it or lose it" -- unused searches do not roll over. Third-party analysis estimates this inflates effective costs by 30-50% for workloads with variable traffic patterns.

## Technical Architecture

<!-- Sources: https://www.smashingmagazine.com/2025/09/serpapi-complete-api-fetching-search-engine-data/, https://serpapi.com/blog/demystify-a-serpapi-request/, https://serpapi.com/security, https://serpapi.com/zero-trace-mode, https://nodemaven.com/blog/serp-scraping-how-to-collect-search-engine-data-safely-with-proxies/ -->

### How It Works

SerpAPI operates as a SERP scraping proxy service. The request lifecycle:

1. **API request received:** Client sends GET request with search parameters (engine, query, location, language, etc.)
2. **Geo-routing:** SerpAPI's optimized routing selects appropriate proxy IPs for the target location, ensuring results match the requested geographic context
3. **Full browser rendering:** Each API request runs in a full browser environment (not simple HTTP requests). This handles JavaScript-heavy SERPs and dynamic content loading
4. **Anti-bot evasion:** The system handles CAPTCHA solving, cookie management, and browser fingerprint rotation to completely mimic human browsing behavior
5. **IP rotation:** Large rotating proxy network (residential and/or datacenter IPs) rotates frequently to avoid detection and throttling
6. **HTML parsing & structuring:** Raw SERP HTML is parsed and transformed into structured JSON with named fields for each SERP feature (organic results, knowledge graph, ads, featured snippets, etc.)
7. **Response delivery:** Structured JSON (or raw HTML if requested) returned to the client

**Key architectural properties:**
- All searches are **live** -- no caching or database of pre-scraped results
- Only **successful** searches count toward quota; cached, errored, and failed searches are not billed
- No AI/ML is described in the parsing pipeline; parsing appears to be rule-based extraction tailored to each search engine's SERP layout
- SerpAPI continuously updates parsers when search engines change their UI/layout

### Models / Infrastructure

- **No AI/ML models** for search or ranking -- SerpAPI is a scraping/parsing service, not a search engine
- Infrastructure handles proxy management, CAPTCHA solving, browser rendering at scale
- "Ludicrous Speed" allocates 2-4x server resources per request for parallel execution
- Global proxy infrastructure for location-accurate results worldwide

### Privacy & Security Features

- **ZeroTrace Mode** (Enterprise): Adds `zero_trace=true` parameter. SerpAPI does not store search parameters, search files, or any search data on their servers. Adds DNT (Do Not Track) headers to outbound requests. Helps with GDPR, HIPAA, CCPA compliance and SOC 2, ISO 27001 certifications
- **SLA:** 99.95% uptime guarantee on all paid plans, with 100% credit penalties for violations
- **Status page:** Reports ~99.911% uptime over trailing 30 days (as observed in benchmarks)

### Self-Host Options

- **Core API:** Cannot be self-hosted. SerpAPI is a proprietary SaaS service; you must use their infrastructure
- **MCP Server:** Open source (MIT license) at https://github.com/serpapi/serpapi-mcp -- can be cloned and run locally, but still requires a SerpAPI API key for actual search requests
- **SDKs:** All language SDKs are open source (MIT) and can be used in any environment

## Integration Surface

<!-- Sources: https://serpapi.com/libraries, https://serpapi.com/blog/serpapis-easy-integration-and-our-supported-language-libraries-curl-ruby-python-node-js-net-java-go-google-sheets/, https://serpapi.com/blog/introducing-serpapis-mcp-server/, https://python.langchain.com/api_reference/community/utilities/langchain_community.utilities.serpapi.SerpAPIWrapper.html, https://docs.crewai.com/en/tools/search-research/serpapi-googleshoppingtool, https://docs.flowiseai.com/integrations/langchain/document-loaders/serpapi-for-web-search -->

| Integration | Status | Notes |
|------------|--------|-------|
| MCP Server | Official, open source | https://github.com/serpapi/serpapi-mcp -- hosted endpoint at `https://mcp.serpapi.com/{KEY}/mcp` or self-run locally. Supports Google, Bing, Yahoo, DuckDuckGo, Yandex, Baidu, YouTube, eBay, Walmart, and more. MIT license. |
| Python SDK | Official (`serpapi` on PyPI) | New: `serpapi` package (~121 GitHub stars). Legacy: `google-search-results` package (~731 stars, deprecated). Install: `pip install serpapi` |
| Node/TS SDK | Official (`serpapi` on npm) | ~55K weekly npm downloads, ~75 GitHub stars. ESM and CommonJS. Supports Node.js and Deno. Install: `npm install serpapi` |
| Ruby SDK | Official (`serpapi` gem) | https://github.com/serpapi/serpapi-ruby (~17 stars) |
| Java SDK | Official | https://github.com/serpapi/serpapi-java |
| Go SDK | Official | https://github.com/serpapi/serpapi-golang |
| .NET SDK | Official | Listed in supported libraries |
| PHP SDK | Official | Listed in supported libraries |
| cURL | Direct REST API | Works with any HTTP client |
| Google Sheets | Official Extension | Native Google Sheets add-on for data fetching without code |
| LangChain | First-class integration | `langchain_community.utilities.SerpAPIWrapper`. Install `google-search-results` package + set `SERPAPI_API_KEY` env var. Supports sync (`run`) and async (`arun`, `aresults`). Can be loaded as a tool: `load_tools(["serpapi"])` |
| LlamaIndex | Via MCP or Composio | No native LlamaIndex tool, but usable through MCP server integration, Composio adapter, or direct API calls with the Python SDK |
| CrewAI | Official tools | `SerpApiGoogleShoppingTool` and `SerpApiGoogleSearchTool` available in `crewai_tools`. Requires `SERPAPI_API_KEY` env var |
| FlowiseAI | Built-in loader | SerpApi For Web Search document loader available |
| n8n | Official integration | SerpApi Official node available for workflow automation |
| Zapier | Integration available | Can be added to Zaps for automated SEO data collection |
| Make (Integromat) | Integration available | SerpAPI connector with 3,000+ app connections |
| Microsoft PromptFlow | Built-in tool | SerpAPI tool reference in PromptFlow docs |

## Performance Data

<!-- Sources: https://hackernoon.com/serp-benchmarks-success-rates-and-latency-at-scale, https://dev.to/ritza/best-serp-api-comparison-2025-serpapi-vs-exa-vs-tavily-vs-scrapingdog-vs-scrapingbee-2jci, https://serpapi.com/blog/who-has-the-fastest-google-search-api/, https://serpapi.com/ludicrous-speed -->

### Published Benchmarks

From third-party benchmark (HackerNoon, 50-query test):
- **Average response time:** 2.972 seconds
- **Success rate:** 100%
- **Response time range:** 0.070s to 13.308s (high variability)
- **P50 latency:** ~2.7 seconds
- **P95 latency:** ~8.2 seconds

### Speed Claims

- **Best Effort:** Standard speed, baseline
- **Ludicrous Speed:** 2.2x faster than Best Effort, claimed average < 1 second when combined with Google Light API
- **Ludicrous Speed Max:** 9.6% faster than Ludicrous Speed on average
- **Google Light API:** Fastest option -- returns only organic_results, knowledge_graph, related_questions, related_searches, and top_stories (skips rich SERP features for speed)

### Accuracy / Coverage Claims

- **Search engines supported:** 80+ search engines, 100+ APIs (SerpAPI's own claims)
- **Uptime SLA:** 99.95% guaranteed on all paid plans (100% credit for violations)
- **Observed uptime:** ~99.911% over trailing 30 days (status page)
- **Data freshness:** All searches are live (no caching/database); real-time scraping on every request
- **SERP coverage:** Parses organic results, featured snippets, knowledge graph, answer boxes, local packs, shopping results, ads, AI overviews, People Also Ask, related searches, top stories, image packs, video results, and more
- **Geo-targeting:** Supports location-specific results worldwide via `location`, `gl` (country), and `hl` (language) parameters

## Limitations

<!-- Sources: https://serpapi.com/faq, https://serpapi.com/api-status-and-error-codes, https://blog.apify.com/best-serpapi-alternatives/, https://www.serphouse.com/blog/serp-api-limitations/ -->

### What It Can't Do

- **No semantic/AI search:** SerpAPI returns raw SERP data as-is. It does not perform any semantic understanding, relevance ranking, or AI-powered search. It is not a search engine -- it is a scraping proxy for existing search engines
- **No content extraction:** Returns SERP snippets and metadata, not full-page content from result URLs. Does not crawl/scrape the pages linked in search results
- **No scheduling or task chaining:** No built-in cron/scheduler for recurring searches. Must be orchestrated externally (via n8n, Zapier, cron jobs, etc.)
- **No data enrichment:** Returns only what the search engine shows on its results page. No entity resolution, deduplication, or cross-referencing
- **No customizable scraping:** Cannot customize what or how things are scraped. The parsing rules are SerpAPI's, not configurable by the user
- **No self-hosting of core service:** The scraping infrastructure is proprietary and cannot be run on your own servers
- **No search result storage/history:** Results are transient (returned and discarded unless ZeroTrace is off, in which case metadata may be retained briefly). No built-in result archiving or comparison over time

### Known Failure Modes

- **High latency variability:** P95 latency can reach 8+ seconds, making it unreliable for real-time UI applications requiring consistent sub-second responses
- **Search engine changes:** When Google or other engines update their SERP layout, parsing may temporarily return incomplete or incorrectly structured data until SerpAPI updates their parsers
- **Quota expiration:** Unused monthly searches expire at billing cycle end and do not roll over, leading to waste for variable workloads
- **Hourly throttling:** The 20% hourly throughput cap can be a bottleneck for burst workloads (e.g., a 5,000 search/month plan can only do 1,000/hour)

### Documented Issues

- **Error codes:** Documented at https://serpapi.com/api-status-and-error-codes. Failed/errored searches are not counted against quota (only successful searches are billed)
- **Newer SERP types:** Some sources note SerpAPI can lag in supporting newer SERP features (e.g., new Google SERP layouts) compared to building custom scrapers
- **No interactive/dynamic queries:** Cannot perform multi-step interactions with search engines (e.g., clicking through pagination is done via `start` parameter, not actual browser interaction)

## Differentiation

> SerpAPI's primary differentiator is **breadth of search engine coverage** (80+ engines including Google, Bing, Baidu, Yandex, Naver, DuckDuckGo, YouTube, Amazon, Walmart, eBay, Apple App Store, and many Google verticals) combined with **depth of structured data extraction** (Knowledge Graph, AI Overview, Featured Snippets, Local Pack, Shopping, Ads, People Also Ask, and dozens of other SERP features parsed into clean JSON). No competitor matches this combination of breadth and structural depth.

**Specific differentiators:**
- **Longest track record:** Operating since 2017, making it one of the oldest and most established SERP API providers
- **Google vertical depth:** Individual APIs for Google Finance, Google Trends, Google Jobs, Google Scholar, Google Patents, Google Flights, Google Maps, Google Play -- not just web search
- **Ludicrous Speed:** Unique speed tier system (2.2x and 4x resource allocation) for time-sensitive applications
- **ZeroTrace privacy:** Enterprise-grade zero data retention mode with DNT headers for compliance (GDPR, HIPAA, CCPA)
- **Google Light API:** Stripped-down, faster API for organic-results-only use cases
- **MCP Server:** Official, open-source MCP server with both hosted and self-hosted deployment options
- **Broad SDK coverage:** Official SDKs in 8 languages (Python, Node.js, Ruby, Java, Go, .NET, PHP) plus Google Sheets and cURL
- **LangChain first-class support:** Built-in `SerpAPIWrapper` in LangChain community, making it arguably the most common search tool in LLM agent frameworks
- **Only successful searches billed:** Failed, cached, and errored searches do not consume quota

## Raw Notes

- SerpAPI was one of the first SERP scraping APIs (2017) and has built significant brand recognition in the developer/SEO community
- The legacy Python package (`google-search-results`) is being deprecated in favor of the newer `serpapi` package -- migration may affect existing integrations
- The "100+ APIs" vs "80+ search engines" discrepancy appears to be because Google verticals (Images, News, Shopping, etc.) are counted as separate APIs but Google is one search engine
- Revenue of ~$3M with 27 employees suggests a lean, profitable operation
- SerpAPI's blog actively publishes competitor comparisons, benchmarks, and AI integration guides -- strong content marketing presence
- The Smashing Magazine article confirms "each API request runs in a full browser" -- this is not lightweight HTTP scraping
- The company appears to be a bootstrapped/self-funded operation (no Crunchbase funding rounds found in search results)
- SerpAPI has 98 repositories on GitHub, indicating significant open-source investment beyond just SDKs (blog code, tutorials, etc.)

## Source Log

<!-- Complete list of all sources consulted for this profile -->

| # | Source | Type | URL | Accessed |
|---|--------|------|-----|----------|
| 1 | SerpAPI Homepage | official-docs | https://serpapi.com/ | 2026-03-01 |
| 2 | SerpAPI Search API Docs | api-reference | https://serpapi.com/search-api | 2026-03-01 |
| 3 | SerpAPI Search Engine APIs List | api-reference | https://serpapi.com/search-engine-apis | 2026-03-01 |
| 4 | SerpAPI Pricing Page | official-docs | https://serpapi.com/pricing | 2026-03-01 |
| 5 | SerpAPI Enterprise Pricing | official-docs | https://serpapi.com/enterprise | 2026-03-01 |
| 6 | SerpAPI Ludicrous Speed | official-docs | https://serpapi.com/ludicrous-speed | 2026-03-01 |
| 7 | SerpAPI Ludicrous Speed Max | official-docs | https://serpapi.com/ludicrous-speed-max | 2026-03-01 |
| 8 | SerpAPI Google Light API | official-docs | https://serpapi.com/google-light-api | 2026-03-01 |
| 9 | SerpAPI ZeroTrace Mode | official-docs | https://serpapi.com/zero-trace-mode | 2026-03-01 |
| 10 | SerpAPI FAQ | official-docs | https://serpapi.com/faq | 2026-03-01 |
| 11 | SerpAPI Status/Error Codes | api-reference | https://serpapi.com/api-status-and-error-codes | 2026-03-01 |
| 12 | SerpAPI Libraries Page | official-docs | https://serpapi.com/libraries | 2026-03-01 |
| 13 | SerpAPI Knowledge Graph API | api-reference | https://serpapi.com/knowledge-graph | 2026-03-01 |
| 14 | SerpAPI Account API | api-reference | https://serpapi.com/account-api | 2026-03-01 |
| 15 | SerpAPI Security | official-docs | https://serpapi.com/security | 2026-03-01 |
| 16 | SerpAPI Blog: Introducing MCP Server | blog-official | https://serpapi.com/blog/introducing-serpapis-mcp-server/ | 2026-03-01 |
| 17 | SerpAPI Blog: Subscription Breakdown | blog-official | https://serpapi.com/blog/breakdown-of-serpapis-subscriptions/ | 2026-03-01 |
| 18 | SerpAPI Blog: Birth of SerpApi | blog-official | https://serpapi.com/blog/the-birth-of-serpapi/ | 2026-03-01 |
| 19 | SerpAPI Blog: ZeroTrace Privacy | blog-official | https://serpapi.com/blog/how-serpapis-zero-trace-keeps-your-data-invisible-to-trackers/ | 2026-03-01 |
| 20 | SerpAPI Blog: Google Light API | blog-official | https://serpapi.com/blog/scrape-google-organic-results-faster-with-the-new-google-light-search-api/ | 2026-03-01 |
| 21 | SerpAPI Blog: SDK Libraries | blog-official | https://serpapi.com/blog/serpapis-easy-integration-and-our-supported-language-libraries-curl-ruby-python-node-js-net-java-go-google-sheets/ | 2026-03-01 |
| 22 | SerpAPI Blog: Web Search API for AI | blog-official | https://serpapi.com/blog/the-web-search-api-for-ai-applications/ | 2026-03-01 |
| 23 | SerpAPI Blog: Fastest Google Search API Benchmark | blog-official | https://serpapi.com/blog/who-has-the-fastest-google-search-api/ | 2026-03-01 |
| 24 | GitHub: serpapi org | github-repo | https://github.com/serpapi | 2026-03-01 |
| 25 | GitHub: google-search-results-python | github-repo | https://github.com/serpapi/google-search-results-python | 2026-03-01 |
| 26 | GitHub: serpapi-python | github-repo | https://github.com/serpapi/serpapi-python | 2026-03-01 |
| 27 | GitHub: serpapi-mcp | github-repo | https://github.com/serpapi/serpapi-mcp | 2026-03-01 |
| 28 | npm: serpapi package | api-reference | https://www.npmjs.com/package/serpapi | 2026-03-01 |
| 29 | PyPI: serpapi package | api-reference | https://pypi.org/project/serpapi/ | 2026-03-01 |
| 30 | DeepWiki: Supported Search Engines | api-reference | https://deepwiki.com/serpapi/serpapi-python/4-supported-search-engines | 2026-03-01 |
| 31 | Crunchbase: SerpApi | news | https://www.crunchbase.com/organization/serpapi | 2026-03-01 |
| 32 | TechCrunch: SerpApi sponsor article | news | https://techcrunch.com/sponsor/serpapi/ | 2026-03-01 |
| 33 | GetLatka: SerpApi revenue data | news | https://getlatka.com/companies/serpapi.com | 2026-03-01 |
| 34 | Smashing Magazine: SerpApi article | blog-third-party | https://www.smashingmagazine.com/2025/09/serpapi-complete-api-fetching-search-engine-data/ | 2026-03-01 |
| 35 | KDnuggets: SerpApi for AI | blog-third-party | https://www.kdnuggets.com/2025/11/serpapi/automating-web-search-data-collection-for-ai-models-with-serpapi | 2026-03-01 |
| 36 | HackerNoon: SERP Benchmarks | benchmark | https://hackernoon.com/serp-benchmarks-success-rates-and-latency-at-scale | 2026-03-01 |
| 37 | DEV.to: SERP API Comparison 2025 | benchmark | https://dev.to/ritza/best-serp-api-comparison-2025-serpapi-vs-exa-vs-tavily-vs-scrapingdog-vs-scrapingbee-2jci | 2026-03-01 |
| 38 | SearchCans: SerpApi Pricing 2026 | blog-third-party | https://www.searchcans.com/blog/serpapi-pricing-alternatives-comparison-2026/ | 2026-03-01 |
| 39 | SearchCans: SERP API Pricing Index 2026 | blog-third-party | https://www.searchcans.com/blog/serp-api-pricing-index-2026/ | 2026-03-01 |
| 40 | LangChain: SerpAPI integration docs | api-reference | https://docs.langchain.com/oss/python/integrations/providers/serpapi | 2026-03-01 |
| 41 | LangChain: SerpAPIWrapper reference | api-reference | https://python.langchain.com/api_reference/community/utilities/langchain_community.utilities.serpapi.SerpAPIWrapper.html | 2026-03-01 |
| 42 | CrewAI: SerpApi Google Shopping Tool | api-reference | https://docs.crewai.com/en/tools/search-research/serpapi-googleshoppingtool | 2026-03-01 |
| 43 | Apify: SerpApi alternatives | blog-third-party | https://blog.apify.com/best-serpapi-alternatives/ | 2026-03-01 |
| 44 | ScrapingBee: Best SERP APIs 2026 | blog-third-party | https://www.scrapingbee.com/blog/best-serp-apis/ | 2026-03-01 |
| 45 | Socket.dev: serpapi npm analysis | api-reference | https://socket.dev/npm/package/serpapi | 2026-03-01 |
| 46 | n8n: SerpApi integration | api-reference | https://n8n.io/integrations/serpapi/ | 2026-03-01 |
| 47 | LinkGo: SerpApi pricing FAQ | blog-third-party | https://linkgo.dev/faq/the-pricing-structure-for-serpapi | 2026-03-01 |
| 48 | Composio: SerpApi integrations | api-reference | https://composio.dev/tools/serpapi/all | 2026-03-01 |
| 49 | FlowiseAI: SerpApi loader | api-reference | https://docs.flowiseai.com/integrations/langchain/document-loaders/serpapi-for-web-search | 2026-03-01 |
| 50 | Microsoft PromptFlow: SerpAPI tool | api-reference | https://microsoft.github.io/promptflow/reference/tools-reference/serp-api-tool.html | 2026-03-01 |

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
