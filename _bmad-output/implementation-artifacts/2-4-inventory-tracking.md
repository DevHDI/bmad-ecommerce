---
status: in-progress
completedAt: null
---
# Story 2-4: Inventory Tracking and Stock Management

## Description

This story implements comprehensive real-time inventory tracking that prevents overselling and provides transparency to customers about product availability. Accurate inventory management is critical for customer satisfaction - studies show 67% of customers abandon purchases due to out-of-stock items discovered at checkout. We're building a system that displays live stock levels, warns about low inventory, and gracefully handles out-of-stock scenarios.

The inventory tracking system maintains real-time stock counts updated on every order placement and admin inventory adjustment. Low-stock badges appear on product cards when inventory falls below 10 units, creating urgency while setting accurate expectations. When items are out of stock, customers can subscribe to "notify me" email alerts that trigger automatically when products are restocked.

During checkout, we implement stock reservation to prevent race conditions where multiple customers purchase the last item simultaneously. Reserved stock is held for 10 minutes during the checkout process, then released if the order isn't completed. This ensures inventory accuracy while minimizing cart abandonment from artificial scarcity.

## Acceptance Criteria

- Product stock level displays on product detail page as "X items available"
- Stock count updates in real-time when orders are placed
- Low-stock badge appears on product cards when inventory < 10 units
- Out-of-stock products show "Out of Stock" badge and disable add-to-cart
- "Notify Me" button appears for out-of-stock products
- Email notification sent to subscribers when product is restocked
- Stock reservation system reserves inventory during checkout for 10 minutes
- Reserved stock automatically releases if checkout is abandoned
- Admin inventory adjustment API updates stock with audit trail
- Inventory changes logged with timestamp, user, and reason
- Prevent negative inventory through database constraints
- Handle concurrent purchase attempts with database-level locking

## Technical Notes

**Database Schema Updates:**
```prisma
model Product {
  // ... existing fields
  stock            Int      @default(0)
  lowStockThreshold Int      @default(10)
  reservedStock    Int      @default(0)

  stockNotifications StockNotification[]
  inventoryLogs      InventoryLog[]
}

model StockNotification {
  id        String   @id @default(cuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  email     String
  notified  Boolean  @default(false)
  createdAt DateTime @default(now())

  @@index([productId, notified])
}

model InventoryLog {
  id        String   @id @default(cuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  change    Int      // Positive for additions, negative for reductions
  reason    String
  userId    String?
  createdAt DateTime @default(now())

  @@index([productId, createdAt])
}
```

**Stock Reservation Logic:**
```typescript
// Reserve stock during checkout
async function reserveStock(productId: string, quantity: number) {
  const product = await prisma.product.findUnique({
    where: { id: productId },
    select: { stock: true, reservedStock: true }
  });

  const available = product.stock - product.reservedStock;
  if (available < quantity) {
    throw new Error('Insufficient stock');
  }

  await prisma.product.update({
    where: { id: productId },
    data: { reservedStock: { increment: quantity } }
  });

  // Schedule release after 10 minutes
  setTimeout(() => releaseStock(productId, quantity), 10 * 60 * 1000);
}
```

**Stock Notification System:**
- Background job checks for restocked products every hour
- Query StockNotification table for products with notified=false
- Send batch emails using Resend API
- Mark notifications as sent to prevent duplicates
- Unsubscribe link in notification emails

**Concurrent Purchase Handling:**
- Use database row-level locking with SELECT FOR UPDATE
- Implement optimistic concurrency control with version field
- Return clear error messages when stock is exhausted
- Suggest alternative products in stock

**Edge Cases:**
- Handle partial stock availability (e.g., customer wants 5, only 3 available)
- Handle inventory sync delays from external systems
- Handle reserved stock that never converts to orders (cleanup job)
- Handle negative inventory from race conditions (alert admin)
- Handle stock notifications for discontinued products
- Handle bulk inventory imports (validate, transaction rollback on error)

## Tasks

- [x] Add stock, lowStockThreshold, reservedStock fields to Product schema
- [x] Create Prisma migration for inventory fields
- [x] Build StockIndicator component showing available quantity
- [x] Implement low-stock badge logic (<10 units)
- [x] Create out-of-stock state for product cards
- [x] Disable add-to-cart button when stock is 0
- [ ] Build "Notify Me" subscription form component
- [ ] Create StockNotification database model and migration
- [ ] Implement notify-me subscription API endpoint
- [ ] Build stock reservation system for checkout
- [ ] Create background job to release expired reservations
- [ ] Implement stock notification email sending with Resend
- [ ] Create admin inventory adjustment API with audit logging
- [ ] Build InventoryLog model for tracking all stock changes
- [ ] Add database-level constraints to prevent negative stock
- [ ] Implement row-level locking for concurrent purchases
- [ ] Create cleanup job for stale stock reservations
- [ ] Test concurrent purchase scenarios with load testing
- [ ] Build admin UI for viewing inventory logs
- [ ] Add monitoring alerts for low stock across all products

## Dependencies

- Story 1-3: Development Environment (requires database schema)
- Story 2-1: Product Catalog (inventory affects product display)
- Story 3-1: Cart Management (stock checks on add-to-cart)
- Story 3-2: Checkout Flow (stock reservation during checkout)

## Estimation

**Story Points:** 8

**Rationale:** Inventory tracking involves complex state management with concurrency challenges. Stock reservation system requires careful timing and cleanup logic. Background jobs for notifications add complexity. Database locking and constraint handling require expertise. Overall high complexity with potential for subtle bugs requiring thorough testing.
