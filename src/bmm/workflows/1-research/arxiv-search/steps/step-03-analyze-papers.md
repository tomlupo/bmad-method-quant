# arXiv Search Step 3: Paper Analysis and Relevance Scoring

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER fabricate analysis for papers not found in step-02 results

- 📖 CRITICAL: ALWAYS read the complete step file before taking any action - partial understanding leads to incomplete decisions
- 🔄 CRITICAL: When loading next step with 'C', ensure the entire file is read and understood before proceeding
- ✅ Analyze paper abstracts and score relevance to research goals
- 📋 YOU ARE A LITERATURE ANALYST AND CRITICAL REVIEWER
- 💬 FOCUS on extracting key contributions, methods, and relevance
- 🔍 Use the paper abstracts from step-02 results for analysis
- 📝 WRITE ANALYSIS TO DOCUMENT IMMEDIATELY
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

## EXECUTION PROTOCOLS:

- 🎯 Show analysis methodology before presenting scored results
- ⚠️ Present [C] continue option after all papers are analyzed and scored
- 📝 WRITE PAPER ANALYSIS TO DOCUMENT IMMEDIATELY
- 💾 ONLY proceed when user chooses C (Continue)
- 📖 Update frontmatter `stepsCompleted: [1, 2, 3]` before loading next step
- 🚫 FORBIDDEN to load next step until C is selected

## CONTEXT BOUNDARIES:

- Current document and frontmatter from steps 01-02 are available
- **Search topic = "{{search_topic}}"** - established from initial discussion
- **Search goals = "{{search_goals}}"** - established from initial discussion
- **Paper catalog from step-02** - load from document (all discovered papers with abstracts)
- Papers already deduplicated in step-02
- No additional arXiv API calls required (analysis is based on already-collected abstracts)

## YOUR TASK:

Analyze each paper's abstract against the research goals, score relevance, categorize by theme, extract key methodologies and findings, and produce a ranked analysis for **{{search_topic}}**.

## PAPER ANALYSIS SEQUENCE:

### 1. Define Scoring Framework

Present the relevance scoring methodology:

"I'll now analyze the **[papers_found] papers** discovered in the arXiv search for **{{search_topic}}**.

**Relevance Scoring Framework:**

| Score | Level | Criteria |
|-------|-------|----------|
| **5** | Essential | Directly addresses core research topic; must-read paper |
| **4** | Highly Relevant | Closely related methodology or findings; strong contribution |
| **3** | Relevant | Related work with applicable methods or context |
| **2** | Tangentially Related | Adjacent topic; useful for background or comparison |
| **1** | Low Relevance | Marginally related; may inform broader context |

**Analysis Dimensions:**

- **Topic Alignment**: How closely does the paper match the search topic?
- **Methodology Relevance**: Are the methods applicable to the research goals?
- **Findings Applicability**: Are the results relevant to the intended application?
- **Recency and Impact**: Publication date and indicators of importance (journal publication, citations mentioned in comments)
- **Novelty**: Does the paper introduce new approaches or insights?

**Beginning systematic paper analysis...**"

### 2. Analyze Papers by Category

**UTILIZE SUBPROCESSES AND SUBAGENTS**: Use subagents or parallel processing if available to analyze papers simultaneously for efficiency.

For each paper in the catalog from step-02, analyze:

#### Per-Paper Analysis Template:

```
**[arXiv ID] - [Title]**
Authors: [Author list]
Published: [Date] | Categories: [Categories]
Link: [Abstract URL]

**Abstract Analysis:**
[2-3 sentence summary of what the paper does and its key contribution]

**Methodology:** [Key methods used - e.g., "LASSO regression, Fama-French factors, rolling window estimation"]
**Key Findings:** [1-2 most important results or contributions]
**Relevance to {{search_topic}}:** [Specific explanation of how this paper relates to the research goals]
**Relevance Score:** [1-5] - [One-line justification]
```

### 3. Group by Relevance Tier

After analyzing all papers, group them into tiers:

```markdown
### Tier 1: Essential Papers (Score 5)

[Papers that are must-reads for this research topic]

### Tier 2: Highly Relevant (Score 4)

[Papers with strong direct relevance]

### Tier 3: Relevant Background (Score 3)

[Papers providing useful methods or context]

### Tier 4: Supporting Literature (Score 2)

[Papers for broader context and comparison]

### Tier 5: Peripheral (Score 1)

[Marginally related papers - brief mention only]
```

### 4. Identify Thematic Clusters

Group papers by research theme/methodology:

"**Thematic Clusters Identified:**

**Cluster A: [Theme Name]** (e.g., "Factor Construction Methods")
- [Paper 1 ID] - [Brief description]
- [Paper 2 ID] - [Brief description]
- _Common thread: [What connects these papers]_

**Cluster B: [Theme Name]** (e.g., "Machine Learning Applications")
- [Paper 1 ID] - [Brief description]
- [Paper 2 ID] - [Brief description]
- _Common thread: [What connects these papers]_

[Continue for all identified clusters]

**Cross-Cluster Connections:**
- [Identify papers that bridge multiple themes]
- [Note methodological overlaps between clusters]
"

### 5. Extract Methodology Summary

Identify the key methodologies across the literature:

"**Methodology Landscape:**

| Method | Papers Using It | Primary Application |
|--------|----------------|---------------------|
| [Method 1] | [IDs] | [How it's applied] |
| [Method 2] | [IDs] | [How it's applied] |
| ... | ... | ... |

**Dominant Approaches:** [Most common methodologies]
**Emerging Methods:** [Newer or less common but promising approaches]
**Methodological Gaps:** [Methods not well-represented in the literature]
"

### 6. Generate Paper Analysis Content

**WRITE IMMEDIATELY TO DOCUMENT**

```markdown
## Paper Analysis and Relevance Scoring

### Scoring Methodology

| Score | Level | Criteria |
|-------|-------|----------|
| 5 | Essential | Directly addresses core research topic |
| 4 | Highly Relevant | Closely related methodology or findings |
| 3 | Relevant | Related work with applicable methods |
| 2 | Tangentially Related | Useful for background or comparison |
| 1 | Low Relevance | Marginally related |

### Tier 1: Essential Papers (Score 5)

[Full analysis for each essential paper using per-paper template]

### Tier 2: Highly Relevant Papers (Score 4)

[Full analysis for each highly relevant paper]

### Tier 3: Relevant Background (Score 3)

[Condensed analysis for relevant papers]

### Tier 4-5: Supporting and Peripheral Literature (Score 1-2)

[Brief listing with one-line descriptions]

### Thematic Clusters

#### [Cluster A Name]

[Papers and common thread]

#### [Cluster B Name]

[Papers and common thread]

[Additional clusters as identified]

### Cross-Cluster Connections

[Bridging papers and methodological overlaps]

### Methodology Landscape

| Method | Papers | Application |
|--------|--------|-------------|
| [method] | [IDs] | [application] |

**Dominant Approaches:** [summary]
**Emerging Methods:** [summary]
**Methodological Gaps:** [summary]

### Analysis Summary

- **Total papers analyzed:** [count]
- **Essential papers (Score 5):** [count]
- **Highly relevant (Score 4):** [count]
- **Relevant background (Score 3):** [count]
- **Supporting literature (Score 1-2):** [count]
- **Thematic clusters identified:** [count]
- **Key methodologies found:** [count]
```

### 7. Present Analysis and Continue Option

"I've completed the **paper analysis and relevance scoring** for **{{search_topic}}**.

**Analysis Summary:**

- **[total] papers analyzed** from the arXiv search results
- **[count] essential papers** (Score 5) identified as must-reads
- **[count] highly relevant papers** (Score 4) with strong contributions
- **[count] thematic clusters** identified in the literature
- **[count] distinct methodologies** mapped across the landscape

**Key Insights:**

- [Most important finding from the literature analysis]
- [Dominant methodological trend]
- [Notable gap or opportunity identified]

**Next Step:** I'll synthesize these findings into a comprehensive literature review with recommendations.

**Ready to proceed to synthesis?**
[C] Continue - Save analysis and proceed to literature synthesis
"

### 8. Handle Continue Selection

#### If 'C' (Continue):

- **CONTENT ALREADY WRITTEN TO DOCUMENT**
- Update frontmatter: `stepsCompleted: [1, 2, 3]`
- Update frontmatter: `papers_analyzed: [count]`
- Load: `./step-04-synthesis.md`

## APPEND TO DOCUMENT:

Content is already written to document when generated in step 6. No additional append needed.

## SUCCESS METRICS:

✅ All papers from step-02 catalog analyzed
✅ Relevance scores assigned with clear justification
✅ Papers grouped into meaningful tiers
✅ Thematic clusters identified with common threads
✅ Cross-cluster connections mapped
✅ Methodology landscape summarized
✅ Analysis written immediately to document
✅ [C] continue option presented and handled correctly
✅ Proper routing to next step (synthesis)

## FAILURE MODES:

❌ Analyzing papers not present in step-02 results
❌ Assigning scores without clear justification
❌ Missing papers in the analysis (incomplete coverage)
❌ Not identifying thematic connections between papers
❌ Not mapping the methodology landscape
❌ Not writing analysis immediately to document
❌ Not presenting [C] continue option after analysis
❌ Not routing to synthesis step

❌ **CRITICAL**: Reading only partial step file - leads to incomplete understanding and poor decisions
❌ **CRITICAL**: Proceeding with 'C' without fully reading and understanding the next step file
❌ **CRITICAL**: Making decisions without complete understanding of step requirements and protocols

## ANALYSIS STANDARDS:

- Base all analysis on actual paper abstracts from step-02
- Provide objective relevance assessments, not inflated scores
- Identify both strengths and limitations of each paper's approach
- Note when papers contradict each other
- Flag potential data-mining or overfitting concerns in empirical papers
- Apply quant finance domain expertise to evaluate methodology soundness

## NEXT STEP:

After user selects 'C', load `./step-04-synthesis.md` to synthesize findings into a comprehensive literature review document.

Remember: Only analyze papers that were actually discovered in step-02. Never fabricate paper analysis!
