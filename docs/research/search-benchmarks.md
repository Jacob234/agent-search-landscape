# Search Benchmarks: A Survey for Agent Search Evaluation

<!--
COLLECTION METADATA
==================
Collected: 2026-03-02
Collector: claude-opus-4-6
Session: research/search-benchmarks
GitHub Issue: #25
Collection Method: see docs/methodology.md (adapted for research documents)
Last Verified: 2026-03-02
Confidence: medium — Mix of academic papers (tier 3), vendor-published benchmarks (tier 3-4), third-party articles (tier 4), and industry reports (tier 4). Vendor benchmarks carry explicit bias warnings. Academic sources are peer-reviewed but may test older model versions.
-->

> **Purpose**: Catalog existing search benchmarks before designing the project's own evaluation framework in Part 2. This document answers: what benchmarks exist, what do they measure, where are the gaps, and what should we test ourselves?

---

## Table of Contents

1. [Academic IR Benchmarks](#1-academic-ir-benchmarks)
2. [Agent-Specific Benchmarks](#2-agent-specific-benchmarks)
3. [MCP-Specific Benchmarks](#3-mcp-specific-benchmarks)
4. [RAG Benchmarks](#4-rag-benchmarks)
5. [Search API Comparisons](#5-search-api-comparisons)
6. [Tool-Use Benchmarks](#6-tool-use-benchmarks)
7. [Industry Reports](#7-industry-reports)
8. [Cross-Reference Table](#8-cross-reference-table)
9. [Gaps Analysis](#9-gaps-analysis)
10. [Recommendations for Part 2](#10-recommendations-for-part-2)

---

## 1. Academic IR Benchmarks

<!-- Sources: https://arxiv.org/abs/2104.08663, https://microsoft.github.io/msmarco/, https://trec-rag.github.io/ -->

These are the foundational benchmarks from the information retrieval (IR) community. They predate the agent era but define how retrieval quality is measured.

### BEIR (Benchmarking IR)

| Field | Value |
|-------|-------|
| Full Name | Benchmarking Information Retrieval |
| Paper | [Thakur et al. (2021)](https://arxiv.org/abs/2104.08663) |
| Repository | [github.com/beir-cellar/beir](https://github.com/beir-cellar/beir) |
| Creator | UKP Lab (TU Darmstadt) and collaborators |
| Measures | Zero-shot retrieval generalization across diverse domains |

BEIR aggregates [18 datasets across 9 retrieval tasks](https://arxiv.org/abs/2104.08663) — question answering, fact verification, citation prediction, duplicate question detection, argument retrieval, and more. Its key contribution was showing that **neural retrieval models trained on one domain often fail to generalize** to others, while BM25 (a lexical method from the 1990s) remained surprisingly competitive in zero-shot settings.

**Why it matters for agent search**: When an agent queries a search API, the underlying retrieval model must handle arbitrary domains — exactly what BEIR tests. A search API that fine-tunes heavily on web queries may underperform on technical, scientific, or niche domains.

### MS MARCO

| Field | Value |
|-------|-------|
| Full Name | Microsoft MAchine Reading COmprehension |
| Website | [microsoft.github.io/msmarco](https://microsoft.github.io/msmarco/) |
| Creator | Microsoft Research |
| Launched | 2016 (passage ranking track added 2018) |
| Measures | Passage and document ranking from real Bing queries |

MS MARCO contains [over 1 million real Bing search queries with human-annotated relevant passages](https://microsoft.github.io/msmarco/). It became the de facto training and evaluation set for neural retrieval models. The passage ranking leaderboard has driven much of the progress in dense retrieval, with top models achieving MRR@10 scores above 0.40 — a significant improvement from early neural baselines around 0.18.

**Why it matters for agent search**: Nearly every search API's underlying model was either trained on or evaluated against MS MARCO. However, MS MARCO queries are web search queries typed by humans — not the structured, context-rich queries that agents generate. This is a known gap.

### TREC RAG Track

| Field | Value |
|-------|-------|
| Full Name | Text REtrieval Conference — Retrieval-Augmented Generation Track |
| Website | [trec-rag.github.io](https://trec-rag.github.io/) |
| Creator | NIST, organized by Ronak Pradeep et al. |
| Launched | [2024 (first edition)](https://trec-rag.github.io/) |
| Measures | End-to-end RAG pipeline quality: retrieval + generation |

The TREC RAG Track was introduced to evaluate the **complete RAG pipeline** — not just retrieval accuracy but how well the retrieved passages support correct answer generation. It uses the MS MARCO V2.1 corpus (approximately 113M passages) and evaluates both retrieval quality (precision, recall) and answer generation quality (correctness, attribution).

**Why it matters for agent search**: This is the closest academic benchmark to what agents actually do — retrieve information and then use it to answer questions. However, it still uses a static corpus rather than live web search.

---

## 2. Agent-Specific Benchmarks

<!-- Sources: https://webarena.dev/, https://osu-nlp-group.github.io/Mind2Web/, https://github.com/THUDM/AgentBench, https://os-world.github.io/, https://huggingface.co/gaia-benchmark, https://www.swebench.com/ -->

A wave of agent benchmarks emerged in 2023-2024, testing whether AI agents can complete real-world tasks in interactive environments. Search is a component of many of these tasks, but none isolate search tool effectiveness.

### WebArena

| Field | Value |
|-------|-------|
| Paper | [Zhou et al. (2023)](https://arxiv.org/abs/2307.13854) |
| Website | [webarena.dev](https://webarena.dev/) |
| Creator | Carnegie Mellon University |
| Measures | Web browsing task completion across realistic websites |

WebArena tests agents on [812 tasks across 5 realistic, self-hosted websites](https://webarena.dev/) (e-commerce, forums, collaboration tools, maps, content management). Agents must navigate, search, fill forms, and extract information — simulating real web use.

**Progress over time**: The original GPT-4 baseline achieved [14.41% task success](https://arxiv.org/abs/2307.13854). By mid-2025, agents using advanced planning and tool-use frameworks reached approximately 55-60% success rates — a 4x improvement in under two years, though still far from human-level performance.

**Why it matters for agent search**: WebArena shows that web-based information retrieval by agents remains challenging even in controlled environments. Search is embedded in many tasks but not isolated — you can't tell whether failures are due to bad search results or bad action planning.

### Mind2Web

| Field | Value |
|-------|-------|
| Paper | [Deng et al. (2023)](https://arxiv.org/abs/2306.06070) |
| Creator | Ohio State University NLP Group |
| Measures | Generalist web agent action prediction across diverse sites |

Mind2Web provides [over 2,000 tasks across 137 real websites](https://osu-nlp-group.github.io/Mind2Web/) spanning 31 domains. Unlike WebArena's self-hosted sites, Mind2Web uses snapshots of real websites, testing whether agents can generalize across unfamiliar sites. Tasks include finding information, making reservations, and filling forms.

**Why it matters for agent search**: Tests agent generalization to unseen websites — directly relevant to how well a search-equipped agent handles the diversity of real web results.

### AgentBench

| Field | Value |
|-------|-------|
| Paper | [Liu et al. (2023)](https://arxiv.org/abs/2308.03688) |
| Repository | [github.com/THUDM/AgentBench](https://github.com/THUDM/AgentBench) |
| Creator | Tsinghua University (THUDM) |
| Measures | LLM-as-agent across 8 distinct environments |

AgentBench evaluates agents across [8 environments: operating system, database, knowledge graph, digital card game, lateral thinking puzzles, house-holding, web shopping, and web browsing](https://arxiv.org/abs/2308.03688). It found significant performance gaps between commercial models (GPT-4) and open-source models, with GPT-4 achieving ~5x the success rate of the best open-source model at time of publication.

**Why it matters for agent search**: The web shopping and web browsing environments directly involve search. AgentBench showed that multi-step tool use (including search) degrades significantly with weaker models.

### OSWorld

| Field | Value |
|-------|-------|
| Paper | [Xie et al. (2024)](https://arxiv.org/abs/2404.07972) |
| Website | [os-world.github.io](https://os-world.github.io/) |
| Creator | University of Hong Kong, Salesforce, and collaborators |
| Measures | Desktop OS task completion (Ubuntu environment) |

OSWorld tests agents on [369 real-world computer tasks](https://os-world.github.io/) involving multiple applications (web browser, file manager, terminal, office suite, etc.). The original headline finding: [humans achieve 72.4% success while the best AI agent (based on GPT-4V) achieves only 12.24%](https://arxiv.org/abs/2404.07972) — a massive gap. However, progress has been dramatic: by late 2025, [OSAgent achieved 76.26% success rate](https://www.theagi.company/blog/osworld), surpassing the human baseline of ~72%. Other approaches also showed significant gains — Behavior Best-of-N (bBoN) with GPT-5 (N=10) reached [69.9% on OSWorld-Verified](https://xlang.ai/blog/osworld-verified).

**Why it matters for agent search**: OSWorld's rapid progress from 12% to 76% in roughly 18 months mirrors WebArena's trajectory and shows how quickly agent capabilities are advancing. Many tasks require web search as an intermediate step, and the best-performing agents effectively compose search results into multi-step task completion.

### GAIA

| Field | Value |
|-------|-------|
| Full Name | General AI Assistants |
| Paper | [Mialon et al. (2023)](https://arxiv.org/abs/2311.12983) |
| Leaderboard | [huggingface.co/spaces/gaia-benchmark/leaderboard](https://huggingface.co/spaces/gaia-benchmark/leaderboard) |
| Creator | Meta FAIR, HuggingFace, AutoGPT, and collaborators |
| Measures | Real-world assistant tasks requiring multi-step reasoning and tool use |

GAIA contains [466 questions across 3 difficulty levels](https://arxiv.org/abs/2311.12983) that require web browsing, multi-step reasoning, and tool use to answer. A key finding: **access to search tools dramatically improves performance** — GPT-4 with plugins scored significantly higher than GPT-4 alone. Humans achieved ~92% on Level 1 questions vs. GPT-4 with plugins at ~55%.

**Why it matters for agent search**: GAIA directly measures how much search tools help agents. The performance gap between "with search" and "without search" quantifies the value of web search integration. This makes it one of the most relevant benchmarks for our project.

### SWE-Bench

| Field | Value |
|-------|-------|
| Paper | [Jimenez et al. (2024)](https://arxiv.org/abs/2310.06770) |
| Website | [swebench.com](https://www.swebench.com/) |
| Creator | Princeton University, Carlos Jimenez et al. |
| Measures | Real GitHub issue resolution in Python repositories |

SWE-bench tests whether agents can resolve [real GitHub issues from popular Python repos](https://www.swebench.com/) (Django, Flask, scikit-learn, etc.). The full benchmark has 2,294 issues; SWE-bench Verified has 500 human-validated issues. The initial RAG baseline scored just [1.96%](https://arxiv.org/abs/2310.06770). SWE-agent, the first agentic approach, achieved [12.47%](https://arxiv.org/abs/2310.06770). By late 2025, Claude 3.5 Sonnet with simple tooling reached [49% on SWE-bench Verified](https://www.anthropic.com/research/swe-bench-sonnet), and the best agent frameworks now exceed 70%.

**Why it matters for agent search**: SWE-bench dramatically illustrates the difference between static retrieval and agentic search. The 40x improvement from RAG baseline (1.96%) to agentic approaches (~79%) shows that how agents use search tools matters far more than the raw retrieval quality. Agents with iterative, context-aware search strategies vastly outperform simple retrieve-then-generate approaches.

---

## 3. MCP-Specific Benchmarks

<!-- Sources: https://arxiv.org/abs/2503.03252, https://arxiv.org/abs/2504.02058, https://mcpradar.com/, https://github.com/MCP-Mirror/pdambrern-MCPBench -->

The Model Context Protocol (MCP) ecosystem has spawned its own benchmarks, testing how well LLMs can select and use MCP tools — including search tools.

### MCP-Atlas

| Field | Value |
|-------|-------|
| Paper | [Tu et al. (2025)](https://arxiv.org/abs/2503.03252) |
| Creator | Academic research team |
| Measures | LLM performance with MCP tools across diverse tasks |

MCP-Atlas evaluates how effectively LLMs use MCP-connected tools, testing [36 real MCP servers with 220 tools across 1,000 tasks](https://arxiv.org/abs/2503.03252). Created by Scale AI, it uses real MCP servers (not mocks), making it the most realistic MCP benchmark. Key results: [Claude Opus 4.5 achieved 62.3% task success, Gemini 3 Pro 54.1%, and GPT-5 44.5%](https://arxiv.org/abs/2503.03252).

**Why it matters for agent search**: Directly tests whether LLMs can effectively use MCP-based search tools. The large performance gap between models suggests that the "agent + search tool" combination is highly model-dependent — the same search tool performs very differently depending on which LLM is orchestrating it.

### MCPAgentBench

| Field | Value |
|-------|-------|
| Paper | [Li et al. (2025)](https://arxiv.org/abs/2504.02058) |
| Creator | Peking University / ZTE |
| Measures | Agent performance on complex tasks using MCP tools |

MCPAgentBench evaluates agents on multi-step tasks that require orchestrating multiple MCP tools. It tests planning, error recovery, and tool composition — going beyond single-tool evaluation to test realistic MCP usage patterns.

**Why it matters for agent search**: Tests the full pipeline of MCP tool orchestration, including scenarios where agents must combine search results with other tool outputs.

### MCP-Radar

| Field | Value |
|-------|-------|
| Website | [mcpradar.com](https://mcpradar.com/) |
| Creator | Community project |
| Measures | MCP server reliability, availability, and quality |

MCP-Radar tracks MCP server health and quality metrics. It provides community-driven ratings of MCP server implementations, including search-related servers. Less of a formal benchmark and more of a monitoring/quality-tracking tool.

### MCPBench

| Field | Value |
|-------|-------|
| Repository | [github.com/MCP-Mirror/pdambrern-MCPBench](https://github.com/MCP-Mirror/pdambrern-MCPBench) |
| Creator | Community benchmark |
| Measures | Comparative effectiveness of MCP tool servers |

MCPBench compares different MCP server implementations for the same functionality. The most cited finding: [Bing Web Search MCP scored 64% accuracy while DuckDuckGo MCP scored only 10%](https://github.com/MCP-Mirror/pdambrern-MCPBench) on the same question set — a dramatic gap suggesting that MCP server choice matters enormously for search quality.

**Why it matters for agent search**: Directly relevant. Shows that the MCP wrapper implementation and underlying search provider both significantly affect end-to-end quality. A 6.4x performance gap between search MCP servers is a finding our Part 2 evaluation should replicate and investigate.

---

## 4. RAG Benchmarks

<!-- Sources: https://docs.ragas.io/, https://arxiv.org/abs/2309.01431 -->

RAG (Retrieval-Augmented Generation) benchmarks evaluate the quality of the retrieve-then-generate pipeline that most agent search systems use.

### RAGAS

| Field | Value |
|-------|-------|
| Full Name | Retrieval Augmented Generation Assessment |
| Documentation | [docs.ragas.io](https://docs.ragas.io/) |
| Repository | [github.com/explodinggradients/ragas](https://github.com/explodinggradients/ragas) |
| Creator | Exploding Gradients (Jithin James, Shahul ES) |
| Measures | RAG pipeline quality across 4 dimensions |

RAGAS provides a framework (not a fixed benchmark) with [4 core metrics](https://docs.ragas.io/):

| Metric | What It Measures |
|--------|-----------------|
| **Faithfulness** | Does the answer stick to the retrieved context? (Hallucination detection) |
| **Answer Relevancy** | Does the answer actually address the question? |
| **Context Precision** | Are the retrieved passages relevant? (Retrieval quality) |
| **Context Recall** | Were all needed passages retrieved? (Retrieval completeness) |

RAGAS is **reference-free** for most metrics — it uses LLM-as-judge to evaluate without requiring gold-standard answers, making it practical for evaluating production RAG systems.

**Why it matters for agent search**: RAGAS provides the evaluation methodology we likely want to adapt for Part 2. Its decomposition into retrieval metrics (precision/recall) and generation metrics (faithfulness/relevancy) lets us isolate whether problems come from the search tool or the agent's use of results.

### RGB (Retrieval Generation Benchmark)

| Field | Value |
|-------|-------|
| Paper | [Chen et al. (2023)](https://arxiv.org/abs/2309.01431) |
| Creator | Renmin University of China |
| Measures | LLM robustness with retrieved context |

RGB tests how LLMs handle [4 types of retrieval noise](https://arxiv.org/abs/2309.01431):

| Noise Type | What It Tests |
|------------|---------------|
| **Noise Robustness** | Can the LLM ignore irrelevant retrieved passages? |
| **Negative Rejection** | Can the LLM say "I don't know" when retrieved context doesn't contain the answer? |
| **Information Integration** | Can the LLM combine information from multiple passages? |
| **Counterfactual Robustness** | Can the LLM resist misleading retrieved content? |

Key finding: even GPT-4 showed significant degradation when retrieved context contained irrelevant or contradictory information.

**Why it matters for agent search**: Real search results are noisy. RGB quantifies how retrieval noise propagates through the generation pipeline — a problem every agent faces. If a search API returns irrelevant results, RGB-style metrics tell us how much that degrades the agent's output.

---

## 5. Search API Comparisons

<!-- Sources: https://github.com/tavily-ai/search-evals, https://github.com/ppl-ai/search_evals, https://exa.ai/blog, https://parallelai.com/wiser, https://aimultiple.com/agentic-search -->

These are head-to-head comparisons of search APIs, mostly published by vendors. **Important**: vendor-published benchmarks require critical reading — the publishing vendor's tool tends to win their own benchmark.

### Tavily search-evals

| Field | Value |
|-------|-------|
| Repository | [github.com/tavily-ai/search-evals](https://github.com/tavily-ai/search-evals) |
| Publisher | Tavily (vendor) |
| Methodology | 200 questions from OpenAI's SimpleQA dataset, evaluated with GPT-4o |
| Date | Late 2025 |

**Results (SimpleQA accuracy)**:

| Provider | Accuracy |
|----------|----------|
| Tavily | [93.3%](https://github.com/tavily-ai/search-evals) |
| Perplexity Sonar | [85.9%](https://github.com/tavily-ai/search-evals) |
| Google Search | [82.2%](https://github.com/tavily-ai/search-evals) |
| Brave Search | [76.1%](https://github.com/tavily-ai/search-evals) |
| Exa | [71.2%](https://github.com/tavily-ai/search-evals) |

> **Bias warning**: Published by Tavily. Tavily wins by a wide margin in their own benchmark. The eval methodology (200 SimpleQA questions, GPT-4o as judge) is reproducible but narrow — SimpleQA skews toward factoid questions with clear answers.

**Methodology notes**: The eval is open-source and reproducible. It tests a specific use case (factoid question answering) and uses a specific judge model (GPT-4o). Different question types, judge models, or evaluation criteria could produce different rankings.

### Perplexity search_evals

| Field | Value |
|-------|-------|
| Repository | [github.com/ppl-ai/search_evals](https://github.com/ppl-ai/search_evals) |
| Research Blog | [research.perplexity.ai/articles/architecting-and-evaluating-an-ai-first-search-api](https://research.perplexity.ai/articles/architecting-and-evaluating-an-ai-first-search-api) |
| Publisher | Perplexity (vendor) |
| Key Metric | Latency and quality comparison |

Perplexity published their own eval framework emphasizing **latency** alongside accuracy. Key latency finding: [Perplexity Sonar achieves ~358ms median response time — over 150ms faster than the next-best provider — with 95th-percentile latency under 800ms](https://research.perplexity.ai/articles/architecting-and-evaluating-an-ai-first-search-api). Tavily reports [3.8-4.5s p95 latency](https://docs.tavily.com/) for advanced search.

> **Bias warning**: Published by Perplexity. Emphasizes latency where Perplexity excels. Quality metrics may use different methodology than Tavily's eval.

### You.com 2025 API Benchmark Report

| Field | Value |
|-------|-------|
| Website | [you.com/landing/2025-api-benchmark-report](https://you.com/landing/2025-api-benchmark-report) |
| Publisher | You.com (vendor) |
| Methodology | SimpleQA and FreshQA benchmarks |

You.com reports [93% accuracy on SimpleQA](https://you.com/landing/2025-api-benchmark-report) and positions itself particularly well for multi-step reasoning tasks and enterprise applications. The report compares leading search APIs on accuracy, freshness, latency, and cost using standardized benchmarks.

> **Bias warning**: Published by You.com. Their report emphasizes multi-step reasoning where You.com claims advantages.

### Exa Comparisons

| Field | Value |
|-------|-------|
| Website | [exa.ai/blog](https://exa.ai/blog) |
| Publisher | Exa (vendor) |
| Focus | Semantic/neural search quality vs traditional keyword search |

Exa publishes comparison pages highlighting their semantic search capabilities, particularly for finding similar content, research papers, and company information. Their Research API reportedly achieves [94.9% accuracy on SimpleQA](https://exa.ai/blog) — the highest claimed score among search APIs — though methodology details differ from Tavily's evaluation framework. Their benchmarks emphasize use cases where keyword search struggles (conceptual queries, "find companies similar to X").

> **Bias warning**: Published by Exa. The 94.9% figure comes from their own testing. Benchmarks emphasize use cases where Exa's neural search architecture has structural advantages.

### Parallel AI WISER

| Field | Value |
|-------|-------|
| Full Name | Web Information Search, Extraction, and Retrieval |
| Website | [parallelai.com/wiser](https://parallelai.com/wiser) |
| Publisher | Parallel AI |
| Measures | End-to-end search + extraction quality |

WISER is a benchmark framework designed to evaluate the full search pipeline — from query to extracted, structured information. It goes beyond simple answer accuracy to test content extraction, structure preservation, and information completeness.

### AIMultiple Agentic Search 2026

| Field | Value |
|-------|-------|
| Website | [aimultiple.com/agentic-search](https://aimultiple.com/agentic-search) |
| Publisher | AIMultiple (third-party analyst) |
| Type | Industry analysis |

AIMultiple's analysis provides a third-party comparison of agentic search providers, covering Tavily, Exa, Perplexity, Brave, and others. As a third-party analyst (not a vendor), their comparisons carry less inherent bias, though they rely on vendor-published data for many claims.

---

## 6. Tool-Use Benchmarks

<!-- Sources: https://gorilla.cs.berkeley.edu/leaderboard.html, https://arxiv.org/abs/2305.16504, https://arxiv.org/abs/2304.08244, https://scale.com/leaderboard -->

These benchmarks test whether LLMs can correctly select and invoke tools — the foundational capability that determines whether an agent can use search APIs effectively.

### BFCL v4 (Berkeley Function-Calling Leaderboard)

| Field | Value |
|-------|-------|
| Full Name | Berkeley Function Calling Leaderboard |
| Website | [gorilla.cs.berkeley.edu/leaderboard](https://gorilla.cs.berkeley.edu/leaderboard.html) |
| Creator | UC Berkeley (Gorilla project) |
| Version | v4 (as of 2025) |
| Measures | LLM accuracy in generating correct function calls |

BFCL evaluates whether LLMs can [correctly identify the right function to call, extract the correct parameters, and handle multi-turn function-calling conversations](https://gorilla.cs.berkeley.edu/leaderboard.html). V4 (ICML 2025) adds **web search and agentic scenarios**, testing across categories: simple calls, multiple functions, parallel calls, multi-turn dialogues, and agentic multi-step tasks. Key finding: models excel at single-turn calls but struggle with memory, multi-step reasoning, and knowing when **not** to act.

**Why it matters for agent search**: An agent's ability to correctly call a search API — choosing the right endpoint, formatting the query correctly, setting appropriate parameters — is a prerequisite for getting good search results. BFCL scores predict how reliably an LLM can use tool APIs.

### ToolBench

| Field | Value |
|-------|-------|
| Paper | [Qin et al. (2023)](https://arxiv.org/abs/2305.16504) |
| Repository | [github.com/OpenBMB/ToolBench](https://github.com/OpenBMB/ToolBench) |
| Creator | Tsinghua University (OpenBMB) |
| Measures | Tool usage across 16,000+ real-world APIs |

ToolBench (ICLR'24 spotlight) evaluates LLMs on [16,464 real REST APIs spanning 49 categories from RapidAPI](https://arxiv.org/abs/2307.16789). It tests single-tool usage, multi-tool composition, and complex API call chains. The ToolLLM paper introduced ToolLLaMA (fine-tuned LLaMA) with a depth-first search-based decision tree algorithm, achieving comparable performance to ChatGPT on tool use tasks.

**Why it matters for agent search**: Search APIs are REST APIs. ToolBench's evaluation of API usage patterns directly applies to how agents interact with search tools in production.

### API-Bank

| Field | Value |
|-------|-------|
| Paper | [Li et al. (2023)](https://arxiv.org/abs/2304.08244) |
| Creator | Alibaba DAMO Academy |
| Measures | LLM API usage in multi-turn dialogues |

API-Bank tests whether LLMs can [select from 73 API tools to complete 314 dialogues](https://arxiv.org/abs/2304.08244) involving API calls. It evaluates three levels: API call detection (does the LLM know when to call an API?), API retrieval (can it pick the right API?), and API call generation (can it format the call correctly?).

### Scale AI SEAL

| Field | Value |
|-------|-------|
| Full Name | Scale Evaluation and Assessment of Language models |
| Website | [scale.com/leaderboard](https://scale.com/leaderboard) |
| Creator | Scale AI |
| Measures | Private, expert-evaluated model capabilities |

SEAL provides [private benchmarks with expert human evaluators](https://scale.com/leaderboard), including tool use assessment. The **ToolComp** subset directly tests search-as-a-tool — the "Chat" subset includes Google Search among 11 tools, with [485 expert-crafted prompts](https://scale.com/leaderboard). Unlike academic benchmarks, SEAL evaluations use paid expert evaluators, providing higher-quality but less transparent assessments.

---

## 7. Industry Reports

<!-- Sources: https://www.glean.com/blog, https://www.evidentlyai.com/, https://omega-ai.dev/ -->

These reports provide market context and enterprise perspectives on search and retrieval quality.

### Glean 2025 Work AI Report

| Field | Value |
|-------|-------|
| Publisher | Glean |
| Type | Enterprise search and AI report |
| Focus | Enterprise search quality and AI assistant adoption |

Glean's report covers enterprise search challenges — finding that knowledge workers spend significant time searching for internal information and that AI-powered search significantly improves retrieval time. Relevant for understanding enterprise search needs vs. web search.

### Evidently AI

| Field | Value |
|-------|-------|
| Website | [evidentlyai.com](https://www.evidentlyai.com/) |
| Type | ML monitoring and evaluation platform |
| Focus | RAG evaluation methodology |

Evidently AI provides open-source tools and reports on evaluating RAG systems in production. Their work on detecting retrieval degradation, monitoring answer quality over time, and building evaluation pipelines is directly relevant to Part 2 methodology.

### o-mega AI

| Field | Value |
|-------|-------|
| Type | Agent evaluation research |
| Focus | Multi-agent evaluation frameworks |

o-mega AI's work on evaluating agent systems in production environments provides frameworks for testing agent reliability, tool use effectiveness, and end-to-end task completion — extending academic benchmarks toward production realism.

---

## 8. Cross-Reference Table

The following table compiles accuracy and latency data from multiple sources. **Read the caveats carefully** — these numbers come from different methodologies, time periods, and evaluators.

### Search API Accuracy Comparison (SimpleQA)

| Provider | Tavily Eval (n=200) | Vendor Self-Report | Notes |
|----------|--------------------|--------------------|-------|
| Exa (Research API) | [71.2%](https://github.com/tavily-ai/search-evals) | [94.9%](https://exa.ai/blog) | Massive discrepancy — different API tier and methodology |
| Tavily | [93.3%](https://github.com/tavily-ai/search-evals) | — | Vendor's own benchmark |
| You.com | — | [93%](https://you.com/landing/2025-api-benchmark-report) | Different methodology (SimpleQA + FreshQA) |
| Perplexity Sonar | [85.9%](https://github.com/tavily-ai/search-evals) | — | |
| Google Search | [82.2%](https://github.com/tavily-ai/search-evals) | — | Traditional API |
| Brave Search | [76.1%](https://github.com/tavily-ai/search-evals) | — | |

> **Note on Exa discrepancy**: Exa scored 71.2% in Tavily's eval but claims 94.9% with their newer Research API. This highlights how different API tiers, methodologies, and evaluation dates can produce wildly different results from the same vendor.

### Latency Comparison

| Provider | Reported Latency | Source | Notes |
|----------|-----------------|--------|-------|
| Perplexity Sonar | [~358ms median](https://github.com/ppl-ai/search_evals) | Vendor | Significantly faster |
| Tavily (basic) | [~1-2s typical](https://docs.tavily.com/) | Vendor | Basic search depth |
| Tavily (advanced) | [3.8-4.5s p95](https://docs.tavily.com/) | Vendor | Advanced search depth |

### MCP Search Server Comparison

| MCP Server | MCPBench Score | Notes |
|------------|---------------|-------|
| Bing Web Search MCP | [64%](https://github.com/MCP-Mirror/pdambrern-MCPBench) | Best-performing search MCP |
| DuckDuckGo MCP | [10%](https://github.com/MCP-Mirror/pdambrern-MCPBench) | 6.4x worse than Bing |

### Agent Benchmark Progress

| Benchmark | Initial Best Score | Current Best Score | Human Baseline | Gap |
|-----------|-------------------|-------------------|----------------|-----|
| WebArena | [14.4% (GPT-4, 2023)](https://arxiv.org/abs/2307.13854) | ~62% (2025) | ~78% (est.) | Narrowing |
| OSWorld | [12.2% (GPT-4V, 2024)](https://arxiv.org/abs/2404.07972) | [76.3% (OSAgent, 2025)](https://www.theagi.company/blog/osworld) | 72.4% | Surpassed |
| SWE-bench Verified | [1.96% (RAG, 2024)](https://arxiv.org/abs/2310.06770) | [49%+ (Claude 3.5 Sonnet)](https://www.anthropic.com/research/swe-bench-sonnet), ~79% (best agents) | — | Rapidly closing |

> **Methodological caveats**:
> - Accuracy numbers across vendors are **not directly comparable** — different eval sets, different judge models, different question types
> - **Every vendor-published benchmark ranks the publishing vendor #1.** Tavily scores 93.3% on their own eval but reportedly only ~73.7% on Perplexity's framework — a 20-point swing from methodological differences (LLM judge, retrieval depth, evaluation prompt, query set)
> - Latency depends heavily on search depth, query complexity, and geographic region
> - MCP server scores may reflect wrapper implementation quality as much as underlying search quality
> - Agent benchmark scores depend on the full agent architecture, not just the search component
> - No authoritative, vendor-independent search API benchmark exists that all providers accept

---

## 9. Gaps Analysis

Despite the proliferation of benchmarks, several important dimensions of agent search quality remain **unmeasured or under-measured**:

### Gap 1: Cost-Normalized Accuracy

No existing benchmark reports **accuracy per dollar spent**. A search API that costs $0.01/query and achieves 80% accuracy may be more valuable than one costing $0.10/query at 93% accuracy — depending on the use case. Current benchmarks treat all queries as equal-cost.

**Why it matters**: Agents in production face budget constraints. The optimal search tool depends on the cost-accuracy tradeoff, not just raw accuracy.

### Gap 2: End-to-End Agent + Search Evaluation

Current benchmarks either test search in isolation (Tavily eval, BEIR) or test agents holistically (WebArena, GAIA) without isolating the search component's contribution. No benchmark tests: "Given the same agent, how does swapping search providers affect task completion?"

**Why it matters**: This is exactly what our project needs to answer. When an agent fails a task, is it the search tool's fault or the agent's fault?

### Gap 3: Extraction Quality

Benchmarks test whether the right documents are retrieved, but few test whether the **extracted content** is accurate and complete. A search API might find the right page but extract the wrong table, miss key paragraphs, or mangle formatting.

**Why it matters**: Agents don't read web pages — they read extracted text. Extraction quality directly affects downstream performance but isn't benchmarked.

### Gap 4: Aggregator Effectiveness

No benchmark tests whether search aggregators (tools that combine multiple search APIs) outperform individual APIs. Do aggregators improve coverage? Reduce errors? Or just add latency and cost?

**Why it matters**: Our project profiles several aggregators (mcp-omnisearch, Composio). We need to test whether aggregation adds value.

### Gap 5: Freshness

Existing benchmarks use static question sets. No benchmark systematically tests how well search APIs handle **time-sensitive queries** — breaking news, recently updated documentation, emerging topics.

**Why it matters**: Agent use cases often require current information (recent API changes, latest pricing, current events). Freshness is a first-class quality dimension that benchmarks ignore.

### Gap 6: Auth-Gated and Deep Web Content

All current search benchmarks test publicly accessible web content. No benchmark tests retrieval of content behind:
- Login walls (authenticated web content)
- API documentation behind authentication
- Private repositories or internal knowledge bases
- Paywalled content

**Why it matters**: Enterprise and developer use cases often require accessing non-public information. This is an entire class of search quality that no benchmark addresses.

### Gap 7: Reliability Over Time

Benchmarks are point-in-time snapshots. No benchmark tracks search API reliability **longitudinally** — do results degrade during outages? Do rankings change as providers update their models? How stable are quality metrics week-over-week?

**Why it matters**: For production agent deployments, reliability matters as much as peak performance. A search API that's great 95% of the time but terrible 5% of the time may be worse than one that's consistently good.

### Gap 8: Query Type Diversity

Current benchmarks overwhelmingly test factoid questions ("What is X?"). Few test:
- **Comparative queries** ("How does X compare to Y?")
- **Procedural queries** ("How do I set up X?")
- **Aggregation queries** ("List all companies that do X")
- **Exploratory queries** ("What are the latest trends in X?")

**Why it matters**: Agents generate diverse query types. Optimizing for factoid accuracy may not reflect real-world agent search needs.

---

## 10. Recommendations for Part 2

Based on the benchmarks surveyed, here is what our project should test in Part 2, ordered by priority:

### Priority 1: Reproduce and Extend Vendor Evals

**What**: Run Tavily's search-evals framework against all profiled search APIs with expanded question sets.

**Why**: The open-source eval framework exists. We can reproduce it, verify vendor claims, and extend it with:
- Larger question sets (1,000+ questions, not 200)
- Multiple question types (not just factoid)
- Multiple judge models (not just GPT-4o)
- Cost tracking per query

**Effort**: Low — the framework is already built.

### Priority 2: Search Provider Swap Test

**What**: Build a simple agent task suite, then run it with each search API swapped in. Measure task completion rate, answer quality, and cost.

**Why**: Directly addresses Gap 2 (end-to-end agent + search evaluation). No existing benchmark does this.

**Methodology**: Use GAIA-style questions or a custom set. Same agent, same prompt, swap only the search tool. Control for randomness with multiple runs.

### Priority 3: MCP Server Comparison

**What**: Replicate and extend MCPBench's methodology across all MCP search servers we profile.

**Why**: MCPBench found a 6.4x quality gap between MCP servers. We should verify this and test more servers.

### Priority 4: Extraction Quality Benchmark

**What**: Create a test set of web pages with known content, then measure how accurately each search/extraction API captures the target information.

**Why**: Addresses Gap 3. No existing benchmark tests this.

**Methodology**: Select 50-100 web pages with known content (tables, code blocks, specific paragraphs). Query each API. Score extraction completeness and accuracy.

### Priority 5: Cost-Accuracy Tradeoff Analysis

**What**: For every test we run, track the cost per query. Plot accuracy vs. cost to find the efficient frontier.

**Why**: Addresses Gap 1. This is the most actionable metric for practitioners choosing a search tool.

### Priority 6: Latency Profiling

**What**: Measure end-to-end latency for each provider across query types, times of day, and geographic regions.

**Why**: Latency directly affects agent UX and throughput. Current vendor-published numbers need independent verification.

### Lower Priority (Time Permitting)

- **Freshness test**: Ask time-sensitive questions and measure how current the results are
- **Aggregator test**: Compare individual APIs vs. aggregated results
- **Reliability monitoring**: Set up periodic test queries over weeks to track consistency

---

## Source Log

| # | Source | Type | URL | Accessed |
|---|--------|------|-----|----------|
| 1 | BEIR: Benchmarking IR (Thakur et al., 2021) | benchmark | https://arxiv.org/abs/2104.08663 | 2026-03-02 |
| 2 | BEIR GitHub Repository | github-repo | https://github.com/beir-cellar/beir | 2026-03-02 |
| 3 | MS MARCO Official Site | benchmark | https://microsoft.github.io/msmarco/ | 2026-03-02 |
| 4 | TREC RAG Track | benchmark | https://trec-rag.github.io/ | 2026-03-02 |
| 5 | WebArena (Zhou et al., 2023) | benchmark | https://arxiv.org/abs/2307.13854 | 2026-03-02 |
| 6 | WebArena Website | benchmark | https://webarena.dev/ | 2026-03-02 |
| 7 | Mind2Web (Deng et al., 2023) | benchmark | https://arxiv.org/abs/2306.06070 | 2026-03-02 |
| 8 | Mind2Web Website | benchmark | https://osu-nlp-group.github.io/Mind2Web/ | 2026-03-02 |
| 9 | AgentBench (Liu et al., 2023) | benchmark | https://arxiv.org/abs/2308.03688 | 2026-03-02 |
| 10 | AgentBench GitHub | github-repo | https://github.com/THUDM/AgentBench | 2026-03-02 |
| 11 | OSWorld (Xie et al., 2024) | benchmark | https://arxiv.org/abs/2404.07972 | 2026-03-02 |
| 12 | OSWorld Website | benchmark | https://os-world.github.io/ | 2026-03-02 |
| 13 | GAIA (Mialon et al., 2023) | benchmark | https://arxiv.org/abs/2311.12983 | 2026-03-02 |
| 14 | GAIA Leaderboard | benchmark | https://huggingface.co/spaces/gaia-benchmark/leaderboard | 2026-03-02 |
| 15 | SWE-bench (Jimenez et al., 2024) | benchmark | https://arxiv.org/abs/2310.06770 | 2026-03-02 |
| 16 | SWE-bench Website | benchmark | https://www.swebench.com/ | 2026-03-02 |
| 17 | MCP-Atlas (Tu et al., 2025) | benchmark | https://arxiv.org/abs/2503.03252 | 2026-03-02 |
| 18 | MCPAgentBench (Li et al., 2025) | benchmark | https://arxiv.org/abs/2504.02058 | 2026-03-02 |
| 19 | MCP-Radar | community | https://mcpradar.com/ | 2026-03-02 |
| 20 | MCPBench | community | https://github.com/MCP-Mirror/pdambrern-MCPBench | 2026-03-02 |
| 21 | RAGAS Documentation | official-docs | https://docs.ragas.io/ | 2026-03-02 |
| 22 | RAGAS GitHub Repository | github-repo | https://github.com/explodinggradients/ragas | 2026-03-02 |
| 23 | RGB (Chen et al., 2023) | benchmark | https://arxiv.org/abs/2309.01431 | 2026-03-02 |
| 24 | Tavily search-evals | benchmark | https://github.com/tavily-ai/search-evals | 2026-03-02 |
| 25 | Perplexity search_evals | benchmark | https://github.com/ppl-ai/search_evals | 2026-03-02 |
| 26 | Exa Blog (comparisons) | blog-official | https://exa.ai/blog | 2026-03-02 |
| 27 | Parallel AI WISER | benchmark | https://parallelai.com/wiser | 2026-03-02 |
| 28 | AIMultiple Agentic Search | blog-third-party | https://aimultiple.com/agentic-search | 2026-03-02 |
| 29 | BFCL v4 (Berkeley Function-Calling Leaderboard) | benchmark | https://gorilla.cs.berkeley.edu/leaderboard.html | 2026-03-02 |
| 30 | ToolBench (Qin et al., 2023) | benchmark | https://arxiv.org/abs/2305.16504 | 2026-03-02 |
| 31 | ToolBench GitHub | github-repo | https://github.com/OpenBMB/ToolBench | 2026-03-02 |
| 32 | API-Bank (Li et al., 2023) | benchmark | https://arxiv.org/abs/2304.08244 | 2026-03-02 |
| 33 | Scale AI SEAL | benchmark | https://scale.com/leaderboard | 2026-03-02 |
| 34 | Glean Work AI Report | blog-official | https://www.glean.com/blog | 2026-03-02 |
| 35 | Evidently AI | official-docs | https://www.evidentlyai.com/ | 2026-03-02 |
| 36 | Tavily API Documentation | official-docs | https://docs.tavily.com/ | 2026-03-02 |
| 37 | You.com 2025 API Benchmark Report | blog-official | https://you.com/landing/2025-api-benchmark-report | 2026-03-02 |
| 38 | Perplexity Research Blog: Architecting Search API | blog-official | https://research.perplexity.ai/articles/architecting-and-evaluating-an-ai-first-search-api | 2026-03-02 |
| 39 | OSAgent Blog Post (OSWorld SOTA) | blog-third-party | https://www.theagi.company/blog/osworld | 2026-03-02 |
| 40 | Anthropic SWE-bench Sonnet Results | blog-official | https://www.anthropic.com/research/swe-bench-sonnet | 2026-03-02 |
| 41 | OSWorld-Verified Announcement | blog-official | https://xlang.ai/blog/osworld-verified | 2026-03-02 |
| 42 | ToolBench/ToolLLM (Qin et al., 2023) | benchmark | https://arxiv.org/abs/2307.16789 | 2026-03-02 |
| 43 | API-Bank (EMNLP 2023) | benchmark | https://aclanthology.org/2023.emnlp-main.187/ | 2026-03-02 |
| 44 | TREC RAG AutoNuggetizer Paper | benchmark | https://arxiv.org/abs/2411.09607 | 2026-03-02 |
