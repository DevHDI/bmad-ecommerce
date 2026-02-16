---
status: done
completedAt: "2026-01-25"
---
# Story 2-1: Product Catalog and Listing Pages

## Description

This story implements the core product browsing experience that serves as the primary entry point for customers. The product listing page displays products in a responsive grid or list layout with high-quality images, pricing information, ratings, and quick-add functionality. This is one of the most critical pages for conversion, as 87% of shoppers begin their journey by browsing product catalogs.

We're building a ProductCard component that balances visual appeal with information density, showing essential product details at a glance while maintaining fast load times through Next.js Image optimization. The grid/list view toggle gives users control over their browsing experience - some prefer dense information (list view) while others prefer visual browsing (grid view). View preference persists across sessions using localStorage.

Performance is paramount: we're implementing skeleton loaders to provide instant perceived feedback while server components fetch product data, and using Next.js server components to render the initial page on the server for optimal First Contentful Paint. SEO optimization through proper meta tags, semantic HTML, and structured data ensures products are discoverable through search engines, driving organic traffic.

## Acceptance Criteria

- Product listing page renders at `/products` with server-side data fetching
- ProductCard component displays product image (4:3 aspect), name (2 lines max), price, rating, and stock status
- Grid view shows 4 products per row on desktop, 2 on tablet, 1 on mobile
- List view shows 1 product per row with expanded details on all screen sizes
- View toggle button persists preference in localStorage across sessions
- Images use Next.js Image component with proper sizing, lazy loading, and WebP format
- Skeleton loaders display while products are loading (minimum 8 skeleton cards)
- "Quick Add" button appears on hover (desktop) or always visible (mobile)
- Stock badges display "Low Stock" warning when inventory < 10 units, "Out of Stock" when 0
- Page achieves 90+ Lighthouse performance score and passes Core Web Vitals
- Structured data (Product schema.org JSON-LD) added for SEO
- Responsive design works flawlessly from 320px to 2560px width

## Technical Notes

**Server Components Architecture:**
- Products page is a server component fetching data directly from Prisma
- ProductCard can be client component for interactivity (hover, quick-add)
- Use Suspense boundaries around product grid for streaming
- Implement pagination server-side to limit initial data transfer

**Image Optimization:**
- Product images stored in /public/products/ or CDN (Cloudinary/S3)
- Use Next.js Image with sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 25vw"
- Implement blur placeholder using base64-encoded thumbnails
- Lazy load images below the fold with loading="lazy"

**ProductCard Component API:**
```typescript
interface ProductCardProps {
  product: {
    id: string;
    name: string;
    slug: string;
    price: number;
    compareAtPrice?: number;
    images: string[];
    rating: number;
    reviewCount: number;
    stock: number;
  };
  view: 'grid' | 'list';
  onQuickAdd: (productId: string) => void;
}
```

**Skeleton Loader:**
- Match exact dimensions and layout of actual ProductCard
- Use shimmer animation with CSS gradient
- Render 8-12 skeletons based on screen size
- Replace with actual products using Suspense

**SEO Optimizations:**
- Meta title: "Shop [Category Name] | ShopFlow"
- Meta description with product count and key features
- Canonical URL to prevent duplicate content
- Open Graph tags for social sharing
- JSON-LD structured data for Product listings

**Edge Cases:**
- Handle products with no images (show placeholder)
- Handle very long product names (truncate with ellipsis)
- Handle missing ratings (show "No reviews yet")
- Handle zero stock (disable quick-add, show "Notify Me" button)
- Handle slow image CDN (show skeleton until loaded)

## Tasks

- [x] Create ProductCard component with grid and list view variants
- [x] Implement image rendering with Next.js Image component and optimizations
- [x] Build ProductGrid layout with responsive columns
- [x] Build ProductList layout with expanded details
- [x] Create view toggle button with icon (grid/list)
- [x] Implement localStorage persistence for view preference
- [x] Create skeleton loader component matching ProductCard dimensions
- [x] Build products page with server component data fetching
- [x] Add Suspense boundary around product grid for streaming
- [x] Implement stock badge logic (low stock warning, out of stock)
- [x] Add "Quick Add to Cart" button functionality
- [x] Configure responsive breakpoints (mobile, tablet, desktop, wide)
- [x] Add SEO meta tags and structured data (JSON-LD)
- [x] Implement canonical URL for product listing
- [x] Test performance with Lighthouse (target 90+ score)
- [x] Test responsive design on real devices
- [x] Add accessibility attributes (ARIA labels, alt text)
- [x] Create unit tests for ProductCard component

## Dependencies

- Story 1-3: Development Environment (requires Product database schema)
- Story 3-1: Cart Management (for Quick Add functionality)

## Estimation

**Story Points:** 8

**Rationale:** This is a substantial feature with multiple complex components (ProductCard, grid/list layouts, skeleton loaders) and critical performance requirements. Image optimization and responsive design add complexity. SEO requirements and accessibility testing increase scope. However, the implementation path is clear with well-established patterns.
