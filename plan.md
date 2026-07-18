# OrderOps Engine Implementation Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task after the plan is approved.

**Goal:** Build a production-style Frappe ecommerce order operations app that visibly demonstrates core DSA concepts through real order/search/delivery/support workflows.

**Architecture:** Frappe DB is the source of truth. Manual DSA modules are rebuildable in-memory index/demo engines built from seeded Frappe records. The app exposes business-first dashboard sections with DSA Explain Mode showing result, steps, complexity, and trace.

**Tech Stack:** Frappe Framework, Python, MariaDB, Frappe Desk, custom Frappe Page, pytest/Frappe tests, browser QA via agent/browser dogfood workflow.

---

## Locked grill decisions

1. **Build target:** Frappe app.
2. **Architecture style:** Hybrid: real Frappe operations app + explicit DSA learning modules.
3. **Domain:** Ecommerce order operations, Flipkart-style order/search/delivery/support.
4. **MVP scope:** Full but phased MVP. Every DSA concept gets one working feature/API.
5. **Planning location:** `/home/labs/orderops_engine/plan.md`; app/bench creation happens after plan approval.
6. **UI:** Frappe Desk DocTypes + custom business-first OrderOps Dashboard page.
7. **Data strategy:** Seeded Frappe DB records are source of truth; DSA structures are built from those records.
8. **Testing:** DSA unit tests + Frappe integration/API tests + agent/browser dashboard QA test with evidence report.
9. **Dashboard style:** Business-first sections with small DSA labels.
10. **Architecture rule:** Dual-path pattern: Frappe DB for durable storage, manual DSA for rebuildable in-memory demo/index layer.
11. **Explain Mode:** Every DSA demo/API returns result, steps, Big O complexity, and trace output.
12. **Repo strategy:** Local Git repo now, GitHub later after plan approval.
13. **DocType naming:** Full names with `OrderOps ...` prefix.
14. **Sample data size:** Default seed = 500 orders, 50 customers, 10 delivery hubs, 50 priority tasks.
15. **Permissions:** Basic roles: OrderOps Manager, OrderOps Support Agent, OrderOps Viewer.
16. **Docs:** Include README, DSA mapping, demo walkthrough, and testing docs.
17. **DSA module style:** Hybrid: classes for larger structures, functions for simple algorithms.

---

## Product summary

**OrderOps Engine** is a Frappe app for ecommerce operations teams. It manages sample customer orders, priority operational tasks, background jobs, delivery route exploration, order timelines, workflow/status maps, and smart search.

This is not a toy DSA visualizer. Each DSA concept powers a business feature:

| DSA concept | Product feature | Why it is used |
|---|---|---|
| Big O | DSA Explain Mode | Shows how work grows with data size |
| Array/List | Recent orders and sorted report arrays | Ordered temporary data and report rows |
| Linear Search | Baseline comparison for search | Shows why naive scanning is slow |
| Binary Search | Search sorted order amounts/date boundaries | Fast search on sorted lists |
| HashMap/Dictionary | Exact order lookup by order ID | Average O(1) key lookup |
| Trie | Search autocomplete for orders/customers/products | Prefix search without scanning every record |
| Heap/Priority Queue | Urgent operations queue | Highest-priority task comes first |
| Queue | Background job monitor/simulation | FIFO processing |
| Stack | Dashboard navigation/undo history | LIFO recent action handling |
| Linked List | Order timeline viewer | Event chain traversal |
| Tree | Workflow/status map | Parent-child status structure |
| BST | High-value order amount index | Ordered insert/search/in-order traversal |
| Graph | Delivery route explorer | Hubs/routes with BFS/DFS traversal |

---

## Target Frappe app structure

After plan approval, create the actual app in a selected bench:

```txt
apps/orderops_engine/
├── orderops_engine/
│   ├── __init__.py
│   ├── hooks.py
│   ├── modules.txt
│   ├── dsa/
│   │   ├── __init__.py
│   │   ├── explain.py
│   │   ├── hash_index.py
│   │   ├── binary_search.py
│   │   ├── trie.py
│   │   ├── priority_queue.py
│   │   ├── fifo_queue.py
│   │   ├── stack.py
│   │   ├── linked_list.py
│   │   ├── workflow_tree.py
│   │   ├── bst.py
│   │   └── graph.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── seed.py
│   │   ├── index_builder.py
│   │   ├── search_service.py
│   │   ├── operations_service.py
│   │   ├── timeline_service.py
│   │   ├── workflow_service.py
│   │   └── route_service.py
│   ├── api/
│   │   ├── __init__.py
│   │   └── dashboard.py
│   ├── www/
│   │   └── orderops-dashboard.py
│   ├── public/
│   │   ├── js/orderops_dashboard.js
│   │   └── css/orderops_dashboard.css
│   ├── fixtures/
│   │   └── roles.json
│   └── tests/
│       ├── test_dsa_hash_index.py
│       ├── test_dsa_binary_search.py
│       ├── test_dsa_trie.py
│       ├── test_dsa_priority_queue.py
│       ├── test_dsa_fifo_queue.py
│       ├── test_dsa_stack.py
│       ├── test_dsa_linked_list.py
│       ├── test_dsa_workflow_tree.py
│       ├── test_dsa_bst.py
│       ├── test_dsa_graph.py
│       ├── test_seed.py
│       ├── test_index_builder.py
│       └── test_dashboard_api.py
├── docs/
│   ├── dsa-mapping.md
│   ├── demo-walkthrough.md
│   └── testing.md
├── qa/
│   ├── browser-report.md
│   └── screenshots/
└── plan.md
```

---

## DocTypes

Use full `OrderOps ...` names.

### OrderOps Customer

Fields:

- `customer_name` Data, required
- `email` Data
- `phone` Data
- `city` Data
- `is_vip` Check
- `customer_score` Int

### OrderOps Product

Fields:

- `product_name` Data, required
- `category` Data
- `price` Currency
- `sku` Data, unique-ish sample value

### OrderOps Order

Fields:

- `order_id` Data, unique, required
- `customer` Link to `OrderOps Customer`
- `order_date` Datetime
- `status` Select: Placed, Payment Failed, Paid, Packed, Shipped, Delivered, Cancelled, Returned
- `amount` Currency
- `priority_level` Int, lower means more urgent
- `delivery_hub` Link to `OrderOps Delivery Hub`
- `is_delayed` Check
- `is_high_value` Check

### OrderOps Order Item

Child table fields:

- `product` Link to `OrderOps Product`
- `qty` Int
- `rate` Currency
- `amount` Currency

Parent: `OrderOps Order`.

### OrderOps Delivery Hub

Fields:

- `hub_name` Data, required
- `city` Data
- `hub_type` Select: Warehouse, Sorting Hub, Local Hub, Delivery Area
- `capacity_score` Int

### OrderOps Delivery Route

Fields:

- `from_hub` Link to `OrderOps Delivery Hub`
- `to_hub` Link to `OrderOps Delivery Hub`
- `distance_km` Float
- `estimated_minutes` Int
- `is_active` Check

### OrderOps Priority Task

Fields:

- `task_title` Data, required
- `order` Link to `OrderOps Order`
- `priority` Int, lower means more urgent
- `task_type` Select: Payment Alert, Same Day Delivery, VIP Complaint, Delay Alert, Normal Followup
- `status` Select: Pending, Processing, Done, Cancelled

### OrderOps Timeline Event

Fields:

- `order` Link to `OrderOps Order`
- `event_type` Data
- `event_message` Small Text
- `event_time` Datetime
- `sequence` Int

### OrderOps Background Job

Fields:

- `job_title` Data
- `job_type` Select: Send Email, Generate PDF, Sync Tracking, Rebuild Index, Send Notification
- `status` Select: Queued, Processing, Done, Failed
- `created_order` Int

---

## Dashboard sections

Route/page target:

```txt
/orderops-dashboard
```

Business-first sections:

1. **Smart Search Center**
   - Exact order lookup powered by HashMap.
   - Autocomplete powered by Trie.
   - Sorted amount/date lookup powered by Binary Search.
   - Displays Explain Mode: steps, complexity, trace.

2. **Urgent Operations Queue**
   - Shows next urgent tasks powered by Heap/Priority Queue.
   - Button: `Pop next urgent task`.
   - Displays priority ordering trace.

3. **Background Jobs Monitor**
   - FIFO job queue simulation.
   - Button: `Process next job`.
   - Shows Queue Explain Mode.

4. **Order Timeline Viewer**
   - Select an order.
   - Render timeline as linked event chain.
   - Shows Linked List traversal trace.

5. **Delivery Route Explorer**
   - Select source and destination hub.
   - Run BFS/DFS traversal on route graph.
   - Shows visited hubs and trace.

6. **Workflow/Status Map**
   - Shows order status hierarchy/tree.
   - Demonstrates DFS/BFS traversal through workflow statuses.

7. **High Value Order Index**
   - Builds BST by order amount.
   - Shows search by amount and in-order traversal output.

Each section must have a small label:

```txt
DSA used: <Concept> | Complexity: <Big O>
```

---

## DSA Explain Mode contract

All DSA APIs should return a consistent object:

```python
{
    "ok": True,
    "concept": "HashMap",
    "complexity": "O(1) average",
    "steps": 1,
    "result": {...},
    "trace": [
        "Looked up key ORD-0100 directly in order_by_id"
    ]
}
```

Create helper dataclass or function in:

```txt
orderops_engine/dsa/explain.py
```

Recommended Python shape:

```python
from dataclasses import dataclass, field
from typing import Any

@dataclass
class ExplainResult:
    ok: bool
    concept: str
    complexity: str
    steps: int = 0
    result: Any = None
    trace: list[str] = field(default_factory=list)

    def to_dict(self):
        return {
            "ok": self.ok,
            "concept": self.concept,
            "complexity": self.complexity,
            "steps": self.steps,
            "result": self.result,
            "trace": self.trace,
        }
```

---

## Phase 0: Repo/app setup planning

### Task 0.1: Confirm bench target before app creation

**Objective:** Avoid installing the app into the wrong bench.

**Files:** None yet.

**Steps:**

1. Inspect `/home/labs` for available benches.
2. Identify a safe development bench or create a new one.
3. Confirm selected bench before running `bench new-app`.

**Verification:** Selected bench path is explicitly documented in `docs/testing.md` before implementation starts.

---

## Phase 1: Create Frappe app shell

### Task 1.1: Create app

**Objective:** Create the `orderops_engine` app in the selected bench.

**Commands:**

```bash
cd <selected-bench>
bench new-app orderops_engine
```

**Expected:** `apps/orderops_engine` exists.

### Task 1.2: Add docs skeleton

**Objective:** Add project docs.

**Files:**

- `apps/orderops_engine/README.md`
- `apps/orderops_engine/plan.md`
- `apps/orderops_engine/docs/dsa-mapping.md`
- `apps/orderops_engine/docs/demo-walkthrough.md`
- `apps/orderops_engine/docs/testing.md`

**Verification:** Files exist and are committed.

---

## Phase 2: DocTypes and roles

### Task 2.1: Create roles

**Objective:** Add basic Frappe roles.

**Roles:**

- OrderOps Manager
- OrderOps Support Agent
- OrderOps Viewer

**Verification:** Roles appear in Frappe Role list and fixtures export succeeds.

### Task 2.2: Create master DocTypes

**Objective:** Create customer, product, hub, and route DocTypes.

**DocTypes:**

- OrderOps Customer
- OrderOps Product
- OrderOps Delivery Hub
- OrderOps Delivery Route

**Verification:** Can create one record for each in Desk.

### Task 2.3: Create order DocTypes

**Objective:** Create order, item, timeline, task, and background job DocTypes.

**DocTypes:**

- OrderOps Order
- OrderOps Order Item
- OrderOps Timeline Event
- OrderOps Priority Task
- OrderOps Background Job

**Verification:** Can create an order with child items and linked customer/hub.

---

## Phase 3: Seed data

### Task 3.1: Add seed service

**Objective:** Generate realistic ecommerce demo data.

**File:**

- `orderops_engine/services/seed.py`

**Default seed:**

```txt
500 orders
50 customers
10 delivery hubs
50 priority tasks
```

**Command target:**

```bash
bench --site <site> execute orderops_engine.services.seed.seed_demo_data
```

**Verification:** Counts match expected values.

### Task 3.2: Add seed tests

**Objective:** Verify seed data creates linked records correctly.

**File:**

- `orderops_engine/tests/test_seed.py`

**Verification command:**

```bash
bench --site <site> run-tests --app orderops_engine --module orderops_engine.tests.test_seed
```

---

## Phase 4: Pure DSA modules

### Task 4.1: ExplainResult helper

**Objective:** Standardize DSA Explain Mode response.

**File:** `orderops_engine/dsa/explain.py`

**Test:** `orderops_engine/tests/test_dsa_explain.py`

### Task 4.2: Hash index

**Objective:** Implement exact order lookup with dict.

**File:** `orderops_engine/dsa/hash_index.py`

**Class/function:** `OrderHashIndex`

**Feature:** `lookup(order_id, explain=True)` returns O(1) trace.

### Task 4.3: Binary search

**Objective:** Implement binary search over sorted orders/amounts.

**File:** `orderops_engine/dsa/binary_search.py`

**Function:** `binary_search_amount(sorted_amounts, target, explain=True)`

### Task 4.4: Trie autocomplete

**Objective:** Implement prefix search for orders/customers/products.

**File:** `orderops_engine/dsa/trie.py`

**Class:** `AutocompleteTrie`

### Task 4.5: Heap priority queue

**Objective:** Implement urgent task priority queue.

**File:** `orderops_engine/dsa/priority_queue.py`

**Class:** `PriorityTaskQueue`

### Task 4.6: FIFO queue

**Objective:** Implement background job queue simulation.

**File:** `orderops_engine/dsa/fifo_queue.py`

**Class:** `BackgroundJobQueue`

### Task 4.7: Stack

**Objective:** Implement navigation/undo stack.

**File:** `orderops_engine/dsa/stack.py`

**Class:** `NavigationStack`

### Task 4.8: Linked list timeline

**Objective:** Implement event chain traversal.

**File:** `orderops_engine/dsa/linked_list.py`

**Classes:** `TimelineNode`, `OrderTimelineList`

### Task 4.9: Workflow tree

**Objective:** Implement order status workflow tree traversal.

**File:** `orderops_engine/dsa/workflow_tree.py`

**Class:** `WorkflowTree`

### Task 4.10: BST amount index

**Objective:** Implement insert/search/in-order traversal by amount.

**File:** `orderops_engine/dsa/bst.py`

**Classes:** `BSTNode`, `OrderAmountBST`

### Task 4.11: Delivery graph

**Objective:** Implement BFS/DFS route traversal.

**File:** `orderops_engine/dsa/graph.py`

**Class:** `DeliveryRouteGraph`

**Verification for Phase 4:**

```bash
pytest orderops_engine/orderops_engine/tests/test_dsa_*.py -v
```

Expected: all pure DSA tests pass.

---

## Phase 5: Index builder services

### Task 5.1: Build indexes from Frappe records

**Objective:** Read Frappe records and build all DSA structures.

**File:** `orderops_engine/services/index_builder.py`

**Function:** `build_orderops_indexes()`

Returns:

```python
{
    "hash_index": OrderHashIndex,
    "trie": AutocompleteTrie,
    "priority_queue": PriorityTaskQueue,
    "fifo_queue": BackgroundJobQueue,
    "workflow_tree": WorkflowTree,
    "amount_bst": OrderAmountBST,
    "delivery_graph": DeliveryRouteGraph,
    "sorted_amounts": list,
}
```

**Verification:** Index builder test confirms structures contain seeded records.

---

## Phase 6: Dashboard APIs

### Task 6.1: Smart search APIs

**File:** `orderops_engine/api/dashboard.py`

Endpoints:

- `lookup_order(order_id)`
- `autocomplete(prefix)`
- `binary_search_amount(amount)`

### Task 6.2: Operations APIs

Endpoints:

- `peek_priority_task()`
- `pop_priority_task()`
- `process_next_background_job()`
- `navigation_stack_demo()`

### Task 6.3: Timeline/workflow/index APIs

Endpoints:

- `get_order_timeline(order_id)`
- `get_workflow_tree()`
- `search_amount_bst(amount)`
- `get_high_value_inorder()`

### Task 6.4: Delivery graph APIs

Endpoints:

- `route_bfs(start_hub)`
- `route_dfs(start_hub)`

**Verification:** API tests assert response includes `ok`, `concept`, `complexity`, `steps`, `result`, `trace`.

---

## Phase 7: Custom dashboard page

### Task 7.1: Create dashboard page

**Files:**

- `orderops_engine/www/orderops-dashboard.py`
- `orderops_engine/public/js/orderops_dashboard.js`
- `orderops_engine/public/css/orderops_dashboard.css`

**Sections:**

- Smart Search Center
- Urgent Operations Queue
- Background Jobs Monitor
- Order Timeline Viewer
- Delivery Route Explorer
- Workflow/Status Map
- High Value Order Index

### Task 7.2: Add DSA labels and trace panels

Each section must show:

```txt
DSA used
Complexity
Step count
Trace lines
```

**Verification:** Browser can open `/orderops-dashboard` and every section renders.

---

## Phase 8: Documentation

### Task 8.1: README

**File:** `README.md`

Must include:

- Product pitch
- Install steps
- Seed command
- Dashboard URL
- Test commands

### Task 8.2: DSA mapping doc

**File:** `docs/dsa-mapping.md`

Must include feature-to-DSA mapping table and Big O explanation.

### Task 8.3: Demo walkthrough

**File:** `docs/demo-walkthrough.md`

Must include a 5-minute demo script:

1. Seed data.
2. Open dashboard.
3. Show Smart Search.
4. Show Priority Queue.
5. Show Delivery Graph.
6. Explain Big O traces.

### Task 8.4: Testing doc

**File:** `docs/testing.md`

Must include:

- DSA unit test command
- Frappe integration test command
- Browser QA workflow
- Known limitations

---

## Phase 9: Browser QA / dogfood test

### Task 9.1: Run browser QA

**Objective:** Validate dashboard like a user, not just with unit tests.

**Output:**

```txt
qa/browser-report.md
qa/screenshots/
```

**Scope:**

- Open dashboard.
- Check console errors.
- Test Smart Search valid/invalid input.
- Test autocomplete.
- Pop priority task.
- Process background job.
- Load order timeline.
- Run delivery route BFS/DFS.
- Verify DSA labels and Explain Mode trace are visible.

**Verification:** `qa/browser-report.md` contains screenshots/evidence and issue summary.

---

## Acceptance criteria

MVP is complete only when all are true:

- [ ] Frappe app installs on selected bench/site.
- [ ] All `OrderOps ...` DocTypes exist.
- [ ] Roles exist: Manager, Support Agent, Viewer.
- [ ] Seed command creates 500 orders, 50 customers, 10 hubs, 50 priority tasks.
- [ ] Each DSA concept has a tested Python module.
- [ ] Index builder builds DSA structures from Frappe DB records.
- [ ] Dashboard APIs return Explain Mode objects.
- [ ] Custom dashboard renders all business-first sections.
- [ ] Browser QA report exists with screenshots/evidence.
- [ ] README and docs explain feature-to-DSA mapping.
- [ ] Tests pass.

---

## Non-goals for MVP

Do not build these in MVP:

- Real payment gateway integration.
- Real shipping provider integration.
- ERPNext Sales Order sync.
- Complex permissions by hub/territory.
- Real-time websockets.
- Large stress dataset beyond default 500 orders.
- Production search engine integration like Elasticsearch/Meilisearch.

These can be Phase 2 after MVP.

---

## Implementation execution style

Use TDD where possible:

1. Write failing DSA test.
2. Implement minimal DSA module.
3. Run test.
4. Wire to service/API.
5. Add Frappe integration test.
6. Commit.

Commit frequently:

```bash
git add <files>
git commit -m "feat: add <specific capability>"
```

Recommended implementation order:

```txt
1. Create app shell
2. Create DocTypes and roles
3. Add seed data
4. Add DSA modules with unit tests
5. Add index builder
6. Add dashboard APIs
7. Add dashboard UI
8. Add docs
9. Run browser QA
```

---

## Open decisions before implementation

These are intentionally left for the next grill/planning step:

1. Which Frappe bench/site should host the app?
2. Should GitHub repo be public or private?
3. Should dashboard use plain JS/CSS or Frappe Desk page APIs/components?
4. Should generated seed data be deterministic using a fixed random seed?
5. Should optional 10,000-order stress seed be implemented in MVP+1?
