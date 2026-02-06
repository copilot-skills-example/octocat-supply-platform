# Search Auto-Complete Multi-Repo Orchestration Plan

**Master Issue:** copilot-skills-example/octocat-supply-platform (current issue)

## Status: 🚀 Ready for Implementation

This document provides the orchestration plan for implementing the Search Auto-Complete functionality across multiple repositories in the OctoCAT Supply Chain application.

---

## Repository Breakdown

### 🔧 Backend API (octocat-supply-api)
**Integration Order:** 1 (No dependencies)  
**Assignee:** @copilot

#### Scope
- Create `GET /api/search/suggestions` endpoint for auto-complete
- Implement fuzzy search algorithm across Products, Suppliers, and Orders
- Add database indexes for search performance optimization
- Swagger documentation updates
- Integration tests for search functionality

#### Deliverables
1. New route handler for `/api/search/suggestions` in Express app
2. Repository methods for multi-entity search queries
3. Database migration adding indexes on searchable columns
4. Updated Swagger spec with search endpoint documentation
5. Test coverage for search algorithm and edge cases

#### Reference
See `/specs/features/search-autocomplete-page.md` Phase 1 for detailed requirements.

---

### 🎨 Frontend UI (octocat-supply-web)
**Integration Order:** 2 (Depends on octocat-supply-api)  
**Assignee:** @copilot

#### Scope
- `SearchAutoComplete` React component with keyboard navigation
- Debounced API calls with React Query integration
- Suggestions dropdown with type indicators and metadata
- Navigation bar integration (desktop and mobile)
- Accessibility features (ARIA attributes, keyboard navigation)
- Tailwind CSS styling with dark mode support

#### Deliverables
1. `src/components/SearchAutoComplete.tsx` component
2. Updated navigation components to include search bar
3. Route integration for suggestion selection
4. React Query hooks for search API calls
5. Unit tests for component behavior
6. Accessibility compliance verification

#### Reference
See `/specs/features/search-autocomplete-page.md` Phase 2 for detailed requirements.

---

## Integration Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. User types in search bar (min 3 characters)              │
│    └─> Input debounced (300ms delay)                        │
│        └─> API call triggered: GET /api/search/suggestions  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. Backend processes search query                           │
│    ├─> Query Products table (LIKE %query%)                  │
│    ├─> Query Suppliers table (LIKE %query%)                 │
│    ├─> Query Orders table (LIKE %query%)                    │
│    └─> Merge and rank results by relevance                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. Frontend displays suggestions dropdown                   │
│    ├─> Render each suggestion with type indicator           │
│    ├─> Enable keyboard navigation (Up/Down/Enter)           │
│    ├─> Highlight on hover                                   │
│    └─> Show loading state during fetch                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. User selects suggestion                                  │
│    ├─> Navigate to detail page based on type                │
│    │   ├─> Product → /products/:id                          │
│    │   ├─> Supplier → /suppliers/:id                        │
│    │   └─> Order → /orders/:id                              │
│    └─> Clear search input and close dropdown                │
└─────────────────────────────────────────────────────────────┘
```

---

## Database Schema Changes

### New Indexes (Migration Required)

Add indexes to optimize search queries:

```sql
-- Migration: 00X_add_search_indexes.sql
CREATE INDEX IF NOT EXISTS idx_products_name ON products(name);
CREATE INDEX IF NOT EXISTS idx_products_sku ON products(sku);
CREATE INDEX IF NOT EXISTS idx_suppliers_name ON suppliers(name);
CREATE INDEX IF NOT EXISTS idx_orders_name ON orders(name);
```

### Tables Affected (Read-Only)
- **products** (SELECT on name, sku)
- **suppliers** (SELECT on name)
- **orders** (SELECT on name, orderDate)

---

## API Contract

### Endpoint: `GET /api/search/suggestions`

#### Query Parameters
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| q | string | ✅ | - | Search query (min 3 chars) |
| entity | string | ❌ | all | Filter by entity type: products, suppliers, orders |
| limit | number | ❌ | 10 | Max results (max 20) |

#### Response Format
```json
{
  "query": "widget",
  "suggestions": [
    {
      "type": "product",
      "id": 5,
      "text": "Widget A",
      "subtext": "SKU: WDG-001 | $29.99",
      "metadata": {
        "price": 29.99,
        "sku": "WDG-001"
      }
    }
  ]
}
```

#### Status Codes
- `200 OK` - Suggestions returned (may be empty array)
- `400 Bad Request` - Query too short or invalid parameters
- `500 Internal Server Error` - Database or server error

---

## Testing Strategy

### Backend Tests (octocat-supply-api)
- [ ] GET /api/search/suggestions with valid query → 200 OK
- [ ] Query < 3 characters → 400 Bad Request
- [ ] Entity filter "products" → only product results
- [ ] Limit parameter enforced → max results respected
- [ ] Search finds products by name (partial match)
- [ ] Search finds products by SKU (partial match)
- [ ] Search finds suppliers by name
- [ ] Search finds orders by name
- [ ] Empty results → 200 OK with empty array
- [ ] Case-insensitive matching verified

### Frontend Tests (octocat-supply-web)
- [ ] Component renders with search input
- [ ] Input < 3 characters → no API call
- [ ] Input >= 3 characters → debounced API call
- [ ] Suggestions dropdown appears on results
- [ ] Arrow Down navigates to next suggestion
- [ ] Arrow Up navigates to previous suggestion
- [ ] Enter key selects highlighted suggestion
- [ ] Escape key closes dropdown
- [ ] Click on suggestion selects it
- [ ] Selection navigates to correct detail page
- [ ] Empty results → "No results" message displayed
- [ ] Loading state shown during API call
- [ ] Dark mode styling verified

### Integration Tests
- [ ] End-to-end: type query → see suggestions → select → navigate
- [ ] Performance: API response < 100ms for typical queries
- [ ] Accessibility: keyboard-only navigation works
- [ ] Screen reader: ARIA attributes announced correctly
- [ ] Mobile: touch interactions work (tap to select)
- [ ] Debouncing: rapid typing doesn't spam API (verified in network tab)

---

## Performance Benchmarks

### API Response Time Targets
| Query Length | Target Response Time |
|--------------|---------------------|
| 3 characters | < 100ms |
| 5 characters | < 80ms |
| 10+ characters | < 60ms |

### Frontend Performance
- **Debounce Delay:** 300ms
- **Time to First Paint (suggestions):** < 500ms from last keystroke
- **Re-render Performance:** < 16ms (60fps)

---

## Accessibility Checklist

- [ ] WCAG 2.1 Level AA compliance
- [ ] Keyboard navigation (Tab, Arrow keys, Enter, Escape)
- [ ] ARIA attributes (`role="combobox"`, `aria-expanded`, `aria-activedescendant`)
- [ ] Screen reader compatibility (tested with NVDA/JAWS)
- [ ] Visible focus indicators (outline on focused elements)
- [ ] Color contrast ratio >= 4.5:1
- [ ] No reliance on color alone for information
- [ ] Focus trap in dropdown (focus stays within suggestions)

---

## Rollout Checklist

- [ ] Database indexes added (octocat-supply-api)
- [ ] API endpoint implemented and tested (octocat-supply-api)
- [ ] Swagger documentation updated
- [ ] API merged to main branch
- [ ] Frontend component implemented (octocat-supply-web)
- [ ] Frontend tested against deployed API
- [ ] Keyboard navigation verified
- [ ] Accessibility audit passed
- [ ] Dark mode verified
- [ ] Responsive design tested (mobile, tablet, desktop)
- [ ] Performance benchmarks met
- [ ] Integration testing completed
- [ ] Documentation updated
- [ ] Feature deployed to staging/production

---

## References

- **Entity Model:** `/architecture/entity-model.md`
- **API Contract:** `/specs/features/search-autocomplete-page.md`
- **Dependencies Map:** `/.github/skills/multi-repo-orchestration/dependencies.json`

---

## Notes for Implementers

### API Implementer
- Use SQLite LIKE queries with wildcards for fuzzy matching: `WHERE name LIKE '%' || ? || '%'`
- Parameterize queries to prevent SQL injection
- Order results by relevance:
  1. Exact match (name = query)
  2. Starts with (name LIKE 'query%')
  3. Contains (name LIKE '%query%')
- Limit results per entity type (e.g., max 5 products, 3 suppliers, 2 orders)
- Use UNION ALL for combining results from multiple tables
- Add indexes BEFORE implementing search to ensure performance
- Cache common queries if needed (consider in-memory cache)

### Frontend Implementer
- Use `lodash.debounce` or `use-debounce` hook for input debouncing
- Implement keyboard navigation state machine (selected index, wrapping)
- Close dropdown on: selection, Escape key, click outside
- Show loading spinner during API call (subtle, inline)
- Use React Query's `useQuery` with proper dependencies
- Maintain focus on input when navigating suggestions (visual only)
- Handle rapid typing gracefully (cancel previous requests)
- Consider using `React.memo` for suggestion items to prevent re-renders
- Test with keyboard-only, screen reader, and mobile touch
- Ensure dropdown doesn't cause layout shift (fixed positioning)

### Shared Concerns
- Agree on minimum character requirement (3 chars)
- Standardize suggestion metadata format
- Define navigation routes for each entity type
- Consider future: search history, personalization
- Document rate limiting if API calls become excessive
