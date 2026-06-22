# Contribution [#]: [Issue Title]

**Contribution Number:** 2  
**Student:** Aditya Devarapalli  
**Issue:** [Link](https://github.com/medusajs/medusa/issues/15178)  
**Status:** Phase II  Complete

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

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

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
