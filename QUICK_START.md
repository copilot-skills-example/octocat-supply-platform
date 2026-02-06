# Quick Start Guide - Cart Page Feature

**For:** Implementers in octocat-supply-api and octocat-supply-web  
**Last Updated:** 2026-02-06

---

## 🚀 Getting Started

This guide helps you quickly understand what needs to be implemented for the Cart Page feature.

### If you're working on the API (octocat-supply-api):

**Your Task:** Implement POST /api/orders endpoint

**Read These Files:**
1. `/specs/features/cart-page.md` → Phase 1: API Foundation section
2. `/architecture/entity-model.md` → Order, OrderDetail, Product entities
3. `/specs/features/cart-issue-templates.md` → Full requirements (API section)

**Key Requirements:**
- Accept: `{ branchId: number, items: [{ productId: number, quantity: number }] }`
- Create Order record with status='pending'
- Create OrderDetail records (snapshot product.price as unitPrice)
- Use transaction for atomicity
- Validate all products exist
- Update Swagger documentation
- Write integration tests

**API Contract:**
```typescript
POST /api/orders
Request: { branchId: 1, items: [{ productId: 5, quantity: 2 }] }
Response: { orderId, branchId, orderDate, status, details: [...] }
```

---

### If you're working on the Frontend (octocat-supply-web):

**Your Task:** Implement cart functionality (context, page, integrations)

**Read These Files:**
1. `/specs/features/cart-page.md` → Phase 2: Frontend Integration section
2. `/docs/design/cart.png` → Visual mockup
3. `/specs/features/cart-issue-templates.md` → Full requirements (Frontend section)

**Key Requirements:**
- Create `CartContext` provider (add/remove/update/clear cart items)
- Create `CartPage` component (item list, summary, place order button)
- Update `Navigation` with cart icon + badge
- Update `ProductsPage` with "Add to Cart" buttons
- Integrate with POST /api/orders via React Query
- Add toast notifications
- Support dark mode and responsive design

**Component Structure:**
```
src/
├── contexts/
│   └── CartContext.tsx (state management)
├── pages/
│   ├── CartPage.tsx (main cart UI)
│   └── ProductsPage.tsx (add "Add to Cart" buttons)
└── components/
    └── Navigation.tsx (add cart icon + badge)
```

---

## 📋 Integration Checklist

- [ ] API endpoint implemented and tested
- [ ] Frontend can call API successfully
- [ ] Cart persists across page navigation
- [ ] Orders appear in database with correct details
- [ ] Error handling works (product not found, etc.)
- [ ] Dark mode renders correctly
- [ ] Mobile/tablet/desktop layouts work
- [ ] Toast notifications appear on actions

---

## 🔗 Quick Links

| Resource | Location |
|----------|----------|
| **Start Here** | `/ORCHESTRATION_COMPLETE.md` |
| **Technical Spec** | `/specs/features/cart-page.md` |
| **Issue Templates** | `/specs/features/cart-issue-templates.md` |
| **Design Mockup** | `/docs/design/cart.png` |
| **Entity Model** | `/architecture/entity-model.md` |
| **Status Tracker** | `/specs/features/cart-implementation-status.md` |

---

## 💡 Pro Tips

### For API Developers:
- Use existing repository pattern (e.g., `OrdersRepository.create()`)
- Capture **current** product.price into orderDetail.unitPrice (prices may change later)
- Wrap Order + OrderDetail inserts in a single transaction
- Default order status to 'pending'
- Set orderDate to current timestamp

### For Frontend Developers:
- Use React Context API (don't add Redux)
- Persist cart to localStorage/sessionStorage
- Use React Query's `useMutation` for POST /api/orders
- Follow existing Tailwind patterns for consistency
- Use ThemeContext for dark mode support
- Show loading states on async operations
- Handle API errors gracefully with user-friendly messages

---

## 🤝 Coordination

- **API should be implemented first** (Frontend depends on the endpoint)
- **branchId**: Use hardcoded value (e.g., 1) or from app context if available
- **Product images**: Use `imgName` field from product entity
- **Order confirmation**: Decide between redirect to orders list or confirmation page

---

## ❓ Questions?

Refer to the comprehensive documentation:
- Full specifications: `/specs/features/cart-page.md`
- Orchestration plan: `/specs/features/cart-orchestration-plan.md`
- Master issue: https://github.com/copilot-skills-example/octocat-supply-platform/issues/1

---

**Ready to start?** Open your assigned issue and begin implementation!
