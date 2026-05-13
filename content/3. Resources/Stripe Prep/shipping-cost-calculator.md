# Shipping Cost Calculator

**Date:** 2026-02-16
**Category:** Data Processing / Financial Logic
**Parts Completed:** 3/3
**Language:** Python

## Problem Summary

Build a shipping cost calculation system for an e-commerce platform. Calculate total shipping costs for orders based on pricing models that differ by country and product type. Progressive parts: fixed pricing → tiered incremental pricing → mixed fixed + incremental pricing.

## Solutions by Part

### Part 1: Fixed Rate Shipping

**Approach:** Build a lookup dict mapping product → cost for the order's country, then iterate items and accumulate quantity × cost.

```python
def calculate_shipping_cost(order: dict, shipping_cost: dict) -> int:
    total_cost = 0
    country = order["country"]
    cost_map = {}
    for item in shipping_cost[country]:
        product, cost = item["product"], item["cost"]
        cost_map[product] = cost
    for item in order["items"]:
        product, quantity = item["product"], item["quantity"]
        product_cost = cost_map[product]
        total_product_cost = product_cost * quantity
        total_cost += total_product_cost
    return total_cost
```

### Part 2: Tiered Incremental Pricing

**Approach:** Map products to their tier lists. For each item, track remaining quantity and walk through tiers sequentially. Tier capacity = maxQuantity - minQuantity (half-open intervals). Unlimited tiers (maxQuantity=None) consume all remaining.

```python
def calculate_shipping_cost(order: dict, shipping_cost: dict) -> int:
    total_cost = 0
    country = order["country"]
    cost_map = {}
    for item in shipping_cost[country]:
        product, costs = item["product"], item["costs"]
        cost_map[product] = costs
    for item in order["items"]:
        product, quantity = item["product"], item["quantity"]
        product_cost = 0
        tiersleft = quantity
        for tier in cost_map[product]:
            minQuantity, maxQuantity, tierCost = tier["minQuantity"], tier["maxQuantity"], tier["cost"]
            if tiersleft <= 0:
                break
            if maxQuantity == None:
                product_cost = tiersleft * tierCost
                tiersleft = 0
            else:
                tier_capacity = maxQuantity - minQuantity
                curr_quantity = min(tiersleft, tier_capacity)
                tiersleft -= curr_quantity
                product_cost = curr_quantity * tierCost
            total_cost += product_cost
    return total_cost
```

### Part 3: Mixed Pricing Models (Fixed + Incremental)

**Approach:** Minimal extension of Part 2 — add a type check per tier. "fixed" adds cost directly; "incremental" multiplies units × cost.

```python
def calculate_shipping_cost(order: dict, shipping_cost: dict) -> int:
    total_cost = 0
    country = order["country"]
    cost_map = {}
    for item in shipping_cost[country]:
        product, costs = item["product"], item["costs"]
        cost_map[product] = costs
    for item in order["items"]:
        product, quantity = item["product"], item["quantity"]
        product_cost = 0
        tiersleft = quantity
        for tier in cost_map[product]:
            minQuantity, maxQuantity, tierCost, tierType = tier["minQuantity"], tier["maxQuantity"], tier["cost"], tier["type"]
            if tiersleft <= 0:
                break
            if maxQuantity == None:
                if tierType == 'fixed':
                    product_cost = tierCost
                else:
                    product_cost = tiersleft * tierCost
                tiersleft = 0
            else:
                tier_capacity = maxQuantity - minQuantity
                curr_quantity = min(tiersleft, tier_capacity)
                tiersleft -= curr_quantity
                if tierType == 'fixed':
                    product_cost = tierCost
                else:
                    product_cost = curr_quantity * tierCost
            total_cost += product_cost
    return total_cost
```

## Edge Cases

- **Negative quantities** — code would skip all tiers and return 0; should raise an error
- **Quantity of 0** — returns 0 cost, correct behavior
- **Quantity exactly at tier boundary** — e.g., qty=2 with tiers [0,2) and [2,∞): all units land in tier 1 correctly
- **Empty items list** — returns 0
- **Missing product in config** — causes KeyError; should raise ValueError with descriptive message
- **Unsorted tiers** — would process in wrong order; fix by sorting at cost_map build time
- **Product with only a fixed tier** — flat fee applied correctly

## Bugs & Issues

1. **Parameter naming** — initially named parameter `order_country` instead of `order`; misleading name
2. **Variable name mismatch** — used `max_qty`/`min_qty` but extracted into `maxQuantity`/`minQuantity`; caused NameError
3. **Accumulation bug** — `total_cost += product_cost` inside tier loop while `product_cost` was a running sum via `+=`; double-counted costs. Fixed by using `=` (not `+=`) for `product_cost` each tier iteration
4. **Wrong dict key** — initially used `tier["quantityCost"]` instead of `tier["cost"]`
5. **Unlimited tier bug** — used `quantity` (total) instead of `tiersleft` (remaining) in unlimited tier branch

## Key Learnings

- **Trace through concrete examples** before submitting — catches accumulation bugs and variable name issues
- **Half-open intervals [min, max)** — capacity = max - min naturally; no off-by-one needed
- **Minimal evolution across parts** — cost_map pattern stayed consistent; Part 3 was a 4-line diff from Part 2
- **Sort once at build time** — when handling unsorted tiers, sort in cost_map construction, not per-item
- **Fail loudly in payments** — raise errors for missing products rather than silently returning 0

## Code Quality Notes

- Good use of lookup dict for O(1) product access
- `tiersleft` should be `tiers_left` or `remaining` for Pythonic snake_case
- Intermediate variables are slightly verbose but aid readability
- Could extract tier cost calculation into a helper function for cleaner separation

## Q&A Highlights

- **Why raise an error for missing products?** In a payments system, silently ignoring = undercharging = real money lost. Failing loudly is the safer default.
- **Where to sort unsorted tiers?** Once at cost_map build time: `cost_map[product] = sorted(costs, key=lambda x: x["minQuantity"])`
- **Testing categories to mention in interview:** happy path, boundary quantities, zero/negative quantities, empty inputs, missing config entries, single-tier products
