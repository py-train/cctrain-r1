---
name: quick-review
description: Quick code review with quality analysis. Use when asked for a quick review to analyze for code style and quality. Typical usage on ongoing basis while developing or initial review of a code-base.
argument-hint: "[target-directory]"
model: haiku
context: fork
---

# Code Review Skill

This skill provides quick and cheap code review capabilities focusing on:

1. **Code Quality**
   - SOLID principles
   - Design patterns
   - Naming conventions
   - Documentation

2. **Maintainability**
   - Code readability
   - Function size (should be < 50 lines)
   - Cyclomatic complexity
   - Dependency management
   - Type safety

## Arguments

- **Target Path (`$0`)**: The relative path to the subdirectory Claude must isolate its operations to. If not provided, default to project/repository root

## Context Boundary
Do not review any files outside of argument directory `$0`.
Access to reference files is allowed.


## Reference Files

This skill includes supporting files that you should read when performing reviews:

- **`templates/review-checklist.md`** — Structured checklist covering security, performance, quality, and testing. Read this file and use it as a guide to ensure no category is missed during review.
- **`templates/finding-template.md`** — Standard template for documenting individual findings with severity, location, code examples, and impact analysis. Read this file and use its format when reporting issues.
- **`scripts/analyze-metrics.py`** — Python script that calculates code metrics (function count, class count, average line length, complexity score). Run this on the file under review to gather quantitative data.

## Review Template

For each piece of code reviewed, provide:

### Summary
- Overall quality assessment (1-5)
- Key findings count
- Recommended priority areas

### Critical Issues (if any)
- **Issue**: Clear description
- **Location**: File and line number
- **Impact**: Why this matters
- **Severity**: Critical/High/Medium
- **Fix**: Code example


---

**Last Updated**: August 4, 2026
**Claude Code Version**: 2.1.220
**Sources**:
- https://code.claude.com/docs/en/skills
**Compatible Models**: Claude Fable 5, Claude Opus 5, Claude Sonnet 5, Claude Sonnet 4.6, Claude Opus 4.8, Claude Haiku 4.5
