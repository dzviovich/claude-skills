---
description: "Conventional Commits format. Reference when writing or reviewing commit messages."
---

# Skill: Commit Message

Write commit messages that follow the [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification.

---

## Format

```
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

---

## Types

| Type | Use for |
|------|---------|
| `feat` | A new user-visible feature |
| `fix` | A bug fix |
| `docs` | Documentation only — no code change |
| `style` | Code formatting only — no logic change |
| `refactor` | Restructuring with no feature add and no bug fix |
| `perf` | A refactor whose primary goal is performance |
| `test` | Adding or correcting tests — no production code change |
| `build` | Build system or external dependency changes |
| `ci` | CI/CD configuration changes |
| `chore` | Housekeeping that fits none of the above |
| `revert` | Reverts a previous commit |

**Type boundaries:**
- `style` is code formatting — not CSS or visual design.
- `perf` is a subset of `refactor`; use it only when performance is the explicit goal.
- `build` = what produces the artifact (`pyproject.toml`, `Makefile`, `Dockerfile`). `ci` = what runs the pipeline (`.github/workflows/`).
- `chore` is the catch-all of last resort — prefer any more specific type.

---

## Subject Line

- **Tense:** Imperative present — "add", not "added" or "adding".
  Test: "If applied, this commit will **[subject]**."
- **Case:** Lowercase first letter after the colon.
- **No period** at the end.
- **Length:** ≤ 50 characters ideal; hard limit 72 characters.

```
# bad
feat(cart): Added quantity validation.   ← past tense, trailing period
fix: Preventing divide-by-zero           ← gerund, capitalised
docs: updates the README                 ← third person

# good
feat(cart): add quantity limit validation
fix: prevent divide-by-zero in tax calculator
docs: add installation section to README
```

---

## Scope

Optional. A **noun** naming the subsystem. Lowercase. No issue IDs.

```
feat(auth): add OAuth2 PKCE flow
fix(api): return 404 for unknown user id
build(deps): bump requests from 2.28.0 to 2.31.0
ci(docker): add ARM64 build step
```

Define project-specific allowed scopes (e.g., `api`, `auth`, `db`, `cli`, `ui`) and register them in `commitlint.config.js`. Omit scope when the change is cross-cutting or the project is small.

---

## Body

- Separated from the subject by **one blank line**.
- Wrap at **72 characters** per line.
- Explains **why**, not what — the code explains what.
- Required for: non-obvious bug fixes, breaking changes, reverts.

```
fix(auth): prevent session tokens from leaking in logs

Previously, the `debug` log level included full request headers,
which could contain Authorization tokens. This exposes credentials
in log aggregation systems.

Replace header logging with a redacted summary that masks any
header matching the configured denylist.

Closes #412
```

---

## Footers

One blank line after the body (or subject if no body). One footer per line.

```
Closes #123
Fixes #456
Co-authored-by: Jane Smith <jane@example.com>
Reviewed-by: Bob <bob@example.com>
BREAKING CHANGE: <description>
```

Token names use hyphens for spaces (`Co-authored-by`). `BREAKING CHANGE` is the only all-caps exception.

---

## Breaking Changes

Two mechanisms — combine them for significant breaks:

**`!` suffix** — signals breaking change at a glance:
```
feat(api)!: remove v1 authentication endpoint
fix!: change config file location
```

**`BREAKING CHANGE:` footer** — provides migration detail:
```
BREAKING CHANGE: `user.full_name` has been removed. Use
`user.first_name` and `user.last_name` instead.
```

**Best practice — use both:**
```
feat(api)!: restructure user response payload

Replace `full_name` with `first_name` + `last_name` to align
with the identity service schema.

BREAKING CHANGE: `user.full_name` has been removed. Clients
must use `user.first_name` and `user.last_name` instead.
Closes #201
```

Either form triggers a **MAJOR** SemVer bump in automated tooling.

---

## SemVer Mapping

| Commit | Version bump |
|--------|-------------|
| `fix:` | PATCH (x.y.**Z**) |
| `feat:` | MINOR (x.**Y**.0) |
| `BREAKING CHANGE` or `!` | MAJOR (**X**.0.0) |
| `docs:`, `style:`, `refactor:`, `test:`, `build:`, `ci:`, `chore:` | None |
| `perf:` | PATCH (tool-dependent) |

Multiple commits in a release: the highest bump wins.

---

## Examples Covering All Types

```
feat(search): add fuzzy matching to product search

fix(checkout): prevent double-submission on slow connections

Debounce the submit button for 2 seconds after first click.
Resolves #887

docs(contributing): add section on running integration tests locally

style: convert all string literals to double quotes

refactor(auth): extract token validation into dedicated service

perf(db): replace N+1 query with single JOIN in user listing

Reduces average /users response time from 340ms to 12ms.

test(parser): add coverage for null byte in input stream

build(deps): bump pytest from 7.3.1 to 8.0.0

ci(github): add ARM64 build target to release workflow

chore: remove deprecated compatibility shim for Python 3.8

revert: feat(api): add experimental streaming endpoint

Reverts commit a1b2c3d. The implementation causes memory leaks
under high concurrency. Will reintroduce after profiling.

feat(config)!: rename `database_url` to `db_connection_string`

Align configuration key names with the infrastructure team's
naming standard across all services.

BREAKING CHANGE: The environment variable `DATABASE_URL` is no
longer read. Set `DB_CONNECTION_STRING` instead. Update all
deployment manifests and .env files before upgrading.
Closes #1042
```

---

## Anti-Patterns

| Anti-pattern | Example | Problem |
|-------------|---------|---------|
| Vague subject | `fix: stuff` | No information |
| Wrong tense | `feat: added login page` | Violates imperative mood |
| Trailing period | `docs: update README.` | Wrong style |
| Capitalised description | `feat: Add new endpoint` | Violates lowercase convention |
| Issue ID as scope | `fix(#123): correct behavior` | Scopes are nouns, not IDs |
| No space after colon | `feat:add thing` | Malformed; fails linting |
| Multiple concerns | `feat: add auth and fix cart bug` | Should be two commits |
| Body without blank line | Subject immediately followed by body text | Git misparses as one paragraph |
| No body on breaking change | `feat!: rename all endpoints` | No migration path |
| Over-long subject | >72 chars | Truncated in GitHub and `git log` |
| WIP commits merged to main | `wip: half-done feature` | Breaks `git bisect` |

---

## Tooling

**commitlint** — lints messages at commit time via a `commit-msg` git hook (Husky).
```js
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'scope-enum': [2, 'always', ['api', 'auth', 'db', 'cli', 'ui']],
    'subject-max-length': [2, 'always', 72],
  },
};
```

**commitizen** — interactive CLI wizard (`git cz`). Python equivalent: `commitizen-tools/commitizen`.

**semantic-release** / **release-please** — reads commit history, bumps version automatically, generates release notes.

---

## Reference

- [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)
- [commitizen/conventional-commit-types](https://github.com/commitizen/conventional-commit-types)
- [Chris Beams — How to Write a Git Commit Message](https://cbea.ms/git-commit/)
- [commitlint](https://commitlint.js.org/)
