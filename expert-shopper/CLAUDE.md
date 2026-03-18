# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code / Cowork skill that acts as a personal shopping assistant with persistent long-term memory. The main deliverable is `SKILL.md` — the runtime prompt that Claude reads when the skill is invoked. Everything else supports development, testing, and packaging of that skill.

## Key Files

- **`SKILL.md`** — The skill itself. This is the file Claude reads at runtime. All behavioral changes go here.
- **`expert-shopper.skill`** — Packaged zip archive of `SKILL.md` + `contexts/`. Binary file, regenerated via `package_skill.py`.
- **`contexts/`** — Seed knowledge files copied to `~/.expert-shopper/` on first user run. Pre-built categories: `electronics/` (laptops, headphones), `kitchen-appliances/` (espresso-machines).
- **`evals/evals.json`** — Test prompts with expected behavior assertions.

## Architecture

The skill operates in two modes detected at session start (Phase 0):
- **Shopping mode** (Phases 0-6): intake → research → comparison → refinement → where-to-buy → save & update memory
- **Purchase logging mode** (Phase 7): receipt extraction → archival → purchase-vs-research comparison → warranty reminders → memory update

Two runtime data stores (created on the user's machine, not in this repo):
- `~/shopper/` — Per-project output (requirements.md, comparison.md, selection.md, purchase_log.md, receipts/)
- `~/.expert-shopper/` — Shared knowledge base that grows over time (global_preferences.md + categories hierarchy)

Each category directory has three files: `_context.md` (buying criteria, pitfalls, price tiers), `_resources.md` (review sites, retailers, used/refurb sources), `_user_prefs.md` (user's history and feedback). These exist at both category and subcategory levels.

## Development Commands

**Run evals** (requires skill-creator skill):
```
"Run the evals for the expert-shopper skill at /path/to/expert-shopper"
```

**Package the skill:**
```bash
python -m scripts.package_skill /path/to/expert-shopper /path/to/output/
```

## Adding a New Category

1. Create `contexts/categories/{new-category}/` with `_context.md`, `_resources.md`, `_user_prefs.md`
2. Follow the structure of existing categories (e.g., `electronics/laptops/`)
3. Re-package with `package_skill.py`

## Iteration Checklist

When modifying `SKILL.md`:
- Intake question changes → update relevant `_context.md` seed files
- Memory format changes → check Phase 6 (save) and Phase 7g (purchase memory update)
- Comparison table changes → ensure `comparison.md` template in Phase 6a matches
- Purchase logging changes → test with a sample receipt
- Re-run evals after changes
- Update `CHANGELOG.md`
- Re-package and redeploy

## Design Principles

- **Never assume** — ask if a fact would change the recommendation
- **Structured multi-dimensional feedback** — each refinement dimension (budget, features, brands, etc.) is tracked separately with tags like `[price]`, `[retailer]`, `[model]`
- **Layered memory** — global → category → subcategory, newest entries at top
- **Rich where-to-buy** — card format per retailer with effective price after coupons/cashback; used/refurb shown separately with condition grades and trust signals
