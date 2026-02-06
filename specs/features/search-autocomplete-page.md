# Search Auto-Complete Feature Specification

**Parent Issue:** copilot-skills-example/octocat-supply-platform (current issue)

## Overview

This specification defines the auto-complete functionality for search bars across the OctoCAT Supply Chain application. The feature will provide real-time suggestions as users type, improving search efficiency and user experience.

## Architecture

This feature spans two repositories:
1. **octocat-supply-api** - Backend search endpoints and suggestion algorithms
2. **octocat-supply-web** - Frontend search UI components with auto-complete

## Implementation Order

### Phase 1: API Foundation (octocat-supply-api)
**Priority:** High (blocks frontend)  
**Integration Order:** 1

#### Tasks

1. Create `GET /api/search/suggestions` endpoint
   - Query parameter: `q` (query string, min 3 characters)
   - Query parameter: `entity` (optional: "products", "suppliers", "orders", defaults to all)
   - Query parameter: `limit` (optional: max results, default 10, max 20)
   - Return fuzzy-matched results from Products, Suppliers, and Orders
   - Response format:
     ```json
     {
       "query": "wid",
       "suggestions": [
         {
           "type": "product",
           "id": 5,
           "text": "Widget A",
           "subtext": "SKU: WDG-001",
           "metadata": { "price": 29.99 }
         },
         {
           "type": "supplier",
           "id": 2,
           "text": "Widget Supplier Inc.",
           "subtext": "Contact: john@widget.com"
         }
       ]
     }
     ```

2. Implement search algorithm
   - Case-insensitive LIKE query with wildcards
   - Search across: Product.name, Product.sku, Supplier.name, Order.name
   - Order results by relevance (exact match prefix > contains)
   - Limit results per entity type for balanced responses

3. Performance optimization
   - Add database indexes on searchable columns
   - Debounce/throttle considerations documented for frontend
   - Response time target: < 100ms for typical queries

4. Update Swagger documentation
   - Document query parameters and validation rules
   - Provide example responses
   - Document error cases (query too short, invalid entity type)

5. Add integration tests
   - Test suggestions with partial matches
   - Test entity filtering
   - Test result limit enforcement
   - Test empty results handling

#### API Contract

```typescript
// Request
GET /api/search/suggestions?q=wid&limit=10

// Success Response (200)
{
  "query": "wid",
  "suggestions": [
    {
      "type": "product",
      "id": 5,
      "text": "Widget A",
      "subtext": "SKU: WDG-001 | $29.99",
      "metadata": { "price": 29.99, "sku": "WDG-001" }
    },
    {
      "type": "supplier",
      "id": 2,
      "text": "Widget Supplier Inc.",
      "subtext": "john@widget.com"
    }
  ]
}

// Error Response (400)
{
  "error": "Query too short",
  "message": "Minimum 3 characters required"
}
```

---

### Phase 2: Frontend Integration (octocat-supply-web)
**Priority:** High  
**Integration Order:** 2  
**Depends on:** octocat-supply-api Phase 1

#### Tasks

##### 1. Auto-Complete Component
Create `src/components/SearchAutoComplete.tsx`:
- **Input field** with search icon
- **Suggestions dropdown** that appears below input
- **Loading state** while fetching suggestions
- **Empty state** when no results found
- **Keyboard navigation**:
  - Arrow Up/Down to navigate suggestions
  - Enter to select highlighted suggestion
  - Escape to close dropdown
- **Mouse interaction**:
  - Hover to highlight
  - Click to select
- **Debounced API calls** (300ms delay after last keystroke)
- **Minimum character requirement** (3 characters before triggering)

##### 2. Suggestion Item Rendering
Each suggestion displays:
- **Icon/badge** indicating type (Product 🏷️, Supplier 🏢, Order 📦)
- **Primary text** (name/title) in bold
- **Secondary text** (SKU, contact, order number) in muted color
- **Hover effect** with background highlight

##### 3. Navigation Integration
Update existing navigation bars to include SearchAutoComplete:
- Desktop: Full-width search bar in header
- Mobile: Collapsible search with icon button
- Maintain existing dark mode support

##### 4. Search Results Handling
On suggestion selection:
- Products → Navigate to `/products/:id` detail page
- Suppliers → Navigate to `/suppliers/:id` detail page
- Orders → Navigate to `/orders/:id` detail page
- Clear search input after navigation
- Track recent searches in localStorage (optional enhancement)

##### 5. React Query Integration
- Use `useQuery` with:
  - `enabled: query.length >= 3` to prevent premature calls
  - `staleTime: 5 minutes` for suggestion caching
  - `keepPreviousData: true` for smooth transitions
  - Error handling with retry logic

##### 6. Accessibility
- `role="combobox"` on input
- `role="listbox"` on suggestions dropdown
- `aria-expanded` state management
- `aria-activedescendant` for keyboard focus
- Proper label association with `htmlFor` / `aria-label`
- Focus management (trap focus in dropdown)
- Screen reader announcements for result counts

##### 7. Styling
- Tailwind CSS utility classes
- Consistent with existing design system
- Dark mode support via ThemeContext
- Responsive: adapt layout for mobile (<640px), tablet (768px), desktop (>1024px)
- Smooth animations for dropdown appearance (slide-in, fade)
- Z-index layering to ensure dropdown appears above other content

##### 8. Performance Considerations
- Debounce user input (300ms)
- Cancel previous requests when new query initiated
- Virtual scrolling if suggestion count exceeds 20 (unlikely with limit)
- Lazy load suggestion metadata only when needed

#### TypeScript Types

```typescript
interface Suggestion {
  type: 'product' | 'supplier' | 'order';
  id: number;
  text: string;
  subtext?: string;
  metadata?: Record<string, any>;
}

interface SuggestionsResponse {
  query: string;
  suggestions: Suggestion[];
}

interface SearchAutoCompleteProps {
  placeholder?: string;
  onSelect?: (suggestion: Suggestion) => void;
  className?: string;
}
```

---

## Integration Testing

Once both phases are complete:
1. Start API server and frontend dev server
2. Navigate to any page with search bar
3. Type at least 3 characters in search input
4. Verify suggestions dropdown appears with relevant results
5. Test keyboard navigation (arrow keys, enter, escape)
6. Test mouse interaction (hover, click)
7. Verify selection navigates to correct detail page
8. Test empty state (query with no results)
9. Test error handling (API unavailable)
10. Verify debouncing works (rapid typing doesn't spam API)

## Performance Targets

- **API Response Time:** < 100ms for typical queries
- **Frontend Debounce Delay:** 300ms
- **Time to First Suggestion:** < 500ms from last keystroke
- **Database Query Optimization:** Use indexes on name/SKU columns

## Database Indexes

Add these indexes for optimal search performance:

```sql
CREATE INDEX IF NOT EXISTS idx_products_name ON products(name);
CREATE INDEX IF NOT EXISTS idx_products_sku ON products(sku);
CREATE INDEX IF NOT EXISTS idx_suppliers_name ON suppliers(name);
CREATE INDEX IF NOT EXISTS idx_orders_name ON orders(name);
```

## Accessibility Requirements

- WCAG 2.1 Level AA compliance
- Keyboard-only navigation support
- Screen reader compatibility (tested with NVDA/JAWS)
- Visible focus indicators
- Sufficient color contrast ratios (4.5:1 minimum)

## Future Enhancements (Out of Scope)

- Search history tracking per user
- Personalized suggestions based on browsing history
- Voice search integration
- Advanced filters in suggestions
- Search analytics and popular searches

## Entity Model Reference

Refer to `/architecture/entity-model.md` in platform repo for full field definitions.

### Entities Involved:
- **Product**: productId, name, sku, price, description
- **Supplier**: supplierId, name, email, contactPerson
- **Order**: orderId, name, orderDate, status, branchId
