---
status: done
completedAt: "2026-01-20"
---
# Story 1-3: Development Environment and Database Setup

## Description

This story establishes the local development environment infrastructure using Docker Compose to provide consistent, reproducible development environments across all team members. We're containerizing PostgreSQL 16 as our primary database and Redis 7 for caching and session management, eliminating the "works on my machine" problem and making onboarding new developers seamless.

Prisma ORM is configured as our database access layer, providing type-safe database queries with excellent TypeScript integration. The initial database schema models our core e-commerce entities: products, categories, users, orders, and carts. Prisma Migrate manages schema changes with version control, enabling safe database evolution as requirements change.

A comprehensive seed script populates the development database with realistic test data including sample products across multiple categories, test users with different roles, and sample orders. This allows developers to immediately start working with realistic data without manual setup. Developer documentation guides new team members through environment setup, database operations, and common troubleshooting scenarios.

## Acceptance Criteria

- Docker Compose file created with PostgreSQL 16 and Redis 7 services
- Services start successfully with `docker-compose up` and persist data between restarts
- Prisma ORM installed and configured with connection to PostgreSQL
- Initial schema defined for User, Product, Category, Order, OrderItem, Cart, CartItem
- Prisma Client generates TypeScript types matching schema
- Database migrations run successfully with `prisma migrate dev`
- Seed script creates 50+ sample products, 10+ categories, 5 test users
- Database connection pooling configured for development performance
- Environment variables documented in .env.example file
- Developer documentation includes setup instructions and common commands

## Technical Notes

**Docker Compose Configuration:**
- PostgreSQL 16 with persistent volume mounted to `./data/postgres`
- Redis 7 with AOF persistence enabled
- Health checks for both services to ensure readiness
- Port mappings: PostgreSQL 5432, Redis 6379
- Network isolation using custom bridge network

**Prisma Schema Design:**
```prisma
model Product {
  id            String   @id @default(cuid())
  name          String
  slug          String   @unique
  description   String?
  price         Decimal  @db.Decimal(10, 2)
  compareAtPrice Decimal? @db.Decimal(10, 2)
  sku           String   @unique
  stock         Int      @default(0)
  categoryId    String
  category      Category @relation(fields: [categoryId], references: [id])
  images        Json     @default("[]")
  status        ProductStatus @default(ACTIVE)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  @@index([categoryId])
  @@index([status])
  @@fulltext([name, description])
}
```

**Seed Data Strategy:**
- Use Faker.js for generating realistic product names and descriptions
- Create hierarchical categories: Electronics → Laptops → Gaming Laptops
- Generate users with bcrypt-hashed passwords (test password: "password123")
- Create sample orders with realistic order totals and timestamps
- Use transactions to ensure atomic seed operations

**Connection Pooling:**
- Configure PgBouncer for connection pooling in Docker Compose
- Set pool size based on expected concurrent developer load (min: 2, max: 10)
- Use connection_limit in Prisma to prevent pool exhaustion

**Edge Cases:**
- Handle database already exists error gracefully in seed script
- Provide reset script to drop and recreate database for clean state
- Ensure seed script is idempotent (can run multiple times safely)
- Handle different OS line endings in shell scripts (CRLF vs LF)

## Tasks

- [x] Create docker-compose.yml with PostgreSQL and Redis services
- [x] Configure PostgreSQL with persistent volume and health checks
- [x] Configure Redis with AOF persistence enabled
- [x] Install Prisma CLI and Prisma Client packages
- [x] Initialize Prisma with `prisma init` and configure DATABASE_URL
- [x] Design initial database schema for all core entities
- [x] Create Prisma schema file with models, relations, and indexes
- [x] Run initial migration with `prisma migrate dev --name init`
- [x] Create seed script with Faker.js for realistic test data
- [x] Generate 50+ products across 10+ categories in seed
- [x] Create test users with admin and customer roles
- [x] Add seed command to package.json scripts
- [x] Create .env.example with all required environment variables
- [x] Write developer documentation in docs/DEVELOPMENT.md
- [x] Test full setup flow on clean machine (Docker Desktop install)

## Dependencies

- Story 1-1: Project Setup (requires Node.js and package.json)

## Estimation

**Story Points:** 5

**Rationale:** Docker and Prisma setup are well-documented, but schema design requires careful thought about relationships and indexes. Seed script development with realistic data takes time. Documentation and testing the setup flow add additional work. Overall moderate complexity with clear implementation path.
