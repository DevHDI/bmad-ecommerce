---
status: done
completedAt: "2026-02-02"
---
# Story 3-1: Shopping Cart Management

## Description

This story implements a persistent shopping cart system that serves as the critical transition between product browsing and checkout. Cart abandonment costs e-commerce businesses $4.6 trillion annually, making cart optimization one of the highest-ROI features we can build. Our cart persists across sessions using localStorage for guest users and database storage for authenticated users, ensuring customers never lose their selections.

The cart appears as a slide-in sidebar triggered by clicking the cart icon in the header, providing immediate access without page navigation. Real-time price calculation updates instantly as quantities change, with running totals for subtotal, tax estimates, and shipping. The cart badge in the header displays item count, creating persistent visibility and encouraging completion.

We're implementing optimistic UI updates for cart operations - when users add items or change quantities, the UI updates immediately while the API call processes in the background. This creates a snappy, responsive experience that feels instant even on slower connections. Cart data syncs between localStorage and the database seamlessly when users log in, merging guest cart items with saved cart items.

## Acceptance Criteria

- Add to cart from product pages and product listing (quick-add)
- Cart sidebar slides in from right with backdrop overlay
- Cart displays all items with image, name, variant, quantity, and price per item
- Quantity controls with +/- buttons update cart instantly (optimistic UI)
- Remove item button (X icon) with confirmation for expensive items (>$100)
- Real-time price calculation showing subtotal updated on every change
- Empty cart state with "Start Shopping" CTA linking to products
- Cart persists in localStorage for guest users between sessions
- Cart syncs to database for authenticated users
- Cart merge logic when guest logs in (combine items, max quantities)
- Cart icon badge in header shows total item count
- Badge animates when items are added (scale pulse effect)
- "Continue Shopping" button closes sidebar and returns to previous page
- "Checkout" button navigates to checkout with current cart state
- Cart handles product variants correctly (same product, different size = separate items)

## Technical Notes

**Cart State Management:**
```typescript
interface CartItem {
  productId: string;
  variantId?: string;
  quantity: number;
  price: number;
  product: {
    name: string;
    slug: string;
    image: string;
  };
}

interface CartState {
  items: CartItem[];
  subtotal: number;
  itemCount: number;
  updatedAt: Date;
}
```

**Context API Setup:**
```typescript
const CartContext = createContext<{
  cart: CartState;
  addItem: (productId: string, quantity: number) => Promise<void>;
  updateQuantity: (productId: string, quantity: number) => Promise<void>;
  removeItem: (productId: string) => Promise<void>;
  clearCart: () => Promise<void>;
}>(null);
```

**Optimistic Updates:**
- Update local state immediately on user action
- Make API call in background
- Rollback local state if API call fails
- Show error toast on failure with retry button

**Cart Persistence Strategy:**
- Guest users: localStorage with key `shopflow_cart`
- Authenticated users: database CartItem table
- Sync on login: merge localStorage cart into database cart
- Sync on logout: keep localStorage cart for continued browsing

**Cart Merge Logic:**
```typescript
function mergeGuestCart(guestCart: CartItem[], userCart: CartItem[]) {
  const merged = [...userCart];

  for (const guestItem of guestCart) {
    const existing = merged.find(
      (item) => item.productId === guestItem.productId &&
                item.variantId === guestItem.variantId
    );

    if (existing) {
      existing.quantity += guestItem.quantity; // Combine quantities
    } else {
      merged.push(guestItem); // Add new item
    }
  }

  return merged;
}
```

**Real-Time Price Calculation:**
- Subtotal: sum of (quantity × price) for all items
- Tax: subtotal × tax rate (based on shipping address, defaults to 0)
- Shipping: calculated based on subtotal and delivery method
- Total: subtotal + tax + shipping

**Edge Cases:**
- Handle products removed from catalog while in cart (show "No longer available")
- Handle price changes between adding to cart and checkout (show notification)
- Handle stock becoming unavailable (show "Out of stock" warning)
- Handle cart with 100+ items (pagination or virtual scrolling)
- Handle concurrent cart updates from multiple tabs (use localStorage events)
- Handle cart data corruption (validate, clear invalid items)

## Tasks

- [x] Create CartContext with React Context API
- [x] Build useCart hook for accessing cart state and actions
- [x] Implement addItem, updateQuantity, removeItem, clearCart functions
- [x] Create CartSidebar component with slide-in animation
- [x] Build CartItem component with image, name, quantity controls, price
- [x] Implement quantity +/- buttons with optimistic updates
- [x] Add remove item button with confirmation for expensive items
- [x] Build empty cart state with illustration and CTA
- [x] Implement real-time subtotal calculation
- [x] Add cart badge to header with item count
- [x] Create badge pulse animation on item add
- [x] Implement localStorage persistence for guest carts
- [x] Create database Cart and CartItem models
- [x] Build cart API endpoints (GET, POST, PATCH, DELETE)
- [x] Implement cart merge logic on user login
- [x] Add cart sync between localStorage and database
- [x] Handle tab synchronization using localStorage events
- [x] Create cart validation logic for corrupted data
- [x] Add unit tests for cart operations and merge logic
- [x] Test cart persistence across browser sessions

## Dependencies

- Story 1-3: Development Environment (requires database schema)
- Story 2-1: Product Catalog (cart contains products)
- Story 4-1: User Registration (cart syncs on authentication)

## Estimation

**Story Points:** 8

**Rationale:** Cart management involves complex state management with optimistic updates and persistence logic. Syncing between localStorage and database adds complexity. Merge logic requires careful handling of edge cases. Real-time calculations and animations add polish but increase scope. Overall substantial feature with moderate-to-high complexity.
