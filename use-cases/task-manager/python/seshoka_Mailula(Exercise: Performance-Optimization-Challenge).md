# Performance Optimization: Inventory Analysis (Python)

## 1. Problem Description

The function `find_product_combinations` is intended to suggest product pairs based on a target price. Currently, with 5,000 products, the execution time is 20–30 seconds. This is due to an O(N²) complexity caused by nested loops comparing every product against every other product.

---

## 2. Optimization Process

- **Identification:** Used time profiling to confirm the nested `for` loops and the `any()` duplicate check were the primary bottlenecks.
- **Refactoring:**
  1. Sorted the list of products by price.
  2. Implemented a two-pointer technique to find combinations within the margin.
  3. Removed the expensive `any()` duplicate check by ensuring the inner loop index always starts after the outer loop index.

### Optimized Logic (Conceptual):

```python
# Instead of nested loops comparing every ID:
products.sort(key=lambda x: x['price'])
# Use pointers (left=0, right=len-1) to find target sums in one pass.
```

---

## 3. Results

| Metric | Before | After |
| :--- | :--- | :--- |
| Execution Time | ~25 seconds | < 1 second |
| User Experience | Page blocks on load | Near-instantaneous |
| Viable for Production | No | Yes |

**Significant?** Yes, this allows the recommendation page to load near-instantaneously, improving user retention.

---

## 4. Reflection Questions

**How did the optimization change your understanding of algorithms?**

It highlighted that for large datasets, "brute force" nested loops are almost never viable. It taught me the importance of sorting data first to enable more efficient search patterns (like two-pointer logic).

**What performance improvements did you achieve?**

The execution time dropped from 25 seconds to under 1 second. This is a massive improvement that turns a blocking operation into a background-friendly task..

* **What did you learn about performance bottlenecks?
I learned that nested operations (loops inside loops) are usually the first place to look when code is slow. I also learned that helper functions like any() or list.append() inside loops can become major hidden costs.

* **How would you approach similar issues in the future?
I would start by profiling the code with the cProfile module to see exactly which functions consume the most time before attempting to rewrite any logic.
