# Agent Search Landscape: Deep Analysis Design

## Project Overview

A multi-part research project to comprehensively analyze the AI agent search and internet access tool landscape — covering every significant tool, how they compare, where the gaps are, and what's worth building.

### Purpose (all of the following)
- Personal reference / knowledge base
- Buying decisions for specific projects
- Published research artifact
- Inform what to build (the analysis reveals the opportunity)

### Project Structure

| Part | Focus | Status |
|------|-------|--------|
| **Part 1** | Data Collection | Active |
| **Part 2** | Analysis & Benchmarking | Future |
| **Part 3** | Synthesis & Opportunity Map | Future |

---

## Part 1: Data Collection

### Goal
Gather comprehensive, structured information on every significant tool in the agent search landscape. Standardized profiles. Raw data. No judgments yet.

### Scope
- ~20-25 tools across all 7 landscape categories
- Data sources: web research + API documentation deep reads
- No hands-on testing in this phase (reserved for Part 2)
- No comparative analysis in this phase (reserved for Part 2)

### Tool List

**Search APIs** (~5 tools)
- Tavily
- Exa
- Perplexity Sonar API
- SerpAPI
- Brave Search API

**Scraping & Extraction** (~4 tools)
- Firecrawl
- Crawl4AI
- Jina Reader
- Apify

**Browser Automation** (~4 tools)
- Playwright
- Browser Use
- Stagehand
- Skyvern

**Cloud Browsers** (~2 tools)
- Browserbase
- Steel

**Deep Research** (~3 tools)
- Firecrawl /agent
- Perplexity Deep Research
- OpenAI Deep Research

**Aggregators** (~2 tools)
- mcp-omnisearch
- Composio

**Built-in** (~2 tools)
- Claude Web Search
- Claude Web Fetch

### Per-Tool Profile Template

Each tool gets a standardized markdown file with these sections:

1. **Identity & Basics** — Name, organization, license (open-source vs proprietary), launch date, GitHub stars, community size
2. **Category Placement** — Primary category + any secondary categories
3. **Core Capability** — What it does in one sentence
4. **API Surface** — Endpoints/methods, authentication method, input/output formats, rate limits
5. **Pricing** — Free tier details, paid tier breakdown, per-unit economics at scale
6. **Technical Architecture** — How it works under the hood (models used, infrastructure, crawling approach, self-host options)
7. **Integration Surface** — MCP server availability, SDK languages, framework integrations (LangChain, LlamaIndex, CrewAI, etc.)
8. **Performance Data** — Published benchmarks, speed claims, accuracy claims, coverage metrics
9. **Limitations** — What it can't do, known failure modes, documented issues
10. **Differentiation** — Unique value proposition vs. same-category alternatives
11. **Raw Notes** — Anything interesting that doesn't fit the template

### Directory Structure

```
data/
  tools/
    search-apis/
      tavily.md
      exa.md
      perplexity-sonar.md
      serpapi.md
      brave-search.md
    scraping-extraction/
      firecrawl.md
      crawl4ai.md
      jina-reader.md
      apify.md
    browser-automation/
      playwright.md
      browser-use.md
      stagehand.md
      skyvern.md
    cloud-browsers/
      browserbase.md
      steel.md
    deep-research/
      firecrawl-agent.md
      perplexity-deep-research.md
      openai-deep-research.md
    aggregators/
      mcp-omnisearch.md
      composio.md
    built-in/
      claude-web-search.md
      claude-web-fetch.md
  index.md
```

### Process
1. Work through categories one at a time
2. For each tool: web research + read actual API/SDK docs
3. Fill the standardized template
4. Save to the appropriate directory
5. Update index.md as tools are completed
6. No analysis, no comparisons — just data

### Deliverable
A complete, structured data set of ~20-25 tool profiles ready for comparative analysis in Part 2.

---

## Part 2: Analysis & Benchmarking (Future)

- Head-to-head comparisons within categories
- Capability-centric mapping (which tools serve which agent needs)
- Hands-on testing for most promising tools (sign up for free tiers, measure latency/accuracy/cost)
- Gap identification

## Part 3: Synthesis & Opportunity Map (Future)

- Build-vs-buy recommendations
- The case for what to build
- Architectural design if applicable
- Published writeup

---

## Existing Artifacts

- `agent-search-landscape.md` — Landscape overview with 7 categories, decision framework, head-to-head comparisons, orchestrator analysis
