# Phase 1 — Tracer bullet search slice

> Read [README.md](README.md) first for shared architecture facts and conventions.

## Goal (the thinnest end-to-end slice)

Ship the smallest working OrderOps slice that travels through every layer: a Frappe app shell, one `OrderOps Order` DocType, a tiny seed command, a HashMap DSA module with Explain Mode, a whitelisted lookup API, a one-card dashboard, and tests. This phase intentionally does **not** include all DocTypes, full seed volume, autocomplete, priority queues, timelines, routes, or browser QA.

## Changes by file

### App shell
- **Create `<selected-bench>/apps/orderops_engine/`** — create the Frappe app after confirming the bench/site.
- **Modify `orderops_engine/hooks.py`** — include app metadata and any required fixtures later.
- **Create `README.md` and copy the approved `plan.md`** — preserve the planning context in the app root.

### DocType
- **Create `orderops_engine/orderops_engine/doctype/orderops_order/`** — add minimal `OrderOps Order` DocType with `order_id`, `customer_name`, `status`, `amount`, and `priority_level` fields.

### DSA
- **Create `orderops_engine/dsa/explain.py`** — add `ExplainResult` helper with `to_dict()`.
- **Create `orderops_engine/dsa/hash_index.py`** — add `OrderHashIndex` with `lookup(order_id, explain=True)`.

### Seed/service
- **Create `orderops_engine/services/seed.py`** — seed 10 deterministic sample orders for the tracer bullet only.
- **Create `orderops_engine/services/index_builder.py`** — build `OrderHashIndex` from `OrderOps Order` records.

### API
- **Create `orderops_engine/api/dashboard.py`** — add whitelisted `lookup_order(order_id)` returning Explain Mode output.

### Dashboard
- **Create `orderops_engine/www/orderops-dashboard.py`** — render a basic page shell.
- **Create `orderops_engine/public/js/orderops_dashboard.js`** — call `lookup_order` and render result/trace.
- **Create `orderops_engine/public/css/orderops_dashboard.css`** — minimal readable layout.

### Tests
- **Create `orderops_engine/tests/test_dsa_hash_index.py`** — pure Python test for O(1) lookup response shape.
- **Create `orderops_engine/tests/test_dashboard_api.py`** — Frappe test for seeded order lookup API.

## Verification

1. Install app on selected site:
   ```bash
   bench --site <site> install-app orderops_engine
   ```
2. Seed tracer data:
   ```bash
   bench --site <site> execute orderops_engine.services.seed.seed_tracer_data
   ```
3. Run pure DSA test:
   ```bash
   pytest orderops_engine/orderops_engine/tests/test_dsa_hash_index.py -v
   ```
4. Run Frappe API test:
   ```bash
   bench --site <site> run-tests --app orderops_engine --module orderops_engine.tests.test_dashboard_api
   ```
5. Open:
   ```txt
   /orderops-dashboard
   ```
   Search an existing order such as `ORD-0001` and verify the dashboard shows `concept: HashMap`, `complexity: O(1) average`, `steps: 1`, and a trace line.

## Done when

A user can open the dashboard, search a seeded order ID, and see a real result plus DSA Explain Mode trace. Tests prove the HashMap module and lookup API work. The slice is deliberately tiny but confirms the full DB → DSA → API → UI path.
