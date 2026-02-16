---
status: done
completedAt: "2026-01-28"
---
# Story 2-3: Search and Advanced Filters

## Description

This story implements powerful product discovery features including full-text search with autocomplete and multi-faceted filtering. Search is a critical conversion driver - 43% of visitors immediately use the search bar, and searches convert 2-3x better than non-search traffic. We're implementing PostgreSQL full-text search with pg_trgm for fuzzy matching, providing fast results (< 200ms) without the complexity of external search services like Elasticsearch.

The autocomplete feature provides instant feedback as users type, showing relevant product suggestions after 2+ characters. Suggestions include product images and prices for visual recognition. Advanced filters allow customers to narrow results by price range, category, rating, and availability, with all filter state reflected in URL query parameters for shareability and SEO.

Filter interactions feel instant through client-side state management with debounced server requests. The price range slider provides intuitive visual feedback, while category and rating filters use checkboxes for multiple selection. Active filters display as removable chips above the product grid. Pagination supports both numbered pages and infinite scroll options for different browsing preferences.

## Acceptance Criteria

- Search bar in header accepts text input with minimum 2 characters
- Autocomplete suggestions appear in dropdown showing top 5 matching products
- Each suggestion shows product image (thumbnail), name, and price
- Full-text search queries product names and descriptions using PostgreSQL FTS
- Search results page displays at `/search?q=[query]` with matching products
- Price range filter uses slider with min/max inputs showing live filtered count
- Category filter shows hierarchical checkboxes with product counts per category
- Rating filter allows selection of minimum star rating (1-5 stars)
- Availability filter toggles "In Stock Only" to hide out-of-stock products
- All active filters display as removable chips with X button
- Filter state persists in URL query params (shareable, bookmarkable, SEO-friendly)
- Pagination displays page numbers with Previous/Next, 20 products per page
- Search results return within 200ms for 95th percentile queries
- Fuzzy matching handles typos (e.g., "laptpo" matches "laptop")

## Technical Notes

**PostgreSQL Full-Text Search:**
```sql
-- Add GIN index for full-text search
CREATE INDEX idx_products_search ON products
USING gin(to_tsvector('english', name || ' ' || description));

-- Search query with ranking
SELECT *, ts_rank(to_tsvector('english', name || ' ' || description), query) AS rank
FROM products, to_tsquery('english', 'laptop | notebook') query
WHERE to_tsvector('english', name || ' ' || description) @@ query
ORDER BY rank DESC;
```

**Fuzzy Matching with pg_trgm:**
```sql
CREATE EXTENSION pg_trgm;
CREATE INDEX idx_products_name_trgm ON products USING gin(name gin_trgm_ops);

-- Fuzzy search for autocomplete
SELECT * FROM products
WHERE name % 'laptpo'  -- % operator for similarity
ORDER BY similarity(name, 'laptpo') DESC
LIMIT 5;
```

**URL Query Parameter Schema:**
- Search: `?q=laptop`
- Price range: `?minPrice=100&maxPrice=500`
- Categories: `?categories=electronics,laptops`
- Rating: `?minRating=4`
- Availability: `?inStock=true`
- Pagination: `?page=2`
- Combined: `?q=laptop&minPrice=500&categories=gaming&minRating=4&page=1`

**Filter State Management:**
```typescript
interface FilterState {
  query: string;
  minPrice?: number;
  maxPrice?: number;
  categories: string[];
  minRating?: number;
  inStockOnly: boolean;
  page: number;
}
```

**Debounced Search:**
- Use debounce (300ms) on search input to avoid excessive API calls
- Show loading spinner during search
- Cancel in-flight requests when new query starts

**Performance Optimization:**
- Cache popular search queries in Redis (5min TTL)
- Preload common filter combinations
- Use database query plan analysis to optimize slow queries
- Implement query result pagination at database level

**Edge Cases:**
- Handle search with no results (show "No products found" with suggestions)
- Handle invalid price ranges (min > max, negative values)
- Handle special characters in search queries (sanitize SQL injection risk)
- Handle empty filter selections (show all products)
- Handle simultaneous filter and search (AND logic)

## Tasks

- [x] Install pg_trgm extension in PostgreSQL for fuzzy matching
- [x] Create GIN indexes for full-text search on products table
- [x] Build search API endpoint with full-text search and ranking
- [x] Implement fuzzy matching for autocomplete suggestions
- [x] Create SearchBar component with input and autocomplete dropdown
- [x] Add debounced search with 300ms delay
- [x] Build autocomplete suggestion item with image, name, price
- [x] Create search results page at `/search` route
- [x] Build FilterPanel component with collapsible sections
- [x] Implement price range slider with min/max inputs
- [x] Create category filter with hierarchical checkboxes
- [x] Add rating filter with star selector (1-5 stars)
- [x] Implement availability toggle (In Stock Only)
- [x] Display active filters as removable chips
- [x] Implement URL query parameter sync for all filters
- [x] Add pagination component with numbered pages
- [x] Optimize database queries for < 200ms response time
- [x] Add loading states for search and filter operations
- [x] Test fuzzy matching with common typos
- [x] Create "No results" state with helpful suggestions

## Dependencies

- Story 1-3: Development Environment (requires PostgreSQL setup)
- Story 2-1: Product Catalog (search operates on product data)
- Story 2-2: Category System (category filter uses category tree)

## Estimation

**Story Points:** 8

**Rationale:** Full-text search with PostgreSQL requires database expertise and performance tuning. Autocomplete with debouncing adds complexity. Multi-faceted filtering with URL sync is moderately complex. Fuzzy matching and query optimization require careful testing. Overall high complexity but well-established patterns exist.
