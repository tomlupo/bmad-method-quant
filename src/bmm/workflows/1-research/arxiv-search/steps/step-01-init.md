# arXiv Search Step 1: Search Scope Confirmation and Query Construction

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER generate content without user confirmation

- 📖 CRITICAL: ALWAYS read the complete step file before taking any action - partial understanding leads to incomplete decisions
- 🔄 CRITICAL: When loading next step with 'C', ensure the entire file is read and understood before proceeding
- ✅ FOCUS EXCLUSIVELY on confirming search scope and constructing arXiv API queries
- 📋 YOU ARE A LITERATURE SEARCH PLANNER, not content generator
- 💬 ACKNOWLEDGE and CONFIRM understanding of search goals
- 🔍 This is SCOPE CONFIRMATION and QUERY DESIGN ONLY - no API calls yet
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

## EXECUTION PROTOCOLS:

- 🎯 Show your analysis before taking any action
- ⚠️ Present [C] continue option after scope confirmation and query design
- 💾 ONLY proceed when user chooses C (Continue)
- 📖 Update frontmatter `stepsCompleted: [1]` before loading next step
- 🚫 FORBIDDEN to load next step until C is selected

## CONTEXT BOUNDARIES:

- Workflow type = "arxiv-search" is already set
- **Search topic = "{{search_topic}}"** - discovered from initial discussion
- **Search goals = "{{search_goals}}"** - captured from initial discussion
- **Target categories = "{{target_categories}}"** - identified arXiv categories
- **Search keywords = "{{search_keywords}}"** - refined keyword list
- Focus on constructing effective arXiv API queries
- Load the arXiv API reference: `{installed_path}/arxiv-api-reference.md`

## YOUR TASK:

Confirm the literature search scope and design arXiv API queries for **{{search_topic}}** with the user's goals in mind. Present the planned queries for user approval before execution.

## SEARCH SCOPE AND QUERY CONSTRUCTION SEQUENCE:

### 1. Load API Reference

Load `{installed_path}/arxiv-api-reference.md` to reference query syntax, field prefixes, Boolean operators, and category codes.

### 2. Present Search Strategy

Start with search scope understanding:

"I understand you want to conduct an **arXiv literature search** for **{{search_topic}}** with these goals: {{search_goals}}

**Search Strategy Overview:**

I'll construct targeted queries using the arXiv API to systematically search for relevant papers. Here's my planned approach:

**Target arXiv Categories:**

{{target_categories}} - with justification for each category

**Search Dimensions:**

I'll search across multiple dimensions to ensure comprehensive coverage:

1. **Primary Topic Search** - Direct keyword matches in titles and abstracts
2. **Category-Specific Search** - Focused searches within the most relevant arXiv categories
3. **Methodology Search** - Papers describing the specific methods or techniques of interest
4. **Cross-Disciplinary Search** - Related work in adjacent fields (ML, statistics, econometrics)
5. **Recent Advances** - Latest submissions sorted by date

### 3. Present Constructed Queries

Design and present the specific arXiv API queries:

"**Proposed arXiv API Queries:**

Based on your research topic and goals, here are the queries I'll execute:

**Query 1 - Primary Topic (Broad):**
```
http://export.arxiv.org/api/query?search_query=all:{{primary_keywords}}+AND+%28cat:{{cat1}}+OR+cat:{{cat2}}%29&sortBy=relevance&max_results=50
```
_Purpose: Cast a wide net for all papers mentioning the core topic in relevant categories_

**Query 2 - Title-Focused (Precision):**
```
http://export.arxiv.org/api/query?search_query=ti:{{title_keywords}}&sortBy=relevance&max_results=30
```
_Purpose: Find papers where the topic is the primary focus_

**Query 3 - Methodology-Focused:**
```
http://export.arxiv.org/api/query?search_query=abs:{{method_keywords}}+AND+%28cat:{{cat1}}+OR+cat:{{cat2}}%29&sortBy=relevance&max_results=30
```
_Purpose: Find papers using or developing the specific methods of interest_

**Query 4 - Recent Advances:**
```
http://export.arxiv.org/api/query?search_query={{topic_keywords}}+AND+%28cat:{{cat1}}+OR+cat:{{cat2}}+OR+cat:{{cat3}}%29&sortBy=submittedDate&sortOrder=descending&max_results=25
```
_Purpose: Discover the latest research in this space_

**Query 5 - Cross-Disciplinary:**
```
http://export.arxiv.org/api/query?search_query=all:{{cross_keywords}}+AND+%28cat:stat.ML+OR+cat:cs.LG+OR+cat:econ.EM%29&sortBy=relevance&max_results=25
```
_Purpose: Find related approaches from adjacent fields_

_Note: Actual query parameters will be filled with specific keywords derived from your topic._

**Estimated Coverage:**

- ~50 broad topic results + ~30 title-focused + ~30 methodology + ~25 recent + ~25 cross-disciplinary
- After deduplication: estimated 80-120 unique papers to screen
- Top 20-40 most relevant papers will receive detailed analysis

**Search Parameters:**

- Sort: By relevance for topic searches, by date for recent advances
- Pagination: Will paginate if needed for thorough coverage
- Rate limiting: 3-second delay between API calls (arXiv requirement)

**Does this search strategy align with your research goals?**

[C] Continue - Proceed with these queries
"

### 4. Refine If Needed

If user requests changes:

- Adjust categories, keywords, or query structure as requested
- Add or remove search dimensions
- Modify result limits or sorting
- Re-present updated queries for confirmation

### 5. Handle Continue Selection

#### If 'C' (Continue):

- Document confirmed scope and queries in research file
- Update frontmatter: `stepsCompleted: [1]`
- Load: `./step-02-execute-search.md`

## APPEND TO DOCUMENT:

When user selects 'C', append scope confirmation:

```markdown
## Literature Search Scope and Strategy

**Search Topic:** {{search_topic}}
**Search Goals:** {{search_goals}}
**Date:** {{date}}

### Target arXiv Categories

[List of targeted categories with justification]

### Search Dimensions

1. **Primary Topic Search** - [description and keywords]
2. **Category-Specific Search** - [description and categories]
3. **Methodology Search** - [description and method keywords]
4. **Cross-Disciplinary Search** - [description and adjacent categories]
5. **Recent Advances** - [description and date parameters]

### Constructed API Queries

[Complete list of queries to be executed with purpose descriptions]

### Expected Coverage

- Estimated total results: [number]
- Estimated unique papers after deduplication: [number]
- Planned detailed analysis: Top [number] most relevant

**Scope Confirmed:** {{date}}
```

## SUCCESS METRICS:

✅ Search scope clearly confirmed with user
✅ arXiv categories identified and justified
✅ Multiple search dimensions designed for comprehensive coverage
✅ Specific API queries constructed with proper syntax
✅ Query purposes clearly explained
✅ Expected coverage estimated
✅ [C] continue option presented and handled correctly
✅ Scope confirmation documented when user proceeds
✅ Proper routing to next step (query execution)

## FAILURE MODES:

❌ Not confirming search scope with user
❌ Missing relevant arXiv categories for the topic
❌ Constructing syntactically invalid API queries
❌ Only using a single search dimension (not systematic)
❌ Not explaining query purposes to user
❌ Not presenting [C] continue option
❌ Proceeding without user scope confirmation
❌ Not routing to next step

❌ **CRITICAL**: Reading only partial step file - leads to incomplete understanding and poor decisions
❌ **CRITICAL**: Proceeding with 'C' without fully reading and understanding the next step file
❌ **CRITICAL**: Making decisions without complete understanding of step requirements and protocols

## NEXT STEP:

After user selects 'C', load `./step-02-execute-search.md` to execute the confirmed arXiv API queries and collect results.

Remember: This is SCOPE CONFIRMATION and QUERY DESIGN ONLY - no arXiv API calls yet, just confirming the search approach and query construction!
