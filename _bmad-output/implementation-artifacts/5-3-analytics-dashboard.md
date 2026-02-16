---
status: backlog
completedAt: null
---
# Story 5-3: Sales Analytics Dashboard

## Description

This story implements a comprehensive analytics dashboard that provides store owners with actionable business intelligence. Data-driven decision making is the competitive advantage of successful e-commerce businesses - merchants using analytics are 23% more profitable than those relying on intuition. We're building a dashboard that visualizes key metrics including revenue trends, top-performing products, customer acquisition, and conversion funnels with interactive charts and real-time data.

The dashboard provides at-a-glance visibility into business health through metric cards showing today's revenue, orders, and average order value compared to previous periods with percentage change indicators. Revenue charts display daily, weekly, or monthly trends using line graphs with the ability to compare time periods. Product rankings highlight best-sellers by revenue and units sold, helping merchants focus inventory and marketing efforts.

Customer metrics track acquisition channels, lifetime value, and retention rates. The conversion funnel visualization shows drop-off points from product view → add-to-cart → checkout → purchase, identifying optimization opportunities. All data updates in real-time as orders are placed, providing merchants with current business performance without manual refresh.

## Acceptance Criteria

- Analytics dashboard displays at `/admin/analytics` (admin role required)
- Top section shows 4 metric cards: Revenue, Orders, Average Order Value, Conversion Rate
- Each metric card displays current value, previous period value, and percentage change
- Percentage change shown in green (positive) or red (negative) with up/down arrow
- Revenue chart displays line graph with selectable time ranges (7d, 30d, 90d, 1y)
- Chart data updates when time range selected without page reload
- Comparison mode overlays previous period on chart for trend analysis
- Top Products section shows table with columns: product name, units sold, revenue, percentage of total revenue
- Table sorts by revenue descending (highest revenue products first)
- Limit to top 10 products with "View All" link to full report
- Customer metrics section shows: Total Customers, New Customers This Month, Repeat Customer Rate
- Acquisition channels chart displays pie/donut chart of traffic sources
- Conversion funnel shows 4 stages: Product Views, Add to Cart, Checkout, Purchase
- Each funnel stage shows count and percentage of previous stage
- Drop-off percentage highlighted between stages
- All charts are responsive and mobile-friendly
- Export button downloads dashboard data as PDF report
- Date range picker allows custom date selection for all metrics

## Technical Notes

**Metrics Calculation:**
```typescript
async function calculateDashboardMetrics(dateRange: { start: Date; end: Date }) {
  // Current period metrics
  const orders = await prisma.order.findMany({
    where: {
      createdAt: { gte: dateRange.start, lte: dateRange.end },
      status: { in: ['processing', 'shipped', 'delivered'] },
    },
  });

  const revenue = orders.reduce((sum, order) => sum + order.total, 0);
  const orderCount = orders.length;
  const avgOrderValue = orderCount > 0 ? revenue / orderCount : 0;

  // Previous period for comparison
  const periodLength = dateRange.end.getTime() - dateRange.start.getTime();
  const prevPeriodEnd = dateRange.start;
  const prevPeriodStart = new Date(prevPeriodEnd.getTime() - periodLength);

  const prevOrders = await prisma.order.findMany({
    where: {
      createdAt: { gte: prevPeriodStart, lte: prevPeriodEnd },
      status: { in: ['processing', 'shipped', 'delivered'] },
    },
  });

  const prevRevenue = prevOrders.reduce((sum, order) => sum + order.total, 0);
  const revenueChange = ((revenue - prevRevenue) / prevRevenue) * 100;

  return {
    revenue: { current: revenue, previous: prevRevenue, change: revenueChange },
    orders: { current: orderCount, previous: prevOrders.length },
    avgOrderValue: { current: avgOrderValue },
  };
}
```

**Top Products Query:**
```typescript
async function getTopProducts(limit: number = 10) {
  const topProducts = await prisma.orderItem.groupBy({
    by: ['productId'],
    _sum: {
      quantity: true,
      total: true,
    },
    orderBy: {
      _sum: {
        total: 'desc',
      },
    },
    take: limit,
  });

  // Enrich with product details
  const productsWithDetails = await Promise.all(
    topProducts.map(async (item) => {
      const product = await prisma.product.findUnique({
        where: { id: item.productId },
      });

      return {
        name: product.name,
        unitsSold: item._sum.quantity,
        revenue: item._sum.total,
        image: product.images[0],
      };
    })
  );

  return productsWithDetails;
}
```

**Revenue Chart Data:**
```typescript
async function getRevenueChartData(days: number) {
  const chartData = [];
  const endDate = new Date();
  const startDate = subDays(endDate, days);

  for (let i = 0; i < days; i++) {
    const date = subDays(endDate, days - i - 1);
    const nextDate = addDays(date, 1);

    const dayRevenue = await prisma.order.aggregate({
      where: {
        createdAt: { gte: date, lt: nextDate },
        status: { in: ['processing', 'shipped', 'delivered'] },
      },
      _sum: { total: true },
    });

    chartData.push({
      date: format(date, 'MMM dd'),
      revenue: dayRevenue._sum.total || 0,
    });
  }

  return chartData;
}
```

**Conversion Funnel Calculation:**
```typescript
async function getConversionFunnel(dateRange: { start: Date; end: Date }) {
  // Product views (tracked via analytics event)
  const productViews = await prisma.analyticsEvent.count({
    where: {
      event: 'product_view',
      createdAt: { gte: dateRange.start, lte: dateRange.end },
    },
  });

  // Add to cart events
  const addToCarts = await prisma.analyticsEvent.count({
    where: {
      event: 'add_to_cart',
      createdAt: { gte: dateRange.start, lte: dateRange.end },
    },
  });

  // Checkout initiated
  const checkouts = await prisma.order.count({
    where: {
      createdAt: { gte: dateRange.start, lte: dateRange.end },
    },
  });

  // Completed purchases
  const purchases = await prisma.order.count({
    where: {
      createdAt: { gte: dateRange.start, lte: dateRange.end },
      status: { in: ['processing', 'shipped', 'delivered'] },
    },
  });

  return [
    { stage: 'Product Views', count: productViews, percentage: 100 },
    { stage: 'Add to Cart', count: addToCarts, percentage: (addToCarts / productViews) * 100 },
    { stage: 'Checkout', count: checkouts, percentage: (checkouts / addToCarts) * 100 },
    { stage: 'Purchase', count: purchases, percentage: (purchases / checkouts) * 100 },
  ];
}
```

**Chart Library (Recharts):**
```typescript
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';

function RevenueChart({ data }: { data: ChartData[] }) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <LineChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="date" />
        <YAxis />
        <Tooltip formatter={(value) => `$${value.toFixed(2)}`} />
        <Line type="monotone" dataKey="revenue" stroke="#2563eb" strokeWidth={2} />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

**Edge Cases:**
- Handle zero revenue periods (show $0, not error)
- Handle division by zero in percentage calculations (show N/A)
- Handle missing previous period data (show "No comparison data")
- Handle very large numbers (format with K, M notation)
- Handle timezone differences in date calculations
- Handle incomplete orders (exclude from revenue calculations)
- Handle refunded orders (subtract from revenue)
- Handle data refresh during viewing (use polling or WebSocket)

## Tasks

- [ ] Create analytics dashboard page at `/admin/analytics`
- [ ] Build metric card component with value, change, and trend arrow
- [ ] Calculate current period metrics (revenue, orders, AOV, conversion)
- [ ] Calculate previous period metrics for comparison
- [ ] Compute percentage change with positive/negative styling
- [ ] Install Recharts library for chart rendering
- [ ] Build revenue line chart component
- [ ] Implement time range selector (7d, 30d, 90d, 1y)
- [ ] Add chart data fetching for selected time range
- [ ] Create top products table component
- [ ] Query top 10 products by revenue
- [ ] Display product image, name, units sold, revenue
- [ ] Build customer metrics section
- [ ] Calculate total customers, new customers, repeat rate
- [ ] Create conversion funnel visualization
- [ ] Track analytics events (product views, add to cart)
- [ ] Calculate funnel drop-off percentages
- [ ] Implement date range picker for custom dates
- [ ] Add PDF export functionality for dashboard
- [ ] Optimize queries for large datasets (indexes, aggregations)
- [ ] Test dashboard performance with 10,000+ orders

## Dependencies

- Story 3-3: Payment Integration (orders data for revenue)
- Story 2-1: Product Catalog (product views tracking)
- Story 3-1: Cart Management (add-to-cart events)
- Story 4-1: User Registration (admin role enforcement)

## Estimation

**Story Points:** 8

**Rationale:** Analytics dashboard involves complex data aggregation queries and calculations. Multiple chart types require integration of charting library (Recharts). Metric calculations with period comparisons add complexity. Conversion funnel requires event tracking implementation. Performance optimization for large datasets is critical. Overall high complexity with substantial data processing requirements.
