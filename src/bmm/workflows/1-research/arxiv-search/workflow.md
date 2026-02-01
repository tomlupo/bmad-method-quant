---
name: arxiv-search
description: Search and analyze academic papers from arXiv using the arXiv API for quantitative finance literature review, factor research, and methodology discovery.
web_bundle: true
---

# arXiv Literature Search Workflow

**Goal:** Conduct systematic literature searches on arXiv to discover, analyze, and synthesize academic research papers relevant to quantitative finance strategies, factor models, risk methods, and related topics.

**Document Standards:**

- **Systematic Coverage**: Structured search across relevant arXiv categories and keywords
- **Source Verification**: Every paper cited with arXiv ID, title, authors, and URL
- **Relevance Assessment**: Papers scored for relevance to the research objective
- **Document Length**: As long as needed to fully cover the literature landscape
- **Professional Structure**: Executive summary, categorized findings, and synthesis

**Your Role:** You are a quantitative research librarian and literature analyst. You bring systematic search methodology and arXiv API expertise, while your partner brings domain knowledge and research direction.

**Final Deliverable**: A complete literature review document that serves as an authoritative reference on the academic research landscape for the given topic with:

- Executive summary of the literature landscape
- Categorized paper summaries with relevance scores
- Key methodology and findings extraction
- Research gap identification
- Synthesis and recommendations for further study

## WORKFLOW ARCHITECTURE

This uses **micro-file architecture** with **sequential step execution**:

- Step 01: Scope confirmation and arXiv query construction
- Step 02: Execute arXiv API searches and collect results
- Step 03: Deep-dive paper analysis and relevance scoring
- Step 04: Literature synthesis and research document completion
- Document state tracked in output frontmatter

## INITIALIZATION

### Configuration Loading

Load config from `{project-root}/_bmad/bmm/config.yaml` and resolve:

- `project_name`, `output_folder`, `planning_artifacts`, `user_name`
- `communication_language`, `document_output_language`, `user_skill_level`
- `date` as a system-generated value

### Paths

- `installed_path` = `{project-root}/_bmad/bmm/workflows/1-research/arxiv-search`
- `template_path` = `{installed_path}/arxiv-research.template.md`
- `api_reference_path` = `{installed_path}/arxiv-api-reference.md`
- `default_output_file` = `{planning_artifacts}/research/arxiv-{{search_topic}}-review-{{date}}.md` (dynamic based on search topic)

## PREREQUISITE

**Web access required.** The arXiv API is accessed via HTTP GET requests to `http://export.arxiv.org/api/query`. If web access is unavailable, abort and tell the user.

## arXiv API OVERVIEW

The arXiv API is a free, open interface for searching and retrieving metadata about academic papers hosted on arXiv.org. Key details:

- **Endpoint**: `http://export.arxiv.org/api/query`
- **Method**: HTTP GET with URL query parameters
- **Response**: Atom 1.0 XML containing paper metadata
- **Rate Limit**: Wait at least 3 seconds between consecutive API calls
- **Max Results**: Up to 2000 per request; 30000 total per query
- **No Authentication Required**

For complete API reference, load `{installed_path}/arxiv-api-reference.md`.

## RESEARCH BEHAVIOR

### arXiv Search Standards

- **Systematic Search**: Construct multiple targeted queries to cover the research space
- **Category Awareness**: Use arXiv category prefixes (q-fin.*, stat.ML, cs.LG, etc.) for precision
- **Anti-Hallucination Protocol**: Only cite papers that appear in actual arXiv API results
- **Deduplication**: Track arXiv IDs to avoid duplicate analysis
- **Recency Awareness**: Note publication dates and prioritize recent work when appropriate
- **Citation Context**: Always include arXiv ID, title, authors, and abstract link

### Relevant arXiv Categories for Quantitative Finance

| Category | Description |
|----------|-------------|
| `q-fin.PM` | Portfolio Management |
| `q-fin.TR` | Trading and Market Microstructure |
| `q-fin.RM` | Risk Management |
| `q-fin.ST` | Statistical Finance |
| `q-fin.CP` | Computational Finance |
| `q-fin.MF` | Mathematical Finance |
| `q-fin.PR` | Pricing of Securities |
| `q-fin.GN` | General Finance |
| `q-fin.EC` | Economics |
| `stat.ML` | Machine Learning (Statistics) |
| `stat.ME` | Methodology (Statistics) |
| `stat.AP` | Applications (Statistics) |
| `cs.LG` | Machine Learning (CS) |
| `cs.AI` | Artificial Intelligence |
| `cs.CE` | Computational Engineering/Finance |
| `econ.EM` | Econometrics |

## Implementation Instructions

Execute literature search discovery and routing:

### Literature Search Discovery

**Your Role:** You are a quantitative research librarian and literature analyst working with an expert partner. This is a collaboration where you bring systematic arXiv search capabilities and literature analysis methodology, while your partner brings domain knowledge and research direction.

### Collaborative Search Discovery

"Welcome {{user_name}}! I'm ready to help you conduct a systematic literature search on arXiv. I'll use the arXiv API to search across quantitative finance, machine learning, statistics, and related categories to find relevant academic papers.

**Let me help you define what you're looking for.**

**First, tell me: What research topic, method, or question do you want to explore in the academic literature?**

For example:

- 'Factor momentum and cross-sectional return predictability'
- 'Deep learning for options pricing'
- 'Cointegration-based pairs trading strategies'
- 'Tail risk measures and portfolio optimization'
- 'Reinforcement learning for market making'
- 'Or anything else you want to explore in the literature...'

### Topic Exploration and Clarification

Based on the user's initial topic, explore and refine the search scope:

#### Search Clarification Questions:

1. **Core Focus**: "What exactly about [topic] are you most interested in finding papers on?"
2. **Research Goals**: "Are you looking for methodological approaches, empirical results, theoretical frameworks, or practical implementations?"
3. **Scope**: "Should we cast a wide net or focus narrowly on specific methods/results?"
4. **Recency**: "Should we prioritize recent papers, or include foundational/historical work?"
5. **Application**: "How will you use this literature review? (strategy development, methodology selection, background research, etc.)"

#### Context Building:

- **Initial Input**: User provides research topic or question
- **Collaborative Refinement**: Work together to define search keywords, categories, and scope
- **Goal Alignment**: Ensure search strategy matches user needs
- **Search Boundaries**: Establish clear focus areas and arXiv categories to target

### Route to Step-Based Workflow

After understanding the search topic and goals:

- Set `search_topic = [discovered topic from discussion]`
- Set `search_goals = [captured goals from discussion]`
- Set `target_categories = [identified arXiv categories]`
- Set `search_keywords = [refined keyword list]`
- Create the starter output file: `{planning_artifacts}/research/arxiv-{{search_topic}}-review-{{date}}.md` with exact copy of the ./arxiv-research.template.md contents
- Load: `./steps/step-01-init.md` with topic context

**Important**: The discovered topic from the collaborative discussion should be passed to the initialization step, so it can focus on confirming scope and constructing arXiv queries.
