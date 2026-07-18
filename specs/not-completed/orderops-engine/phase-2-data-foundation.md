# Phase 2 — Data foundation and roles

> Read [README.md](README.md) first for shared architecture facts and conventions.

## Goal (the thinnest end-to-end slice)

Expand the tracer into a realistic Frappe data foundation: all MVP DocTypes, basic roles/permissions, and deterministic seed data with 500 orders, 50 customers, 10 hubs, and 50 priority tasks. This phase does **not** add the full DSA dashboard features yet; it prepares trustworthy data for them.

## Changes by file

### Roles and fixtures
- **Create/Modify `orderops_engine/fixtures/roles.json`** — export `OrderOps Manager`, `OrderOps Support Agent`, and `OrderOps Viewer`.
- **Modify `orderops_engine/hooks.py`** — include role fixtures.

### DocTypes
- **Create `OrderOps Customer`** — customer profile fields: name, email, phone, city, VIP flag, score.
- **Create `OrderOps Product`** — product fields: product name, category, price, SKU.
- **Create `OrderOps Order`** — extend tracer DocType with links to customer and hub, dates, status, amount, priority, delay/high-value flags, and child items.
- **Create `OrderOps Order Item`** — child table for products, qty, rate, amount.
- **Create `OrderOps Delivery Hub`** — hub name, city, type, capacity score.
- **Create `OrderOps Delivery Route`** — from hub, to hub, distance, ETA, active flag.
- **Create `OrderOps Priority Task`** — task title, linked order, priority, type, status.
- **Create `OrderOps Timeline Event`** — linked order, event type/message/time/sequence.
- **Create `OrderOps Background Job`** — job title/type/status/created order.

### Seed
- **Modify `orderops_engine/services/seed.py`** — replace tracer seed with deterministic full seed command while keeping `seed_tracer_data` for tests.
- **Add deterministic random seed constant** — make repeated demo runs stable.

### Tests
- **Create `orderops_engine/tests/test_seed.py`** — verify counts and links after default seed.
- **Create/Modify DocType smoke tests** — verify an order can link customer, hub, items, timeline events, and priority tasks.

### Docs
- **Create/Modify `docs/testing.md`** — document seed and role verification commands.

## Verification

1. Migrate site:
   ```bash
   bench --site <site> migrate
   ```
2. Seed default data:
   ```bash
   bench --site <site> execute orderops_engine.services.seed.seed_demo_data
   ```
3. Verify counts in bench console or tests:
   ```bash
   bench --site <site> run-tests --app orderops_engine --module orderops_engine.tests.test_seed
   ```
4. In Desk, confirm `OrderOps Order` records can open and show linked customer, items, hub, timeline events, and tasks.

## Done when

The app has realistic ecommerce operations data in Frappe, basic roles exist, and tests prove the seeded records are linked correctly. Later phases can build every DSA structure from this source of truth.
