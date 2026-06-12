---
name: arxiv-autonomous-search
description: Fully autonomous arXiv literature search that discovers the research topic from project context and produces a complete literature review without user interaction.
web_bundle: true
---

# Autonomous arXiv Literature Search

**Goal:** Autonomously discover the research topic from project context, construct arXiv API queries, execute searches, analyze papers, and produce a complete literature review document - all without requiring user input at intermediate steps.

**Key Difference from Interactive Workflow:** This workflow does NOT pause for `[C] Continue` confirmation. It runs end-to-end autonomously, producing the final document in one continuous execution.

## AUTONOMOUS EXECUTION RULES

- 🤖 **FULLY AUTONOMOUS**: Do NOT stop for user confirmation at any point. Execute all phases sequentially without pausing.
- 🛑 **NO [C] GATES**: Skip all `[C] Continue` prompts. Proceed immediately from one phase to the next.
- 📄 **SINGLE OUTPUT**: Produce the complete literature review document at the end.
- ✅ **SELF-DIRECTED**: Make all decisions about topic scope, query construction, and paper relevance autonomously based on project context.
- 🔍 **REAL API CALLS**: Execute actual arXiv API calls. Do not simulate or fabricate results.
- ⏱️ **RATE LIMITING**: Still respect the 3-second minimum between arXiv API calls.
- 🚫 **ANTI-HALLUCINATION**: Only cite papers that appear in actual arXiv API results. Never fabricate arXiv IDs, titles, or authors.
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

## INITIALIZATION

### Configuration Loading

Load config from `{project-root}/_bmad/bmm/config.yaml` and resolve:

- `project_name`, `output_folder`, `research_artifacts`, `user_name`
- `communication_language`, `document_output_language`, `user_skill_level`
- `date` as a system-generated value

### Reference Loading

Load the arXiv API reference: `{installed_path}/arxiv-api-reference.md`

### Paths

- `installed_path` = `{project-root}/_bmad/bmm/workflows/1-research/arxiv-search`
- `template_path` = `{installed_path}/arxiv-research.template.md`
- `default_output_file` = `{research_artifacts}/research/arxiv-autonomous-{{search_topic}}-{{date}}.md`

## PREREQUISITE

**Web fetch capability required.** The arXiv API is accessed via HTTP GET requests to `http://export.arxiv.org/api/query`. If web fetch is unavailable, abort and tell the user.

## arXiv API Details

- **Endpoint**: `http://export.arxiv.org/api/query`
- **Method**: HTTP GET with URL query parameters
- **Authentication**: None required
- **Rate Limit**: 3 seconds minimum between consecutive calls
- **Response Format**: Atom 1.0 XML
- **Max Results Per Request**: 2000 (use 50 for focused queries)

## AUTONOMOUS EXECUTION SEQUENCE

Execute all five phases below **sequentially and without stopping**. Do not pause for user input between phases.

---

## PHASE 1: TOPIC DISCOVERY AND SCOPE DEFINITION

### 1.1 Discover Research Topic from Project Context

Scan available project context to determine the research topic autonomously. Check these sources in order:

1. **Research Brief** - Look for files in `{research_artifacts}/` matching `*research-brief*` or `*product-brief*`
2. **Strategy Design** - Look for files matching `*strategy-design*` or `*strategy-spec*`
3. **Project Context** - Check `{project-root}/_bmad/bmm/data/project-context-template.md` for filled-in context
4. **Recent Research** - Look for any recent research documents in `{research_artifacts}/research/`
5. **Project Name** - Fall back to `{{project_name}}` from config as the base topic

**From the available context, extract:**

- **search_topic**: The core research subject (e.g., "momentum factor models", "deep learning portfolio optimization")
- **search_goals**: What the research aims to achieve (e.g., "identify state-of-the-art methods for cross-sectional momentum")
- **target_categories**: 3-5 most relevant arXiv categories from:

| Category | Focus |
|----------|-------|
| `q-fin.PM` | Portfolio Management |
| `q-fin.ST` | Statistical Finance |
| `q-fin.TR` | Trading and Market Microstructure |
| `q-fin.RM` | Risk Management |
| `q-fin.CP` | Computational Finance |
| `q-fin.MF` | Mathematical Finance |
| `q-fin.PR` | Pricing of Securities |
| `q-fin.EC` | Economics |
| `q-fin.GN` | General Finance |
| `stat.ML` | Machine Learning (Stats) |
| `stat.ME` | Methodology (Stats) |
| `cs.LG` | Machine Learning (CS) |
| `cs.AI` | Artificial Intelligence |
| `cs.CE` | Computational Engineering |
| `econ.EM` | Econometrics |
| `math.OC` | Optimization and Control |

- **search_keywords**: 5-10 refined keywords and phrases

**If no project context is available:** Use `{{project_name}}` combined with general quantitative finance as the topic. Set search_goals to "Comprehensive literature survey on [project_name] within quantitative finance."

### 1.2 Construct Search Queries

Build exactly 5 search queries covering different dimensions:

**Query 1 - Primary Topic (Broad):**
```
http://export.arxiv.org/api/query?search_query=all:{{primary_keywords}}+AND+%28cat:{{cat1}}+OR+cat:{{cat2}}+OR+cat:{{cat3}}%29&sortBy=relevance&max_results=50
```

**Query 2 - Title-Focused (Precision):**
```
http://export.arxiv.org/api/query?search_query=ti:{{core_phrase}}+AND+%28cat:{{cat1}}+OR+cat:{{cat2}}%29&sortBy=submittedDate&sortOrder=descending&max_results=30
```

**Query 3 - Methodology-Focused:**
```
http://export.arxiv.org/api/query?search_query=all:{{method_keywords}}+AND+%28cat:{{method_cats}}%29&sortBy=relevance&max_results=30
```

**Query 4 - Recent Advances (Last 2 Years):**
```
http://export.arxiv.org/api/query?search_query=all:{{topic_keywords}}+AND+%28cat:{{cat1}}+OR+cat:{{cat2}}%29&sortBy=submittedDate&sortOrder=descending&max_results=25
```

**Query 5 - Cross-Disciplinary:**
```
http://export.arxiv.org/api/query?search_query=all:{{topic_keywords}}+AND+%28cat:stat.ML+OR+cat:cs.LG+OR+cat:cs.AI%29&sortBy=relevance&max_results=25
```

### 1.3 Initialize Output Document

Create the output file from template at `{default_output_file}` with frontmatter:

```yaml
stepsCompleted: [1]
search_topic: '{{search_topic}}'
search_goals: '{{search_goals}}'
target_categories: [{{categories}}]
autonomous: true
date: '{{date}}'
```

Append scope summary to document. **Do not pause. Proceed immediately to Phase 2.**

---

## PHASE 2: EXECUTE arXiv API SEARCHES

### 2.1 Execute All 5 Queries

Execute each query constructed in Phase 1 sequentially. **Wait at least 3 seconds between each API call.**

For each query:

1. Fetch the URL using web fetch
2. Parse the Atom XML response
3. Extract from each `<entry>`:
   - **arXiv ID** (from `<id>` URL, e.g., `2103.00496v2`)
   - **Title** (from `<title>`)
   - **Authors** (from `<author><name>`, list all)
   - **Abstract** (from `<summary>`)
   - **Published Date** (from `<published>`)
   - **Updated Date** (from `<updated>`)
   - **Primary Category** (from `<arxiv:primary_category term>`)
   - **All Categories** (from `<category term>`)
   - **PDF Link** (from `<link rel="related" title="pdf" href>`)
   - **Abstract Link** (from `<link rel="alternate" href>`)
   - **DOI** (from `<arxiv:doi>` if present)
   - **Journal Reference** (from `<arxiv:journal_ref>` if present)
   - **Comment** (from `<arxiv:comment>` if present)
4. Record the total results count from `<opensearch:totalResults>`

### 2.2 Deduplicate Results

Track all arXiv IDs across queries. Remove duplicate entries (same arXiv ID). Keep the first occurrence.

### 2.3 Record Search Results

Note the search execution summary (total per query, unique papers found, categories represented).

Update frontmatter: `stepsCompleted: [1, 2]`, `papers_found: [count]`

**Do not pause. Proceed immediately to Phase 3.**

---

## PHASE 3: ANALYZE AND SCORE PAPERS

### 3.1 Score Every Unique Paper

For each unique paper discovered, assign a relevance score:

| Score | Label | Criteria |
|-------|-------|----------|
| 5 | Essential | Directly addresses core research topic, foundational or seminal work |
| 4 | Highly Relevant | Closely related methodology or findings applicable to research goals |
| 3 | Relevant | Related work with applicable methods or context |
| 2 | Tangentially Related | Useful for background or peripheral understanding |
| 1 | Low Relevance | Marginally related, minimal direct application |

**For each paper, determine:**

- **Abstract Analysis**: 1-2 sentence summary of the paper's contribution
- **Methodology**: Key methods used
- **Key Findings**: Most important results
- **Relevance Justification**: Why it received its score relative to the search topic
- **Relevance Score**: 1-5

### 3.2 Identify Thematic Clusters

Group the scored papers (score 3+) into thematic clusters based on:

- Research methodology (statistical, ML, analytical, simulation)
- Problem domain (pricing, portfolio, risk, execution, factor)
- Application approach (theoretical, empirical, hybrid)

Name each cluster and identify cross-cluster connections.

### 3.3 Map the Methodology Landscape

Create a methodology summary across all analyzed papers:

| Method | Papers Using It | Application Context |
|--------|----------------|---------------------|
| [method] | [arXiv IDs] | [how it's applied] |

Update frontmatter: `stepsCompleted: [1, 2, 3]`, `papers_analyzed: [count]`

**Do not pause. Proceed immediately to Phase 4.**

---

## PHASE 4: SYNTHESIZE AND PRODUCE LITERATURE REVIEW

### 4.1 Generate Executive Summary

Summarize:
- Breadth and depth of the literature found
- Most impactful papers and their contributions
- Dominant themes and methodological approaches
- Most significant research gaps
- High-level recommendations

### 4.2 Synthesize Thematic Findings

For each thematic cluster from Phase 3:

- **State of the Art**: Current best understanding/approach
- **Evolution**: How research in this theme has progressed
- **Consensus vs. Debate**: Where researchers agree and disagree
- **Key Contributors**: Most active research groups
- **Practical Implications**: What it means for the research goals

### 4.3 Assess Methodology Landscape

- **Proven Methods**: Methods with strong empirical support
- **Emerging Methods**: Newer approaches showing promise
- **Methodological Concerns**: Common pitfalls found in the literature
- **Best Practices**: Recommended approaches based on literature consensus

### 4.4 Identify Research Gaps

- **Unexplored Areas**: Topics not well-covered
- **Contradictions**: Where papers disagree
- **Extension Opportunities**: How existing work could be improved
- **Integration Opportunities**: How different approaches could be combined

### 4.5 Generate Recommendations

- **Recommended Reading Order**: Priority papers with rationale
- **Methodology Recommendations**: Best approaches to adopt
- **Research Direction**: Suggested questions based on gaps
- **Follow-Up Searches**: Additional arXiv queries to explore

---

## PHASE 5: PRODUCE FINAL DOCUMENT

### 5.1 Write Complete Literature Review

Append the complete literature review to the output document using this structure:

```markdown
# arXiv Literature Review: {{search_topic}}

## Executive Summary

[2-3 paragraphs synthesizing the literature landscape]

**Literature Landscape at a Glance:**

- **Papers Discovered:** [count]
- **Papers Analyzed:** [count]
- **Essential Papers (Score 5):** [count]
- **Thematic Clusters:** [count]
- **Date Range:** [oldest] to [newest]
- **Primary Categories:** [list]

**Key Findings:**

- [Finding 1]
- [Finding 2]
- [Finding 3]
- [Finding 4]

**Top Recommendations:**

- [Recommendation 1]
- [Recommendation 2]
- [Recommendation 3]

## Table of Contents

1. Research Motivation and Scope
2. Search Methodology and Coverage
3. Literature Landscape Overview
4. Thematic Synthesis
5. Methodology Assessment and Best Practices
6. Research Gaps and Opportunities
7. Recommendations and Suggested Reading
8. Complete Reference List
9. Appendices

## 1. Research Motivation and Scope

### Research Objectives

**Topic:** {{search_topic}}
**Goals:** {{search_goals}}
**Discovery Method:** Autonomous extraction from project context

### Scope

- **Target Categories:** [list with descriptions]
- **Search Dimensions:** 5 (primary, title-focused, methodology, recent, cross-disciplinary)
- **Keyword Coverage:** [list]

## 2. Search Methodology and Coverage

### Systematic Search Approach

- **API Used:** arXiv API (http://export.arxiv.org/api/query)
- **Queries Executed:** 5
- **Total Results Retrieved:** [count before dedup]
- **Unique Papers:** [count after dedup]
- **Execution Mode:** Fully autonomous

### Query Log

| # | Dimension | Query Summary | Results |
|---|-----------|---------------|---------|
| 1 | Primary Topic | [summary] | [count] |
| 2 | Title-Focused | [summary] | [count] |
| 3 | Methodology | [summary] | [count] |
| 4 | Recent Advances | [summary] | [count] |
| 5 | Cross-Disciplinary | [summary] | [count] |

### Coverage Assessment

[Assessment of completeness and potential blind spots]

## 3. Literature Landscape Overview

### Temporal Distribution
[How papers are distributed over time]

### Category Distribution
[Which categories are most represented]

### Key Research Groups
[Active institutions and authors]

## 4. Thematic Synthesis

### Theme A: [Name]

**State of the Art:**
[Current understanding from essential papers]

**Key Papers:**
- [arXiv ID] - [Title] - [Contribution]
- [arXiv ID] - [Title] - [Contribution]

**Evolution:** [Research progression]
**Consensus and Debates:** [Agreements and disagreements]
**Implications:** [For the research goals]

### Theme B: [Name]
[Same structure]

### Theme C: [Name]
[Same structure]

### Cross-Theme Connections
[How themes relate to each other]

## 5. Methodology Assessment and Best Practices

### Proven Methods

| Method | Papers | Strengths | Limitations |
|--------|--------|-----------|-------------|
| [method] | [IDs] | [strengths] | [limitations] |

### Emerging Methods

| Method | Papers | Promise | Maturity |
|--------|--------|---------|----------|
| [method] | [IDs] | [promise] | [stage] |

### Best Practices
[From the literature]

### Common Pitfalls
[Risks flagged in papers - bias, overfitting, data-mining, etc.]

## 6. Research Gaps and Opportunities

### Unexplored Areas
[Not well-covered]

### Contradictions
[Disagreements needing resolution]

### Extension Opportunities
[How to build on existing work]

### Integration Opportunities
[Combining approaches]

## 7. Recommendations and Suggested Reading

### Priority 1 - Essential Foundation
1. [arXiv ID] - [Title] - _[Why read first]_
2. [arXiv ID] - [Title] - _[Why foundational]_
3. [arXiv ID] - [Title] - _[Core contribution]_

### Priority 2 - Methodology Deep-Dive
4. [arXiv ID] - [Title] - _[Key methodology]_
5. [arXiv ID] - [Title] - _[Important technique]_

### Priority 3 - Broader Context
6. [arXiv ID] - [Title] - _[Useful context]_
7. [arXiv ID] - [Title] - _[Alternative view]_

### Methodology Recommendations
[What methods to adopt]

### Suggested Research Directions
[Questions to pursue]

### Follow-Up Searches
- Query: [specific query] - _Purpose: [reason]_
- Category: [category] - _Purpose: [reason]_

## 8. Complete Reference List

### Essential Papers (Score 5)
1. **[Authors] ([Year]).** [Title]. arXiv:[ID]. [Categories].
   URL: [abstract link]

### Highly Relevant (Score 4)
[References]

### Relevant Background (Score 3)
[References]

### Supporting Literature (Score 1-2)
[References]

## 9. Appendices

### Appendix A: Full Query URLs
[All 5 arXiv API URLs executed]

### Appendix B: Complete Paper Metadata
[Extended table with all papers and scores]

### Appendix C: Category Glossary
[Descriptions of all categories searched]

---

**Literature Review Date:** {{date}}
**Execution Mode:** Autonomous
**Papers Discovered:** [count]
**Papers Analyzed:** [count]
**Essential Papers:** [count]

_This literature review was produced through autonomous systematic arXiv API search and analysis._
```

### 5.2 Update Frontmatter and Finalize

Update frontmatter: `stepsCompleted: [1, 2, 3, 4, 5]`

### 5.3 Present Summary to User

After the document is complete, present a brief summary to the user:

"I've completed an **autonomous arXiv literature review** on **{{search_topic}}**.

**Results:**
- **Papers discovered:** [count]
- **Papers analyzed:** [count]
- **Essential papers (score 5):** [count]
- **Thematic clusters:** [count]

**Output document:** `{{output_file_path}}`

The document includes executive summary, thematic synthesis, methodology assessment, research gaps, recommended reading order, and complete references.

Would you like me to dive deeper into any specific theme, paper, or research direction?"

---

## ERROR HANDLING

- **API call fails**: Log the error, wait 5 seconds, retry once. If still fails, skip that query and continue with remaining queries.
- **No results for a query**: Note "0 results" and continue with next query.
- **All queries return 0 results**: Produce a document noting no results were found, suggest alternative search terms, and list the queries attempted.
- **No project context found**: Use `{{project_name}}` as the topic with general quant finance scope.

## QUALITY STANDARDS

- Every paper citation must come from actual API results
- Score justifications must reference specific aspects of the abstract
- Thematic clusters must be grounded in actual paper content
- Gap analysis must be based on what was NOT found relative to the search scope
- Recommendations must trace back to specific papers and findings
