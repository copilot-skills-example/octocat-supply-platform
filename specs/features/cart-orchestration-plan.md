# Cart Page Multi-Repo Orchestration Plan

**Master Issue:** [copilot-skills-example/octocat-supply-platform#1](https://github.com/copilot-skills-example/octocat-supply-platform/issues/1)

## Status: 🚀 Ready for Implementation

This document provides the orchestration plan for implementing the Cart Page functionality across multiple repositories in the OctoCAT Supply Chain application.

---

## Repository Breakdown

### 🔧 Backend API (octocat-supply-api)
**Integration Order:** 1 (No dependencies)  
**Assignee:** @copilot

#### Scope
- Create `POST /api/orders` endpoint for order placement from cart
- Implement order validation logic (product existence, stock levels)
- Transaction-based order + order details creation
- Swagger documentation updates
- Integration tests for new endpoint

#### Deliverables
1. New route handler in Express app
2. Repository method for order creation with details
3. Updated Swagger spec
4. Test coverage for happy path and error cases

#### Reference
See `/specs/features/cart-page.md` Phase 1 for detailed requirements.

---

### 🎨 Frontend UI (octocat-supply-web)
**Integration Order:** 2 (Depends on octocat-supply-api)  
**Assignee:** @copilot

#### Scope
- `CartContext` React provider for cart state management
- Cart page component with item list, summary, and order placement
- Navigation updates (cart icon with badge)
- Products page "Add to Cart" integration
- Toast notifications for user feedback
- Tailwind CSS styling with dark mode support

#### Deliverables
1. `src/contexts/CartContext.tsx`
2. `src/pages/CartPage.tsx`
3. Updated `src/components/Navigation.tsx`
4. Updated `src/pages/ProductsPage.tsx`
5. Route configuration for `/cart`
6. React Query integration for API calls

#### Reference
See `/specs/features/cart-page.md` Phase 2 for detailed requirements.

---

## Integration Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. User browses Products page                                │
│    └─> Clicks "Add to Cart" on product card                  │
│        └─> CartContext.addToCart() updates state             │
│            └─> Toast notification confirms addition          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. User navigates to /cart                                   │
│    └─> CartPage renders items from CartContext              │
│        └─> User can update quantities or remove items        │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. User clicks "Place Order"                                 │
│    └─> Frontend calls POST /api/orders with cart payload    │
│        ├─> API validates products exist                      │
│        ├─> API creates Order + OrderDetail records           │
│        └─> API returns created order                         │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. Success handling                                          │
│    ├─> Frontend shows success toast                          │
│    ├─> CartContext.clearCart() empties cart                  │
│    └─> User redirected to orders or confirmation page        │
└─────────────────────────────────────────────────────────────┘
```

---

## Entity Interactions

### Order Creation
```typescript
// Frontend prepares cart data
const cartPayload = {
  branchId: currentBranchId, // From app context or hardcoded
  items: cartItems.map(item => ({
    productId: item.productId,
    quantity: item.quantity
  }))
};

// Backend processes
// 1. Fetch all products by IDs (validate existence)
// 2. Check stock levels (if applicable)
// 3. Create Order record
// 4. Create OrderDetail records with current product prices
// 5. Return enriched order object
```

### Database Tables Affected
- **orders** (INSERT)
- **order_details** (INSERT multiple)
- **products** (SELECT for validation)

---

## Testing Strategy

### Backend Tests
- [ ] POST /api/orders with valid payload → 201 Created
- [ ] POST /api/orders with nonexistent product → 400 Bad Request
- [ ] POST /api/orders with missing branchId → 400 Bad Request
- [ ] Verify Order and OrderDetail records created in transaction
- [ ] Verify unitPrice captured from current Product.price

### Frontend Tests
- [ ] CartContext adds/removes/updates items correctly
- [ ] Cart page renders items from context
- [ ] Cart page calculates subtotal correctly
- [ ] Place Order button calls API with correct payload
- [ ] Cart clears after successful order
- [ ] Toast notifications appear on actions

### Integration Tests
- [ ] End-to-end flow: add to cart → view cart → place order
- [ ] Verify order appears in database
- [ ] Verify cart cleared in frontend after success

---

## Rollout Checklist

- [ ] API endpoint implemented and tested (octocat-supply-api)
- [ ] API merged to main branch
- [ ] Frontend cart functionality implemented (octocat-supply-web)
- [ ] Frontend tested against deployed API
- [ ] Dark mode verified on cart page
- [ ] Responsive design tested (mobile, tablet, desktop)
- [ ] Integration testing completed
- [ ] Documentation updated
- [ ] Feature deployed to staging/production

---

## References

- **Entity Model:** `/architecture/entity-model.md`
- **API Contract:** `/specs/features/cart-page.md`
- **Design Mockup:** `/docs/design/cart.png`
- **Dependencies Map:** `/.github/skills/multi-repo-orchestration/dependencies.json`

---

## Notes for Implementers

### API Implementer
- Use existing repository pattern (e.g., `OrdersRepository`, `ProductsRepository`)
- Wrap Order + OrderDetail creation in a transaction
- Capture current `product.price` into `orderDetail.unitPrice` (price may change later)
- Default order status to 'pending'
- Set `orderDate` to current timestamp
- Return full order object with nested details and product info

### Frontend Implementer
- Use React Context API for cart state (avoid prop drilling)
- Persist cart to localStorage/sessionStorage for page reload resilience
- Use React Query for POST /api/orders mutation
- Follow existing Tailwind class patterns for consistency
- Support ThemeContext for dark mode toggle
- Add loading states to "Place Order" button
- Handle API errors gracefully with user-friendly messages

### Shared Concerns
- Coordinate on `branchId` value (hardcoded default or from user context?)
- Agree on product image display (use `imgName` field, construct URL)
- Decide on order confirmation page vs. redirect to orders list
- Consider future: stock level checks, inventory decrement
