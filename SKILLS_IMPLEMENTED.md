# Research Paper Writing: Implemented Claude Skills

> Implementation record for 30 skills deployed at `~/.claude/skills/`.
> Companion to [SKILLS_DESIGN.md](./SKILLS_DESIGN.md) which contains the original design specifications.

---

## Summary

| Category | Count | With Scripts | Prompt-Only |
|----------|-------|-------------|-------------|
| Research Discovery & Planning | 6 | 4 | 2 |
| Method Design | 4 | 0 | 4 |
| Experiment Pipeline | 4 | 2 | 2 |
| Paper Writing | 4 | 1 | 3 |
| Figures, Tables & Citations | 4 | 4 | 0 |
| LaTeX & Compilation | 3 | 3 | 0 |
| Review & Polish | 5 | 3 | 2 |
| **Total** | **30** | **17** | **13** |

Scripts total: 27 Python + 7 CJS (Excalidraw) across 15 skill directories.

---

## Phase 0: Research Discovery & Planning

### 1. `deep-research` — Systematic Literature Survey

**Design ref:** Pre-dates SKILLS_DESIGN.md (not in S01-S28). Most script-heavy skill.

**How it works:**
- **Prompt:** Orchestrates a 6-phase literature survey: Frontier (latest conferences) → Survey (35-80 papers) → Deep Dive (8-15 detailed reads) → Code & Tools (GitHub extraction) → Synthesis (cross-paper analysis) → Compilation (final report).
- **Scripts (7):**
  - `search_semantic_scholar.py` — Semantic Scholar API search, returns JSONL with title/authors/year/venue/abstract/citations
  - `search_arxiv.py` — arXiv API search via `arxiv` package
  - `download_papers.py` — Download PDFs from URLs or arXiv IDs
  - `extract_pdf.py` — Extract text from PDF using PyMuPDF
  - `paper_db.py` — Merge, deduplicate, and manage paper databases (JSONL format)
  - `bibtex_manager.py` — Convert JSONL paper records to BibTeX entries
  - `compile_report.py` — Assemble final survey report from phase outputs

**Usage pattern:** Scripts handle API calls, PDF processing, and data management. Prompt guides Claude through analysis, synthesis, and gap identification at each phase.

---

### 2. `literature-search` — Academic Paper Search

**Design ref:** S03

**How it works:**
- **Prompt:** Expands user query into 2-4 complementary searches, runs across 3+ APIs, merges and deduplicates results, ranks by citations (0.3) + recency (0.3) + venue quality (0.2) + relevance (0.2).
- **Scripts (4 own + shared from deep-research):**
  - `search_crossref.py` — CrossRef API search with BibTeX generation, type mapping (article/inproceedings/book). Stdlib-only. *New.*
  - `download_arxiv_source.py` — Search arXiv by title, download source tarball, extract .tex files. Stdlib-only (urllib + xml.etree). *New.*
  - `search_openalex.py` — OpenAlex API with citation count and year filtering
  - Also uses: `deep-research/scripts/search_semantic_scholar.py`, `search_arxiv.py`, `paper_db.py`, `bibtex_manager.py`

**Usage pattern:** Scripts call search APIs and return structured JSONL. Prompt handles query expansion, result ranking, and relevance filtering.

---

### 3. `literature-review` — Multi-Perspective Literature Review

**Design ref:** S04

**How it works:**
- **Prompt:** Generates 3-5 expert personas from different research perspectives. Each persona conducts a multi-turn grounded Q&A conversation (3-5 turns). All conversations are synthesized into a unified knowledge base with inline citations.
- **Scripts:** Shares search scripts from `literature-search` and `deep-research`.

**Usage pattern:** Prompt-driven role-play simulation. Scripts provide the search backbone for grounding expert answers in real literature.

---

### 4. `idea-generation` — Research Idea Generation

**Design ref:** S02

**How it works:**
- **Prompt:** Generates 3-5 diverse research ideas, each with Name/Title/Experiment plan. Iterative reflection (up to 5 rounds) to refine. Scores each idea on Interestingness, Feasibility, Novelty (1-10 scale).
- **Scripts (1):**
  - `novelty_check.py` — Searches Semantic Scholar for similar work, evaluates overlap in multiple rounds

**Usage pattern:** Prompt generates and refines ideas. Script validates novelty against existing literature.

---

### 5. `novelty-assessment` — Research Novelty Evaluation

**Design ref:** S27

**How it works:**
- **Prompt:** Adopts a harsh critic persona. Runs up to 10 rounds of search-evaluate loops. Final output is a binary Novel/Not Novel decision with justification. Identifies the most similar existing papers and explains differentiation.
- **Scripts:** Shares `idea-generation/scripts/novelty_check.py` and `deep-research/scripts/search_semantic_scholar.py`.

**Usage pattern:** Prompt drives the adversarial evaluation. Scripts perform iterative literature searches.

---

### 6. `research-planning` — Research Plan Design

**Design ref:** S01

**How it works:**
- **Prompt:** 4-stage planning framework: Overall Plan → Architecture Design (UML) → Logic Design (task list) → Configuration. Outputs paper structure, section-by-section plan, dependency-ordered task graph, baselines, datasets, metrics, and risk flags.
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Claude produces structured research plans as JSON or Markdown.

---

## Phase 1: Method Design

### 7. `atomic-decomposition` — Atomic Concept Decomposition

**Design ref:** S08

**How it works:**
- **Prompt:** Decomposes a complex research idea into atomic, self-contained concepts. For each concept, performs a Paper Survey (extract math formula from papers) and Code Survey (find implementation in repos). Creates bidirectional mapping: `math_formula ↔ code_implementation`.
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Claude reads papers and code to build a structured knowledge base of atomic definitions.

---

### 8. `algorithm-design` — Algorithm Design & Pseudocode

**Design ref:** S07

**How it works:**
- **Prompt:** Formalizes algorithms with clear input/output/parameters. Generates LaTeX pseudocode using `algorithm` + `algpseudocode` environments. Creates Mermaid UML class diagrams and sequence diagrams. Verifies consistency: every pseudocode step maps to a code module.
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Outputs LaTeX algorithm environments and Mermaid diagram code.

---

### 9. `math-reasoning` — Mathematical Reasoning

**Design ref:** S06

**How it works:**
- **Prompt:** Six task modes:
  - `derive` — Step-by-step equation derivation with justifications, boxed final result
  - `prove` — Formal theorem proof using appropriate technique (induction, contradiction, construction)
  - `formalize` — Problem setting formalization with variable definitions and assumptions
  - `stats` — Statistical test selection and proper reporting
  - `notation` — Generate notation table with all symbols used in the paper
  - `verify` — Check mathematical correctness of existing derivations
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Outputs LaTeX math notation.

---

### 10. `symbolic-equation` — Scientific Equation Discovery

**Design ref:** S28

**How it works:**
- **Prompt:** Implements the LLM-SR framework: LLM-guided evolutionary search over symbolic expression space. Multi-island algorithm with softmax-based cluster sampling for diversity. Island reset mechanism prevents premature convergence. LLM proposes candidate equations, evaluates fitness against data, iteratively improves.
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Claude generates and evaluates candidate equations in an evolutionary loop.

---

## Phase 2: Experiment Pipeline

### 11. `experiment-design` — Experiment Plan Design

**Design ref:** S11

**How it works:**
- **Prompt:** 4-stage progressive framework (from AI-Scientist-v2):
  - Stage 1: Initial Implementation — basic working baseline on simple dataset
  - Stage 2: Baseline Tuning — hyperparameters on 2+ datasets, no architecture changes
  - Stage 3: Creative Research — novel improvements on 3+ datasets
  - Stage 4: Ablation Studies — systematic component analysis
- **Scripts (1):**
  - `design_experiments.py` — Generates experiment design JSON/Markdown: baseline list, ablation matrix, hyperparameter grid, metric selection by task type (classification/regression/generation/detection/segmentation/retrieval), total run estimation. Stdlib-only. *New.*

**Usage pattern:** Script generates the structural experiment plan. Prompt fills in domain-specific details and rationale.

---

### 12. `experiment-code` — Experiment Code Writing

**Design ref:** S09

**How it works:**
- **Prompt:** Three actions:
  - `generate` — Create complete training/evaluation pipeline with logging and figure generation
  - `improve` — Read results, reflect on what worked, apply targeted edits, re-run and compare
  - `debug` — Identify root cause, apply minimal fix, up to 4 retries
- Enforces constraints: no placeholder code, must use real datasets, must generate figures, PyTorch/scikit-learn only.
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Claude writes, executes, and iteratively improves experiment code.

---

### 13. `code-debugging` — Code Debugging

**Design ref:** S10

**How it works:**
- **Prompt:** Structured error analysis pipeline:
  1. Categorize error (SyntaxError, ImportError, RuntimeError, TimeoutError, OutputError, LogicError)
  2. Analyze root cause from traceback
  3. Apply targeted fix (up to 4 retries)
  4. Reflect: explain error, identify lines, describe fix, note patterns to avoid
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Claude follows the categorize→analyze→fix→reflect workflow.

---

### 14. `data-analysis` — Statistical Data Analysis

**Design ref:** S12

**How it works:**
- **Prompt:** Generates analysis code in 7 sections (IMPORT → LOAD DATA → DATASET PREPARATIONS → DESCRIPTIVE STATISTICS → PREPROCESSING → ANALYSIS → SAVE). Then runs a 4-round code review: Round 1 (code flaws) → Round 2 (data handling) → Round 3 (per-table) → Round 4 (cross-table completeness). Statistical test selection table guides appropriate test choice.
- **Scripts (2):**
  - `stat_summary.py` — Loads CSV/JSON, detects data types, recommends statistical tests (t-test, Mann-Whitney, Wilcoxon, ANOVA, Kruskal-Wallis), computes effect sizes (Cohen's d), outputs significance stars. Requires numpy + scipy. *New.*
  - `format_pvalue.py` — Formats p-values as text, significance stars (`*`/`**`/`***`/`ns`), LaTeX notation, or JSON. Supports batch processing from CLI values, CSV, or stdin. Stdlib-only. *New.*

**Usage pattern:** Scripts handle statistical computation and formatting. Prompt performs the 4-round review and generates the full analysis code.

---

## Phase 3: Paper Writing

### 15. `paper-writing-section` — Section-by-Section Writing

**Design ref:** S15

**How it works:**
- **Prompt:** Writes each paper section with section-specific guidance:

  | Section | Guidance |
  |---------|----------|
  | Abstract | TL;DR → why hard → contribution → how verified. Single paragraph. |
  | Introduction | Longer abstract; list contributions as bullet points. |
  | Background | Problem setting with formal notation; define all symbols. |
  | Related Work | Compare and contrast, not just describe. Organize by theme. |
  | Methods | Precise equations; what we do AND why. |
  | Experiments | Datasets, metrics, hyperparameters. No hallucinated numbers. |
  | Results | Only from actual logs. Include ablations and all figures. |
  | Discussion | Brief recap → limitations → future work. |

  Two-pass refinement: Pass 1 (fix errors, unenclosed math, broken refs, hallucinated numbers) → Pass 2 (remove redundancies, compress, smooth transitions).
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Claude reads existing materials, writes the section, then self-refines in two passes.

---

### 16. `related-work-writing` — Related Work Section

**Design ref:** S05

**How it works:**
- **Prompt:** 4-step process:
  1. Analyze paper's key contributions and novelty claims
  2. Organize literature into thematic clusters
  3. Write each theme paragraph: topic sentence → describe key works → compare/contrast with this paper
  4. Refine: verify citation reasons are clear, novelty is explicit, all cite keys resolve
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Focused specifically on the Related Work section with compare-and-contrast emphasis.

---

### 17. `survey-generation` — Survey Paper Generation

**Design ref:** S24

**How it works:**
- **Prompt:** AutoSurvey pipeline:
  1. Collect 50-200 papers via Semantic Scholar/arXiv
  2. Generate N outlines in parallel, merge best elements
  3. RAG-based subsection writing: retrieve relevant papers per subsection, generate with inline citations
  4. Validate citations: check titles match, verify claims are supported
  5. Enhance local coherence: read adjacent sections, refine transitions
  6. Convert paper title citations to BibTeX `\cite{key}` format
- **Scripts:** Shares `deep-research/scripts/search_semantic_scholar.py`.

**Usage pattern:** Prompt orchestrates the multi-step RAG pipeline. Script provides the search backend.

---

### 18. `paper-to-code` — Paper to Runnable Code

**Design ref:** S25

**How it works:**
- **Prompt:** Paper2Code 3-stage pipeline:
  1. Planning — Overall plan, architecture design (UML), task breakdown, configuration extraction
  2. Analysis — Per-file detailed logic analysis
  3. Coding — Dependency-ordered code generation, each file sees all previously generated files
  4. Debugging — If execution fails, identify root cause and apply fixes
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Claude reads the paper, designs the architecture, then generates code file by file in dependency order.

---

## Phase 4: Figures, Tables & Citations

### 19. `figure-generation` — Scientific Figure Generation

**Design ref:** S13

**How it works:**
- **Prompt:** 3-phase pipeline:
  1. Query Expansion — expand user description into step-by-step coding specifications
  2. Code Generation with Execution Loop — generate matplotlib/seaborn script, execute, fix errors (up to 4 retries)
  3. Visual Refinement — read generated PNG, inspect using VLM feedback, generate corrections
- **Scripts (1):**
  - `figure_template.py` — Generates starter matplotlib code for 10 figure types: `bar`, `line`, `heatmap`, `scatter`, `training_curve`, `ablation`, `radar`, `violin`, `tsne`, `attention`. The last 4 templates were added in the upgrade. *Updated.*

**Usage pattern:** Script generates code scaffolds for common figure types. Prompt handles the iterative refinement loop with VLM feedback.

---

### 20. `table-generation` — LaTeX Table Generation

**Design ref:** S14

**How it works:**
- **Prompt:** Converts JSON/CSV experiment results to publication-ready LaTeX tables with `booktabs` styling, bold best results, proper captions and labels. Supports comparison, ablation, descriptive, and multi-dataset table types.
- **Scripts (1):**
  - `results_to_table.py` — Converts JSON/CSV data to LaTeX. 4 table types: `comparison` (methods × metrics), `ablation` (variants × metrics), `descriptive` (dataset stats), `multi-dataset` (methods × datasets × metrics). Flags: `--bold-best`, `--significance` (p-value stars), `--underline-second` (second-best results). *Updated.*

**Usage pattern:** Script handles the mechanical conversion. Prompt decides table layout and which results to include.

---

### 21. `citation-management` — BibTeX Citation Management

**Design ref:** S17

**How it works:**
- **Prompt:** Four actions:
  - `harvest` — Iterative citation harvesting: scan draft for uncited claims, search Semantic Scholar, add candidate BibTeX entries (up to 20 rounds)
  - `validate` — Pre-compilation check: every `\cite{key}` must exist in `.bib`, every `\includegraphics` must exist
  - `add` — Add a specific paper by title or DOI
  - `format` — Standardize and deduplicate `.bib` file
- **Scripts (2 own + shared):**
  - `harvest_citations.py` — Scans .tex for sentences lacking `\cite`, searches Semantic Scholar API, outputs candidate BibTeX. Stdlib-only. *New.*
  - `validate_citations.py` — Checks cite keys vs .bib, label vs ref consistency, figure file existence. `--fix` mode generates placeholder entries for missing keys. *Updated.*
  - Also uses: `deep-research/scripts/bibtex_manager.py`, `search_semantic_scholar.py`

**Usage pattern:** Scripts automate search and validation. Prompt handles citation selection and integration into paper text.

---

### 22. `backward-traceability` — Numeric Value Traceability

**Design ref:** S23

**How it works:**
- **Prompt:** Ensures every number in the final PDF traces to the exact code line that produced it. Workflow:
  1. Tag code outputs with `\hypertarget{label}{value}`
  2. Reference in paper with `\hyperlink{label}{value}`
  3. Use `\num{formula}` for derived values (compile-time evaluation)
  4. Generate appendix code listing with hypertarget anchors
  5. Verify all hyperlinks resolve correctly
- **Scripts (1):**
  - `ref_numeric_values.py` — Two modes: `--scan` (report all hypertarget/hyperlink usage, orphan references, unreferenced numbers) and `--verify` (cross-reference integrity between .tex and code output, value mismatch detection). Stdlib-only. *New.*

**Usage pattern:** Script scans and verifies traceability. Prompt guides how to add hypertarget/hyperlink tags.

---

## Phase 5: LaTeX & Compilation

### 23. `latex-formatting` — LaTeX Formatting & Templates

**Design ref:** S16

**How it works:**
- **Prompt:** Three actions:
  - `setup` — Create project directory with conference template (ICML, ICLR, NeurIPS, AAAI, ACL)
  - `fix` — Fix common LaTeX issues: unescaped characters, math mode errors, float placement, cross-references
  - `check` — Pre-submission validation: word count, section structure, anonymization, citation consistency
- **Scripts (2):**
  - `latex_checker.py` — Checks word count, section completeness (flags missing expected sections), citation/figure/equation counts, venue-specific rules, anonymization. `--fix` mode calls `clean_latex.py` after checking. *Updated.*
  - `clean_latex.py` — Replaces special characters with LaTeX equivalents (28 special chars + 20 non-UTF8 chars). Skips math environments, comments, command definitions, tabular environments, and LaTeX commands. Stdlib-only. *New.*

**Usage pattern:** Scripts handle automated checking and cleaning. Prompt does venue-specific template setup and complex formatting fixes.

---

### 24. `paper-compilation` — LaTeX Compilation

**Design ref:** S22

**How it works:**
- **Prompt:** Full compilation pipeline: pdflatex → bibtex → pdflatex → pdflatex. Pre-compilation validation, up to 5 rounds of error correction, post-compilation report (page count, warnings, style issues).
- **Scripts (2):**
  - `compile_paper.py` — Runs the full pdflatex+bibtex pipeline, optional `chktex` style checking. `--auto-fix` flag runs `fix_latex_errors.py` + recompile up to 3 rounds automatically. *Updated.*
  - `fix_latex_errors.py` — Parses pdflatex `.log` files, classifies errors (undefined commands, missing math mode, mismatched environments, missing files), applies automated fixes: HTML tag conversion, environment balancing, missing figure commenting. `--dry-run` mode for preview. Stdlib-only. *New.*

**Usage pattern:** Scripts automate the compile-fix-recompile cycle. Prompt handles complex errors that require understanding paper content.

---

### 25. `excalidraw-skill` — Excalidraw Diagramming

**Design ref:** Not in S01-S28. Uses MCP tools, not traditional scripts.

**How it works:**
- **Prompt:** Programmatic canvas control via MCP Server tools. Mandatory quality gate after every diagram: check text truncation, element overlap, arrow crossing, spacing, and readability. Workflows include: Draw (plan grid → create elements → bind arrows → verify), Iterative Refinement (screenshot → evaluate → fix), File I/O (export/import .excalidraw), and Sharing (export to excalidraw.com URL).
- **Scripts (7 CJS):** MCP server implementation files (not called directly by users).

**Usage pattern:** Claude calls MCP tools (`create_element`, `batch_create_elements`, `describe_scene`, `get_canvas_screenshot`) to manipulate a live Excalidraw canvas in the browser.

---

## Phase 6: Review & Polish

### 26. `self-review` — Automated Paper Review

**Design ref:** S18

**How it works:**
- **Prompt:** Simulates peer review using the NeurIPS review form. Three independent reviewer personas (harsh-fair, harsh-critical, open-minded) each produce a full review. Reflection refinement loop (up to 3 rounds). Reviews are aggregated into a meta-review with averaged scores. Scores: Originality, Quality, Clarity, Significance, Soundness, Presentation, Contribution (1-4), Overall (1-10).
- **Scripts (2):**
  - `extract_pdf_text.py` — Extracts raw text from PDF, outputs as plain text or markdown
  - `parse_pdf_sections.py` — Parses PDF into structured sections using PyMuPDF font-size analysis. Detects title (largest font), headings (ALL CAPS or larger font), and body text. Outputs `{title, pages, sections: [{name, text, page}]}`. Requires pymupdf. *New.*

**Usage pattern:** Scripts extract paper content from PDF. Prompt runs 3 independent reviews, refines, and aggregates.

---

### 27. `paper-revision` — Paper Revision from Reviews

**Design ref:** S20

**How it works:**
- **Prompt:** 5-step process:
  1. Parse reviewer concerns — extract, classify (major/minor), map to specific paper sections, prioritize
  2. Plan revisions — create mapping: Concern → Section → Action → New Content
  3. Execute revisions — read section, apply edits, run additional experiments if needed, mark changes with `\revised{}`
  4. Verify improvements — re-run self-review, check all concerns addressed, check page count
  5. Write revision summary — list all changes with cross-references to reviewer concerns
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Claude systematically addresses each reviewer concern.

---

### 28. `rebuttal-writing` — Rebuttal Writing

**Design ref:** S19

**How it works:**
- **Prompt:** Extracts reviewer concerns one by one. For each concern, generates a response following the pattern: Acknowledge → Respond with evidence → Describe what was done (not what will be done). Outputs formatted rebuttal with `## Reviewer #N` / `**Concern #N**` / `**Author Response**` structure.
- **Scripts:** None.

**Usage pattern:** Purely prompt-driven. Emphasis on evidence-based responses to specific concerns.

---

### 29. `slide-generation` — Presentation Slide Generation

**Design ref:** S26

**How it works:**
- **Prompt:** Converts a completed paper into Beamer LaTeX slides or poster. Standard flow: extract key content → design 15-20 slide structure → generate Beamer code → simplify for presentation (max 1 key message per slide, max 6 bullets, large figures). Optional poster layout with 4-column structure.
- **Scripts (1):**
  - `extract_paper_elements.py` — Parses .tex (resolves `\input{}` directives), extracts title, authors, abstract, sections (with hierarchy), figures (path + caption + label), equations, and tables. Generates complete Beamer skeleton or raw JSON. Supports theme selection. Stdlib-only. *New.*

**Usage pattern:** Script extracts paper structure and generates slide skeleton. Prompt fills in content and optimizes the narrative flow.

---

### 30. `paper-assembly` — End-to-End Paper Orchestrator

**Design ref:** S21

**How it works:**
- **Prompt:** Manages the full 9-phase paper pipeline: literature → planning → code → results → figures → tables → bibliography → sections → compilation. Supports checkpointing after each phase and resumption. Quality gates verify outputs before proceeding. State propagation passes results to downstream phases.
- **Scripts (1):**
  - `assembly_checker.py` — Scans a paper project directory, checks completeness of all 9 pipeline phases, analyzes .tex section coverage (abstract, introduction, method, experiment, conclusion), verifies citation cross-references, reports missing artifacts, suggests next steps with skill recommendations. Stdlib-only. *New.*

**Usage pattern:** Script assesses current pipeline state. Prompt orchestrates calls to other skills in dependency order.

---

## Skill Dependency Graph

Every SKILL.md includes a `## Related Skills` section linking upstream, downstream, and complementary skills. The full graph:

```
deep-research ──→ literature-search ──→ citation-management ──→ paper-compilation
                  literature-review      related-work-writing     latex-formatting
                  novelty-assessment     survey-generation

idea-generation ──→ research-planning ──→ experiment-design ──→ experiment-code
                    atomic-decomposition                        code-debugging

experiment-code ──→ data-analysis ──→ figure-generation ──→ paper-writing-section
                                      table-generation      related-work-writing
                                      backward-traceability

paper-writing-section ──→ latex-formatting ──→ paper-compilation ──→ self-review
citation-management                                                   paper-revision
                                                                      rebuttal-writing

paper-assembly (orchestrator) ──→ all skills above ──→ slide-generation
```

---

## Scripts Inventory

### New Scripts (12)

| Script | Skill | Lines | Dependencies | Source |
|--------|-------|-------|-------------|--------|
| `search_crossref.py` | literature-search | ~260 | stdlib | data-to-paper crossref.py |
| `download_arxiv_source.py` | literature-search | ~230 | stdlib | AI-Researcher arxiv.py |
| `harvest_citations.py` | citation-management | ~245 | stdlib | AI-Scientist citation loop pattern |
| `clean_latex.py` | latex-formatting | ~240 | stdlib | data-to-paper clean_latex.py |
| `fix_latex_errors.py` | paper-compilation | ~305 | stdlib | data-to-paper + AI-Scientist patterns |
| `parse_pdf_sections.py` | self-review | ~260 | pymupdf | ChatReviewer get_paper_from_pdf.py |
| `ref_numeric_values.py` | backward-traceability | ~265 | stdlib | data-to-paper ref_numeric_values.py |
| `stat_summary.py` | data-analysis | ~320 | numpy, scipy | data-to-paper 4-round review pattern |
| `format_pvalue.py` | data-analysis | ~145 | stdlib | data-to-paper pvalue.py |
| `design_experiments.py` | experiment-design | ~275 | stdlib | AI-Scientist-v2 4-stage pattern |
| `assembly_checker.py` | paper-assembly | ~290 | stdlib | New |
| `extract_paper_elements.py` | slide-generation | ~270 | stdlib | New |

### Updated Scripts (5)

| Script | Skill | Change |
|--------|-------|--------|
| `validate_citations.py` | citation-management | Added `--fix` auto-fix mode |
| `compile_paper.py` | paper-compilation | Added `--auto-fix` flag (fix + recompile 3 rounds) |
| `latex_checker.py` | latex-formatting | Added `--fix` flag (calls clean_latex.py) |
| `figure_template.py` | figure-generation | Added 4 templates: radar, violin, tsne, attention |
| `results_to_table.py` | table-generation | Added multi-dataset type, --significance, --underline-second |

### Pre-existing Scripts (17)

| Skill | Scripts |
|-------|---------|
| deep-research | search_semantic_scholar.py, search_arxiv.py, download_papers.py, extract_pdf.py, paper_db.py, bibtex_manager.py, compile_report.py |
| literature-search | search_openalex.py |
| citation-management | validate_citations.py (pre-upgrade) |
| figure-generation | figure_template.py (pre-upgrade) |
| idea-generation | novelty_check.py |
| latex-formatting | latex_checker.py (pre-upgrade) |
| paper-compilation | compile_paper.py (pre-upgrade) |
| self-review | extract_pdf_text.py |
| table-generation | results_to_table.py (pre-upgrade) |
| excalidraw-skill | 7 CJS files (MCP server) |

---

## Verification Results

Tested against real paper: RIGID metamaterials (OpenResearcher/2401.00003, with main.tex + references.bib + 14 figures).

| Test | Script | Result |
|------|--------|--------|
| CrossRef API search | `search_crossref.py --query "attention mechanism transformer" --rows 3` | PASS — 3 results |
| LaTeX error fixer | `fix_latex_errors.py --tex main.tex --dry-run` | PASS — no fixes needed |
| PDF section parser | `parse_pdf_sections.py --pdf report.pdf --format json` | PASS — title + 7 sections |
| Citation validator | `validate_citations.py --tex main.tex --bib references.bib` | PASS — 71 citations, 7 unused entries |
| Beamer skeleton | `extract_paper_elements.py --tex main.tex --format beamer` | PASS — 19 sections, 12 figures, 3 equations |
| Assembly checker | `assembly_checker.py --dir paper/ --verbose` | PASS — 2/9 phases complete |
| LaTeX cleaner | `clean_latex.py --input main.tex --dry-run` | PASS — 2 legitimate changes |
| LaTeX checker | `latex_checker.py main.tex` | PASS — 7515 words, 12 sections |
| Traceability scan | `ref_numeric_values.py --scan main.tex` | PASS — 0 targets (paper has none) |
| P-value formatter | `format_pvalue.py --values "0.0001 0.003 0.012 0.048 0.067 0.5" --format latex` | PASS |
| Experiment design | `design_experiments.py --method "inverse design" --task classification` | PASS |
| Element extraction | `extract_paper_elements.py --tex main.tex --format json` | PASS |

Bugs found and fixed during verification:
1. `parse_pdf_sections.py`: `len(doc)` called after `doc.close()` — fixed by saving page count before closing
2. `clean_latex.py`: Escaped `%` in comments and `#` in `\newcommand` — fixed by adding comment/command-definition skip patterns; removed `~` from escape chars (it's a valid LaTeX tie); added `tabular`/`array` to skip environments
