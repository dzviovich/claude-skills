# Expert Shopper Skill

A Cowork/Claude Code skill that acts as a personal shopping assistant with growing long-term
memory. It researches products, compares alternatives, surfaces deals and coupons, tracks
purchases, and learns from every session to become more useful over time.

---

## What it does

**Shopping flow (Phases 0–6)**
1. Checks `~/shopper/projects.md` to see if this search was started before — asks if ambiguous
2. Creates a dated project folder: `~/shopper/yyyy-mm-dd - descriptive name/`
3. Loads category knowledge and your past preferences before asking a single question
4. Asks targeted intake questions (never assumes; always asks when in doubt)
5. Researches review sites and pricing for the specific category
6. Presents 3–5 alternatives in a comparison table with pros/cons and images
7. Collects structured refinement feedback across distinct dimensions (budget, features, brands, etc.)
8. Shows where to buy — rich cards per retailer with price, coupon codes, cashback, and shipping
9. Separately presents used/open-box/refurbished options with condition grades and trust signals
10. Saves `requirements.md`, `comparison.md`, and `selection.md` to the project folder
11. Updates shared knowledge base (category context, resources, user preferences)

**Purchase logging flow (Phase 7)**
- Triggered when you share a receipt file, order URL, or say "I bought it"
- Extracts purchase details, archives the receipt, compares actual vs. researched
- Issues warranty action items (registration URL, return window, extended coverage)
- Collects post-purchase feedback across separate dimensions (price, retailer, model choice, reviews, impressions)
- Updates all memory files with real-world outcome learnings

---

## File structure

```
~/shopper/                              ← one folder per shopping project
├── projects.md                         ← registry of every project
└── 2026-03-18 - product name/
    ├── requirements.md
    ├── comparison.md
    ├── selection.md
    ├── purchase_log.md
    └── receipts/

~/.expert-shopper/                      ← shared knowledge base
├── global_preferences.md
└── categories/
    └── {category}/
        ├── _context.md                 ← what matters when buying in this category
        ├── _resources.md               ← trusted review sites, stores, APIs
        ├── _user_prefs.md              ← your feedback and past choices
        └── {subcategory}/
            ├── _context.md
            ├── _resources.md
            └── _user_prefs.md
```

### Seed categories included

The skill ships with pre-built knowledge for:
- `electronics/` → `laptops/`, `headphones/`
- `kitchen-appliances/` → `espresso-machines/`

New categories are created automatically as you shop.

---

## How to invoke

| Situation | What to say |
|---|---|
| New search | `/expert_shopper I need a [product]` |
| Resume search | `/expert_shopper` (it will check your history) |
| Log a purchase | Share a receipt/URL, or say "I bought the [product]" |
| Explicit command | `/expert_shopper` followed by any shopping-related request |

The skill also triggers automatically on phrases like "help me buy X", "what's the best X for
my needs", "compare X vs Y", "where can I get X cheapest".

---

## Installation

Double-click `expert-shopper.skill` (or drag it into Cowork) to install.

In Claude Code (terminal), place the `expert-shopper/` directory inside your skills folder
and ensure it's referenced in your skills manifest.

---

## Project layout (this repo)

```
expert-shopper/
├── SKILL.md                   ← the skill itself (Claude reads this at runtime)
├── README.md                  ← this file
├── DEVELOPMENT.md             ← how to continue development
├── CHANGELOG.md               ← version history
├── expert-shopper.skill       ← packaged skill (zip), ready to install
├── contexts/                  ← seed files copied to ~/.expert-shopper/ on first run
│   ├── projects.md
│   ├── global_preferences.md
│   └── categories/
│       ├── electronics/
│       │   ├── _context.md / _resources.md / _user_prefs.md
│       │   ├── laptops/
│       │   └── headphones/
│       └── kitchen-appliances/
│           ├── _context.md / _resources.md / _user_prefs.md
│           └── espresso-machines/
└── evals/
    └── evals.json             ← test cases for skill evaluation
```
