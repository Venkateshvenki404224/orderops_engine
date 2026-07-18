# Phase 3 — Smart search and ordered indexes

> Read [README.md](README.md) first for shared architecture facts and conventions.

## Goal (the thinnest end-to-end slice)

Implement the search-focused DSA modules and expose them through business dashboard sections: exact order lookup, autocomplete, sorted amount search, and high-value ordered index. This phase covers HashMap, Trie, Binary Search, BST, Array/List, Linear Search baseline, and Big O Explain Mode.

## Changes by file

### DSA modules
- **Modify `orderops_engine/dsa/hash_index.py`** — support full `OrderOps Order` payloads and compare HashMap lookup against linear search baseline.
- **Create `orderops_engine/dsa/binary_search.py`** — implement amount/date boundary search with trace.
- **Create `orderops_engine/dsa/trie.py`** — implement `AutocompleteTrie` for order IDs, customer names, product names, and SKUs.
- **Create `orderops_engine/dsa/bst.py`** — implement `OrderAmountBST` with insert, search, and in-order traversal.

### Services
- **Modify `orderops_engine/services/index_builder.py`** — build search indexes from seeded Frappe records: order hash map, autocomplete trie, sorted amount list, and amount BST.
- **Create `orderops_engine/services/search_service.py`** — provide business methods for exact lookup, autocomplete, linear-vs-hash comparison, binary amount search, and high-value traversal.

### API
- **Modify `orderops_engine/api/dashboard.py`** — add `autocomplete(prefix)`, `binary_search_amount(amount)`, `compare_linear_vs_hash(order_id)`, `search_amount_bst(amount)`, and `get_high_value_inorder()`.

### Dashboard
- **Modify `orderops_engine/public/js/orderops_dashboard.js`** — add Smart Search Center and High Value Order Index interactions.
- **Modify `orderops_engine/public/css/orderops_dashboard.css`** — add trace panel styling.
- **Modify `orderops_engine/www/orderops-dashboard.py`** — render sections for Smart Search Center and High Value Order Index.

### Tests
- **Create `test_dsa_binary_search.py`** — found/missing/edge cases with step count.
- **Create `test_dsa_trie.py`** — prefix suggestions and missing prefix.
- **Create `test_dsa_bst.py`** — insert/search/in-order traversal and unbalanced warning if included.
- **Modify `test_dsa_hash_index.py`** — include linear comparison Explain Mode.
- **Modify `test_dashboard_api.py`** — verify new API response contract.

### Docs
- **Create/Modify `docs/dsa-mapping.md`** — explain search features and Big O: O(1), O(n), O(log n), prefix length behavior, and BST caveats.

## Verification

1. Run pure DSA tests:
   ```bash
   pytest orderops_engine/orderops_engine/tests/test_dsa_hash_index.py \
          orderops_engine/orderops_engine/tests/test_dsa_binary_search.py \
          orderops_engine/orderops_engine/tests/test_dsa_trie.py \
          orderops_engine/orderops_engine/tests/test_dsa_bst.py -v
   ```
2. Run dashboard API tests:
   ```bash
   bench --site <site> run-tests --app orderops_engine --module orderops_engine.tests.test_dashboard_api
   ```
3. Open dashboard and verify:
   - exact order lookup shows HashMap trace;
   - autocomplete returns order/customer/product suggestions;
   - binary search shows comparison steps;
   - high-value index shows BST in-order output.

## Done when

Search features are useful from the dashboard, all search/index DSA modules are tested, and Explain Mode makes the difference between linear scan, HashMap, Trie, Binary Search, and BST visible with real seeded records.
