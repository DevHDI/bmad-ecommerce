---
status: done
completedAt: "2026-01-26"
---
# Story 2-2: Category and Navigation System

## Description

This story implements a hierarchical category system that helps customers navigate the product catalog efficiently. Categories are organized in a tree structure supporting up to 3 levels of depth (e.g., Electronics → Laptops → Gaming Laptops), providing intuitive product organization that mirrors how customers think about product discovery.

The category navigation appears in a collapsible sidebar on desktop and as a bottom sheet on mobile, with expandable sections for parent categories. Breadcrumb navigation at the top of every page shows the current location in the category hierarchy and provides quick navigation back to parent categories. Each category has its own landing page with featured products, category-specific banners, and optimized SEO.

The database schema uses a self-referencing relationship (parentId) to create the tree structure, enabling efficient queries for category hierarchies using recursive CTEs in PostgreSQL. Category slugs are generated automatically from names and ensure SEO-friendly URLs like `/category/electronics/laptops/gaming-laptops`.

## Acceptance Criteria

- Category tree supports up to 3 levels of nesting with parent-child relationships
- Sidebar navigation displays category tree with collapsible parent categories
- Click on parent category expands/collapses children with smooth animation
- Active category and its ancestors are highlighted in the tree
- Breadcrumb component appears on all category and product pages
- Breadcrumbs show full path: Home > Electronics > Laptops > Gaming Laptops
- Each breadcrumb link is clickable and navigates to that category
- Category landing pages display at `/category/[slug]` with featured products
- Category pages include category description, image, and product count
- URL structure supports nested categories: `/category/electronics/laptops`
- Category sidebar is sticky on scroll and remains visible on desktop
- Mobile category navigation appears as bottom sheet with close button

## Technical Notes

**Database Schema:**
```prisma
model Category {
  id          String     @id @default(cuid())
  name        String
  slug        String     @unique
  description String?
  image       String?
  parentId    String?
  parent      Category?  @relation("CategoryTree", fields: [parentId], references: [id])
  children    Category[] @relation("CategoryTree")
  products    Product[]
  sortOrder   Int        @default(0)
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  @@index([parentId])
  @@index([slug])
}
```

**Category Tree Query:**
```typescript
// Get full category tree with children
const categoryTree = await prisma.category.findMany({
  where: { parentId: null }, // Root categories only
  include: {
    children: {
      include: {
        children: true, // 3 levels deep
      },
    },
  },
  orderBy: { sortOrder: 'asc' },
});
```

**Breadcrumb Generation:**
- Build breadcrumb array by traversing category.parent recursively
- Reverse array to show root → current order
- Include schema.org BreadcrumbList structured data for SEO

**Category URL Structure:**
- Single level: `/category/electronics`
- Two levels: `/category/electronics/laptops`
- Three levels: `/category/electronics/laptops/gaming-laptops`
- Use Next.js dynamic catch-all route: `[...slug]`

**Sidebar Component:**
- Use React state for expand/collapse
- Use CSS transitions for smooth animations
- Desktop: sticky positioning with `top-24` offset
- Mobile: full-screen bottom sheet with backdrop

**Edge Cases:**
- Handle categories with no products (show "Coming soon" message)
- Handle orphaned categories (parentId references non-existent category)
- Handle circular references in category tree (validate on save)
- Handle very long category names (truncate in sidebar, full in breadcrumb)
- Handle deep nesting beyond 3 levels (prevent in admin, flatten in display)

## Tasks

- [x] Design Category database schema with self-referencing parentId
- [x] Create Prisma migration for Category model
- [x] Build CategoryTree component with recursive rendering
- [x] Implement expand/collapse functionality with state management
- [x] Add smooth animations for category expansion
- [x] Create Breadcrumb component with schema.org structured data
- [x] Build category landing page at `/category/[...slug]` route
- [x] Implement breadcrumb generation by traversing parent chain
- [x] Create sidebar layout with sticky positioning on desktop
- [x] Build mobile bottom sheet for category navigation
- [x] Add category product count to sidebar display
- [x] Implement category filtering in product listing
- [x] Create category page with description, image, and featured products
- [x] Generate category slugs automatically from names
- [x] Add URL routing for nested categories
- [x] Test category tree with 50+ categories across 3 levels
- [x] Add loading states for category navigation
- [x] Implement active category highlighting in tree

## Dependencies

- Story 1-3: Development Environment (requires database setup)
- Story 2-1: Product Catalog (category pages display products)

## Estimation

**Story Points:** 5

**Rationale:** Category tree rendering is moderately complex but well-understood. Recursive data structures require careful handling. Breadcrumb generation and URL routing add complexity. Mobile bottom sheet and animations require attention to UX details. Overall moderate complexity with clear implementation patterns.
