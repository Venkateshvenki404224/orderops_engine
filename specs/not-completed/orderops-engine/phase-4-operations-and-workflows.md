# Phase 4 — Operations queues, timelines, and workflow maps

> Read [README.md](README.md) first for shared architecture facts and conventions.

## Goal (the thinnest end-to-end slice)

Implement operations-focused DSA modules and dashboard sections: urgent task priority queue, background job FIFO queue, navigation/undo stack, order timeline linked list, and workflow/status tree. This phase covers Heap, Queue, Stack, Linked List, and Tree with Explain Mode.

## Changes by file

### DSA modules
- **Create `orderops_engine/dsa/priority_queue.py`** — implement `PriorityTaskQueue` using Python heap behavior with lower priority number first.
- **Create `orderops_engine/dsa/fifo_queue.py`** — implement `BackgroundJobQueue` with FIFO enqueue/dequeue trace.
- **Create `orderops_engine/dsa/stack.py`** — implement `NavigationStack` for demo navigation/undo operations.
- **Create `orderops_engine/dsa/linked_list.py`** — implement timeline nodes and traversal from ordered timeline events.
- **Create `orderops_engine/dsa/workflow_tree.py`** — implement order status hierarchy and DFS/BFS traversal.

### Services
- **Create `orderops_engine/services/operations_service.py`** — build priority task heap, background job queue, and stack demos from records.
- **Create `orderops_engine/services/timeline_service.py`** — build linked-list timelines from `OrderOps Timeline Event` records.
- **Create `orderops_engine/services/workflow_service.py`** — expose workflow tree traversal and status map.
- **Modify `orderops_engine/services/index_builder.py`** — include operations/timeline/workflow structures in the common builder where useful.

### API
- **Modify `orderops_engine/api/dashboard.py`** — add `peek_priority_task()`, `pop_priority_task()`, `process_next_background_job()`, `navigation_stack_demo()`, `get_order_timeline(order_id)`, and `get_workflow_tree()`.

### Dashboard
- **Modify `orderops_engine/www/orderops-dashboard.py`** — add sections for Urgent Operations Queue, Background Jobs Monitor, Order Timeline Viewer, and Workflow/Status Map.
- **Modify `orderops_engine/public/js/orderops_dashboard.js`** — add buttons and rendering for heap pop, queue dequeue, stack demo, timeline traversal, and workflow traversal.
- **Modify `orderops_engine/public/css/orderops_dashboard.css`** — style operation cards and timeline/tree output.

### Tests
- **Create `test_dsa_priority_queue.py`** — assert urgent task pops before lower priority tasks.
- **Create `test_dsa_fifo_queue.py`** — assert FIFO behavior.
- **Create `test_dsa_stack.py`** — assert LIFO behavior.
- **Create `test_dsa_linked_list.py`** — assert timeline chain traversal.
- **Create `test_dsa_workflow_tree.py`** — assert tree traversal and status map.
- **Modify `test_dashboard_api.py`** — verify operations API response contract.

### Docs
- **Modify `docs/dsa-mapping.md`** — add operations concepts and Big O.
- **Modify `docs/demo-walkthrough.md`** — add how to demo urgent task pop, job dequeue, timeline, and workflow.

## Verification

1. Run operations DSA tests:
   ```bash
   pytest orderops_engine/orderops_engine/tests/test_dsa_priority_queue.py \
          orderops_engine/orderops_engine/tests/test_dsa_fifo_queue.py \
          orderops_engine/orderops_engine/tests/test_dsa_stack.py \
          orderops_engine/orderops_engine/tests/test_dsa_linked_list.py \
          orderops_engine/orderops_engine/tests/test_dsa_workflow_tree.py -v
   ```
2. Run Frappe API tests:
   ```bash
   bench --site <site> run-tests --app orderops_engine --module orderops_engine.tests.test_dashboard_api
   ```
3. On dashboard, verify:
   - same-day/VIP/delay tasks appear before normal tasks;
   - background jobs process FIFO;
   - stack demo removes last action first;
   - timeline renders in sequence;
   - workflow tree renders statuses and traversal trace.

## Done when

The dashboard feels like a real operations center, and each operations DSA concept has a working feature, tests, and Explain Mode trace backed by seeded Frappe records.
