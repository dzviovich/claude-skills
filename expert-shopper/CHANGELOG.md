# Changelog — Expert Shopper Skill

All notable changes to this skill are documented here.
Entries are in reverse chronological order (newest first).

---

## [1.4.0] — 2026-03-18

### Added
- **Rich where-to-buy cards (Phase 5)** — each retailer now displayed as a formatted card
  with price, coupon codes, cashback portal rates, shipping, price-match eligibility, and notes
- **Active coupon search** — skill now explicitly searches RetailMeNot, Honey, and CouponCabin
  for each retailer before presenting results
- **Cashback portal lookup** — Rakuten, TopCashback, Capital One Shopping checked per retailer
- **Effective price sorting** — retailers sorted by price after coupons + cashback applied
- **Best Deal callout** — single highlighted line showing cheapest effective option with math
- **Savings Summary table** — for items over ~$200, compact table showing new vs. refurb vs. used
  savings vs. MSRP
- **Rich used/refurb cards** — same card format for used/refurb section, with condition grade,
  warranty period, trust signals (seller rating, manufacturer-certified, Amazon-backed)

---

## [1.3.0] — 2026-03-18

### Added
- **Structured multi-dimensional feedback (Phase 4)** — refinement questions now use
  `AskUserQuestion multiSelect` across distinct axes: budget, features, brands, condition,
  retailers, form factor, compatibility, aesthetics, timeline, coverage
- **Per-dimension follow-up** — each flagged dimension gets its own targeted follow-up question;
  nothing batched into a single "other" field
- **Tagged feedback tracking** — feedback items tracked as `[budget]`, `[brands]`, `[features]`
  etc. and addressed/reported individually in the revised comparison
- **Post-purchase multi-dimensional feedback (Phase 7d)** — purchase logging now also collects
  feedback across separate axes: price, retailer, condition, model deviation, reviews, external
  factors, first impressions
- **Tagged memory storage** — each feedback dimension stored as a separate labeled bullet in
  `purchase_log.md` and `_user_prefs.md` for precise future reasoning

---

## [1.2.0] — 2026-03-18

### Added
- **Purchase Logging mode (Phase 7)** — triggered when user shares receipt, invoice, order
  URL, or says "I bought it"
- Receipt/invoice extraction from file, URL, or verbal description
- Receipt archival to `~/shopper/{project}/receipts/` with dated filename
- Purchase vs. research comparison table
- Warranty action item checklist (registration URL, return window, extended coverage)
- `purchase_log.md` template written to project folder
- Memory updates from real-world purchase outcome (price delta, retailer, model choice)
- Updated skill description to trigger on purchase logging signals
- Added `purchased` project status to registry

### Changed
- Phase 0 now detects session type (PURCHASE LOGGING vs SHOPPING) before any other action

---

## [1.1.0] — 2026-03-18

### Added
- **Project management system (Phase 0)** — every session creates a dated project folder
  under `~/shopper/yyyy-mm-dd - descriptive name/`
- `~/shopper/projects.md` registry tracking all projects with status
- Project lookup at session start — detects new vs. existing, asks when ambiguous
- Resume existing project — loads `requirements.md` and `comparison.md` to restore context
- Fallback when folder missing: tells user, asks them to identify directory or start fresh
- `requirements.md`, `comparison.md`, `selection.md` saved progressively during session
- Project status lifecycle: `in-progress` → `completed` → `purchased`

---

## [1.0.0] — 2026-03-18

### Added
- **Memory/context system** — persistent knowledge base at `~/.expert-shopper/`
- `global_preferences.md` — cross-category user preferences loaded every session
- Category context files (`_context.md`) — key buying criteria, common pitfalls, price tiers
- Category resource files (`_resources.md`) — trusted review sites, retailers, used/refurb sources
- Category user preference files (`_user_prefs.md`) — personal history per category
- Subcategory hierarchy — category files can reference subcategory files
- Phase 0 (Context Loading) — loads relevant files before intake questions
- Phase 6 (Memory Update) — updates all knowledge files at end of session
- Seed knowledge files for: `electronics/`, `electronics/laptops/`, `electronics/headphones/`,
  `kitchen-appliances/`, `kitchen-appliances/espresso-machines/`

---

## [0.1.0] — 2026-03-18 (initial)

### Added
- Core shopping workflow: intake → research → comparison table → refinement → where-to-buy
- Never-assume principle: always asks when information would change recommendation
- Comparison table with pros/cons, key specs, top pick indicator, and images
- Used/open-box/refurbished as a separate section in where-to-buy
- Category-aware review sources and retailers
- General principles: budget respect, honest trade-offs, scannable output
