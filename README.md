# claude-skills

Skills for [Claude Code](https://code.claude.com/).

## Available Skills

### deep-research

Systematic academic literature review in 6 phases. Searches arXiv, Semantic Scholar, and conference proceedings. Produces structured notes, a curated paper database, and a synthesized final report.

**Includes:**
- 7 Python scripts (search, download, extract, database management, BibTeX, report compilation)
- 3 reference guides (API reference, note templates, workflow phases)
- `/research <topic>` slash command

## Installation

### Step 1: Install the skill

```bash
npx skills add lingzhi/claude-skills -g -a claude-code
```

> **Important:** Use the `-g` (global) flag. The scripts use `~/.claude/skills/deep-research/` paths that require global installation.

### Step 2: Install the `/research` command + check dependencies

```bash
git clone https://github.com/lingzhi/claude-skills.git /tmp/claude-skills
/tmp/claude-skills/install.sh
rm -rf /tmp/claude-skills
```

This installs the `/research` slash command and optionally sets up PyMuPDF for PDF extraction.

### Step 3: Configure (optional)

1. **Semantic Scholar API key** (recommended for higher rate limits):
   - Get one at https://www.semanticscholar.org/product/api#api-key
   - Save in `~/keys.md`:
     ```
     S2_API_Key: your-key-here
     ```

2. **Output directory**: Research outputs go to `~/deep-research-output/` by default.

## Usage

In Claude Code:

```
/research transformer architectures for long-context reasoning
```

Or just ask naturally: *"Do a literature review on protein folding with LLMs"*

## Requirements

- Python 3
- PyMuPDF (`pip install PyMuPDF`) — optional, for PDF text extraction
