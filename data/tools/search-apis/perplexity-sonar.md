# Perplexity Sonar API

<!--
COLLECTION METADATA
==================
Collected: 2026-03-01
Collector: claude-opus-4-6
Session: agent-search-landscape research
GitHub Issue: N/A
Collection Method: WebSearch with cross-referencing against official docs URLs
Last Verified: 2026-03-01
Confidence: medium — Assembled from WebSearch results referencing official docs; WebFetch to docs.perplexity.ai was unavailable, so direct page verification was not possible. Pricing and model details cross-referenced across multiple sources.
-->

## Identity & Basics

| Field | Value | Source |
|-------|-------|--------|
| Organization | Perplexity AI (founded August 2022 by Aravind Srinivas, Denis Yarats, Andy Konwinski, Johnny Ho) | <!-- https://en.wikipedia.org/wiki/Perplexity_AI --> |
| Website | https://sonar.perplexity.ai/ (API platform); https://www.perplexity.ai/ (consumer product) | <!-- https://www.perplexity.ai/api-platform --> |
| License | Proprietary / Commercial API. Governed by Perplexity API Terms of Service. Commercial use permitted but AUP prohibits building competitive products or redistributing the service. | <!-- https://www.perplexity.ai/hub/legal/perplexity-api-terms-of-service --> |
| Launch Date | Sonar API: January 21, 2025. Search API: September 25, 2025. | <!-- https://techcrunch.com/2025/01/21/perplexity-launches-sonar-an-api-for-ai-search/ ; https://www.perplexity.ai/hub/blog/introducing-the-perplexity-search-api --> |
| GitHub (Official) | https://github.com/perplexityai (org); key repos: perplexity-py, perplexity-node, modelcontextprotocol | <!-- https://github.com/perplexityai --> |
| GitHub Stars | Not confirmed via direct page load — repos are relatively new (2025) | <!-- as of 2026-03-01 --> |
| Community Size | Discord: ~375K members; API forum: community.perplexity.ai (~12K queries/month); Developer integrations: ~85K active (as of mid-2025) | <!-- https://discord.com/servers/perplexity-1047197230748151888 ; https://sqmagazine.co.uk/perplexity-ai-statistics/ --> |

## Category Placement

- **Primary**: Search API (AI-powered web search with synthesized answers)
- **Secondary**: LLM API (chat completions with grounding), Research API (deep research mode)

## Core Capability

> Perplexity Sonar provides an API for AI-powered web search that returns synthesized natural-language answers with inline citations, grounded in real-time web data from an index of hundreds of billions of pages -- not just raw search results, but actual answers.

## API Surface

<!-- Sources: https://docs.perplexity.ai/api-reference/chat-completions-post ; https://docs.perplexity.ai/api-reference/search-post ; https://docs.perplexity.ai/guides/getting-started ; https://docs.perplexity.ai/guides/chat-completions-guide -->

### Endpoints / Methods

| Endpoint | Purpose | Input | Output |
|----------|---------|-------|--------|
| `POST /chat/completions` | AI-synthesized answers with web search grounding and citations | OpenAI-compatible messages array + model name | Streamed or complete chat response with citations array and search_results metadata |
| `POST /search` | Raw ranked web search results (no AI synthesis) | Query string + optional filters (domain, date, region) | Array of ranked result objects with title, URL, snippet, date, last_updated |

### Models Available (Sonar Family)

| Model ID | Category | Context Window | Description |
|----------|----------|---------------|-------------|
| `sonar` | Search | 128K tokens | Lightweight, fast grounded search. Built on Llama 3.3 70B. |
| `sonar-pro` | Search | 200K tokens | Deeper retrieval, multi-step queries, ~2x citations vs sonar |
| `sonar-reasoning` | Reasoning | 128K tokens | Real-time reasoning with search (chain-of-thought) |
| `sonar-reasoning-pro` | Reasoning | 128K tokens | Advanced reasoning powered by DeepSeek-R1 with search |
| `sonar-deep-research` | Research | 128K tokens | Long-form, source-dense research reports; multi-step search |

**Deprecated model IDs:** `llama-3.1-sonar-*-128k-online`, `pplx-*` (migrated to `sonar*` family).

### Search Modes (for Sonar, Sonar Pro, Sonar Reasoning Pro)

| Mode | Behavior | Cost Impact |
|------|----------|-------------|
| `high` | Maximum search depth and context for complex queries | Highest per-request fee |
| `medium` | Balanced depth for moderately complex questions | Mid-range per-request fee |
| `low` | Cost-optimized, strong accuracy for straightforward queries | Lowest per-request fee |

### Authentication

- **Method:** Bearer token via `Authorization` header
- **Format:** `Authorization: Bearer <PERPLEXITY_API_KEY>`
- **Key generation:** Via Perplexity web console at https://www.perplexity.ai/account/api/keys (requires creating an API Group first)
- **Programmatic key generation:** `POST https://api.perplexity.ai/generate_auth_token` with Bearer auth and optional `token_name` parameter

### Rate Limits

| Model | Requests Per Minute (RPM) | Notes |
|-------|--------------------------|-------|
| `sonar` | Up to 4,000 RPM | Tier-dependent |
| `sonar-pro` | Up to 4,000 RPM | Tier-dependent |
| `sonar-reasoning-pro` | Up to 4,000 RPM | Tier-dependent |
| `sonar-deep-research` | 100 RPM | Lower due to multi-step nature |
| Search API | 50 req/sec (burst: 50) | Independent of usage tier |

**Usage Tiers (0-5):** Rate limits scale with cumulative spend. Tier 0 (new accounts) has the most restrictions; Tier 5 ($5,000+ lifetime spend) offers highest limits.

**Rate limiting algorithm:** Leaky bucket (allows burst traffic while maintaining long-term rate control).

### Input/Output Formats

**Chat Completions (OpenAI-compatible):**
- Input: JSON body with `model`, `messages` (array of role/content), optional `temperature` (0-2), `max_tokens`, `stream` (boolean), `search_context_size` ("low"/"medium"/"high"), `search_recency_filter`, `search_domain_filter` (up to 20 domains), `response_format` (JSON schema for structured output), `return_images` (Tier 2+)
- Output: Standard chat completion object + `citations` array (URLs) + `search_results` array (title, URL, snippet, date, source)
- Streaming: Supported via server-sent events (SSE)

**Image input:** Supported. Base64 data URIs or HTTPS URLs. Formats: PNG, JPEG, WEBP, GIF. Max 50MB.

**Structured output:** JSON Schema via `response_format` parameter. Note: first request with a new schema may take 10-30s. Recursive schemas and unconstrained objects not supported.

**Search API:**
- Input: Query string + optional `search_recency_filter`, `search_domain_filter`, region targeting
- Output: Array of result objects: `{ title, url, snippet, date, last_updated }`

## Pricing

<!-- Sources: https://docs.perplexity.ai/docs/getting-started/pricing ; https://www.glbgpt.com/hub/perplexity-api-cost-2025/ ; https://pricepertoken.com/pricing-page/provider/perplexity ; https://www.finout.io/blog/perplexity-pricing-in-2026 -->
<!-- Pricing as of: 2026-03-01 -->

### Token Pricing

| Model | Input (per 1M tokens) | Output (per 1M tokens) |
|-------|-----------------------|------------------------|
| `sonar` | $1.00 | $1.00 |
| `sonar-pro` | $3.00 | $15.00 |
| `sonar-reasoning` | $2.00 | $8.00 |
| `sonar-reasoning-pro` | $2.00 | $8.00 |
| `sonar-deep-research` | $2.00 (input) | $8.00 (output) |

**Note on sonar-deep-research:** Pricing has additional components -- $5/1,000 searches performed during research; reasoning tokens at $3/1M tokens; input tokens include both prompt tokens and citation tokens from search runs.

### Per-Request Fees (by search_context_size)

| Model Family | Low (per 1K req) | Medium (per 1K req) | High (per 1K req) |
|-------------|-------------------|---------------------|---------------------|
| Sonar / Sonar Reasoning | $5 | $8 | $12 |
| Sonar Pro / Sonar Reasoning Pro | $6 | $10 | $14 |

### Search API Pricing

| Endpoint | Price |
|----------|-------|
| Search API (`POST /search`) | $5 per 1,000 requests (no additional token-based pricing) |

### Citation Token Billing

As of 2025/2026: Citation tokens are **no longer billed** for Sonar and Sonar Pro models (except Sonar Deep Research). This simplifies billing and reduces effective costs.

### Per-Unit Economics at Scale

**Example (sonar, low context):** A basic query with 500 input tokens + 200 output tokens at low search context = ~$0.0057 per query ($0.005 request fee + ~$0.0007 tokens).

**Example (sonar-pro, high context):** A complex query with 1,000 input tokens + 500 output tokens at high search context = ~$0.0215 per query ($0.014 request fee + $0.003 input + $0.0075 output).

**At 1M queries/month (sonar, low):** ~$5,700/month in request fees + token costs.

**At 1M queries/month (sonar-pro, high):** ~$21,500/month.

## Technical Architecture

<!-- Sources: https://www.perplexity.ai/hub/blog/introducing-the-perplexity-search-api ; https://rankstudio.net/articles/en/perplexity-llm-tech-stack ; https://www.perplexity.ai/hub/blog/meet-new-sonar -->

### How It Works

**Web Index:** Perplexity maintains a proprietary web index covering hundreds of billions of pages. The index processes tens of thousands of update requests per second to maintain freshness. Machine learning models prioritize crawling and indexing, predicting URL importance based on factors like update frequency.

**Content Understanding:** An AI-powered content understanding module dynamically generates parsing logic (using frontier LLMs) to handle diverse website layouts. The module self-improves via iterative evaluation loops, processing millions of queries hourly.

**Retrieval Pipeline (multi-stage):**
1. **Hybrid retrieval** -- generates initial candidates using combined semantic + lexical methods
2. **Pre-filtering** -- removes noise
3. **Progressive ranking** -- applies lexical scoring, embedding-based similarity, and cross-encoder models
4. **Sub-document chunking** -- documents are divided into fine-grained units; individual chunks are scored against the query for precise relevance

**Answer Generation:** Retrieved content is passed to the Sonar model family (built on Llama 3.3 70B for base Sonar), which synthesizes a natural-language answer with inline citations. The model has been specifically fine-tuned for web-grounded, real-time answers.

**Inference:** Sonar runs on Cerebras inference infrastructure, achieving up to 1,200 tokens/second for fast answer generation.

### Models / Infrastructure

| Component | Detail |
|-----------|--------|
| Base model (Sonar) | Fine-tuned Llama 3.3 70B, optimized for search-grounded Q&A |
| Reasoning backbone | DeepSeek-R1 (for sonar-reasoning-pro) |
| Inference provider | Cerebras (high-speed inference) |
| Search index | Proprietary; hundreds of billions of pages; real-time updates |
| Content parsing | AI-powered dynamic parsing with LLM-driven adaptation |

### Consumer Product vs API Relationship

The consumer product (perplexity.ai / app) and the Sonar API share the same underlying search infrastructure and index. However:
- The API model may differ from the UI model for equivalent queries
- The API provides configurable parameters (search context size, domain filters, structured output) not exposed in the consumer UI
- Deep Research was integrated into Sonar's API architecture in April 2025

### Search API vs Chat Completions API

| Dimension | Chat Completions (`/chat/completions`) | Search API (`/search`) |
|-----------|----------------------------------------|----------------------|
| Output | AI-synthesized answer + citations + search_results | Raw ranked result objects (title, URL, snippet) |
| Use case | Conversational search, Q&A, research synthesis | Programmatic access to ranked web results |
| Pricing | Token-based + per-request fee | Per-request only ($5/1K) |
| AI synthesis | Yes (full LLM generation) | No (structured data only) |
| Customization | Messages, temperature, streaming, JSON schema | Domain filters, date range, region targeting |

### Self-Host Options

None. Perplexity Sonar is a fully managed cloud API. No self-hosting, on-premise, or open-source deployment option.

## Integration Surface

<!-- Sources: https://docs.perplexity.ai/guides/perplexity-sdk ; https://github.com/perplexityai ; https://docs.perplexity.ai/guides/mcp-server ; https://docs.llamaindex.ai/en/stable/examples/llm/perplexity/ ; https://python.langchain.com/docs/integrations/chat/perplexity/ ; https://ai-sdk.dev/providers/ai-sdk-providers/perplexity -->

| Integration | Status | Notes |
|------------|--------|-------|
| MCP Server | Official | `perplexityai/modelcontextprotocol` on GitHub. Install via `npx -y @perplexity-ai/mcp-server`. Supports all Sonar models + Search API. |
| Python SDK | Official | `perplexityai/perplexity-py` on GitHub. Python 3.9+. Sync + async (httpx). Install: `pip install perplexity-ai` (or similar). |
| Node/TS SDK | Official | `perplexityai/perplexity-node` on GitHub. npm: `@perplexity-ai/perplexity_ai`. Node 20+, Deno 1.28+, Bun 1.0+, Cloudflare Workers, Vercel Edge Runtime. Generated with Stainless. |
| OpenAI SDK Compatible | Yes (drop-in) | Change `base_url` to `https://api.perplexity.ai` in any OpenAI client. Works with Python `openai` and JS `openai` packages. |
| LangChain | Available | `ChatPerplexity` class in LangChain. Docs at python.langchain.com/docs/integrations/chat/perplexity/. |
| LlamaIndex | Available | `llama-index-llms-perplexity` package on PyPI. Supports sync/async chat completions and streaming. Default model: sonar-pro. |
| Vercel AI SDK | Available | `@ai-sdk/perplexity` npm package. Documented at ai-sdk.dev. |
| LiteLLM | Available | Provider: `perplexity`. Documented at docs.litellm.ai/docs/providers/perplexity. |
| CrewAI | Community/Partial | Works via LiteLLM integration. Some users report issues with newer Sonar models vs deprecated PPLX models. Not an official, dedicated integration. |
| Cloudflare AI Gateway | Available | Documented proxy support at developers.cloudflare.com/ai-gateway. |
| OpenRouter | Available | All Sonar models available via OpenRouter as a provider. |
| Promptfoo | Available | Provider integration documented at promptfoo.dev. |

## Performance Data

<!-- Sources: https://www.perplexity.ai/hub/blog/perplexity-sonar-dominates-new-search-arena-evolution ; https://www.perplexity.ai/hub/blog/new-sonar-search-modes-outperform-openai-in-cost-and-performance ; https://artificialanalysis.ai/providers/perplexity ; https://www.perplexity.ai/hub/blog/meet-new-sonar -->

### Published Benchmarks

| Benchmark | Model | Score | Comparison |
|-----------|-------|-------|------------|
| Search Arena | sonar-reasoning-pro | Score 1136 | Statistically tied with Gemini-2.5-Pro-Grounding (1142); beat it 53% in head-to-head |
| SimpleQA (F-score) | sonar-pro | 0.858 | Leads the benchmark |
| SimpleQA (F-score) | sonar | 0.773 | Strong for lightweight model |

**Claim:** Sonar and Sonar Pro outperform search-enabled GPT-4o on factuality while maintaining lower pricing.

### Speed Claims

| Metric | Value | Source |
|--------|-------|--------|
| Peak token generation (Cerebras) | ~1,200 tokens/sec | Perplexity blog (Meet New Sonar) |
| Measured output speed | 121.1 tokens/sec | Artificial Analysis benchmarks |
| Time to first token (TTFT) | 1.33 seconds | Artificial Analysis benchmarks |

### Accuracy / Coverage Claims

- Index covers "hundreds of billions of web pages" with "tens of thousands of index update requests per second"
- Sonar Pro provides approximately 2x the citations per search compared to base Sonar
- ~780 million queries processed monthly on the consumer product (as of Sept 2025)
- Citation tokens are free (not billed), encouraging comprehensive sourcing

## Limitations

<!-- Sources: https://docs.perplexity.ai/faq/faq ; https://community.perplexity.ai/ ; https://ai-sdk.dev/providers/ai-sdk-providers/perplexity -->

### What It Can't Do

- **No self-hosting or on-premise deployment** -- fully managed cloud API only
- **No image generation** -- can return images from search (Tier 2+) but does not generate them
- **No fine-tuning** -- you cannot fine-tune Sonar models; they are fixed
- **No guaranteed uptime SLA** -- Perplexity does not guarantee service uptime, failure frequency, or target recovery time
- **No recursive JSON schemas** -- structured output does not support recursive structures or unconstrained objects
- **No real-time chat memory** -- each API call is stateless; no built-in conversation persistence

### Known Failure Modes

- **sonar-deep-research timeouts:** Users report consistent timeouts after ~60 seconds on complex report generation tasks. Simple requests work fine; complex multi-step research can fail.
- **Reasoning token leakage:** sonar-reasoning-pro outputs a `<think>` section containing reasoning tokens before the actual JSON response. The `response_format` parameter does not remove these reasoning tokens.
- **Cold-start on structured output:** First request with a new JSON schema may take 10-30 seconds to process.
- **Rate limit 429 errors:** Especially on Tier 0 accounts with low RPM limits; leaky bucket algorithm means burst traffic may be silently queued.
- **API vs UI divergence:** The underlying AI model may differ between API and consumer UI for equivalent queries.

### Documented Issues

- **Domain filter limit:** Expanded to 20 entries as of August 2025 (previously lower)
- **search_recency_filter:** Filters by `last_updated` date; format is `%m/%d/%Y`. Can miss content that was published recently but indexed with older dates.
- **Token counting inconsistencies:** Some third-party integrations (e.g., LangChain `max_tokens`) have reported parameter handling issues (see langchain-ai/langchain#28229).
- **Deprecated model migration:** Older model IDs (`llama-3.1-sonar-*-128k-online`, `pplx-*`) have been deprecated. Applications using old IDs need migration to new `sonar*` family.

## Differentiation

> **What makes Perplexity Sonar unique vs. same-category alternatives:**
>
> 1. **Synthesized answers, not just search results.** Unlike Google Custom Search, Bing Web Search, or Brave Search API which return ranked links/snippets, Sonar returns a complete natural-language answer synthesized from multiple web sources. It is search + LLM in one API call.
>
> 2. **Built-in citations.** Every response includes inline citations with source URLs, enabling verifiable, grounded AI responses without building your own RAG pipeline.
>
> 3. **Real-time web grounding.** The proprietary index of hundreds of billions of pages with sub-second update rates means responses reflect current information, not stale training data.
>
> 4. **Model family spanning speed-to-depth spectrum.** From lightweight `sonar` (fast, cheap) to `sonar-deep-research` (thorough, multi-step) -- one API, multiple depth levels.
>
> 5. **OpenAI-compatible drop-in.** Uses the standard chat completions format, so any OpenAI client works with a base URL change. Minimal integration friction.
>
> 6. **Search + Chat API duality.** Both raw search results (Search API) and synthesized answers (Chat Completions) from the same infrastructure, letting developers choose their level of control.
>
> 7. **Cerebras-powered inference.** Peak speeds of ~1,200 tokens/sec enable near-instant answer generation for the search use case.

## Raw Notes

- Perplexity AI crossed $100M ARR by March 2025 and ~$200M by late 2025. Valuation reached $20B in Sept 2025.
- The company processes ~360M+ monthly API calls (separate from consumer product volume).
- The Sonar model was initially announced January 21, 2025 (TechCrunch coverage). The "Meet New Sonar" refresh (built on Llama 3.3 70B) came February 11, 2025.
- Deep Research was integrated into Sonar API in April 2025.
- The standalone Search API (returning raw results, no synthesis) launched September 25, 2025.
- Three search modes (high/medium/low) introduced alongside updated pricing in 2025.
- Perplexity's GitHub org operates under both `perplexityai` and `ppl-ai` namespaces.
- The MCP server is available as both an official npm package (`@perplexity-ai/mcp-server`) and via several community implementations.
- OpenRouter lists all Sonar models as available providers.

## Source Log

<!-- Complete list of all sources consulted for this profile -->

| # | Source | Type | URL | Accessed |
|---|--------|------|-----|----------|
| 1 | Perplexity API Docs (main) | official-docs | https://docs.perplexity.ai/ | 2026-03-01 |
| 2 | Perplexity API Pricing | official-docs | https://docs.perplexity.ai/docs/getting-started/pricing | 2026-03-01 |
| 3 | Perplexity Chat Completions API Reference | api-reference | https://docs.perplexity.ai/api-reference/chat-completions-post | 2026-03-01 |
| 4 | Perplexity Search API Reference | api-reference | https://docs.perplexity.ai/api-reference/search-post | 2026-03-01 |
| 5 | Perplexity SDK Quickstart | official-docs | https://docs.perplexity.ai/guides/perplexity-sdk | 2026-03-01 |
| 6 | Perplexity MCP Server Guide | official-docs | https://docs.perplexity.ai/guides/mcp-server | 2026-03-01 |
| 7 | Perplexity OpenAI Compatibility Guide | official-docs | https://docs.perplexity.ai/guides/chat-completions-guide | 2026-03-01 |
| 8 | Perplexity Image Attachments Guide | official-docs | https://docs.perplexity.ai/guides/image-attachments | 2026-03-01 |
| 9 | Perplexity API Key Management | official-docs | https://docs.perplexity.ai/guides/api-key-management | 2026-03-01 |
| 10 | Perplexity Changelog | official-docs | https://docs.perplexity.ai/changelog/changelog | 2026-03-01 |
| 11 | Perplexity API Platform (Sonar) | official-docs | https://sonar.perplexity.ai/ | 2026-03-01 |
| 12 | TechCrunch: Perplexity launches Sonar | news | https://techcrunch.com/2025/01/21/perplexity-launches-sonar-an-api-for-ai-search/ | 2026-03-01 |
| 13 | Blog: Introducing Sonar Pro API | blog-official | https://www.perplexity.ai/hub/blog/introducing-the-sonar-pro-api | 2026-03-01 |
| 14 | Blog: Meet New Sonar | blog-official | https://www.perplexity.ai/hub/blog/meet-new-sonar | 2026-03-01 |
| 15 | Blog: Introducing the Perplexity Search API | blog-official | https://www.perplexity.ai/hub/blog/introducing-the-perplexity-search-api | 2026-03-01 |
| 16 | Blog: Improved Sonar Models | blog-official | https://www.perplexity.ai/hub/blog/new-sonar-search-modes-outperform-openai-in-cost-and-performance | 2026-03-01 |
| 17 | Blog: Sonar Dominates Search Arena | blog-official | https://www.perplexity.ai/hub/blog/perplexity-sonar-dominates-new-search-arena-evolution | 2026-03-01 |
| 18 | GitHub: perplexityai/perplexity-py | github-repo | https://github.com/perplexityai/perplexity-py | 2026-03-01 |
| 19 | GitHub: perplexityai/perplexity-node | github-repo | https://github.com/perplexityai/perplexity-node | 2026-03-01 |
| 20 | GitHub: perplexityai/modelcontextprotocol | github-repo | https://github.com/perplexityai/modelcontextprotocol | 2026-03-01 |
| 21 | npm: @perplexity-ai/perplexity_ai | api-reference | https://www.npmjs.com/package/@perplexity-ai/perplexity_ai | 2026-03-01 |
| 22 | PyPI: llama-index-llms-perplexity | api-reference | https://pypi.org/project/llama-index-llms-perplexity/ | 2026-03-01 |
| 23 | LangChain Perplexity Integration | official-docs | https://python.langchain.com/docs/integrations/chat/perplexity/ | 2026-03-01 |
| 24 | LlamaIndex Perplexity Docs | official-docs | https://docs.llamaindex.ai/en/stable/examples/llm/perplexity/ | 2026-03-01 |
| 25 | Vercel AI SDK: Perplexity Provider | official-docs | https://ai-sdk.dev/providers/ai-sdk-providers/perplexity | 2026-03-01 |
| 26 | LiteLLM: Perplexity Provider | official-docs | https://docs.litellm.ai/docs/providers/perplexity | 2026-03-01 |
| 27 | Artificial Analysis: Perplexity | benchmark | https://artificialanalysis.ai/providers/perplexity | 2026-03-01 |
| 28 | OpenRouter: Perplexity Models | api-reference | https://openrouter.ai/perplexity | 2026-03-01 |
| 29 | Helicone: Sonar Pricing Calculator | blog-third-party | https://www.helicone.ai/llm-cost/provider/perplexity/model/sonar | 2026-03-01 |
| 30 | pricepertoken.com: Perplexity API Pricing | blog-third-party | https://pricepertoken.com/pricing-page/provider/perplexity | 2026-03-01 |
| 31 | Global GPT: Perplexity API Cost 2025 | blog-third-party | https://www.glbgpt.com/hub/perplexity-api-cost-2025/ | 2026-03-01 |
| 32 | Finout: Perplexity Pricing in 2026 | blog-third-party | https://www.finout.io/blog/perplexity-pricing-in-2026 | 2026-03-01 |
| 33 | SQ Magazine: Perplexity AI Statistics 2025 | news | https://sqmagazine.co.uk/perplexity-ai-statistics/ | 2026-03-01 |
| 34 | Perplexity API Terms of Service | official-docs | https://www.perplexity.ai/hub/legal/perplexity-api-terms-of-service | 2026-03-01 |
| 35 | Perplexity Acceptable Use Policy | official-docs | https://www.perplexity.ai/hub/legal/aup | 2026-03-01 |
| 36 | Wikipedia: Perplexity AI | news | https://en.wikipedia.org/wiki/Perplexity_AI | 2026-03-01 |
| 37 | RankStudio: Perplexity LLM Tech Stack | blog-third-party | https://rankstudio.net/articles/en/perplexity-llm-tech-stack | 2026-03-01 |
| 38 | Perplexity API Forum: Sonar Deep Research Timeout | community | https://community.perplexity.ai/t/perplexity-sonar-deep-research-model-timeout-issue-seeking-solution/2094 | 2026-03-01 |
| 39 | Perplexity API Forum: August 2025 Updates | community | https://community.perplexity.ai/t/sonar-api-updates-august-29-2025/1252 | 2026-03-01 |
| 40 | Cloudflare AI Gateway: Perplexity | official-docs | https://developers.cloudflare.com/ai-gateway/usage/providers/perplexity/ | 2026-03-01 |
| 41 | InfoQ: Perplexity Search API Launch | news | https://www.infoq.com/news/2025/09/perplexity-search-api/ | 2026-03-01 |
| 42 | Promptfoo: Perplexity Provider | official-docs | https://www.promptfoo.dev/docs/providers/perplexity/ | 2026-03-01 |
| 43 | DeepWiki: perplexity-node Search API | blog-third-party | https://deepwiki.com/perplexityai/perplexity-node/4.1-search-api | 2026-03-01 |

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
