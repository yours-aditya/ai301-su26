# Contribution [5]: [Issue Title]

**Contribution Number:** 5  
**Student:** Aditya Devarapalli  
**Issue:** [Link](https://github.com/omnigent-ai/omnigent/issues/1921)  
**Status:** Phase V  Complete

---

## Why I Chose This Issue

Adding cache tags lets Medusa invalidate the stale cache whenever related variants or inventory items change. Disabling caching is simpler and safer if inventory availability changes often, but tagged caching keeps performance benefits while fixing correctness.

---

## Understanding the Issue

### Problem Description

The cache stores an empty `product_variant_inventory_items` result when a variant starts with inventory management off. Later, even after inventory is enabled and an item is linked, Medusa keeps reading the stale cached empty result.

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. Enable the Medusa caching module
2. Create a product with at least one variant
3. While creating the variant, keep Inventory Management off
4. Open the product detail page
5. This triggers `product_variant_inventory_items` query
6. Since no inventory item is linked yet, the query returns `[]`
7. That empty result gets cached without cache tags
8. Edit the variant and turn Inventory Management on
9. Create an inventory item and link it to the variant
10. Reopen the product detail page.
11. The variant still shows incorrect inventory, usually `inventory_quantity: null`, because Medusa keeps reading the stale cached `[]`
12. Add stock to the linked inventory item
13. Reopen the product or query it from the storefront
14. Inventory is still incorrect until the cache TTL expires

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

The `product_variant_inventory_items` query is cached, but it has no cache invalidation tags. So when a variant later gets linked to an inventory item, Medusa has no way to know that this cached query result should be invalidated.

### Proposed Solution

Change the cache config for `product_variant_inventory_items` cache. This makes the cache invalidate when the variant changes, inventory items change, and variant-to-inventory links change.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The `product_variant_inventory_items` query inside `getDataForComputation` was being cached without cache tags. If a variant initially had inventory management disabled, the query returned an empty array and cached that empty result. Later, when inventory management was enabled and an inventory item was linked, the cache was not invalidated, so Medusa continued returning stale inventory availability data.

**Match:** The nearby `sales_channel_locations` query already uses cache tags to invalidate cached data when related sales channel or stock location data changes. The fix follows the same pattern by attaching relevant cache tags to the `product_variant_inventory_items` query

**Plan:**
1. Modify `packages/core/utils/src/product/get-variant-availability.ts`
2. Add cache tags to the `product_variant_inventory_items query`
3. Tag the cache by queried product variant IDs and add inventory-related list tags so inventory item changes or variant-inventory links can invalidate the cached result.
4. Run existing tests around product variant availability and inventory behavior and manually verify the reproduction flow with caching enabled.

**Implement:** Implemented by updating the cache configuration for the `product_variant_inventory_items` query to include invalidation tags such as:

tags: [
  ...data.variant_ids.map((id) => `ProductVariant:${id}`),
  "InventoryItem:list:*",
  "ProductVariantInventoryItem:list:*",
]

This ensures the cached query result is invalidated when a product variant, inventory item, or variant-inventory-item relationship changes.

**Review:**
- Change is scoped to the availability computation path.

- Cache behavior now matches the surrounding query patterns.

- No unrelated product or inventory logic was changed.

- Cache tags are tied to the relevant entities.

- Code follows the existing Medusa style.

- Existing behavior remains unchanged when caching is disabled.

**Evaluate:** The fix should prevent stale empty inventory-item results from being reused after inventory management is enabled and an inventory item is linked to a variant.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: Variant with inventory management disabled returns no linked inventory items
- [ ] Test case 2: After enabling inventory management and linking an inventory item, the query returns the new inventory item instead of stale cached `[]`
- [ ] Test case 3: Updating inventory item or stock level invalidates availability-related cached data

### Integration Tests

- [ ] Integration scenario 1: Create product variant with inventory management disabled, load product once, then enable inventory management and link inventory item. Verify `variant.inventory_quantity` updates correctly
- [ ] Integration scenario 2: Add stock to the linked inventory item and verify product/admin/storefront queries return the updated inventory quantity.

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
