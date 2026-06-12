# arXiv Academic Enrichment Protocol

This reference is loaded by research workflow steps that augment web research with academic papers from arXiv. It provides the instructions for adding academic depth to any research topic.

## When to Enrich with arXiv

Apply arXiv enrichment when the current research step covers topics where academic literature adds value:

- **Quantitative methods** - statistical models, factor analysis, optimization
- **Machine learning applications** - prediction, classification, NLP for finance
- **Risk management** - VaR models, stress testing, tail risk
- **Market microstructure** - order flow, liquidity, price impact
- **Portfolio theory** - allocation, optimization, rebalancing
- **Econometrics** - time series, causal inference, regime detection
- **Technology trends** - algorithms, computational methods, data processing
- **Regulatory modeling** - compliance algorithms, systemic risk measurement

If the current research topic does not benefit from academic literature, skip enrichment and note "arXiv enrichment: not applicable for this section."

## How to Execute arXiv Enrichment

### Step 1: Construct a Targeted Query

Build 1-2 focused arXiv API queries related to the current research section's topic. Use the endpoint:

```
http://export.arxiv.org/api/query?search_query=[QUERY]&sortBy=submittedDate&sortOrder=descending&max_results=10
```

**Query Construction Rules:**

- Use `all:` prefix for broad keyword search
- Use `ti:` prefix for title-specific search
- Use `cat:` prefix to limit to relevant categories
- Combine with `AND`, `OR` operators
- URL-encode spaces as `+`, parentheses as `%28`/`%29`

**Category Quick Reference for Quant Finance:**

| Category | Focus |
|----------|-------|
| `q-fin.PM` | Portfolio Management |
| `q-fin.ST` | Statistical Finance |
| `q-fin.TR` | Trading and Market Microstructure |
| `q-fin.RM` | Risk Management |
| `q-fin.CP` | Computational Finance |
| `q-fin.MF` | Mathematical Finance |
| `q-fin.PR` | Pricing of Securities |
| `stat.ML` | Machine Learning (Stats) |
| `cs.LG` | Machine Learning (CS) |
| `econ.EM` | Econometrics |

**Example Queries:**

For a section on momentum factor research:
```
http://export.arxiv.org/api/query?search_query=all:momentum+factor+AND+%28cat:q-fin.PM+OR+cat:q-fin.ST%29&sortBy=submittedDate&sortOrder=descending&max_results=10
```

For a section on risk management models:
```
http://export.arxiv.org/api/query?search_query=all:risk+management+AND+cat:q-fin.RM&sortBy=submittedDate&sortOrder=descending&max_results=10
```

### Step 2: Fetch and Parse Results

Fetch the API URL. The response is Atom 1.0 XML. Extract from each `<entry>`:

| Field | XML Path |
|-------|----------|
| Title | `<title>` |
| Authors | `<author><name>` |
| Abstract | `<summary>` |
| Published | `<published>` |
| arXiv ID | Extract from `<id>` URL |
| Categories | `<category term>` |
| PDF Link | `<link rel="related" title="pdf" href>` |
| Abstract Link | `<link rel="alternate" href>` |

### Step 3: Select Relevant Papers

From the results, select the **top 3-5 most relevant** papers based on:

1. Direct relevance to the current research section
2. Recency (prefer papers from the last 3 years unless foundational)
3. Methodological value (techniques applicable to the research topic)

### Step 4: Integrate into Research Section

Add an **"Academic Research Foundations"** subsection within the current research section:

```markdown
### Academic Research Foundations

Recent academic research from arXiv provides additional depth on [section topic]:

**[Paper Title]** ([Year]) - [Authors]
_[1-2 sentence summary of key findings or methodology relevant to this section]_
_arXiv: [ID] | [Primary Category] | [Abstract Link]_

**[Paper Title]** ([Year]) - [Authors]
_[1-2 sentence summary]_
_arXiv: [ID] | [Primary Category] | [Abstract Link]_

**[Paper Title]** ([Year]) - [Authors]
_[1-2 sentence summary]_
_arXiv: [ID] | [Primary Category] | [Abstract Link]_

_Academic sources supplement web research and provide methodological foundations for the analysis above._
```

## Rate Limiting

**CRITICAL**: Wait at least 3 seconds between consecutive arXiv API calls. If multiple enrichment sections are needed, space them appropriately.

## Anti-Hallucination Protocol

- **ONLY cite papers that appear in actual arXiv API results**
- **NEVER fabricate arXiv IDs, titles, or author names**
- If the API returns no relevant results, state: "No directly relevant academic papers were found on arXiv for this specific topic."
- If the API call fails, note the failure and continue with web sources only

## Scope Control

arXiv enrichment is a supplement, not a replacement for web research. Keep the enrichment subsection concise (3-5 papers max per section). The primary research should still come from web sources with URL citations.
