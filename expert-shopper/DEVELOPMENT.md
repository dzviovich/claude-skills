# Development Guide — Expert Shopper Skill

This guide explains how to continue developing this skill in Claude Code (terminal mode)
or in Cowork. It covers the skill architecture, how to run evals, how to iterate, and
how to package and deploy.

---

## Prerequisites

- Claude Code installed (`npm install -g @anthropic-ai/claude-code`)
- The `skill-creator` skill available (comes with Cowork / Claude Code)
- Python 3.8+ (for packaging and eval scripts)

---

## Working with the skill in Claude Code

### Directory layout

Place this folder inside your Claude Code skills directory. The typical location is:
```
~/.claude/skills/expert-shopper/     ← on Mac/Linux
%APPDATA%\Claude\skills\expert-shopper\   ← on Windows
```

Or reference it from wherever you keep your skills projects.

### Opening the skill for editing

In Claude Code terminal:
```bash
cd /path/to/expert-shopper
claude
```

Then ask Claude to:
- "Update the SKILL.md to add [feature]"
- "Run the evals and show me results"
- "Package the skill"

---

## Architecture overview

The skill has **two data stores** that work together:

### 1. `~/shopper/` — per-project files
Created fresh for each shopping session. Contains the research output:
- `requirements.md` — what the user asked for
- `comparison.md` — the alternatives table
- `selection.md` — where-to-buy results
- `purchase_log.md` — post-purchase record
- `receipts/` — archived receipt files

### 2. `~/.expert-shopper/` — shared knowledge base
Persists across all sessions. Gets richer over time:
- `global_preferences.md` — cross-category user preferences
- `categories/{cat}/_context.md` — what matters when buying in this category
- `categories/{cat}/_resources.md` — trusted review sites and stores
- `categories/{cat}/_user_prefs.md` — the user's history and feedback

**Seed files** (in `contexts/`) are copied into `~/.expert-shopper/` on first run.
These give the skill pre-built knowledge for electronics and kitchen appliances.

---

## Adding a new category

1. Create a directory in `contexts/categories/{new-category}/`
2. Add three files: `_context.md`, `_resources.md`, `_user_prefs.md`
3. Follow the structure of an existing category (e.g., `electronics/laptops/`)
4. `_context.md` should cover:
   - Critical intake questions for this category
   - Key specs to include in comparison tables
   - Common pitfalls buyers fall into
   - Price tiers with approximate ranges
5. `_resources.md` should list:
   - Best review sources with URLs
   - Community/forum sources
   - New purchase retailers
   - Used/refurbished sources
6. Re-run `package_skill.py` to include the new files in the `.skill` bundle

---

## Running evaluations

Eval test cases are in `evals/evals.json`. To run them:

### With the skill-creator

In a Cowork or Claude Code session with the skill-creator skill available:
```
"Run the evals for the expert-shopper skill at /path/to/expert-shopper"
```

### Manually

Spawn Claude subagents with and without the skill using the test prompts in `evals/evals.json`.
Compare outputs against the assertions in each eval's `assertions` array.

### Adding new test cases

Edit `evals/evals.json` and add entries following this schema:
```json
{
  "id": 3,
  "prompt": "The user's test message",
  "expected_output": "Description of what a good response looks like",
  "files": []
}
```

Good test cases to add:
- A purchase logging session (share a fake receipt)
- A refinement loop (user changes budget mid-session)
- A second session for the same category (tests memory loading)
- An edge case: vague request, unknown category, or empty registry

---

## Packaging the skill

From the `skill-creator` directory (or wherever `package_skill.py` lives):

```bash
python -m scripts.package_skill /path/to/expert-shopper /path/to/output/
```

This produces `expert-shopper.skill` (a zip archive containing `SKILL.md` and all `contexts/`
files, excluding `evals/`). The `.skill` file can be installed by:
- Dragging into Cowork
- Using "Copy to your skills" button
- Placing in the Claude Code skills directory manually

---

## Skill phases reference

| Phase | Name | Description |
|---|---|---|
| 0 | Session Detection & Project Lookup | Detects purchase logging vs. shopping mode; finds or creates project folder |
| 0b | Context Loading | Loads `~/.expert-shopper/` knowledge files for the detected category |
| 1 | Intake | Gathers user requirements via targeted questions |
| 2 | Research | Searches review sites and pricing sources |
| 3 | Present Alternatives | Comparison table with pros/cons, images, top pick |
| 4 | Refinement Loop | Multi-dimensional structured feedback collection |
| 5 | Where to Buy | Rich retailer cards with price, coupons, cashback; separate used/refurb section |
| 6 | Save & Update Memory | Writes project files; updates knowledge base |
| 7 | Purchase Logging | Extracts receipt data, archives it, compares vs. research, updates memory |

---

## Key design principles

- **Never assume** — if a fact would change the recommendation, ask for it
- **Structured feedback** — each feedback dimension (budget, features, brands, etc.) is
  tracked and addressed separately, never merged into a single "other" bucket
- **Memory is layered** — global → category → subcategory, from broad to specific
- **Separate feedback tags** — every piece of user feedback is labeled `[price]`, `[retailer]`,
  `[model]`, etc. so future sessions can reason about each signal independently
- **Rich where-to-buy** — card format per retailer with effective price after coupons/cashback,
  sorted by value; used/refurb shown separately with condition grades and trust signals

---

## Iteration checklist

When making changes to the skill, run through this checklist:

- [ ] Does the change affect intake questions? → Update the category `_context.md` files too
- [ ] Does it affect how memory is written? → Check all 4 memory update steps in Phase 6 & 7g
- [ ] Does it affect the comparison table? → Ensure `comparison.md` template in Phase 6a matches
- [ ] Does it affect purchase logging? → Test with a sample receipt (PDF or text)
- [ ] Re-run evals after changes to check for regressions
- [ ] Update `CHANGELOG.md` with the change
- [ ] Re-package with `package_skill.py`
- [ ] Re-deploy via Cowork "Copy to your skills"

---

## Troubleshooting

**Skill doesn't trigger automatically**
- Check that the `description` field in `SKILL.md` frontmatter covers the trigger phrases
- In Claude Code, ensure the skill is in the correct skills directory and listed in the manifest
- Try invoking explicitly with `/expert_shopper`

**Memory files aren't being created**
- The skill bootstraps `~/.expert-shopper/` from `contexts/` on first run
- Check that the skill has write access to the home directory
- In Claude Code, the `~` path resolves to the user running the claude process

**`~/shopper/` folder location**
- On Windows: `C:\Users\{username}\shopper\`
- On Mac/Linux: `/Users/{username}/shopper/` or `/home/{username}/shopper/`
- You can change this by editing the data directory path in `SKILL.md`
