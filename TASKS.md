# Meal Tracker — Task List

Tracks every task from project setup to MVP completion, plus stretch goals. Check items off as you go. See `PROJECT_PLAN.md` for the reasoning behind each decision referenced here.

---

## Phase 0 — Project Setup

- [ ] Create GitHub repo
- [ ] Add `PROJECT_PLAN.md` and this `TASKS.md` to repo root
- [ ] Create `.gitignore` (Python + Node + `.env` + `__pycache__` + `node_modules`)
- [ ] Create backend folder structure (`/backend`)
- [ ] Create frontend folder structure (`/frontend`)
- [ ] Set up Python virtual environment for backend
- [ ] Install FastAPI, Uvicorn, SQLAlchemy, Alembic, Pydantic, python-dotenv, pytest
- [ ] Create `requirements.txt`
- [ ] Install PostgreSQL locally
- [ ] Create local database and local dev user/credentials
- [ ] Create `.env` file for DB connection string (and add `.env` to `.gitignore`)
- [ ] Confirm FastAPI "hello world" endpoint runs locally via Uvicorn
- [ ] Set up React app (Vite recommended) in `/frontend`
- [ ] Confirm React dev server runs and can hit the FastAPI hello-world endpoint

## Phase 1 — Database & Core Data Model

- [ ] Define SQLAlchemy model: `Ingredient` (name, unit, calories_per_unit)
- [ ] Define SQLAlchemy model: `Meal` (name, method as ordered steps, created_at)
- [ ] Define SQLAlchemy model: `MealIngredient` (meal_id, ingredient_id, quantity)
- [ ] Define SQLAlchemy model: `PantryStock` (ingredient_id, quantity_on_hand, updated_at)
- [ ] Define SQLAlchemy model: `PriceRecord` (ingredient_id, source, price, unit, scraped_at)
- [ ] Decide and implement `method` storage shape (ordered array of `{step_number, text}`)
- [ ] Set up Alembic and generate initial migration
- [ ] Run migration against local database, confirm tables created
- [ ] Add DB constraints: unit enum restriction, `calories_per_unit > 0`, `quantity > 0`, `quantity_on_hand >= 0`, `price >= 0`
- [ ] Write a seed script with a handful of sample ingredients and one sample meal for local testing

## Phase 2 — Backend API: Ingredients

- [ ] Define Pydantic schema for Ingredient create/response
- [ ] Implement `POST /api/ingredients` (create)
- [ ] Implement `GET /api/ingredients` (list all)
- [ ] Implement `GET /api/ingredients/:id` (single)
- [ ] Implement `PUT /api/ingredients/:id` (update)
- [ ] Implement `DELETE /api/ingredients/:id`
- [ ] Add validation: reject negative/zero `calories_per_unit`, enforce unique name
- [ ] Write pytest tests for ingredient endpoints (happy path + validation failure)

## Phase 3 — Backend API: Meals (with method steps)

- [ ] Define Pydantic schema for Meal create/response, including nested ingredients list and nested ordered steps list
- [ ] Implement `POST /api/meals` (composite create: meal + ingredients + steps in one call)
- [ ] Implement `GET /api/meals` (list, with computed total calories per meal)
- [ ] Implement `GET /api/meals/:id` (single meal, full ingredient breakdown + calories + method steps)
- [ ] Implement `PUT /api/meals/:id` (update meal, including replacing ingredients/steps)
- [ ] Implement `DELETE /api/meals/:id`
- [ ] Implement calorie calculation logic (per-ingredient and per-meal total)
- [ ] Add validation: at least one ingredient required, at least one method step required, positive quantities
- [ ] Write pytest tests for meal endpoints and calorie calculation logic

## Phase 4 — Pantry & Shortfall Logic

- [ ] Define Pydantic schema for PantryStock
- [ ] Implement `GET /api/pantry` (list stock levels for all ingredients)
- [ ] Implement `PUT /api/pantry/:ingredient_id` (update quantity on hand)
- [ ] Add validation: hard-block negative `quantity_on_hand`
- [ ] Implement shortfall calculation function: `max(0, quantity_needed - quantity_on_hand)` per ingredient for a given meal
- [ ] Expose shortfall in `GET /api/meals/:id` response (which ingredients are missing and how much)
- [ ] Write pytest tests for shortfall calculation, including edge cases (exact stock, zero stock, surplus stock)

## Phase 5 — Pricing & Cost Calculation

- [ ] Define Pydantic schema for PriceRecord
- [ ] Implement `POST /api/ingredients/:id/prices` (manual price entry)
- [ ] Implement `GET /api/ingredients/:id/prices` (price history for one ingredient)
- [ ] Implement "latest price" lookup function per ingredient
- [ ] Add validation: hard-block negative price, enforce unit match with ingredient's unit
- [ ] Implement cost-to-make calculation: shortfall quantity × latest price, summed per meal
- [ ] Expose cost-to-make in `GET /api/meals/:id` and `GET /api/meals` responses
- [ ] Write pytest tests for cost calculation, including missing-price edge case

## Phase 6 — Recommendation Engine

- [ ] Implement `GET /api/recommendations` endpoint
- [ ] Implement sort logic: cost-to-make ascending, then calories descending as tiebreaker
- [ ] Handle edge case: meals with no price data available for one or more ingredients
- [ ] Write pytest tests for recommendation ordering with sample data

## Phase 7 — Frontend Setup & Shared Structure

- [ ] Set up React Router (or equivalent) with routes for each screen
- [ ] Set up API client/fetch wrapper for calling the FastAPI backend
- [ ] Set up basic layout/navigation shell
- [ ] Decide and set up styling approach (CSS modules, Tailwind, etc.)

## Phase 8 — Frontend Screens

- [ ] Build Ingredient management screen (list, add, edit, delete)
- [ ] Build Meal list / dashboard screen (name, total calories, cost-to-make at a glance)
- [ ] Build Meal detail screen (full ingredient breakdown, per-ingredient calories, method steps)
- [ ] Build Add/Edit Meal form, including ingredient picker with quantities
- [ ] Build ordered step editor for method (add/remove/reorder steps)
- [ ] Build Pantry screen (view and edit stock levels per ingredient)
- [ ] Build Recommendation view (cheapest-by-calories sorted list)
- [ ] Add loading and error states for all screens
- [ ] Add form validation matching backend rules (positive quantities, required fields) for immediate user feedback

## Phase 9 — Testing & Quality Pass

- [ ] Run full pytest suite, confirm all passing
- [ ] Manually test full flow: create ingredients → create meal → set pantry stock → enter prices → check recommendation view
- [ ] Test edge cases: meal with all ingredients in stock (cost = $0), meal with missing price data, empty pantry
- [ ] Fix bugs found during manual pass

## Phase 10 — MVP Polish & Documentation

- [ ] Write `README.md` (setup instructions, how to run locally, screenshots)
- [ ] Clean up unused code/dependencies
- [ ] Confirm `PROJECT_PLAN.md` and `TASKS.md` reflect final decisions made during the build
- [ ] Tag/release MVP version in GitHub (e.g. `v1.0-mvp`)

---

## Stretch Phase A — Price Scraping (may or may not build)

- [ ] Research and pick a target source (grocery site or price dataset/API) with the least fragile access
- [ ] Build scraper module as an isolated component behind a simple interface (e.g. `get_price(ingredient_name)`)
- [ ] Add caching/rate-limiting so the scraper doesn't hit the source excessively
- [ ] Add "stale price" threshold logic (fall back to last known price if scrape fails or is too old)
- [ ] Add UI indicator for stale vs. fresh prices
- [ ] Set up scheduled job (APScheduler) to run the scraper daily
- [ ] Write tests for the scraper using mocked responses (no live network calls in tests)
- [ ] Add manual "refresh now" trigger as a fallback/override

## Stretch Phase B — Extras

- [ ] Build price history/trend chart per ingredient
- [ ] Build shopping list generator (aggregate shortfall across multiple selected meals)
- [ ] Add "mark meal as cooked" action that auto-deducts pantry stock
- [ ] Revisit multi-user support if ever needed (auth, per-user pantry/meals)
- [ ] Investigate hosting options if moving off local-only (revisit Deployment Plan in `PROJECT_PLAN.md`)
