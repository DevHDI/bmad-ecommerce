---
status: done
completedAt: "2026-01-15"
---
# ShopFlow - Architecture Document

## 1. System Overview

ShopFlow is a modern e-commerce platform built on a headless architecture with Next.js as the frontend framework and a RESTful API backend. The system is designed for scalability, performance, and developer experience.

### 1.1 Architecture Style

The platform follows a **modular monolith** architecture pattern, organized by business domains (products, orders, users, payments). Each module has clear boundaries and communicates through well-defined interfaces, allowing future extraction into microservices if needed.

### 1.2 Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Frontend Framework | Next.js 14 (App Router) | Server components, streaming, built-in optimizations |
| Database | PostgreSQL 16 | ACID compliance, JSON support, full-text search |
| ORM | Prisma | Type-safe queries, migration management, excellent DX |
| Cache | Redis 7 | Session storage, cart persistence, rate limiting |
| Search | PostgreSQL FTS + pg_trgm | Avoids external dependency, sufficient for current scale |
| Payments | Stripe | Industry standard, extensive API, webhook support |
| Auth | NextAuth.js v5 | Native Next.js integration, multiple providers |
| Styling | Tailwind CSS v4 | Utility-first, design system consistency |
| Deployment | Vercel + AWS | Edge rendering + managed infrastructure |

## 2. System Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                    CDN (Vercel Edge)                  │
├─────────────────────────────────────────────────────┤
│              Next.js Application                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │  Pages &  │ │  Server  │ │   API    │            │
│  │ Layouts   │ │ Actions  │ │  Routes  │            │
│  └──────────┘ └──────────┘ └──────────┘            │
├─────────────────────────────────────────────────────┤
│                 Service Layer                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐│
│  │ Product  │ │  Order   │ │   User   │ │Payment ││
│  │ Service  │ │ Service  │ │ Service  │ │Service ││
│  └──────────┘ └──────────┘ └──────────┘ └────────┘│
├─────────────────────────────────────────────────────┤
│              Data Access Layer                        │
│  ┌──────────────┐  ┌───────────┐  ┌──────────────┐ │
│  │  PostgreSQL   │  │   Redis   │  │    Stripe    │ │
│  │  (Prisma)     │  │  (Cache)  │  │  (Payments)  │ │
│  └──────────────┘  └───────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────┘
```

## 3. Data Model

### 3.1 Core Entities

**User**
- id, email, name, passwordHash, role (customer | admin)
- Relations: orders, addresses, wishlist, reviews

**Product**
- id, name, slug, description, price, compareAtPrice, sku
- categoryId, inventory (stock, lowStockThreshold)
- images[], tags[], status (active | draft | archived)
- Relations: category, reviews, orderItems

**Category**
- id, name, slug, parentId (self-referencing for hierarchy)
- description, image, sortOrder
- Relations: parent, children, products

**Order**
- id, userId, status (pending | processing | shipped | delivered | cancelled)
- subtotal, tax, shipping, total, currency
- shippingAddressId, billingAddressId
- Relations: user, items, payments, shippingAddress

**OrderItem**
- id, orderId, productId, quantity, unitPrice, total
- Relations: order, product

**Cart**
- id, userId (nullable for guest carts), sessionId
- items: CartItem[] (productId, quantity, addedAt)
- Relations: user, items

### 3.2 Database Indexes

```sql
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_status ON products(status) WHERE status = 'active';
CREATE INDEX idx_products_search ON products USING gin(to_tsvector('english', name || ' ' || description));
CREATE INDEX idx_orders_user ON orders(user_id, created_at DESC);
CREATE INDEX idx_orders_status ON orders(status);
```

## 4. API Design

### 4.1 Public API Routes

| Method | Path | Description |
|--------|------|-------------|
| GET | /api/products | List products with filters |
| GET | /api/products/:slug | Get product details |
| GET | /api/categories | Get category tree |
| POST | /api/cart | Add item to cart |
| PATCH | /api/cart/:itemId | Update cart item |
| DELETE | /api/cart/:itemId | Remove from cart |
| POST | /api/checkout | Create checkout session |
| POST | /api/webhooks/stripe | Handle Stripe webhooks |

### 4.2 Admin API Routes

| Method | Path | Description |
|--------|------|-------------|
| GET | /api/admin/products | List all products |
| POST | /api/admin/products | Create product |
| PATCH | /api/admin/products/:id | Update product |
| GET | /api/admin/orders | List orders |
| PATCH | /api/admin/orders/:id | Update order status |

## 5. Security Considerations

- **Authentication**: JWT tokens with HttpOnly cookies, CSRF protection
- **Authorization**: Role-based access control (RBAC) with middleware
- **Data Validation**: Zod schemas on all API inputs
- **SQL Injection**: Prevented by Prisma's parameterized queries
- **XSS Prevention**: React's built-in escaping + CSP headers
- **Rate Limiting**: Redis-based rate limiting on auth and checkout endpoints
- **PCI Compliance**: No card data touches our servers (Stripe Elements)
- **HTTPS**: Enforced via Vercel edge network

## 6. Performance Strategy

- **Server Components**: Reduce client-side JavaScript bundle
- **Streaming**: Progressive page rendering with Suspense
- **Image Optimization**: Next.js Image component with automatic WebP/AVIF
- **Database**: Connection pooling via PgBouncer, query optimization
- **Caching**: Redis for sessions, cart, product catalog (5min TTL)
- **CDN**: Static assets and ISR pages cached at edge
- **Search**: PostgreSQL full-text search with materialized views for popular queries

## 7. Deployment Architecture

- **Frontend**: Vercel (automatic deployments from GitHub)
- **Database**: AWS RDS PostgreSQL (Multi-AZ for production)
- **Cache**: AWS ElastiCache Redis
- **File Storage**: AWS S3 + CloudFront for product images
- **Monitoring**: Vercel Analytics + Sentry for error tracking
- **CI/CD**: GitHub Actions → Vercel Preview/Production
