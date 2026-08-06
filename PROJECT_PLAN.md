# Meal Tracker — Project Plan

## Overview

A web app (future mobile app) that stores meals and their ingredients, calculates calories per ingredient and per meal, tracks pantry stock, and recommends meals based on cost and calories. Live ingredient price scraping is a stretch feature, not a dependency for the core app.

## Goals

- Persist meals and their ingredient breakdowns
- Show calories per ingredient and total calories per meal
- Track pantry stock levels and compute what's missing for a given meal
- Compute cost-to-make per meal (based on shortfall, not full ingredient cost)
- Recommend the cheapest meal to make right now, sorted by highest calories
- (Stretch) Automatically refresh ingredient prices daily from an external source

## Non-goals (for v1)

- Multi-user accounts / auth
- Mobile app (planned for later, not part of this build)
- Nutritional tracking beyond calories (macros, micronutrients)
- Recipe recommendation based on taste/preference, only cost/calories

## Functional & Non-Functional Requirements

Fill these out before implementation starts — they force decisions the feature list glosses over.

**Functional** (what the system must do — expand beyond the Goals above with specifics, e.g. exact fields, validation rules)
- [x] Ingredient: `name` (required, unique), `unit` (fixed enum, e.g. g/ml/each — no free text), `calories_per_unit` (required, > 0)
- [x] Meal: `name` (required), `method` (ordered array of steps, each `{step_number, text}`, at least one step required), `created_at` (auto)
- [x] MealIngredient: `quantity` (required, > 0) per ingredient per meal
- [x] PantryStock: `quantity_on_hand` (required, >= 0, defaults to 0 for new ingredients)
- [x] PriceRecord: `price` (required, >= 0), `unit` must match the ingredient's unit, `source` (manual or scraped), `scraped_at`/`entered_at` (auto)
- [x] Calorie calc: per-ingredient = `quantity * calories_per_unit`; per-meal = sum across its ingredients
- [x] Cost calc: per meal = sum of `max(0, quantity_needed - quantity_on_hand) * latest_price` per ingredient
- [x] Recommendation: filter/sort meals by cost-to-make ascending, then calories descending, so "cheapest, tie-broken toward highest calories" surfaces first
- [x] No login/registration flow — single implicit user, no auth tables needed

**Non-functional** (qualities the system must have)
- [x] Performance: single local user, small data volume — no indexing/pagination work needed for v1; recommendation view should just recompute on request, sub-second is trivial at this scale
- [x] Reliability: if a scraped price is missing or stale (older than a set threshold, e.g. 24–48h), fall back to the last known `PriceRecord` and show a "stale" indicator in the UI rather than blocking the calculation
- [x] Data integrity: negative stock or negative price updates are **hard-blocked** at the API validation layer (rejected with a clear error), not clamped or silently allowed
- [x] Usability: built for a single user (you) — no accounts, no permissions logic; UI can prioritize fast data entry over polish for v1
- [x] Portability: runs locally only for v1 (e.g. `docker-compose up` or local npm/uvicorn scripts against a local Postgres). Still worth keeping DB connection config and secrets in environment variables from day one, so moving to hosting later is a config change, not a rewrite

## Feature Scope

### MVP (must have)
- [ ] Create/edit/delete ingredients (name, unit, calories per unit)
- [ ] Create/edit/delete meals, each composed of ingredients + quantities
- [ ] Store cooking method/instructions per meal (steps to prepare it)
- [ ] Auto-calculated per-ingredient and per-meal calorie totals
- [ ] Pantry table: quantity on hand per ingredient
- [ ] Shortfall calculation: what's missing from pantry for a given meal
- [ ] Manual/CSV price entry per ingredient
- [ ] Cost-to-make calculation per meal (based on shortfall only)
- [ ] "Cheapest meal, sorted by highest calories" recommendation view

### Stretch (may or may not build)
- [ ] Automated daily price scraping job
- [ ] Price history / trend view
- [ ] Shopping list generator from shortfall across multiple planned meals
- [ ] Swap manual pantry updates for "mark meal as cooked" auto-deduction
- [ ] Multi-user support

## Data Model

```
Ingredient
  id, name, unit, calories_per_unit

Meal
  id, name, method, created_at

MealIngredient (join table)
  meal_id, ingredient_id, quantity

PantryStock
  ingredient_id, quantity_on_hand, updated_at

PriceRecord
  id, ingredient_id, source, price, unit, scraped_at
```

Notes:
- `method` on `Meal` is an ordered array of steps (e.g. `[{step_number, text}]`), not a single text block. Chosen to support future step-by-step UI (numbered steps, per-step timers).
- `PriceRecord` is a time series (append-only), not a single field on `Ingredient`, so a failed scrape falls back to the last known price instead of erroring.
- Cost-to-make for a meal = sum over ingredients of `max(0, quantity_needed - quantity_on_hand) * current_price`.
- Units must be consistent per ingredient (e.g. always grams, or always "each") to avoid conversion bugs.

## API Design

Sketch the main endpoints before coding — even a rough table catches mismatches between frontend needs and backend structure early.

| Method | Endpoint | Purpose |
|---|---|---|
| GET | /api/ingredients | ... |
| POST | /api/ingredients | ... |
| GET | /api/meals | ... |
| POST | /api/meals | ... |
| GET | /api/meals/:id | ... |
| GET | /api/pantry | ... |
| PUT | /api/pantry/:ingredient_id | ... |
| GET | /api/recommendations | ... |

- [ ] Decide: REST or something else (GraphQL is probably overkill here, but worth a conscious decision not a default)
- [ ] Decide: how errors are shaped in responses (status codes, error body format)
- [ ] Decide: pagination approach once meal/ingredient lists grow

## Architecture

- **Frontend**: React
- **Backend**: Python, FastAPI (Uvicorn as the server, Pydantic for request/response validation)
- **Database**: PostgreSQL, accessed via SQLAlchemy (+ Alembic for migrations)
- **Scraper (stretch)**: standalone module behind an interface, so it can be swapped for manual entry without touching the rest of the app. `httpx`/`requests` + `BeautifulSoup` for static pages, `Playwright` if a site needs JS rendering. Scheduled via APScheduler, not a full task queue at this scale.

## UI / UX & Screens

List the screens/views you need, then sketch (paper or Figma/Excalidraw is fine) before building components.

- [ ] Meal list / dashboard
- [ ] Meal detail (ingredients, calories, method steps)
- [ ] Add/edit meal form (including ordered step editor for method)
- [ ] Pantry view (stock levels, edit quantities)
- [ ] Recommendation view (cheapest meal by calories)
- [ ] Ingredient management (CRUD, base prices)

## Build Order / Roadmap

1. **Core data model** — ingredients, meals, meal-ingredients, calorie calculation
2. **Pantry tracking** — stock levels, shortfall calculation
3. **Manual pricing** — CSV/manual entry, cost-to-make calculation, cheapest-meal recommendation
4. **Polish MVP** — UI cleanup, edge cases (missing prices, zero-stock ingredients)
5. **Stretch: scraping** — automated price refresh, with graceful fallback to last known price
6. **Stretch: extras** — price history, shopping list generator, auto-deduct pantry on "cooked"

## Testing Strategy

- [ ] Decide unit test coverage for calorie/cost calculation logic (highest-value tests, since a bug here silently gives wrong recommendations)
- [ ] Decide whether to test the API layer (integration tests) and with what tool
- [ ] Decide how to test the scraper independently from the rest of the app (mock responses so tests don't depend on a live site)
- [ ] Decide manual test checklist for the UI, if not automating frontend tests

## Deployment Plan

- [ ] Where will this run — local only for now, or hosted (e.g. same AWS EC2 approach as Teapot Invoicing, or a simpler PaaS like Railway/Render/Fly.io)?
- [ ] Database hosting — local Postgres vs. a managed instance
- [ ] Environment variables / secrets handling (DB credentials, any scraper config)
- [ ] CI: run tests on push? (GitHub Actions is the natural fit given this is going on GitHub)

## Timeline & Milestones

Fill in target dates once you know your available hours per week.

| Milestone | Target date | Status |
|---|---|---|
| Core data model working (Step 1) | | |
| Pantry + shortfall logic (Step 2) | | |
| Manual pricing + recommendation view (Step 3) | | |
| MVP polish (Step 4) | | |
| Stretch: scraping (Step 5) | | |

## Success Criteria / Definition of Done

- [ ] What does "MVP is done" mean concretely — all MVP checkboxes above, or a specific demo scenario?
- [ ] Is there a specific use case you'll test it against (e.g. "can plan a week of meals from what's in my pantry")?

## Assumptions & Constraints

- [x] Single user (you), single pantry — confirmed for v1, no auth/multi-tenancy
- [x] Unit system: metric only for v1 (no imperial support)
- [x] Offline: not required — local-only for v1 means this is moot for now; revisit if/when hosted

## Open Decisions

Running list of things still undecided — move items here as they come up instead of deciding ad hoc mid-build.

- [x] Backend: Python, FastAPI + SQLAlchemy (locked in)
- [ ] Hosting choice
- [ ] Method step format details (plain text per step vs. structured fields like duration/temperature)

## Key Risks / Tradeoffs

- **Live price scraping is unreliable by nature.** Major AU grocery sites (Woolworths, Coles) actively fingerprint/rate-limit automated requests and prohibit scraping in their terms of service. Treat scraping as an optional enrichment layer, not a load-bearing dependency — the app must work fully with manually entered prices.
- **Relational vs document DB**: chose Postgres over Mongo because this data has genuine foreign-key relationships (meal → ingredients → prices → stock).
- **Unit conversion**: locking down consistent units per ingredient early avoids retrofitting pain later.
- **Single-user first**: skip auth/multi-tenancy for v1; add later if needed rather than building it in prematurely.

## Tech Stack Summary

| Layer | Choice |
|---|---|
| Frontend | React |
| Backend | Python, FastAPI |
| Validation | Pydantic |
| Database | PostgreSQL |
| ORM / Migrations | SQLAlchemy + Alembic |
| Testing | pytest |
| Scraper (stretch) | Python (httpx/requests + BeautifulSoup, or Playwright) as a swappable module |
| Scheduling (stretch) | APScheduler |

## Future Work (post-MVP)

- Mobile app
- Multi-user support with auth
- Macro/micronutrient tracking
- Smarter recommendations (e.g. weekly meal planning within a budget)
