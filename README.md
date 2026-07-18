# OrderOps Engine

OrderOps Engine is a planned Frappe app for ecommerce order operations that combines real Frappe DocTypes with explicit DSA modules.

The app will demonstrate DSA through real business workflows:

- Smart order search: HashMap, Trie, Binary Search
- Urgent task processing: Heap / Priority Queue
- Background jobs: Queue
- Navigation/undo: Stack
- Order timeline: Linked List
- Workflow/status map: Tree
- High-value order index: BST
- Delivery route explorer: Graph BFS/DFS
- Performance explanation: Big O / DSA Explain Mode

See [`plan.md`](./plan.md) for the implementation plan.

## Current status

Planning repo created. Frappe app implementation has not started yet.

## Locked direction

- Build as a Frappe app.
- Use ecommerce order operations as the domain.
- Use Frappe DB as source of truth.
- Build manual DSA structures as rebuildable in-memory demo/index layers.
- Include DSA unit tests, Frappe integration/API tests, and browser QA evidence.
