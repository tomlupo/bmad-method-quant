# arXiv Search Step 2: Execute API Searches and Collect Results

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER fabricate or hallucinate paper results - only use actual arXiv API responses

- 📖 CRITICAL: ALWAYS read the complete step file before taking any action - partial understanding leads to incomplete decisions
- 🔄 CRITICAL: When loading next step with 'C', ensure the entire file is read and understood before proceeding
- ✅ Execute arXiv API queries and collect real paper metadata
- 📋 YOU ARE A SYSTEMATIC LITERATURE SEARCHER, not content generator
- 💬 FOCUS on executing queries and organizing results
- 🔍 WEB FETCH REQUIRED - retrieve actual arXiv API results
- 📝 WRITE SEARCH RESULTS TO DOCUMENT IMMEDIATELY
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

## EXECUTION PROTOCOLS:

- 🎯 Execute each query and show results before proceeding to next
- ⏱️ WAIT AT LEAST 3 SECONDS between consecutive arXiv API calls (rate limit requirement)
- ⚠️ Present [C] continue option after all searches are complete and results documented
- 📝 WRITE SEARCH RESULTS TO DOCUMENT IMMEDIATELY
- 💾 ONLY proceed when user chooses C (Continue)
- 📖 Update frontmatter `stepsCompleted: [1, 2]` before loading next step
- 🚫 FORBIDDEN to load next step until C is selected

## CONTEXT BOUNDARIES:

- Current document and frontmatter from step-01 are available
- **Search topic = "{{search_topic}}"** - established from initial discussion
- **Search goals = "{{search_goals}}"** - established from initial discussion
- **Confirmed queries from step-01** - load from document
- arXiv API reference available at `{installed_path}/arxiv-api-reference.md`
- Web access is required to call the arXiv API

## YOUR TASK:

Execute the confirmed arXiv API queries from step-01, collect and organize paper metadata, deduplicate results, and produce a structured catalog of discovered papers.

## SEARCH EXECUTION SEQUENCE:

### 1. Load Confirmed Queries

Read the research document to retrieve the confirmed API queries from step-01.

"Now I'll execute the **confirmed arXiv API queries** for **{{search_topic}}**.

**Execution Plan:**

- Execute each query sequentially with 3-second delays between calls
- Parse Atom XML responses to extract paper metadata
- Track all paper IDs for deduplication
- Organize results by search dimension

**Beginning systematic literature search...**"

### 2. Execute Queries Systematically

For each confirmed query from step-01:

#### Query Execution Protocol:

1. **Fetch the URL**: Use web fetch to call the arXiv API endpoint
2. **Parse the response**: Extract paper metadata from the Atom XML response
3. **Extract key fields per paper**:
   - arXiv ID (from `<id>` element, extract ID portion)
   - Title (from `<title>`)
   - Authors (from `<author><name>` elements)
   - Abstract (from `<summary>`)
   - Published date (from `<published>`)
   - Updated date (from `<updated>`)
   - Primary category (from `<arxiv:primary_category>`)
   - All categories (from `<category>` elements)
   - PDF link (from `<link rel="related" title="pdf">`)
   - Abstract link (from `<link rel="alternate">`)
   - DOI (from `<arxiv:doi>` if present)
   - Journal reference (from `<arxiv:journal_ref>` if present)
   - Comment (from `<arxiv:comment>` if present)
4. **Record total results**: Note `<opensearch:totalResults>` for coverage assessment
5. **Wait 3 seconds** before next API call

#### Progress Reporting:

After each query, report:

"**Query [N] Results: [Query Description]**

- Total matching papers on arXiv: [totalResults]
- Retrieved in this batch: [itemsPerPage]
- New unique papers found: [count after deduplication]
- Notable finds: [1-2 particularly relevant titles]
"

### 3. Pagination Decision

After initial queries complete, assess if pagination is needed:

- If `totalResults` significantly exceeds `max_results` for a high-value query
- And the initial results show high relevance
- Then execute additional paginated requests (start=50, start=100, etc.)
- Cap at 200 total results per query dimension to maintain manageability

### 4. Deduplicate and Organize Results

After all queries are complete:

"**Search Execution Complete.**

**Results Summary:**

| Search Dimension | Query | Total on arXiv | Retrieved | Unique New |
|-----------------|-------|----------------|-----------|------------|
| Primary Topic | [query] | [total] | [retrieved] | [new] |
| Title-Focused | [query] | [total] | [retrieved] | [new] |
| Methodology | [query] | [total] | [retrieved] | [new] |
| Recent Advances | [query] | [total] | [retrieved] | [new] |
| Cross-Disciplinary | [query] | [total] | [retrieved] | [new] |

**Total Unique Papers Discovered: [count]**

**Deduplication:** [number] duplicates removed across search dimensions

### 5. Generate Paper Catalog

Organize all unique papers into a structured catalog:

**WRITE IMMEDIATELY TO DOCUMENT**

```markdown
## arXiv Search Results

### Search Execution Summary

| Search Dimension | Total on arXiv | Retrieved | Unique |
|-----------------|----------------|-----------|--------|
| [dimension] | [total] | [retrieved] | [unique] |
| ... | ... | ... | ... |
| **TOTAL** | | | **[total unique]** |

**Queries Executed:** [number]
**Total API Calls:** [number]
**Deduplication:** [duplicates] duplicates removed

### Paper Catalog

#### Papers from Primary Topic Search

| # | arXiv ID | Title | Authors | Published | Categories |
|---|----------|-------|---------|-----------|------------|
| 1 | [id] | [title] | [first author et al.] | [YYYY-MM-DD] | [categories] |
| 2 | [id] | [title] | [first author et al.] | [YYYY-MM-DD] | [categories] |
| ... | ... | ... | ... | ... | ... |

#### Papers from Title-Focused Search

| # | arXiv ID | Title | Authors | Published | Categories |
|---|----------|-------|---------|-----------|------------|
| ... | ... | ... | ... | ... | ... |

#### Papers from Methodology Search

| # | arXiv ID | Title | Authors | Published | Categories |
|---|----------|-------|---------|-----------|------------|
| ... | ... | ... | ... | ... | ... |

#### Papers from Recent Advances

| # | arXiv ID | Title | Authors | Published | Categories |
|---|----------|-------|---------|-----------|------------|
| ... | ... | ... | ... | ... | ... |

#### Papers from Cross-Disciplinary Search

| # | arXiv ID | Title | Authors | Published | Categories |
|---|----------|-------|---------|-----------|------------|
| ... | ... | ... | ... | ... | ... |
```

### 6. Present Results and Continue Option

"I've completed the **systematic arXiv search** for **{{search_topic}}**.

**Key Statistics:**

- **[total unique] unique papers** discovered across [N] search dimensions
- **[N] queries** executed against the arXiv API
- **[N] duplicates** removed through deduplication
- Papers span from [oldest date] to [newest date]

**Notable Discoveries:**

- [Highlight 3-5 particularly relevant or highly-cited papers]
- [Note any unexpected or cross-disciplinary finds]

**Next Step:** I'll analyze each paper's abstract for relevance to your research goals and score them for detailed review.

**Ready to proceed to paper analysis?**
[C] Continue - Save results and proceed to paper analysis
"

### 7. Handle Continue Selection

#### If 'C' (Continue):

- **CONTENT ALREADY WRITTEN TO DOCUMENT**
- Update frontmatter: `stepsCompleted: [1, 2]`
- Update frontmatter: `papers_found: [total unique count]`
- Load: `./step-03-analyze-papers.md`

## APPEND TO DOCUMENT:

Content is already written to document when generated in step 5. No additional append needed.

## SUCCESS METRICS:

✅ All confirmed queries executed against arXiv API
✅ 3-second delay observed between API calls
✅ Paper metadata correctly extracted from Atom XML responses
✅ Results deduplicated by arXiv ID
✅ Structured paper catalog generated and written to document
✅ Search statistics clearly reported
✅ Notable discoveries highlighted
✅ [C] continue option presented and handled correctly
✅ Proper routing to next step (paper analysis)

## FAILURE MODES:

❌ Fabricating paper titles, IDs, or metadata not present in API responses
❌ Not respecting 3-second rate limit between API calls
❌ Missing papers due to incomplete XML parsing
❌ Not deduplicating results across search dimensions
❌ Not writing results immediately to document
❌ Not presenting [C] continue option after results collection
❌ Not routing to paper analysis step

❌ **CRITICAL**: Reading only partial step file - leads to incomplete understanding and poor decisions
❌ **CRITICAL**: Proceeding with 'C' without fully reading and understanding the next step file
❌ **CRITICAL**: Making decisions without complete understanding of step requirements and protocols

## API ERROR HANDLING:

If an arXiv API call fails:

1. **Report the error** to the user with the query URL and error message
2. **Retry once** after waiting 5 seconds
3. If retry fails, **skip that query** and note it in the results
4. **Continue with remaining queries** - do not abort the entire search
5. At the end, offer to retry failed queries

## NEXT STEP:

After user selects 'C', load `./step-03-analyze-papers.md` to analyze paper abstracts, score relevance, and identify the most important papers for detailed review.

Remember: Only report papers that actually appear in arXiv API responses. Never fabricate or hallucinate paper metadata!
