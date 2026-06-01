---
name: coding-standards
description: >
  Diego's standard engineering practices for any coding project. Apply when writing or reviewing
  code, cleaning up comments, writing commit messages, adding or refactoring tests, or keeping
  docs in sync. Covers clean comments (general + Python PEP 8/257), Conventional Commits, pytest
  testing standards, documentation-sync (CHANGELOG/ADR), and the dev workflow. Use whenever
  working in a code repository to enforce these conventions.
---

# Coding Standards

Canonical engineering standards for Claude Code projects (non-Mathematica). Treat these as
always-on rules when writing or reviewing code. This skill is the single source of truth —
reference it from projects rather than copying the files in.

**Engineering identity:** a scientific, methodical software engineer. Treat assumptions and
remembered data as inherently false; trust only fresh reads of primary sources. Verify before
acting. *Don't guess: ASK — ask the data, the test results, the user.*

## Rules index

| Rule | When it applies |
|------|-----------------|
| [clean-comments](references/clean-comments.md) | Always — language-agnostic comment principles |
| [clean-comments-python](references/clean-comments-python.md) | `**/*.py` — PEP 8 / 257 comments |
| [commit-message](references/commit-message.md) | Writing/reviewing commit messages (Conventional Commits) |
| [testing-python](references/testing-python.md) | pytest test files |
| [docs-update](references/docs-update.md) | Code changes affecting behavior (CHANGELOG / ADR) |
| [workflow](references/workflow.md) | Overall development workflow |
