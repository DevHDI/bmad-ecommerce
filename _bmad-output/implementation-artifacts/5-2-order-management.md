---
status: backlog
completedAt: null
---
# Story 5-2: Order Management and Fulfillment

## Description

This story implements the admin order management system that enables efficient order processing and fulfillment. Order management is the operational heart of e-commerce - delays in processing directly impact customer satisfaction and merchant reputation. We're building a comprehensive order dashboard with status workflows, bulk processing capabilities, and refund handling that mirrors industry-leading platforms like Shopify and WooCommerce.

The order management interface provides a real-time view of all orders with advanced filtering by status, date range, customer, and payment method. Each order displays essential information (order number, customer, total, status) at a glance, with expandable details showing the complete order breakdown. The status workflow guides orders through fulfillment stages: pending → processing → shipped → delivered, with timestamps and notes logged for each transition.

Fulfillment actions include printing packing slips, generating shipping labels (integration with ShipStation or similar), and bulk status updates for efficiency. Refund processing integrates with Stripe to handle partial or full refunds while updating inventory automatically. Export functionality allows downloading order data as CSV for accounting and analytics purposes.

## Acceptance Criteria

- Order management page displays at `/admin/orders` (admin role required)
- Data table shows orders with columns: order number, date, customer, total, status, actions
- Default sort shows newest orders first
- Search bar filters by order number, customer name, or email
- Status filter allows selecting multiple statuses with checkboxes
- Date range filter with presets (today, last 7 days, last 30 days, custom)
- Payment method filter (card, PayPal, Apple Pay, etc.)
- Server-side pagination displays 25 orders per page
- Click order row expands inline detail view
- Detail view shows: customer info, shipping address, billing address, items list, totals
- Each item shows product image, name, SKU, quantity, unit price, total
- Order status displayed with color-coded badge
- Status update dropdown allows changing status (pending → processing → shipped → delivered)
- Status change requires confirmation and optional note
- "Mark as Shipped" action prompts for tracking number and carrier selection
- Tracking information saved and displayed in order timeline
- Timeline shows all status changes with timestamps and admin names
- "Print Packing Slip" button generates printer-friendly order summary
- "Refund" button opens refund modal for Stripe integration
- Refund modal shows refundable amount and partial refund option
- Refund updates inventory (returns items to stock)
- Cancelled orders show cancellation reason if provided
- Bulk actions toolbar for multi-select: Mark as Processing, Mark as Shipped, Export
- Export button downloads selected orders as CSV file
- Mobile view uses card layout instead of table

## Technical Notes

**Order List Query:**
```typescript
interface OrderFilters {
  search?: string;
  statuses?: OrderStatus[];
  dateRange?: { start: Date; end: Date };
  paymentMethod?: string;
  page: number;
  pageSize: number;
}

async function getOrders(filters: OrderFilters) {
  return prisma.order.findMany({
    where: {
      AND: [
        filters.search ? {
          OR: [
            { number: { contains: filters.search, mode: 'insensitive' } },
            { user: { email: { contains: filters.search, mode: 'insensitive' } } },
            { user: { name: { contains: filters.search, mode: 'insensitive' } } },
          ],
        } : {},
        filters.statuses?.length ? { status: { in: filters.statuses } } : {},
        filters.dateRange ? {
          createdAt: {
            gte: filters.dateRange.start,
            lte: filters.dateRange.end,
          },
        } : {},
      ],
    },
    orderBy: { createdAt: 'desc' },
    skip: (filters.page - 1) * filters.pageSize,
    take: filters.pageSize,
    include: {
      user: true,
      items: { include: { product: true } },
      shippingAddress: true,
      billingAddress: true,
    },
  });
}
```

**Status Update Workflow:**
```typescript
enum OrderStatus {
  PENDING = 'pending',
  PROCESSING = 'processing',
  SHIPPED = 'shipped',
  DELIVERED = 'delivered',
  CANCELLED = 'cancelled',
}

const statusTransitions = {
  pending: ['processing', 'cancelled'],
  processing: ['shipped', 'cancelled'],
  shipped: ['delivered', 'cancelled'],
  delivered: [],
  cancelled: [],
};

async function updateOrderStatus(
  orderId: string,
  newStatus: OrderStatus,
  note?: string,
  tracking?: { number: string; carrier: string }
) {
  const order = await prisma.order.findUnique({ where: { id: orderId } });

  // Validate transition
  if (!statusTransitions[order.status].includes(newStatus)) {
    throw new Error(`Invalid status transition from ${order.status} to ${newStatus}`);
  }

  // Update order
  await prisma.order.update({
    where: { id: orderId },
    data: {
      status: newStatus,
      trackingNumber: tracking?.number,
      trackingCarrier: tracking?.carrier,
      statusHistory: {
        create: {
          status: newStatus,
          note,
          adminId: currentAdminId,
        },
      },
    },
  });

  // Send notification email if shipped
  if (newStatus === 'shipped') {
    await sendShippingNotification(orderId);
  }
}
```

**Refund Processing:**
```typescript
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

async function processRefund(
  orderId: string,
  amount: number,
  reason?: string
) {
  const order = await prisma.order.findUnique({
    where: { id: orderId },
    include: { items: { include: { product: true } } },
  });

  // Create Stripe refund
  const refund = await stripe.refunds.create({
    payment_intent: order.paymentIntentId,
    amount: Math.round(amount * 100), // Convert to cents
    reason: reason || 'requested_by_customer',
  });

  // Update order
  await prisma.order.update({
    where: { id: orderId },
    data: {
      status: 'cancelled',
      refundedAmount: amount,
      refundedAt: new Date(),
    },
  });

  // Return items to stock
  for (const item of order.items) {
    await prisma.product.update({
      where: { id: item.productId },
      data: {
        stock: { increment: item.quantity },
      },
    });
  }

  return refund;
}
```

**CSV Export:**
```typescript
import { stringify } from 'csv-stringify/sync';

async function exportOrders(orderIds: string[]) {
  const orders = await prisma.order.findMany({
    where: { id: { in: orderIds } },
    include: { user: true, items: { include: { product: true } } },
  });

  const csvData = orders.map(order => ({
    'Order Number': order.number,
    'Date': order.createdAt.toISOString(),
    'Customer Name': order.user.name,
    'Customer Email': order.user.email,
    'Status': order.status,
    'Items': order.items.length,
    'Subtotal': order.subtotal,
    'Tax': order.tax,
    'Shipping': order.shipping,
    'Total': order.total,
    'Payment Method': order.paymentMethod,
  }));

  return stringify(csvData, { header: true });
}
```

**Edge Cases:**
- Handle refunds exceeding paid amount (validate before processing)
- Handle partial refunds (track refunded vs remaining amount)
- Handle refund failures (Stripe errors, insufficient funds)
- Handle orders with no stock to return (digital products)
- Handle status updates for delivered orders (prevent regression)
- Handle concurrent status updates by multiple admins (optimistic locking)
- Handle deleted customers (preserve order data)
- Handle bulk actions on thousands of orders (queue processing)

## Tasks

- [ ] Create order management page at `/admin/orders`
- [ ] Build order data table with sortable columns
- [ ] Implement search by order number, customer name, email
- [ ] Add status filter with multi-select
- [ ] Create date range filter with presets
- [ ] Add payment method filter dropdown
- [ ] Build expandable order detail view
- [ ] Display customer info and addresses in detail
- [ ] Show complete items list with product details
- [ ] Create status update dropdown with workflow validation
- [ ] Build "Mark as Shipped" modal with tracking input
- [ ] Implement status timeline with timestamps
- [ ] Create printable packing slip template
- [ ] Build refund modal with Stripe integration
- [ ] Implement partial refund calculation
- [ ] Add inventory return on refund
- [ ] Create bulk status update functionality
- [ ] Implement CSV export for orders
- [ ] Add mobile-responsive card layout
- [ ] Test with large order volume (1000+ orders)
- [ ] Add email notifications for status changes

## Dependencies

- Story 3-3: Payment Integration (orders have payment data)
- Story 3-4: Order Confirmation (shipping notifications)
- Story 4-1: User Registration (admin role enforcement)

## Estimation

**Story Points:** 13

**Rationale:** Order management is a complex feature with intricate workflows and external integrations. Status transition validation and timeline tracking add complexity. Refund processing with Stripe requires careful implementation. CSV export and bulk actions increase scope. Edge case handling (concurrent updates, refund failures) requires thorough testing. Overall this is a large, complex feature with substantial scope.
