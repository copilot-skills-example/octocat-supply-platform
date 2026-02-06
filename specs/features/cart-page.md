# Cart Page Feature Specification

**Parent Issue:** copilot-skills-example/octocat-supply-platform#1

## Overview

This specification breaks down the Cart Page functionality into repository-specific tasks that must be implemented in parallel across the OctoCAT Supply Chain application.

## Architecture

This feature spans two repositories:
1. **octocat-supply-api** - Backend API endpoints and data persistence
2. **octocat-supply-web** - Frontend UI components and state management

## Implementation Order

### Phase 1: API Foundation (octocat-supply-api)
**Priority:** High (blocks frontend)  
**Integration Order:** 1

#### Tasks
1. Create `POST /api/orders` endpoint
   - Accept payload: `{ branchId: number, items: [{ productId: number, quantity: number }] }`
   - Validate all products exist and have sufficient stock
   - Create Order record with current timestamp
   - Create OrderDetail records for each cart item (capture current product price)
   - Use transaction to ensure atomicity
   - Return created order with full details

2. Update Swagger documentation
   - Document request/response schemas
   - Add validation error examples

3. Add integration tests
   - Test successful order creation
   - Test product not found validation
   - Test insufficient stock validation
   - Test transaction rollback on error

#### Entity Mapping
- **Order**: `orderId`, `branchId`, `orderDate`, `status` (default: 'pending')
- **OrderDetail**: `orderDetailId`, `orderId`, `productId`, `quantity`, `unitPrice`
- **Product**: Read `productId`, `name`, `price` for validation and order detail creation

#### API Contract
```typescript
// Request
POST /api/orders
Content-Type: application/json

{
  "branchId": 1,
  "items": [
    { "productId": 5, "quantity": 2 },
    { "productId": 12, "quantity": 1 }
  ]
}

// Success Response (201)
{
  "orderId": 42,
  "branchId": 1,
  "orderDate": "2026-02-06T18:30:00.000Z",
  "status": "pending",
  "details": [
    {
      "orderDetailId": 100,
      "productId": 5,
      "quantity": 2,
      "unitPrice": 29.99,
      "product": {
        "productId": 5,
        "name": "Widget A",
        "sku": "WDG-001"
      }
    }
  ]
}

// Error Response (400)
{
  "error": "Product not found",
  "productId": 999
}
```

---

### Phase 2: Frontend Integration (octocat-supply-web)
**Priority:** High  
**Integration Order:** 2  
**Depends on:** octocat-supply-api Phase 1

#### Tasks

##### 1. Cart Context Provider
Create `src/contexts/CartContext.tsx`:
- State: `cartItems: { productId, name, price, quantity, imgName }[]`
- Actions:
  - `addToCart(product, quantity)` - Add or increment item
  - `removeFromCart(productId)` - Remove item entirely
  - `updateQuantity(productId, quantity)` - Update quantity (remove if 0)
  - `clearCart()` - Empty the cart
- Persist state in sessionStorage or localStorage
- Calculate derived values: `totalItems`, `subtotal`

##### 2. Navigation Updates
Update `src/components/Navigation.tsx`:
- Add cart icon with badge showing `totalItems` count
- Link to `/cart` route
- Badge should hide when cart is empty

##### 3. Cart Page Component
Create `src/pages/CartPage.tsx`:
- **Empty State**: Show message + "Browse Products" CTA when cart empty
- **Cart Items List**: 
  - Product image, name, price
  - Quantity selector (increment/decrement buttons + input)
  - Line total (price × quantity)
  - Remove button
- **Cart Summary**:
  - Subtotal
  - Item count
  - "Continue Shopping" link
  - "Place Order" button (primary CTA)
- **Place Order Flow**:
  1. Show loading state on button
  2. Call `POST /api/orders` with `branchId` and cart items
  3. On success: Show toast, clear cart, redirect to orders page
  4. On error: Show error toast with message

##### 4. Products Page Integration
Update `src/pages/ProductsPage.tsx`:
- Add "Add to Cart" button to each product card
- On click: call `addToCart`, show success toast
- Button should show current quantity if item already in cart

##### 5. Routing
Update `src/App.tsx` or router config:
- Add `/cart` route rendering `CartPage`

##### 6. Styling
- Use existing Tailwind theme classes
- Support dark mode via `ThemeContext`
- Responsive: mobile-first, adjust layout for tablet/desktop
- Match design mockup in `docs/design/cart.png`

##### 7. Toast Notifications
- Install/use existing toast library
- Show on "Add to Cart" action
- Show on "Place Order" success/error

#### TypeScript Types
```typescript
interface CartItem {
  productId: number;
  name: string;
  price: number;
  quantity: number;
  imgName?: string;
}

interface CartContextValue {
  items: CartItem[];
  totalItems: number;
  subtotal: number;
  addToCart: (product: Product, quantity: number) => void;
  removeFromCart: (productId: number) => void;
  updateQuantity: (productId: number, quantity: number) => void;
  clearCart: () => void;
}
```

---

## Integration Testing

Once both phases are complete:
1. Start API server and frontend dev server
2. Navigate to Products page
3. Add multiple products to cart
4. Navigate to Cart page
5. Update quantities and remove items
6. Place order and verify API call succeeds
7. Verify cart clears and user redirected
8. Check database for created Order and OrderDetail records

## Design Reference

See `/docs/design/cart.png` in the platform repo for UI mockup.

## Entity Model Reference

Refer to `/architecture/entity-model.md` in platform repo for full field definitions.
