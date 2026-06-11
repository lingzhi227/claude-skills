# Security Policy

## Overview

This repository contains research automation skills that interact with external APIs and process third-party academic content. This document outlines security considerations and best practices.

## API Key Management

### Semantic Scholar API Key

The literature search functionality can optionally use a Semantic Scholar API key for higher rate limits.

**✅ SECURE METHOD (Recommended):**

Use environment variables to store your API key:

```bash
# Add to your shell profile (~/.zshrc, ~/.bashrc, etc.)
export S2_API_KEY="your-api-key-here"
```

Then restart your terminal or run `source ~/.zshrc`

The scripts will automatically use this environment variable when available.

**❌ INSECURE METHODS (DO NOT USE):**

- Hardcoding API keys in scripts or command lines
- Storing keys in plaintext files that are version controlled
- Passing keys as command-line arguments (visible in process list)

### Getting an API Key

1. Visit <https://www.semanticscholar.org/product/api#api-key>
2. Sign up for a free API key
3. Set the environment variable as shown above

### Running Without an API Key

All scripts work without an API key, with lower rate limits:

- Without key: ~100 requests per 5 minutes
- With key: ~1 request per second sustained

## Third-Party Content Risks

### Untrusted Data Sources

The literature search and deep research skills download and process content from external sources:

- **arXiv preprints**: User-submitted, not peer-reviewed
- **OpenAlex**: Aggregates from multiple sources
- **CrossRef**: Publisher metadata
- **LaTeX source files**: May contain arbitrary commands

### Risk Mitigation

1. **Treat all downloaded content as untrusted**
   - Review downloaded papers before using citations
   - Be cautious when compiling LaTeX source from arXiv
   - Verify claims from preprints against peer-reviewed sources

2. **Content Sanitization**
   - The skills prioritize peer-reviewed conference papers over preprints
   - Scripts mark preprint status explicitly
   - Use `--peer-reviewed-only` flag when available

3. **LaTeX Compilation Safety**
   - Review `.tex` files before compilation
   - Use `--shell-escape` carefully (required by some packages)
   - Consider compiling in isolated environments

## Installation Security

### Global Installation

This skill package is designed for global installation at `~/.agent/skills/`:

```bash
# Recommended: Review the repository before installation
git clone https://github.com/lingzhi227/agent-research-skills.git
cd agent-research-skills
# Review install.sh and skill contents
./install.sh
```

### Path Configuration

All skills use `~/.agent/skills/` as the base directory, making them harness-agnostic and compatible with:

- Claude Code
- Other AI agent frameworks
- Direct command-line usage

## Script Execution

### Python Scripts

All Python scripts:

- Use only standard library where possible
- Accept input via `argparse` (prevents injection)
- Do not use `eval()` or `exec()` on user input
- Validate file paths before operations

### Command Injection Prevention

Scripts use parameterized arguments instead of shell interpolation:

```bash
# ✅ SAFE: Parameterized
python script.py --query "$USER_INPUT"

# ❌ UNSAFE: Shell interpolation
python script.py --query "$(echo $USER_INPUT)"
```

## Reporting Security Issues

If you discover a security vulnerability:

1. **DO NOT** open a public issue
2. Email the maintainer directly
3. Include:
   - Description of the vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)

## Security Checklist for Users

Before using these skills:

- [ ] Set `S2_API_KEY` environment variable (optional)
- [ ] Review what external APIs are accessed
- [ ] Understand that downloaded papers are untrusted content
- [ ] Keep Python and dependencies updated
- [ ] Use peer-reviewed sources when possible
- [ ] Review LaTeX files before compilation

## Updates and Maintenance

- Check for security updates regularly
- Review CHANGELOG.md for security-related changes
- Update Python dependencies: `pip install --upgrade PyMuPDF numpy scipy`

## License

See LICENSE file for terms and conditions.
