---
description: "Documentation sync protocol. Reference when making code changes that affect user-facing behavior."
---

# Skill: Docs Update

Keep documentation in sync with code. Docs that lag code are worse than no docs — they actively mislead. Update docs **in the same PR as the code change**, never in a follow-up.

---

## What Triggers a Docs Update

| Change type | Docs to update |
|-------------|---------------|
| New public API / endpoint / function | API reference, README usage section, docstrings, CHANGELOG → Added |
| Bug fix | CHANGELOG → Fixed; docstring if it described the wrong behavior |
| Breaking change | CHANGELOG → Changed / Removed, migration guide, README examples, ADR if architectural |
| New config option | README configuration section, example config files |
| Dependency change | README prerequisites, install docs |
| Performance improvement | CHANGELOG → Changed |
| Deprecation | CHANGELOG → Deprecated, docstring with deprecation notice |
| Removed feature | CHANGELOG → Removed, migration guide, README (remove examples) |
| New architectural decision | ADR in `docs/decisions/` |
| Release | Promote CHANGELOG `[Unreleased]` to version + date |

---

## CHANGELOG — Keep a Changelog Format

**File:** `CHANGELOG.md` in the project root.

### Structure

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Fuzzy matching to product search (#301)

### Fixed
- Race condition in connection pool under high concurrency (#287)

## [1.2.0] - 2025-11-14

### Added
- `--dry-run` flag to the deploy command
- OAuth2 PKCE flow for web clients

### Changed
- `authenticate()` now accepts a `Credentials` object instead of raw strings

### Deprecated
- `authenticate(password=)` keyword argument; use `Credentials` instead. Removed in v2.0.0.

### Removed
- Python 3.8 support

### Fixed
- Double-submission on slow network connections (#887)

### Security
- Upgrade `cryptography` to 42.0.5 (CVE-2024-12345)

## [1.1.0] - 2025-09-01
...

[Unreleased]: https://github.com/org/repo/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/org/repo/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/org/repo/releases/tag/v1.1.0
```

### The Six Change Categories

| Category | Use for |
|----------|---------|
| `Added` | New features |
| `Changed` | Changes to existing functionality |
| `Deprecated` | Features flagged for future removal |
| `Removed` | Features removed in this version |
| `Fixed` | Bug fixes |
| `Security` | Vulnerability patches |

### Rules

- **Latest version first.**
- **Dates in ISO 8601:** `YYYY-MM-DD`.
- **Written for humans** — not a raw list of commit messages.
- **The `[Unreleased]` section** accumulates entries throughout development. At release, rename it to the version + date, then create a fresh `[Unreleased]` section above it.
- **Comparison links** at the bottom keep every version linkable.
- **Do not include:** individual commit SHAs, internal refactors invisible to users, test-only changes, formatting changes.

### Conventional Commits → CHANGELOG

| Commit type | CHANGELOG section |
|-------------|------------------|
| `feat:` | Added |
| `fix:` | Fixed |
| `BREAKING CHANGE` / `!` | Changed or Removed |
| `perf:` | Changed |
| `docs:`, `style:`, `refactor:`, `test:`, `build:`, `ci:`, `chore:` | Omit |
| `feat:` + security advisory | Security |

### Deprecation Entry Pattern

```markdown
### Deprecated
- `authenticate(password=)` keyword argument. Use `authenticate(Credentials(...))`
  instead. Will be removed in v2.0.0.
```

---

## Architecture Decision Records (ADRs)

### When to Write an ADR

Write an ADR whenever a decision has significant, lasting impact on the codebase or team.

**Write an ADR when:**
- Choosing between two viable frameworks, databases, or protocols
- Adopting a new architectural pattern (event sourcing, CQRS, microservices)
- Changing a security or authentication model
- Deciding to *not* do something significant (captures the rejection for future reference)
- Any decision that, if reversed later, would require a team-wide migration

**Skip the ADR for:**
- Small config changes or version bumps
- Bug fixes with no design implications
- Decisions already universally understood

### Storage

```
docs/
└── decisions/
    ├── 0001-use-postgresql-as-primary-datastore.md
    ├── 0002-use-opentelemetry-for-tracing.md
    └── 0003-adopt-event-sourcing-for-orders.md
```

Number sequentially. **Never delete or renumber.** When a decision is reversed, mark the old ADR as "Superseded by ADR-0007" and write a new one.

### Nygard Format (1–2 pages, sufficient for most decisions)

```markdown
# ADR 0001: Use PostgreSQL as the primary datastore

## Status
Accepted

## Context
The application requires ACID-compliant transactions, complex relational
queries, and JSON document storage. The team has strong PostgreSQL
expertise. Hosting is on AWS.

## Decision
Use PostgreSQL 16 on AWS RDS. Use `jsonb` columns for semi-structured
data rather than a separate document store.

## Consequences
- Positive: Single datastore reduces operational complexity.
- Positive: Full ACID guarantees simplify error handling.
- Negative: Less horizontal write scalability than DynamoDB.
- Neutral: Schema migrations managed via Alembic.
```

**Status values:** `Proposed` → `Accepted` → `Deprecated` / `Superseded by ADR-0007`

### MADR Format (for higher-stakes decisions requiring more rigor)

```markdown
---
status: accepted
date: 2025-11-14
decision-makers: [Alice, Bob]
consulted: [Platform Team]
---

# Use OpenTelemetry for distributed tracing

## Context and Problem Statement
We need consistent trace propagation across 12 services in Python, Go,
and Node.js. Currently each service uses a vendor-specific SDK.

## Decision Drivers
- Avoid vendor lock-in
- W3C TraceContext compliance
- Single SDK per language

## Considered Options
- OpenTelemetry SDK
- Datadog APM SDK directly
- Jaeger client libraries only

## Decision Outcome
Chosen: OpenTelemetry SDK — vendor-neutral; all target vendors have
OTLP receivers.

### Consequences
- Good: Single instrumentation pattern across all languages.
- Bad: Python SDK is beta-quality at time of decision.

## Pros and Cons of the Options

### OpenTelemetry SDK
- Pro: Vendor neutral, W3C compliant
- Con: Beta status for Python SDK

### Datadog APM SDK
- Pro: Mature, well-documented
- Con: Vendor lock-in
```

---

## README — Sections That Go Stale

| Section | Staleness risk | Rule |
|---------|---------------|------|
| Installation / Prerequisites | High | Review on every `build:` commit; pin versions explicitly |
| Quick Start / Usage examples | High | Test examples in CI; review on every `feat:` or breaking change |
| Configuration reference | High | Generate from schema if possible; review on every config option change |
| Screenshots / GIF demos | Very high | Label with version; regenerate at each minor release |
| Roadmap | High | Derive from issue labels or remove entirely |
| Build / coverage badges | Medium | Use dynamic badge services |
| Contributing guide | Medium | Link to `CONTRIBUTING.md` rather than embedding |

**Rule:** Any code example in a README must be tested. If it cannot be tested automatically, it will eventually be wrong.

---

## Docs-Update Checklist

Run this checklist on every PR before marking it ready for review.

### Every PR
- [ ] Does the change affect public API, CLI, or user-visible behavior?
  - → Update docstrings on all changed public functions/classes
  - → Add an entry to `CHANGELOG.md` under `[Unreleased]`

### `feat:` commits
- [ ] CHANGELOG → Added
- [ ] README: update usage or configuration section if needed
- [ ] Docstrings written for all new public symbols

### `fix:` commits
- [ ] CHANGELOG → Fixed
- [ ] Docstring corrected if it described the wrong behavior

### `BREAKING CHANGE` commits
- [ ] CHANGELOG → Changed or Removed
- [ ] Migration guide created or updated at `docs/migration/vX.md`
- [ ] All affected README examples updated
- [ ] ADR written if this reflects an architectural direction change

### `build:` / dependency bumps
- [ ] README prerequisites updated if minimum versions changed

### At release time
- [ ] Rename `[Unreleased]` to `[X.Y.Z] - YYYY-MM-DD`
- [ ] Create fresh `[Unreleased]` section above it
- [ ] Update comparison links at bottom of `CHANGELOG.md`
- [ ] Tag the release in git
- [ ] Update static version badges in README if applicable

---

## Docs-as-Code Principles

1. **Docs live in the same repo as code** — versioned together, reviewed in the same PR.
2. **Plain text formats** — Markdown, reStructuredText, AsciiDoc.
3. **Docs reviewed like code** — inline comments, approval required.
4. **Automated checks in CI** — link checkers, spell checkers, doc build validation.
5. **Single source of truth** — link rather than copy.
6. **Generated API reference** — derive from docstrings and type annotations (Sphinx, pdoc, MkDocs).
7. **Docs updated in the same PR** — never in a follow-up.

---

## Anti-Patterns

| Anti-pattern | Consequence | Remedy |
|-------------|-------------|--------|
| Outdated code examples | New users follow broken instructions | Test examples in CI |
| Version number in multiple places | Version drift | Single source of truth in `pyproject.toml`; read from it everywhere |
| CHANGELOG written at release time | Forgotten changes, vague entries | Write entries per PR |
| No migration notes on breaking changes | Users upgrade silently and break | Every `BREAKING CHANGE` entry must include a migration path |
| ADR written months after the decision | History reconstructed from memory, inaccurate | Write ADR at decision time |
| ADRs deleted when decisions change | History lost | Mark as "Superseded by ADR-0NNN", never delete |
| Docs in a separate repo | Docs lag code by weeks | Docs-as-code: same repo |
| Describing "how" instead of "why" | Docs duplicate the code | Docs explain rationale and usage; code explains implementation |
| `[Unreleased]` never maintained | CHANGELOG missing entries at release | Require CHANGELOG entry in PR checklist |

---

## Tooling

**git-cliff** — generates CHANGELOG from conventional commits using a customizable template.
```bash
git cliff --output CHANGELOG.md
```

**towncrier** — Python ecosystem; stores fragments as individual files (`changelog.d/412.bugfix.md`), assembled at release. Eliminates merge conflicts.
```bash
towncrier build --version 2.0.0
```

**release-please** (Google) — GitHub Action that opens a "Release PR" with CHANGELOG updates and version bumps. Merging the PR triggers the release.

**semantic-release** — fully automated: reads commits, bumps version, writes CHANGELOG, publishes.

---

## Reference

- [Keep a Changelog v1.1.0](https://keepachangelog.com/en/1.1.0/)
- [Semantic Versioning 2.0.0](https://semver.org/)
- [Michael Nygard — Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [MADR — Markdown Architectural Decision Records](https://adr.github.io/madr/)
- [Spotify — When Should I Write an ADR](https://engineering.atspotify.com/2020/04/when-should-i-write-an-architecture-decision-record)
- [Write the Docs — Docs as Code](https://www.writethedocs.org/guide/docs-as-code/)
- [git-cliff](https://git-cliff.org/)
- [towncrier](https://towncrier.readthedocs.io/)
