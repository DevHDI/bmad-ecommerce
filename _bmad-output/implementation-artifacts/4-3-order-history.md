---
status: backlog
completedAt: null
---
# Story 4-3: Order History and Tracking

## Description

This story implements order history and tracking features that provide customers with complete visibility into their past and current orders. Order history is a critical self-service feature - 52% of customers check order status online rather than contact support. We're building a comprehensive order management interface that displays all orders with filtering, detailed views, and real-time status tracking.

The order history page presents orders in reverse chronological order (newest first) with key information visible at a glance: order number, date, total, and current status. Advanced filtering allows narrowing by date range, status (pending, processing, shipped, delivered), and total amount ranges. Each order is clickable, expanding to show the complete item list, shipping details, and status timeline.

Real-time status tracking uses a visual timeline showing order progression through fulfillment stages. Each status change includes timestamp and optional notes from the merchant. The reorder functionality allows customers to quickly repeat previous purchases by adding all items from a past order to their current cart with one click, dramatically reducing time to repeat purchase.

## Acceptance Criteria

- Order history page displays at `/account/orders` for authenticated users
- Orders displayed in table/card format showing order number, date, status, total
- Default sort is newest first (most recent orders at top)
- Filter sidebar includes date range picker (last 30 days, 90 days, year, custom)
- Status filter allows selecting multiple statuses (pending, processing, shipped, delivered, cancelled)
- Amount range filter with min/max inputs
- Active filters display as removable chips above order list
- Pagination displays 20 orders per page with page numbers
- Click on order row expands or navigates to detailed order view
- Detailed view shows complete item list with images, names, quantities, prices
- Shipping address and billing address displayed
- Payment method shown (last 4 digits of card)
- Status timeline shows progression with timestamps for each status change
- Current status highlighted in timeline
- Delivered orders show delivery date and time
- Cancelled orders show cancellation reason if provided
- "Reorder" button adds all items from order to current cart
- Reorder handles out-of-stock items gracefully (skip or notify)
- Empty state for users with no orders shows call-to-action to browse products
- Mobile design uses card layout instead of table

## Technical Notes

**Order List Query:**
```typescript
interface OrderFilters {
  dateRange?: { start: Date; end: Date };
  statuses?: OrderStatus[];
  minTotal?: number;
  maxTotal?: number;
  page: number;
  pageSize: number;
}

async function getOrders(userId: string, filters: OrderFilters) {
  return prisma.order.findMany({
    where: {
      userId,
      createdAt: filters.dateRange ? {
        gte: filters.dateRange.start,
        lte: filters.dateRange.end,
      } : undefined,
      status: filters.statuses?.length ? {
        in: filters.statuses,
      } : undefined,
      total: {
        gte: filters.minTotal,
        lte: filters.maxTotal,
      },
    },
    orderBy: { createdAt: 'desc' },
    skip: (filters.page - 1) * filters.pageSize,
    take: filters.pageSize,
    include: {
      items: {
        include: { product: true },
      },
      shippingAddress: true,
    },
  });
}
```

**Status Timeline Component:**
```typescript
interface TimelineEvent {
  status: OrderStatus;
  timestamp: Date;
  note?: string;
}

const statusFlow = [
  'pending',
  'processing',
  'shipped',
  'delivered',
] as const;

function OrderTimeline({ events, currentStatus }: TimelineProps) {
  return (
    <div className="relative">
      {statusFlow.map((status) => {
        const event = events.find((e) => e.status === status);
        const isCompleted = event !== undefined;
        const isCurrent = status === currentStatus;

        return (
          <TimelineNode
            key={status}
            status={status}
            event={event}
            isCompleted={isCompleted}
            isCurrent={isCurrent}
          />
        );
      })}
    </div>
  );
}
```

**Reorder Functionality:**
```typescript
async function reorderItems(orderId: string) {
  const order = await prisma.order.findUnique({
    where: { id: orderId },
    include: { items: { include: { product: true } } },
  });

  const cartItems = [];
  const unavailableItems = [];

  for (const item of order.items) {
    // Check if product still exists and is in stock
    if (item.product.status === 'active' && item.product.stock >= item.quantity) {
      cartItems.push({
        productId: item.productId,
        quantity: item.quantity,
      });
    } else {
      unavailableItems.push(item.product.name);
    }
  }

  // Add available items to cart
  await addItemsToCart(order.userId, cartItems);

  // Notify about unavailable items
  if (unavailableItems.length > 0) {
    return {
      success: true,
      message: `${cartItems.length} items added to cart. ${unavailableItems.length} items no longer available: ${unavailableItems.join(', ')}`,
    };
  }

  return {
    success: true,
    message: `${cartItems.length} items added to cart`,
  };
}
```

**Date Range Presets:**
```typescript
const dateRangePresets = [
  {
    label: 'Last 30 days',
    getValue: () => ({
      start: subDays(new Date(), 30),
      end: new Date(),
    }),
  },
  {
    label: 'Last 90 days',
    getValue: () => ({
      start: subDays(new Date(), 90),
      end: new Date(),
    }),
  },
  {
    label: 'This year',
    getValue: () => ({
      start: startOfYear(new Date()),
      end: new Date(),
    }),
  },
  {
    label: 'Custom',
    getValue: () => null, // Opens date picker
  },
];
```

**Edge Cases:**
- Handle orders with cancelled items (show in item list with strikethrough)
- Handle refunded orders (show refund amount and date)
- Handle split shipments (multiple tracking numbers)
- Handle orders placed before user registration (associate by email)
- Handle very old orders (pagination prevents performance issues)
- Handle deleted products in order history (show placeholder)
- Handle reorder when cart already has items (merge, notify)
- Handle reorder with price changes (show notification about price difference)

## Tasks

- [ ] Create order history page at `/account/orders`
- [ ] Build order list component with table/card layouts
- [ ] Fetch orders from database with pagination
- [ ] Implement date range filter with presets
- [ ] Add status filter with multi-select checkboxes
- [ ] Create amount range filter with min/max inputs
- [ ] Display active filters as removable chips
- [ ] Implement filter clearing functionality
- [ ] Build detailed order view component
- [ ] Display complete item list with product details
- [ ] Show shipping and billing addresses
- [ ] Create status timeline component with visual progression
- [ ] Add timestamps to each status in timeline
- [ ] Highlight current status in timeline
- [ ] Implement reorder functionality
- [ ] Handle unavailable items during reorder gracefully
- [ ] Show notification for successfully added items
- [ ] Create empty state for users with no orders
- [ ] Add mobile-responsive card layout
- [ ] Test filtering and pagination performance

## Dependencies

- Story 3-3: Payment Integration (orders created from payments)
- Story 3-1: Cart Management (reorder adds to cart)
- Story 4-1: User Registration (orders associated with users)

## Estimation

**Story Points:** 8

**Rationale:** Order history involves complex filtering with multiple criteria and date ranges. Detailed order view requires careful layout of nested data. Status timeline component needs visual design. Reorder functionality involves cart integration and stock checking. Pagination and filtering performance optimization required. Overall high complexity with substantial scope.
