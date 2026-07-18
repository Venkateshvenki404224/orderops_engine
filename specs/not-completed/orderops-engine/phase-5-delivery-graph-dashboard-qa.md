# Phase 5 — Delivery graph, final dashboard, docs, and browser QA

> Read [README.md](README.md) first for shared architecture facts and conventions.

## Goal (the thinnest end-to-end slice)

Complete the MVP by adding delivery route graph traversal, polishing the business-first dashboard, finishing docs, and running agent/browser QA with screenshots and a report. This phase covers Graph BFS/DFS and validates the entire app from a user perspective.

## Changes by file

### DSA module
- **Create `orderops_engine/dsa/graph.py`** — implement `DeliveryRouteGraph` with adjacency list, BFS, DFS, visited tracking, and Explain Mode traces.

### Service
- **Create `orderops_engine/services/route_service.py`** — build graph from `OrderOps Delivery Hub` and `OrderOps Delivery Route` records.
- **Modify `orderops_engine/services/index_builder.py`** — include delivery graph in common rebuilt indexes if the dashboard uses one entry point.

### API
- **Modify `orderops_engine/api/dashboard.py`** — add `route_bfs(start_hub)`, `route_dfs(start_hub)`, and optional `route_between(start_hub, end_hub)` if kept simple.

### Dashboard
- **Modify `orderops_engine/www/orderops-dashboard.py`** — add Delivery Route Explorer and final layout structure.
- **Modify `orderops_engine/public/js/orderops_dashboard.js`** — add hub selection, BFS/DFS buttons, trace rendering, and empty/error states.
- **Modify `orderops_engine/public/css/orderops_dashboard.css`** — polish business-first cards, trace panels, badges, and responsive layout.

### Tests
- **Create `test_dsa_graph.py`** — verify BFS/DFS order, visited set prevents cycles, missing node handled cleanly.
- **Modify `test_dashboard_api.py`** — verify graph APIs return Explain Mode contract.

### Docs
- **Modify `README.md`** — finalize install, seed, dashboard, and test commands.
- **Modify `docs/dsa-mapping.md`** — complete feature-to-DSA table.
- **Modify `docs/demo-walkthrough.md`** — complete 5-minute demo script.
- **Modify `docs/testing.md`** — add full unit/integration/browser QA instructions.

### Browser QA
- **Create `qa/browser-report.md`** — record exploratory test report.
- **Create `qa/screenshots/`** — store screenshots captured during browser test.

## Verification

1. Run all pure DSA tests:
   ```bash
   pytest orderops_engine/orderops_engine/tests/test_dsa_*.py -v
   ```
2. Run Frappe tests:
   ```bash
   bench --site <site> run-tests --app orderops_engine
   ```
3. Open `/orderops-dashboard` and manually smoke test all sections:
   - Smart Search Center
   - Urgent Operations Queue
   - Background Jobs Monitor
   - Order Timeline Viewer
   - Delivery Route Explorer
   - Workflow/Status Map
   - High Value Order Index
4. Run agent/browser QA using the dogfood workflow:
   - Navigate to dashboard.
   - Check browser console after load and after interactions.
   - Test valid and invalid search inputs.
   - Test priority task pop and job dequeue.
   - Test timeline selection.
   - Test BFS/DFS route traversal.
   - Capture screenshots and write `qa/browser-report.md`.

## Done when

Graph traversal works from seeded delivery routes, the dashboard demonstrates every learned DSA concept in business context, docs explain how to run and demo it, all tests pass, and browser QA evidence exists with no untriaged critical/high issues.
