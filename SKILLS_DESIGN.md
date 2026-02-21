# Research Paper Writing: Claude Skill Modules

> Extracted from 17 GitHub repos studying LLM-agent-driven research automation.
> Each skill is a self-contained, reusable Claude Code skill for the research paper lifecycle.

---

## Skill Map Overview

```
Phase 0: Research Planning
  [S01] research-planning        — 研究规划与论文架构设计
  [S02] idea-generation          — 研究想法生成与新颖性评估

Phase 1: Literature & Knowledge
  [S03] literature-search        — 文献检索（Semantic Scholar / arXiv / Google Scholar）
  [S04] literature-review        — 文献综述生成（多视角对话模拟）
  [S05] related-work-writing     — Related Work 段落撰写

Phase 2: Formalization
  [S06] math-reasoning           — 数学推理、公式推导、定理证明
  [S07] algorithm-design         — 算法设计与伪代码生成
  [S08] atomic-decomposition     — 原子概念分解（数学公式 ↔ 代码双向映射）

Phase 3: Implementation
  [S09] experiment-code          — 实验代码编写（ML training/eval pipeline）
  [S10] code-debugging           — 代码调试与自动修复
  [S11] experiment-design        — 实验设计（baselines、ablation、hyperparameter）

Phase 4: Results & Visualization
  [S12] data-analysis            — 数据分析与统计检验
  [S13] figure-generation        — 科研图表生成（matplotlib + VLM反馈闭环）
  [S14] table-generation         — LaTeX 表格生成（实验结果 → 发表级表格）

Phase 5: Writing
  [S15] paper-writing-section    — 逐节论文撰写（Abstract → Conclusion）
  [S16] latex-formatting         — LaTeX 格式、模板、编译
  [S17] citation-management      — BibTeX 引用管理与引文插入

Phase 6: Review & Polish
  [S18] self-review              — 自动审稿（NeurIPS/ICLR review form）
  [S19] rebuttal-writing         — Rebuttal 撰写（逐条回复审稿意见）
  [S20] paper-revision           — 基于审稿意见的论文修改

Phase 7: Assembly & Delivery
  [S21] paper-assembly           — 论文全流程整合（数学+图表+数据+引用 → 完整论文）
  [S22] paper-compilation        — LaTeX 编译与错误修复
  [S23] backward-traceability    — 数据可追溯（PDF中数字 → 生成代码行的超链接）

Phase 8: Extended
  [S24] survey-generation        — 综述论文自动生成
  [S25] paper-to-code            — 论文 → 可运行代码仓库
  [S26] slide-generation         — 论文 → 演示幻灯片/Poster
  [S27] novelty-assessment       — 研究新颖性评估与文献对比
  [S28] symbolic-equation        — 科学方程发现（符号回归 + LLM引导）
```

---

## Detailed Skill Specifications

---

### S01: research-planning — 研究规划与论文架构设计

**Source repos:** AI-Researcher (plan_agent), AgentLaboratory (plan_formulation), Paper2Code (1_planning.py)

**What it does:**
- Takes a research topic/idea as input
- Outputs: research questions, methodology outline, paper structure, section-by-section plan
- Generates Mermaid class/sequence diagrams for system architecture (from Paper2Code)
- Creates a dependency graph of what needs to be done first

**Key techniques extracted:**
1. **4-turn planning conversation** (Paper2Code): Overall plan → Architecture design (UML) → Logic analysis (task list) → Config extraction
2. **Diverge-converge framework** (AI-Researcher): Generate 5 orthogonal research directions, score by novelty/reliability/potential, deepen the best
3. **Plan formulation dialogue** (AgentLaboratory): Postdoc-PhD role-play to refine plan iteratively

**Prompt template core:**
```
You are an experienced research advisor. Given a research topic and optional reference papers:

1. Identify the core research question and its significance
2. Propose a methodology with clear steps
3. Design the paper structure (8 sections: Abstract, Introduction, Background, Related Work, Methods, Experiments, Results, Discussion)
4. Create a dependency-ordered task list for implementation
5. Identify key baselines, datasets, and evaluation metrics
6. Flag risks and potential failure modes

Output format: structured JSON with sections, tasks, dependencies, and timeline.
```

---

### S02: idea-generation — 研究想法生成与新颖性评估

**Source repos:** AI-Scientist (generate_ideas.py), SciMON, AI-Researcher (idea_agent)

**What it does:**
- Given a research area + existing codebase, generates novel research ideas
- Iterative reflection (up to 5 rounds) to refine each idea
- Novelty check against Semantic Scholar / arXiv
- Scores ideas on: Interestingness, Feasibility, Novelty (1-10)

**Key techniques:**
1. **Seed idea bootstrapping** (AI-Scientist): Start with 1-2 hand-written ideas, generate more conditioned on archive
2. **Reflection convergence** (AI-Scientist): Refine until LLM says "I am done"
3. **Literature-grounded novelty check** (AI-Scientist): Up to 10 rounds of Semantic Scholar queries
4. **Novelty optimization loop** (SciMON): Retrieve inspirations → iteratively optimize for novelty

**Output per idea:**
```json
{
  "Name": "adaptive_attention_pruning",
  "Title": "Adaptive Attention Head Pruning via Gradient-Guided Importance Scoring",
  "Experiment": "detailed implementation plan...",
  "Interestingness": 8,
  "Feasibility": 7,
  "Novelty": 9
}
```

---

### S03: literature-search — 文献检索

**Source repos:** STORM (rm.py), AI-Scientist (generate_ideas.py), data-to-paper (literature_search.py), OpenResearcher

**What it does:**
- Multi-source academic search: Semantic Scholar, arXiv, OpenAlex, CrossRef, Google Scholar
- Returns structured results: title, authors, year, venue, abstract, citation count, BibTeX
- Supports embedding-based similarity sorting
- Filters by citation count, recency, relevance

**API integrations:**
- Semantic Scholar: `api.semanticscholar.org/graph/v1/paper/search` (fields: title, authors, venue, year, abstract, citationCount, citationStyles)
- arXiv: `arxiv` Python package
- OpenAlex: `pyalex` library
- CrossRef: REST API for DOI-based lookup

---

### S04: literature-review — 文献综述生成

**Source repos:** STORM (knowledge_curation.py), AutoSurvey, AgentLaboratory (literature_review phase)

**What it does:**
- Multi-perspective dialogue simulation: generates N expert personas, each conducts multi-turn Q&A
- Each persona asks questions from their unique angle, grounded by live search results
- Synthesizes all conversations into a structured knowledge base
- Citation precision ~85%, recall ~85% (STORM benchmark)

**Key techniques:**
1. **Persona generation** (STORM): Find related Wikipedia topics → extract TOC → LLM generates diverse editor personas
2. **Grounded expert answers** (STORM): TopicExpert converts questions to search queries, retrieves top-k results, synthesizes answer with inline citations
3. **Parallel persona execution** (STORM): ThreadPoolExecutor runs all persona conversations concurrently
4. **Semantic retrieval** (STORM): SentenceTransformer embeddings + cosine similarity for per-section information retrieval

---

### S05: related-work-writing — Related Work 段落撰写

**Source repos:** STORM, LitLLM, AI-Scientist (perform_writeup.py)

**What it does:**
- Takes collected literature + paper draft as input
- Generates a Related Work section with proper citations
- "Compare and contrast" style, not just descriptions
- Sentence-level planning (LitLLM): keyword extraction → retrieval → re-ranking → generation

**Prompt core (from AI-Scientist per_section_tips):**
```
Related Work should compare and contrast prior work with your approach.
Don't just describe what others did — explain how your work differs and why.
Organize by theme, not chronologically.
Cite broadly — not just the most popular papers.
```

---

### S06: math-reasoning — 数学推理、公式推导、定理证明

**Source repos:** AI-Researcher (survey_agent math extraction), LLM-SR, data-to-paper (analysis coding)

**What it does:**
- Formal mathematical derivation with step-by-step reasoning
- Theorem statement and proof generation in LaTeX
- Equation numbering and cross-referencing
- Statistical test selection and interpretation
- Symbolic equation discovery (LLM-SR: evolutionary search for scientific equations)

**Key techniques:**
1. **Math formula extraction from papers** (AI-Researcher Paper Survey Agent): Navigate paper → find formula → extract LaTeX
2. **Backward-traceable computations** (data-to-paper): `\num{formula}` → evaluated at compile time → hyperlinked to source
3. **LLM-guided symbolic regression** (LLM-SR): LLM proposes candidate equations → evolutionary search optimizes fit

---

### S07: algorithm-design — 算法设计与伪代码生成

**Source repos:** Paper2Code (planning stage), AI-Researcher (plan_agent)

**What it does:**
- Generates algorithm pseudocode in LaTeX `algorithmic` environment
- Creates UML class diagrams (Mermaid syntax)
- Designs sequence diagrams for system flow
- Ensures consistency between pseudocode and actual implementation

**Prompt pattern:**
```
Given a method description:
1. Formalize the algorithm with clear input/output
2. Write pseudocode using \begin{algorithmic} environment
3. Generate a Mermaid classDiagram showing data structures
4. Generate a Mermaid sequenceDiagram showing execution flow
5. Verify consistency: every pseudocode step must map to a code module
```

---

### S08: atomic-decomposition — 原子概念分解

**Source repos:** AI-Researcher (survey_agent.py) — this is its core innovation

**What it does:**
- Decomposes a research idea into atomic, self-contained concepts
- For each atom: extracts math formula from papers + finds code implementation in repos
- Bidirectional mapping: `math_formula ↔ code_implementation`
- Creates a structured note for each concept with: definition, formula, code, references

**Output format:**
```json
{
  "definition": "Kernelized Gumbel-Softmax Operator",
  "math_formula": "Z = softmax((log π + g) / τ), g ~ Gumbel(0,1)",
  "code_implementation": "def gumbel_softmax(logits, tau=1.0): ...",
  "reference_papers": ["NodeFormer: A Scalable Graph Structure Learning Transformer"],
  "reference_codebases": ["LarFii/nodeformer"]
}
```

**Why this matters:** Ensures every mathematical concept in the paper has a verified code implementation, and every code module traces back to a formal mathematical definition.

---

### S09: experiment-code — 实验代码编写

**Source repos:** AI-Scientist (perform_experiments.py), AgentLaboratory (mlesolver.py), AI-Researcher (ml_agent.py)

**What it does:**
- Generates complete ML training/evaluation pipelines
- Iterative code improvement via REPLACE/EDIT operations
- Self-debugging: runs code, captures errors, fixes automatically (up to 4 retries)
- Hill-climbing optimization: maintains pool of best-scoring code candidates

**Key techniques:**
1. **Aider diff-mode editing** (AI-Scientist): LLM outputs unified diffs applied to experiment files
2. **REPLACE/EDIT operations** (AgentLaboratory): Two editing primitives with line-range targeting
3. **Mandatory project structure** (AI-Researcher): `data/`, `model/`, `training/`, `testing/`, `run_training_testing.py`
4. **Code reflection** (AgentLaboratory): After improvement, reflect on what worked → inject insights into next iteration

**Constraints enforced:**
- No placeholder code (`pass`, `...`, `raise NotImplementedError`)
- Must use actual datasets (not toy data)
- Must generate figures (Figure_1.png, Figure_2.png minimum)
- PyTorch or scikit-learn only (no TensorFlow/Keras)

---

### S10: code-debugging — 代码调试与自动修复

**Source repos:** AI-Scientist (perform_experiments.py), data-to-paper (debugger.py), AI-Scientist-v2 (parallel_agent.py _debug())

**What it does:**
- Captures runtime errors from experiment execution
- Feeds truncated stderr + code context back to LLM
- Structured error categorization: ImportError, FileNotFoundError, TimeoutError, etc.
- State machine for fix strategy: "repost" / "leave" / "regen"
- Up to 5 repair attempts with code reflection

**Key techniques:**
1. **Truncated error feedback** (AI-Scientist): Last N lines of stderr fed back
2. **Structured RunIssue objects** (data-to-paper): category + description + fix instructions
3. **Monkey-patched sandbox** (data-to-paper): Override pandas, scipy, sklearn to track operations and enforce safety
4. **Automated code repair tool** (AgentLaboratory): Dedicated "repair" system prompt distinct from "generate" prompt

---

### S11: experiment-design — 实验设计

**Source repos:** AI-Scientist-v2 (agent_manager.py 4-stage), AI-Researcher (exp_analyser.py)

**What it does:**
- Designs complete experiment plans: baselines, ablations, hyperparameter sweeps
- 4-stage progressive experiment management:
  - Stage 1: Initial implementation (simple dataset, working baseline)
  - Stage 2: Baseline tuning (hyperparameters, multiple datasets)
  - Stage 3: Creative research (novel improvements, 3+ datasets)
  - Stage 4: Ablation studies (systematic component analysis)
- Multi-seed evaluation for statistical significance
- VLM-evaluated stage completion (training curves checked for convergence)

**Key techniques:**
1. **Progressive staging** (AI-Scientist-v2): Each stage has explicit goals and completion criteria
2. **Ablation study design** (AI-Researcher): Analyzes results → proposes component removal experiments
3. **Sensitivity analysis** (AI-Researcher): Tests key hyperparameters across ranges
4. **Best-node selection** (AI-Scientist-v2): LLM holistically selects best experiment considering metrics + VLM plot analysis

---

### S12: data-analysis — 数据分析与统计检验

**Source repos:** data-to-paper (analysis coding), AgentLaboratory (results_interpretation)

**What it does:**
- Generates Python code for statistical analysis of experimental results
- Selects appropriate statistical tests (t-test, ANOVA, chi-square, etc.)
- Interprets p-values, effect sizes, confidence intervals
- Produces structured analysis reports with numerical evidence

**Key techniques:**
1. **4-round code review** (data-to-paper):
   - Round 1: Fundamental math/statistics flaws
   - Round 2: Data handling issues
   - Round 3: Per-table review
   - Round 4: Cross-table completeness
2. **Allowed packages whitelist** (data-to-paper): pandas, numpy, scipy, statsmodels, sklearn only
3. **Results interpretation dialogue** (AgentLaboratory): Postdoc guides PhD to extract meaningful insights

---

### S13: figure-generation — 科研图表生成

**Source repos:** MatPlotAgent, AI-Scientist-v2 (VLM feedback), AI-Scientist (plot.py)

**What it does:**
- Generates publication-quality matplotlib/seaborn figures
- 3-module closed loop: Query expansion → Code generation → VLM visual feedback
- VLM (GPT-4V) evaluates figure quality: data representation, labels, colors, readability
- Automatic code retry on execution errors (up to 4 attempts)
- Professional styling with consistent color palettes

**Key techniques:**
1. **Query expansion** (MatPlotAgent): Raw instruction → step-by-step coding instructions before code generation
2. **VLM feedback loop** (MatPlotAgent): Generated PNG + original query → GPT-4V → improvement instructions → regenerate code
3. **Plot aggregation** (AI-Scientist-v2): Multiple experiment `.npy` files → unified ~12 final figures
4. **Figure-caption alignment review** (AI-Scientist-v2): VLM checks each figure's caption accuracy and informativeness

**Score improvement:** GPT-4 baseline 48.86 → with MatPlotAgent 61.16 (+12.3 points)

---

### S14: table-generation — LaTeX 表格生成

**Source repos:** data-to-paper (displayitems stage), AI-Researcher (comparison tables)

**What it does:**
- Converts experimental results (JSON/CSV/DataFrame) to publication-ready LaTeX tables
- Handles: `booktabs` styling, bold best results, multi-row/multi-column layouts
- Baseline comparison tables from prior work (scraped from papers)
- Ablation study tables with component-wise analysis

**Prompt pattern:**
```
Given experimental results in JSON format:
1. Design table layout (rows = methods, columns = metrics/datasets)
2. Generate LaTeX using booktabs package (\toprule, \midrule, \bottomrule)
3. Bold the best result in each column
4. Add proper captions and labels
5. Ensure all numbers match the actual experimental logs — do not hallucinate
```

---

### S15: paper-writing-section — 逐节论文撰写

**Source repos:** AI-Scientist (perform_writeup.py), AgentLaboratory (papersolver.py), data-to-paper (writing stages)

**What it does:**
- Writes each paper section individually with section-specific guidance
- Two refinement passes per section (AI-Scientist): criticize → fix errors → compress
- Scaffold generation (AgentLaboratory): skeleton → fill section by section
- Word count enforcement (~4000 words for 8-page paper)

**Per-section tips (from AI-Scientist + AgentLaboratory):**

| Section | Key Guidance |
|---------|-------------|
| **Abstract** | TL;DR → why it's hard → contribution → how verified. Single paragraph only. |
| **Introduction** | Longer abstract; list contributions as bullet points. |
| **Background** | Problem setting with formal notation; define all symbols. |
| **Related Work** | Compare and contrast, not just describe. Organize by theme. |
| **Methods** | Precise mathematical equations; what we do AND why. |
| **Experimental Setup** | Datasets, metrics, hyperparameters. Don't hallucinate hardware details. |
| **Results** | Only report numbers from actual logs. Include ablations. Include all figures. |
| **Discussion/Conclusion** | Brief recap → limitations → future work as "academic offspring." |

**Two-pass refinement (AI-Scientist):**
```
Pass 1: Criticize the section. Fix: unenclosed math, broken refs, LaTeX errors,
        hallucinated numbers, duplicate labels, verbosity.
Pass 2: Identify redundancies. Save space. Be more concise without weakening the message.
```

---

### S16: latex-formatting — LaTeX 格式、模板、编译

**Source repos:** AI-Scientist (template.tex), AgentLaboratory (papersolver.py auto-inject), data-to-paper (latex_to_pdf.py)

**What it does:**
- Manages LaTeX templates for major venues (ICML, ICLR, NeurIPS, AAAI, ACL, ICBINB)
- Auto-injects required packages (amsmath, booktabs, hyperref, algorithm, etc.)
- Handles common formatting issues: overfull hboxes, missing labels, duplicate sections
- Conference-specific formatting rules

**Auto-injected packages (from AgentLaboratory):**
```
amsmath, amssymb, array, algorithm, algorithmicx, algpseudocode,
booktabs, colortbl, color, enumitem, fontawesome5, float, graphicx,
hyperref, listings, makecell, multicol, multirow, pgffor, pifont,
soul, sidecap, subcaption, titletoc, footmisc, url, wrapfig, xcolor, xspace
```

---

### S17: citation-management — BibTeX 引用管理

**Source repos:** AI-Scientist (perform_writeup.py citation harvesting), data-to-paper (literature_search.py)

**What it does:**
- Iterative citation harvesting: LLM reads draft → identifies most needed citation → searches Semantic Scholar → selects from results → injects BibTeX
- Pre-compilation validation: every `\cite{key}` must exist in `.bib`
- Deduplication of citation entries
- Proper BibTeX formatting with all required fields

**Citation loop (from AI-Scientist, 20 rounds):**
```
Round N:
1. LLM reads current draft, identifies gap needing a citation
2. LLM generates a Semantic Scholar search query
3. API returns top-10 results with title/abstract/BibTeX
4. LLM selects the most relevant paper(s)
5. BibTeX appended to references.bib
6. LLM integrates \cite{key} into the paper text
```

---

### S18: self-review — 自动审稿

**Source repos:** AI-Scientist (perform_review.py), AgentLaboratory (ReviewersAgent), ChatReviewer

**What it does:**
- Structured review using NeurIPS/ICLR review form
- Ensemble voting: 5 independent reviews → meta-review aggregation
- Three reviewer personas: harsh-fair, harsh-critical, open-minded
- Reflection loop: refine review up to 5 rounds
- Scores: Originality, Quality, Clarity, Significance, Soundness, Presentation, Contribution (1-4), Overall (1-10)

**Review form fields:**
```json
{
  "Summary": "...",
  "Strengths": ["..."],
  "Weaknesses": ["..."],
  "Originality": 3,
  "Quality": 3,
  "Clarity": 3,
  "Significance": 3,
  "Soundness": 3,
  "Presentation": 3,
  "Contribution": 3,
  "Overall": 6,
  "Confidence": 4,
  "Decision": "Accept/Reject"
}
```

**Few-shot calibration (AI-Scientist):** Uses 3 real ICLR papers + reviews as reference:
- "Attention Is All You Need" (scored 8/10 Accept)
- Real ICLR submissions with known scores

---

### S19: rebuttal-writing — Rebuttal 撰写

**Source repos:** ChatReviewer (chat_response.py)

**What it does:**
- Takes reviewer comments as input
- Extracts concerns one by one
- Generates point-by-point responses
- Key instruction: "Reply with what we have done, not what we will do"
- Proper rebuttal formatting: Concern → Response per reviewer

**Output format:**
```
# Response to Reviewers

## Reviewer #1
**Concern #1:** [extracted concern]
**Author Response:** [detailed response with evidence]

**Concern #2:** [extracted concern]
**Author Response:** [detailed response with evidence]

## Reviewer #2
...
```

---

### S20: paper-revision — 基于审稿意见修改论文

**Source repos:** AI-Scientist (perform_improvement), AgentLaboratory (report_refinement with second_round)

**What it does:**
- Takes review feedback and current paper draft
- Maps each weakness/concern to specific sections needing changes
- Applies targeted edits preserving paper structure
- Re-compiles and re-reviews to verify improvement
- Supports iterative revision loops

**Key technique (AgentLaboratory):** When PhD decides to revise:
1. Copy current state to `prev_*` fields
2. Reset all downstream phases (experiments, writing)
3. Re-run the pipeline with reviewer feedback injected as notes
4. Compare new scores vs previous scores

---

### S21: paper-assembly — 论文全流程整合

**Source repos:** AI-Scientist (launch_scientist.py), AI-Researcher (main_ai_researcher.py), AgentLaboratory (ai_lab_repo.py)

**What it does:**
- Orchestrates the entire paper pipeline end-to-end
- Manages state propagation between phases
- Handles checkpointing and resumption
- Integrates: literature → plan → code → experiments → figures → tables → writing → review

**Orchestration patterns:**
1. **Sequential pipeline** (AI-Scientist): generate_ideas → experiments → writeup → review
2. **Multi-agent state broadcasting** (AgentLaboratory): `set_agent_attr()` propagates results to all agents
3. **FlowModule caching** (AI-Researcher): Cache each agent's output, support resume/replay
4. **Copilot mode checkpoints** (AgentLaboratory): Human can intervene at any phase boundary

---

### S22: paper-compilation — LaTeX 编译与错误修复

**Source repos:** AI-Scientist (perform_writeup.py generate_latex), data-to-paper (latex_to_pdf.py)

**What it does:**
- Full LaTeX compilation pipeline: `pdflatex → bibtex → pdflatex → pdflatex`
- Pre-compilation validation:
  - All `\cite{key}` exist in `.bib`
  - All `\includegraphics{file}` exist as `.png`
  - No duplicate figure references
  - No duplicate `\section{}` headers
- Error correction loop: `chktex` analysis → LLM fixes → recompile (up to 5 rounds)
- Handles common errors: unescaped underscores, HTML syntax in LaTeX, unclosed environments

---

### S23: backward-traceability — 数据可追溯

**Source repos:** data-to-paper (the defining innovation)

**What it does:**
- Every number in the final PDF hyperlinks back to the exact code line that produced it
- Uses `\hypertarget{label}{value}` and `\hyperlink{label}{value}` LaTeX commands
- `\num{formula, "explanation"}` evaluated at compile time via `eval()`
- Appendix contains full code listing with hyperlink anchors
- Calculation Notes section shows `formula = result` for each computed value

**Implementation:**
1. Code output: Numbers tagged with `\hypertarget{R1a}{45.3}`
2. Paper text: Author writes `\num{\hyperlink{R1a}{45.3}, "mean age"}`
3. Compile time: `eval()` verifies formula, generates hyperlink
4. Appendix: Code listing with `\hypertarget` at relevant lines
5. Result: Click any number in PDF → jumps to code line that produced it

---

### S24: survey-generation — 综述论文自动生成

**Source repos:** AutoSurvey, STORM

**What it does:**
- Generates complete academic survey papers
- Parallel multi-LLM sub-section generation
- RAG-based real-time knowledge updates
- Multi-perspective coverage for comprehensive topic review

---

### S25: paper-to-code — 论文 → 可运行代码仓库

**Source repos:** Paper2Code (PaperCoder)

**What it does:**
- Takes an ML paper PDF and generates a complete, runnable code repository
- 3-stage pipeline: Planning (UML + dependency graph) → Analysis (per-file logic) → Coding (dependency-ordered generation)
- Dependency-ordered code generation: each file sees all previously generated files
- Mermaid diagrams as rigid interface contracts across all stages

**Key metrics:** PaperBench 44.26% vs baseline 16.4%, 85% rated helpful, only 0.48% code lines need human modification

---

### S26: slide-generation — 论文 → 演示幻灯片/Poster

**Source repos:** (Gap identified in research — no dedicated tool exists)

**What it does:**
- Converts a completed paper into presentation slides (Beamer/PowerPoint)
- Extracts key figures, tables, and equations
- Creates a narrative flow suitable for oral presentation
- Poster layout for conference poster sessions

**Note:** This is an identified gap. This skill would be novel — to be designed from best practices.

---

### S27: novelty-assessment — 研究新颖性评估

**Source repos:** AI-Scientist (check_idea_novelty), data-to-paper (assess_novelty stage), SciMON

**What it does:**
- Takes a research idea and searches literature systematically
- Multi-round search-evaluate loop (up to 10 rounds)
- Harsh critic persona for novelty evaluation
- Binary decision: Novel / Not Novel with justification
- Tracks "most similar papers" for positioning

**System prompt (from AI-Scientist):**
```
Be a harsh critic for novelty. Ensure there is a sufficient contribution
for a new conference or workshop paper. You will be given access to the
Semantic Scholar API to survey the literature.
```

---

### S28: symbolic-equation — 科学方程发现

**Source repos:** LLM-SR

**What it does:**
- Uses LLMs to discover scientific equations from data
- LLM-guided evolutionary search over symbolic expression space
- Combines neural reasoning with symbolic optimization
- Outputs interpretable mathematical relationships

---

## Cross-Cutting Patterns Used Across Skills

### Pattern 1: Reflection Convergence
Used in: S02, S15, S18
```
Loop until LLM says "I am done" or max_rounds:
  1. Generate output
  2. LLM critiques own output
  3. LLM refines output
```

### Pattern 2: Code-Execute-Fix Loop
Used in: S09, S10, S12, S13
```
Loop up to max_retries:
  1. LLM generates code
  2. Execute in subprocess
  3. If error: feed error back → LLM fixes
  4. If success: evaluate output quality
```

### Pattern 3: Multi-Source Search + Selection
Used in: S03, S17, S27
```
1. LLM generates search query
2. API returns top-k results
3. LLM selects most relevant
4. Repeat with refined queries
```

### Pattern 4: VLM Feedback Loop
Used in: S13, S11
```
1. Generate visual artifact (figure/plot)
2. Send image to VLM (GPT-4V)
3. VLM provides improvement instructions
4. Regenerate based on feedback
```

### Pattern 5: Ensemble + Meta-Aggregation
Used in: S18
```
1. Generate N independent outputs (e.g., 5 reviews)
2. Aggregate via meta-review prompt
3. Average numerical scores
```

---

## GitHub Repos Cloned → `/Users/lingzhi/Code/research-engine/github/`

| Repo | Primary Skills | Stars |
|------|---------------|-------|
| AI-Scientist | S01, S02, S09, S15, S17, S18, S22 | 12.1k |
| AI-Scientist-v2 | S11, S13, S15, S21 | 2.1k |
| AI-Researcher | S01, S08, S09, S10, S11, S12 | 4.5k |
| AgentLaboratory | S01, S04, S09, S15, S18, S20, S21 | 5.3k |
| data-to-paper | S06, S10, S12, S14, S22, S23 | 756 |
| storm | S04, S05, S24 | 27.9k |
| AutoSurvey | S24 | 458 |
| gpt-researcher | S03, S04 | 25.4k |
| Paper2Code | S07, S25 | 4.2k |
| MatPlotAgent | S13 | 105 |
| ChatReviewer | S18, S19 | — |
| SciMON | S02, S27 | — |
| LitLLM | S05 | 21 |
| LLM-SR | S28 | — |
| MLR-Copilot | S01, S09, S11 | 67 |
| OpenResearcher | S03 | 492 |
| ReviewAdvisor | S18 | — |

---

## Recommended Priority for Implementation

### Tier 1 — Core (must-have, daily use)
1. **S15** paper-writing-section — 最常用，每篇论文都需要
2. **S03** literature-search — 基础设施
3. **S17** citation-management — 每篇论文都需要
4. **S06** math-reasoning — 理论论文核心
5. **S13** figure-generation — 每篇论文都需要
6. **S16** latex-formatting — 基础设施
7. **S22** paper-compilation — 基础设施
8. **S18** self-review — 提交前必做

### Tier 2 — High Value (significant time savings)
9. **S01** research-planning — 大幅提升效率
10. **S02** idea-generation — 创新起点
11. **S09** experiment-code — 加速实现
12. **S14** table-generation — 节省大量格式化时间
13. **S05** related-work-writing — 耗时最多的部分之一
14. **S04** literature-review — 深度文献综述
15. **S19** rebuttal-writing — 审稿后必需

### Tier 3 — Advanced (specialized scenarios)
16. **S11** experiment-design — 完善实验设计
17. **S12** data-analysis — 数据密集型研究
18. **S10** code-debugging — 复杂实验调试
19. **S20** paper-revision — 大修场景
20. **S27** novelty-assessment — 提交前评估
21. **S08** atomic-decomposition — 复杂系统论文
22. **S07** algorithm-design — 算法论文
23. **S21** paper-assembly — 端到端自动化

### Tier 4 — Extended (nice-to-have)
24. **S23** backward-traceability — 可重复性
25. **S24** survey-generation — 综述论文
26. **S25** paper-to-code — 复现工作
27. **S26** slide-generation — 报告展示
28. **S28** symbolic-equation — 特定领域
