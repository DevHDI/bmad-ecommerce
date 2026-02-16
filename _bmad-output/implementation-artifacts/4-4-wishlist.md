---
status: backlog
completedAt: null
---
# Story 4-4: Wishlist and Favorites

## Description

This story implements wishlist functionality that allows customers to save products for future consideration. Wishlists increase customer engagement and provide valuable intent signals - 40% of wishlist items eventually convert to purchases. We're building a simple, intuitive wishlist system with heart icons for saving products, a dedicated wishlist page for managing saved items, and sharing capabilities for gift registries or collaborative shopping.

The wishlist toggle appears as a heart icon on product cards and detail pages. Clicking toggles the saved state with immediate visual feedback (filled vs outline heart) and optimistic UI updates. The wishlist page displays all saved items in a grid matching the product catalog layout, making it easy to browse and compare saved products. Each item includes a "Move to Cart" action for quick purchase conversion.

Sharing functionality generates a unique URL for each user's wishlist, enabling gift registries, collaborative shopping lists, or sharing favorites with friends. Shared wishlists are view-only for non-owners but display the same product information. The wishlist persists in the database for authenticated users and syncs across devices, ensuring customers never lose their saved items.

## Acceptance Criteria

- Heart icon appears on all product cards in listing and search results
- Heart icon appears on product detail page near add-to-cart button
- Clicking heart toggles wishlist state with animation (outline → filled)
- Unauthenticated users redirected to login when clicking heart
- Wishlist state updates immediately with optimistic UI
- Wishlist page displays at `/account/wishlist` for authenticated users
- Saved items shown in grid layout matching product catalog
- Each item shows product image, name, price, rating, stock status
- Out-of-stock items display "Out of Stock" badge
- "Move to Cart" button adds item to cart and optionally removes from wishlist
- Remove from wishlist button (X icon) with confirmation for expensive items
- Empty wishlist state shows message and link to browse products
- "Share Wishlist" button generates unique shareable URL
- Shared wishlist displays at `/wishlist/[shareId]` publicly
- Shared wishlist is view-only, cannot be edited by non-owners
- Share modal includes copy-to-clipboard button and social share links
- Wishlist count badge appears in header navigation next to cart icon

## Technical Notes

**Database Schema:**
```prisma
model WishlistItem {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  addedAt   DateTime @default(now())

  @@unique([userId, productId])
  @@index([userId])
}

model Wishlist {
  id        String   @id @default(cuid())
  userId    String   @unique
  user      User     @relation(fields: [userId], references: [id])
  shareId   String?  @unique
  isPublic  Boolean  @default(false)
  createdAt DateTime @default(now())
}
```

**Wishlist Toggle API:**
```typescript
// Add to wishlist
POST /api/wishlist
Body: { productId: string }

// Remove from wishlist
DELETE /api/wishlist/:productId

// Check if product is in wishlist
GET /api/wishlist/check/:productId
Response: { isInWishlist: boolean }
```

**Move to Cart Functionality:**
```typescript
async function moveToCart(userId: string, productId: string, removeFromWishlist: boolean = true) {
  // Add to cart
  await addItemToCart(userId, productId, 1);

  // Optionally remove from wishlist
  if (removeFromWishlist) {
    await prisma.wishlistItem.delete({
      where: {
        userId_productId: { userId, productId },
      },
    });
  }

  return { success: true };
}
```

**Share URL Generation:**
```typescript
import { nanoid } from 'nanoid';

async function generateShareLink(userId: string) {
  const shareId = nanoid(10); // Generate short unique ID

  await prisma.wishlist.update({
    where: { userId },
    data: {
      shareId,
      isPublic: true,
    },
  });

  return `${process.env.NEXT_PUBLIC_APP_URL}/wishlist/${shareId}`;
}
```

**Optimistic UI Update:**
```typescript
const toggleWishlist = async (productId: string) => {
  const newState = !isInWishlist;

  // Update UI immediately
  setIsInWishlist(newState);
  setLocalWishlistCount(prev => newState ? prev + 1 : prev - 1);

  try {
    if (newState) {
      await fetch('/api/wishlist', {
        method: 'POST',
        body: JSON.stringify({ productId }),
      });
    } else {
      await fetch(`/api/wishlist/${productId}`, {
        method: 'DELETE',
      });
    }
  } catch (error) {
    // Rollback on error
    setIsInWishlist(!newState);
    setLocalWishlistCount(prev => newState ? prev - 1 : prev + 1);
    toast.error('Failed to update wishlist');
  }
};
```

**Edge Cases:**
- Handle adding out-of-stock items to wishlist (allow, notify when back in stock)
- Handle deleted products in wishlist (show placeholder, allow removal)
- Handle products with price changes (show current price)
- Handle wishlist size limits (max 100 items to prevent abuse)
- Handle concurrent wishlist updates from multiple tabs
- Handle sharing when wishlist is empty (show empty state to viewer)
- Handle unauthenticated access to heart icon (redirect to login)
- Handle revoked share links (admin can disable public sharing)

## Tasks

- [ ] Create WishlistItem and Wishlist database models
- [ ] Run Prisma migration for wishlist tables
- [ ] Build heart icon toggle component with animation
- [ ] Add heart icon to ProductCard component
- [ ] Add heart icon to product detail page
- [ ] Implement wishlist toggle API endpoints (add/remove)
- [ ] Add optimistic UI updates for wishlist toggle
- [ ] Create wishlist page at `/account/wishlist`
- [ ] Build wishlist grid layout matching product catalog
- [ ] Display out-of-stock badges on wishlist items
- [ ] Implement "Move to Cart" functionality
- [ ] Add remove from wishlist with confirmation dialog
- [ ] Create empty wishlist state with CTA
- [ ] Build share modal with URL generation
- [ ] Implement share link generation with unique ID
- [ ] Create public wishlist view at `/wishlist/[shareId]`
- [ ] Add copy-to-clipboard button in share modal
- [ ] Add wishlist count badge to header navigation
- [ ] Test wishlist sync across multiple devices
- [ ] Add social share links (Facebook, Twitter, Email)

## Dependencies

- Story 4-1: User Registration (wishlist requires authentication)
- Story 3-1: Cart Management (move to cart functionality)
- Story 2-1: Product Catalog (wishlist displays products)

## Estimation

**Story Points:** 5

**Rationale:** Wishlist is conceptually similar to cart with simpler state management. Heart toggle with optimistic UI is straightforward. Wishlist page layout reuses product grid components. Share link generation adds moderate complexity. Public wishlist view is simple read-only page. Overall moderate complexity with clear implementation patterns.
