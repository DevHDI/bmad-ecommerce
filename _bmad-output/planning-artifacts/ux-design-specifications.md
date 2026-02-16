---
status: done
completedAt: "2026-01-12"
---
# ShopFlow - UX Design Specifications

## 1. Design Principles

### 1.1 Core Principles

1. **Speed is a Feature**: Every interaction should feel instant. Skeleton loaders, optimistic updates, and prefetching create perceived performance.
2. **Progressive Disclosure**: Show essential information first, details on demand. Product cards show name/price/image; full specs are on the detail page.
3. **Mobile-First**: Design for touch screens first, enhance for desktop. 68% of e-commerce traffic is mobile.
4. **Accessibility**: WCAG 2.1 AA compliance is a requirement, not a nice-to-have. All interactive elements must be keyboard-navigable.

### 1.2 Design System

**Typography**:
- Headings: Inter (600/700 weight)
- Body: Inter (400/500 weight)
- Scale: 12px / 14px / 16px / 18px / 24px / 32px / 48px

**Color Palette**:
- Primary: #2563EB (blue-600) - CTAs, links, active states
- Secondary: #7C3AED (violet-600) - Accents, badges
- Success: #16A34A (green-600) - Confirmations, in-stock
- Warning: #D97706 (amber-600) - Low stock, alerts
- Error: #DC2626 (red-600) - Errors, out-of-stock
- Neutral: Slate scale (50-950)

**Spacing**: 4px base unit (4, 8, 12, 16, 24, 32, 48, 64, 96)

**Border Radius**: 4px (small), 8px (medium), 12px (large), 9999px (pill)

**Shadows**:
- sm: 0 1px 2px rgba(0,0,0,0.05)
- md: 0 4px 6px rgba(0,0,0,0.07)
- lg: 0 10px 15px rgba(0,0,0,0.1)

## 2. Page Specifications

### 2.1 Product Listing Page

**Layout**: 12-column grid
- Desktop: 4 products per row (3-column cards)
- Tablet: 2 products per row
- Mobile: 1 product per row (full-width cards)

**Product Card Components**:
- Image container: 4:3 aspect ratio, hover zoom effect
- Product name: 2 lines max, truncated with ellipsis
- Price: Bold, primary color for sale prices, strikethrough for compare-at
- Rating: 5-star display with count
- Quick-add button: Appears on hover (desktop), always visible (mobile)
- Stock badge: "Low stock" warning when < 10 units

**Filters Sidebar** (desktop) / **Filter Sheet** (mobile):
- Price range slider with min/max inputs
- Category checkboxes (collapsible tree)
- Rating filter (star selector)
- Availability toggle
- Clear all filters button

### 2.2 Product Detail Page

**Above the Fold**:
- Image gallery with thumbnails (left) and main image (right)
- Product title, price, rating
- Variant selector (size, color) as button groups
- Quantity selector with +/- buttons
- Add to Cart primary CTA (full-width on mobile)
- Wishlist heart icon

**Below the Fold**:
- Tab navigation: Description | Specifications | Reviews
- Related products carousel (4 items)

### 2.3 Cart Sidebar

**Trigger**: Cart icon in header (shows item count badge)
**Behavior**: Slides in from right, overlay on content

**Contents**:
- Cart items list (image, name, variant, quantity, price)
- Quantity +/- controls inline
- Remove item (X button with confirmation)
- Subtotal calculation
- "Continue Shopping" secondary button
- "Checkout" primary button
- Free shipping progress bar ("$X more for free shipping!")

### 2.4 Checkout Flow

**Step 1 - Shipping Address**:
- Address form with autocomplete (Google Places API)
- Saved addresses for logged-in users
- "Same as shipping" checkbox for billing

**Step 2 - Delivery Method**:
- Radio cards: Standard (free, 5-7 days), Express ($9.99, 2-3 days), Next Day ($19.99)
- Estimated delivery date displayed

**Step 3 - Review & Pay**:
- Order summary with editable quantities
- Stripe Elements card input
- Apple Pay / Google Pay express buttons
- Order total with tax breakdown
- "Place Order" CTA

### 2.5 Admin Dashboard

**Sidebar Navigation**:
- Dashboard (overview)
- Products (list, create, edit)
- Orders (list, detail)
- Customers (list)
- Analytics (charts)
- Settings

**Dashboard Overview**:
- Revenue stat cards (today, this week, this month)
- Sales chart (line graph, 30-day view)
- Recent orders table (last 10)
- Low stock alerts

## 3. Interaction Patterns

### 3.1 Loading States
- **Skeleton screens** for page loads (not spinners)
- **Optimistic updates** for cart operations
- **Progress indicators** for checkout steps
- **Shimmer effect** on loading skeletons

### 3.2 Error Handling
- **Inline validation** on form fields (real-time)
- **Toast notifications** for transient errors (3s auto-dismiss)
- **Error pages** for 404/500 with clear messaging and back navigation
- **Retry buttons** for failed network requests

### 3.3 Animations
- **Page transitions**: Fade (200ms ease-out)
- **Cart sidebar**: Slide-in (300ms ease-out)
- **Hover effects**: Scale 1.02 (150ms)
- **Button feedback**: Scale 0.98 on press (100ms)
- **Skeleton shimmer**: Linear gradient animation (1.5s infinite)

## 4. Responsive Breakpoints

| Breakpoint | Width | Columns | Behavior |
|------------|-------|---------|----------|
| Mobile | < 640px | 4 | Stack layout, bottom nav |
| Tablet | 640-1024px | 8 | Compact sidebar, 2-col grid |
| Desktop | 1024-1280px | 12 | Full sidebar, 3-col grid |
| Wide | > 1280px | 12 | Max-width container, 4-col grid |

## 5. Accessibility Requirements

- All images must have descriptive alt text
- Color contrast ratio ≥ 4.5:1 for text, ≥ 3:1 for large text
- Focus indicators visible on all interactive elements
- Skip navigation link on every page
- ARIA labels on icon-only buttons
- Form inputs must have associated labels
- Error messages must be announced by screen readers
- Keyboard navigation for all functionality
