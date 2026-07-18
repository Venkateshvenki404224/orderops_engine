> **Status:** ⬜ Not started — spec authored, no implementation yet

# Spec: OrderOps Engine

OrderOps Engine is a Frappe ecommerce operations app that turns core DSA concepts into real business workflows: smart order search, urgent task processing, job queues, order timelines, workflow maps, high-value order indexes, and delivery route exploration. Frappe records remain the source of truth; manual DSA structures are rebuildable in-memory demo/index layers with Explain Mode so the dashboard can show result, steps, complexity, and trace.

Branch: `feature/orderops-engine` · Base: `master` for the planning repo; use the selected Frappe bench repo default branch when implementation starts.

## Build strategy — tracer bullets

Build the thinnest end-to-end slice first: one Frappe app shell, one real DocType, seed data, one DSA module, one API, one dashboard card, and one test. Once that proves the full pipe, widen the app phase by phase until all DSA-backed operations sections are present and browser-tested.

| Phase | Tracer-bullet capability | Proves |
|------|---------------------------|--------|
| **[1](phase-1-tracer-bullet-search.md)** | Minimal Frappe app with `OrderOps Order`, seed 10 orders, HashMap lookup API, and one Smart Search dashboard card | The DB → DSA index → API → dashboard → test loop works end to end |
| **[2](phase-2-data-foundation.md)** | Full DocType/role foundation plus default 500-record seed | The app has realistic ecommerce operations data and permissions |
| **[3](phase-3-smart-search-indexes.md)** | Search/index features: HashMap, Trie, Binary Search, BST, Smart Search and High Value Index dashboard sections | Search DSA concepts are visible, tested, and built from Frappe records |
| **[4](phase-4-operations-and-workflows.md)** | Operations features: Heap, Queue, Stack, Linked List, Tree with dashboard sections | Operational DSA concepts power realistic task/job/timeline/workflow flows |
| **[5](phase-5-delivery-graph-dashboard-qa.md)** | Delivery Graph BFS/DFS, final dashboard polish, docs, and browser QA evidence | Full MVP is demoable, documented, and validated through agent/browser testing |

## Confirmed decisions (apply to all phases)

- Build target is a Frappe app named `orderops_engine`.
- Domain is ecommerce order operations, Flipkart-style order/search/delivery/support.
- Architecture is hybrid: real Frappe DocTypes plus explicit DSA modules.
- Frappe DB is the durable source of truth.
- Manual DSA structures are rebuildable in-memory demo/index layers built from Frappe records.
- Dashboard is business-first, with small DSA labels and Explain Mode trace panels.
- Every DSA demo/API returns `ok`, `concept`, `complexity`, `steps`, `result`, and `trace`.
- Default demo seed is 500 orders, 50 customers, 10 delivery hubs, and 50 priority tasks.
- DocTypes use full `OrderOps ...` names.
- Roles are `OrderOps Manager`, `OrderOps Support Agent`, and `OrderOps Viewer`.
- Testing includes DSA unit tests, Frappe integration/API tests, and browser QA evidence.
- Docs include README, DSA mapping, demo walkthrough, and testing guide.

## Shared architecture facts (verified from `plan.md`)

- Target DSA package: `orderops_engine/dsa/` with `explain.py`, `hash_index.py`, `binary_search.py`, `trie.py`, `priority_queue.py`, `fifo_queue.py`, `stack.py`, `linked_list.py`, `workflow_tree.py`, `bst.py`, and `graph.py`.
- Service layer: `orderops_engine/services/seed.py`, `index_builder.py`, `search_service.py`, `operations_service.py`, `timeline_service.py`, `workflow_service.py`, and `route_service.py`.
- API layer: `orderops_engine/api/dashboard.py`.
- Dashboard assets: `orderops_engine/www/orderops-dashboard.py`, `orderops_engine/public/js/orderops_dashboard.js`, and `orderops_engine/public/css/orderops_dashboard.css`.
- Browser QA output: `qa/browser-report.md` and `qa/screenshots/`.
- Planning artifact: `/home/labs/orderops_engine/plan.md`.

## Conventions (apply to all phases)

- Keep the business feature real first; show DSA as the engine/trace, not as a toy visualizer.
- Do not use in-memory DSA structures as durable storage. Rebuild them from Frappe records.
- Keep configurable values admin-editable where reasonable. Seed size, priority thresholds, high-value amount threshold, dashboard toggles, and route/search defaults should live in a Settings DocType or documented config surface before production use.
- Use tests before expanding each module. Pure DSA modules should be independently testable without Frappe when possible.
- Keep functions/classes small and readable. Prefer clear names over clever code.
- Do not integrate real payment/shipping/search providers in MVP.
