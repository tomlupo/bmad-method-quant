# arXiv API Reference Guide

This document provides complete reference for constructing and executing arXiv API queries within the literature search workflow.

## API Endpoint

```
http://export.arxiv.org/api/query
```

All requests are HTTP GET with URL query parameters. No authentication required.

## Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `search_query` | string | None | Search expression using field prefixes and Boolean operators |
| `id_list` | comma-delimited | None | Specific arXiv paper IDs to retrieve |
| `start` | integer | 0 | 0-based index for pagination |
| `max_results` | integer | 10 | Number of results per request (max 2000) |
| `sortBy` | string | `relevance` | Sort field: `relevance`, `lastUpdatedDate`, `submittedDate` |
| `sortOrder` | string | `descending` | Sort direction: `ascending`, `descending` |

## Search Field Prefixes

| Prefix | Field | Example |
|--------|-------|---------|
| `ti:` | Title | `ti:momentum+factor` |
| `au:` | Author | `au:fama` |
| `abs:` | Abstract | `abs:risk+premium` |
| `co:` | Comment | `co:accepted+journal` |
| `jr:` | Journal Reference | `jr:journal+of+finance` |
| `cat:` | Subject Category | `cat:q-fin.PM` |
| `all:` | All fields | `all:pairs+trading` |

## Boolean Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `AND` | Both conditions must match | `ti:momentum+AND+cat:q-fin.PM` |
| `OR` | Either condition matches | `ti:momentum+OR+ti:reversal` |
| `ANDNOT` | Exclude matching results | `ti:momentum+ANDNOT+ti:physics` |

### Grouping and Phrases

- Parentheses for grouping: URL-encode as `%28` and `%29`
- Phrase search with double quotes: URL-encode as `%22`

**Example - Grouped OR:**
```
search_query=ti:momentum+AND+%28cat:q-fin.PM+OR+cat:q-fin.ST%29
```

**Example - Exact phrase:**
```
search_query=ti:%22factor+investing%22
```

## Date Filtering

Filter by submission date using the `submittedDate` field in `search_query`:

```
search_query=cat:q-fin.PM+AND+submittedDate:[202301010000+TO+202412312359]
```

Format: `YYYYMMDDTTTT` in GMT 24-hour time.

## Constructing Effective Queries for Quant Finance

### Strategy 1: Category + Keyword Search

Search within specific arXiv categories with topic keywords:

```
http://export.arxiv.org/api/query?search_query=cat:q-fin.PM+AND+all:momentum+factor&sortBy=submittedDate&sortOrder=descending&max_results=50
```

### Strategy 2: Multi-Category Sweep

Search across multiple related categories:

```
http://export.arxiv.org/api/query?search_query=%28cat:q-fin.PM+OR+cat:q-fin.ST+OR+cat:q-fin.TR%29+AND+all:factor+model&max_results=50
```

### Strategy 3: Author-Focused Search

Find papers by specific researchers:

```
http://export.arxiv.org/api/query?search_query=au:fama+AND+au:french&max_results=20
```

### Strategy 4: Title + Abstract Precision

Combine title and abstract searches for precision:

```
http://export.arxiv.org/api/query?search_query=ti:deep+learning+AND+abs:portfolio+optimization&sortBy=submittedDate&sortOrder=descending&max_results=30
```

### Strategy 5: Recent Papers in a Category

Get the latest papers in a specific category:

```
http://export.arxiv.org/api/query?search_query=cat:q-fin.RM&sortBy=submittedDate&sortOrder=descending&max_results=25
```

### Strategy 6: Retrieve Specific Papers

Fetch specific papers by their arXiv IDs:

```
http://export.arxiv.org/api/query?id_list=2103.00496,2009.14794,1907.12830
```

## Response Format (Atom 1.0 XML)

### Feed-Level Elements

```xml
<feed xmlns="http://www.w3.org/2005/Atom"
      xmlns:opensearch="http://a9.com/-/spec/opensearch/1.1/"
      xmlns:arxiv="http://arxiv.org/schemas/atom">
  <title>ArXiv Query: ...</title>
  <id>http://arxiv.org/api/...</id>
  <updated>2024-01-01T00:00:00-05:00</updated>
  <opensearch:totalResults>150</opensearch:totalResults>
  <opensearch:startIndex>0</opensearch:startIndex>
  <opensearch:itemsPerPage>10</opensearch:itemsPerPage>
  <!-- entries follow -->
</feed>
```

### Entry-Level Elements (Per Paper)

```xml
<entry>
  <id>http://arxiv.org/abs/2103.00496v2</id>
  <updated>2021-06-15T17:00:00Z</updated>
  <published>2021-02-28T19:00:00Z</published>
  <title>Deep Learning for Portfolio Optimization</title>
  <summary>We propose a deep learning framework for...</summary>

  <author>
    <name>John Smith</name>
    <arxiv:affiliation>MIT</arxiv:affiliation>
  </author>
  <author>
    <name>Jane Doe</name>
  </author>

  <category term="q-fin.PM" scheme="http://arxiv.org/schemas/atom"/>
  <category term="cs.LG" scheme="http://arxiv.org/schemas/atom"/>
  <arxiv:primary_category term="q-fin.PM"/>

  <!-- Links -->
  <link href="http://arxiv.org/abs/2103.00496v2" rel="alternate" type="text/html"/>
  <link href="http://arxiv.org/pdf/2103.00496v2" rel="related" title="pdf" type="application/pdf"/>
  <link href="http://dx.doi.org/10.xxxx/xxxxx" rel="related" title="doi" type="text/html"/>

  <!-- Optional extended metadata -->
  <arxiv:comment>25 pages, 8 figures, accepted at Journal of Finance</arxiv:comment>
  <arxiv:journal_ref>Journal of Finance, 2021</arxiv:journal_ref>
  <arxiv:doi>10.xxxx/xxxxx</arxiv:doi>
</entry>
```

### Key Fields to Extract Per Paper

| Field | XML Path | Description |
|-------|----------|-------------|
| arXiv ID | `<id>` | Unique identifier (extract from URL) |
| Title | `<title>` | Paper title |
| Authors | `<author><name>` | List of author names |
| Abstract | `<summary>` | Paper abstract |
| Published | `<published>` | First submission date |
| Updated | `<updated>` | Latest version date |
| Categories | `<category term>` | arXiv categories |
| Primary Category | `<arxiv:primary_category term>` | Main category |
| PDF Link | `<link rel="related" title="pdf">` | Direct PDF URL |
| Abstract Link | `<link rel="alternate">` | Abstract page URL |
| DOI | `<arxiv:doi>` | DOI if available |
| Journal Ref | `<arxiv:journal_ref>` | Journal publication info |
| Comment | `<arxiv:comment>` | Author comments (pages, figures, acceptance) |

## Pagination

Use `start` and `max_results` for multi-page retrieval:

```
# First page
http://export.arxiv.org/api/query?search_query=all:momentum&start=0&max_results=50

# Second page
http://export.arxiv.org/api/query?search_query=all:momentum&start=50&max_results=50
```

**Important**: Wait at least 3 seconds between consecutive API calls.

## Rate Limiting

- **Minimum delay**: 3 seconds between consecutive requests
- **Max per request**: 2000 results
- **Max total**: 30,000 results per query (use refined queries or OAI-PMH for bulk)
- **Cache**: Results are cached daily on arXiv servers; same-day repeat queries return cached data

## Error Handling

Errors return Atom feeds with a single entry containing:
- `<summary>`: Error message description
- `<id>`: Error reference (e.g., `http://arxiv.org/api/errors#incorrect_id_format`)

Common errors:
- Invalid `start` or `max_results` (must be non-negative integers)
- Malformed arXiv IDs in `id_list`
- Requests exceeding 30,000 results (HTTP 400)

## Quant Finance Category Reference

### Quantitative Finance (q-fin)

| Category | Name | Focus |
|----------|------|-------|
| `q-fin.CP` | Computational Finance | Numerical methods, simulation, pricing algorithms |
| `q-fin.EC` | Economics | Economic models, behavioral finance |
| `q-fin.GN` | General Finance | General financial topics |
| `q-fin.MF` | Mathematical Finance | Stochastic calculus, derivatives theory |
| `q-fin.PM` | Portfolio Management | Portfolio optimization, asset allocation |
| `q-fin.PR` | Pricing of Securities | Options, derivatives, fixed income pricing |
| `q-fin.RM` | Risk Management | VaR, CVaR, stress testing, risk models |
| `q-fin.ST` | Statistical Finance | Empirical finance, financial econometrics |
| `q-fin.TR` | Trading and Market Microstructure | Trading strategies, market microstructure |

### Related Categories

| Category | Name | Relevance |
|----------|------|-----------|
| `stat.ML` | Machine Learning (Stats) | ML methods applicable to finance |
| `stat.ME` | Methodology | Statistical methods for quant research |
| `stat.AP` | Applications | Applied statistics including finance |
| `cs.LG` | Machine Learning (CS) | Deep learning, reinforcement learning for trading |
| `cs.AI` | Artificial Intelligence | AI methods for financial applications |
| `cs.CE` | Computational Engineering | Computational methods for finance |
| `econ.EM` | Econometrics | Time series, causal inference, financial econometrics |
| `math.OC` | Optimization and Control | Portfolio optimization, optimal execution |
| `math.PR` | Probability | Stochastic processes, measure theory |
