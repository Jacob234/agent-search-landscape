# Collection Methodology

Standard operating procedure for data collection in the agent search landscape project.

## Provenance Tracking

Every profile uses three layers of provenance, applied simultaneously:

### Layer 1: Collection Metadata Block

Every profile starts with an HTML comment block recording:
- **Collected**: Date the profile was initially created
- **Collector**: Who/what did the research (e.g., `human`, `claude-opus-4-6`)
- **Session**: Git branch name or session identifier for traceability
- **GitHub Issue**: The issue number this profile addresses
- **Collection Method**: Reference to this methodology doc
- **Last Verified**: Date the profile was last checked for accuracy
- **Confidence**: Overall confidence level (see definitions below)

### Layer 2: Section-Level Sources

Each major section includes a `<!-- Sources: [...] -->` comment listing the URLs consulted for that section's data. This allows reviewers to trace which sources informed which parts of the profile.

### Layer 3: Inline Citations for Key Claims

Specific factual claims that are:
- **Quantitative** (pricing, benchmarks, star counts, dates)
- **Contested** (different sources disagree)
- **Surprising** (counter-intuitive or likely to be questioned)

...get inline citations: `claim text [source](url)`.

Claims that are common knowledge within the tool's own documentation (e.g., "Tavily is a search API") do not need inline citations.

### Source Log

Every profile ends with a complete source log table documenting every URL consulted, its type, and when it was accessed. This is the audit trail.

## Source Types and Authority

Sources are categorized by type, which implies a trust hierarchy:

| Tier | Source Type | Authority | Notes |
|------|------------ |-----------|-------|
| 1 | `official-docs` | Highest | Official documentation, API reference |
| 1 | `api-reference` | Highest | SDK docs, OpenAPI specs |
| 2 | `github-repo` | High | README, code, release notes |
| 2 | `blog-official` | High | Blog posts from the tool's own org |
| 3 | `benchmark` | Medium-High | Independent benchmarks (check methodology) |
| 3 | `news` | Medium | News articles, press releases |
| 4 | `blog-third-party` | Medium | Comparison articles, tutorials |
| 4 | `community` | Medium-Low | Forums, Discord, Stack Overflow |
| 5 | `inference` | Low | Derived from other data; flag for verification |

When sources conflict, prefer higher-tier sources. When same-tier sources conflict, note the discrepancy in the profile.

## Confidence Levels

Each profile gets an overall confidence rating:

| Level | Definition |
|-------|------------|
| **high** | Majority of data from tier 1-2 sources. API docs read directly. Pricing verified on official site. No significant gaps. |
| **medium** | Mix of tier 1-3 sources. Some sections rely on third-party reports. Minor gaps noted. |
| **low** | Significant reliance on tier 4-5 sources. Key sections have gaps or unverified claims. Needs follow-up. |

## Collection Procedure

For each tool:

### Step 1: Official Sources First
1. Visit the tool's official website
2. Read the main documentation / getting started guide
3. Read the API reference (endpoints, auth, rate limits, input/output)
4. Check the pricing page
5. Find the GitHub repository (if open-source)

### Step 2: Technical Deep Dive
6. Read the GitHub README thoroughly
7. Check recent releases / changelog for current state
8. Look at the GitHub issues for known limitations and failure modes
9. Read any architecture or "how it works" documentation

### Step 3: Integration and Ecosystem
10. Search for MCP server implementations
11. Check SDK availability (PyPI, npm)
12. Search for framework integrations (LangChain, LlamaIndex, CrewAI)

### Step 4: Performance and Benchmarks
13. Search for independent benchmarks and comparisons
14. Note any official performance claims and their methodology
15. Cross-reference with third-party test results

### Step 5: Fill Template
16. Fill each template section with collected data
17. Add section-level source comments
18. Add inline citations for quantitative/contested claims
19. Complete the source log table
20. Set confidence level
21. Note any gaps or items needing verification in Raw Notes

## Handling Data Freshness

Web data decays. To manage this:

- **Pricing**: Note the date checked. Pricing changes frequently.
- **GitHub stars**: Note "as of YYYY-MM-DD" — these grow daily.
- **Benchmarks**: Note publication date. Benchmarks older than 6 months may reflect outdated versions.
- **API surface**: Note version if available. APIs change with releases.
- **Rate limits / quotas**: These change with plan restructuring.

## Handling Conflicts

When two sources disagree:
1. Note both values in the profile
2. Cite both sources
3. Indicate which source you consider more authoritative and why
4. If unresolvable, flag in Raw Notes for hands-on verification in Part 2

## Git Workflow Integration

Provenance is reinforced by the git workflow:

- **Branch name**: `profile/<tool-name>` — ties to the specific tool
- **Commit messages**: Describe what data was collected and from where
- **PR description**: Summarizes research procedure, lists key sources, notes gaps
- **PR links issue**: `Closes #N` connects the profile to its tracking issue
- **Git history**: Provides immutable timestamp of when each data point was added

This means even if a profile is later edited, the original collection is preserved in git history.

## Quality Checklist

Before submitting a PR for a profile, verify:

- [ ] Collection metadata block is complete
- [ ] Every section has a source comment (even if "Sources: none found")
- [ ] Quantitative claims have inline citations
- [ ] Source log table is complete with all URLs consulted
- [ ] Confidence level is set and justified
- [ ] Any gaps or uncertainties are noted in Raw Notes
- [ ] Pricing date is recorded
- [ ] GitHub stars have "as of" date
- [ ] No section is left completely empty without explanation
