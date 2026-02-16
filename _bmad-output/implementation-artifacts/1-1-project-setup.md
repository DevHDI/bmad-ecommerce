---
status: done
completedAt: "2026-01-18"
---
# Story 1-1: Project Setup and Foundation

## Description

This foundational story establishes the core technical infrastructure for the ShopFlow e-commerce platform. We're initializing a Next.js 14 project using the App Router architecture, which provides server components, streaming, and built-in performance optimizations critical for our e-commerce use case. The setup includes TypeScript for type safety across the codebase, Tailwind CSS v4 for our design system implementation, and ESLint with Prettier for code quality enforcement.

The folder structure follows a domain-driven design approach, organizing code by business concerns (products, orders, users) rather than technical layers. This makes the codebase more maintainable as it scales and allows for potential future extraction of modules into microservices. We're establishing conventions for component organization, API routes, and shared utilities that will be followed throughout the project.

This story also includes the creation of essential configuration files (tsconfig.json, next.config.js, .eslintrc) with settings optimized for performance, security, and developer experience. We're configuring absolute imports using the `@/` prefix to avoid deeply nested relative imports, and setting up path aliases for common directories like components, lib, and types.

## Acceptance Criteria

- Next.js 14.0+ project initialized with App Router and TypeScript support
- Tailwind CSS v4 configured with `@theme inline` in globals.css (no tailwind.config file)
- ESLint and Prettier configured with recommended rules for Next.js and TypeScript
- Folder structure created following domain-driven design: `/app`, `/components`, `/lib`, `/types`, `/services`
- Absolute imports configured with `@/` prefix working in all TypeScript files
- Development server starts successfully on `localhost:3000` without errors
- Hot module replacement (HMR) working for component changes
- Build process completes successfully with `next build` producing optimized output
- Basic root layout and page created with proper metadata and SEO tags
- All linting and type-checking passes with zero errors

## Technical Notes

**Next.js Configuration:**
- Enable experimental features: `serverActions: true` for form handling
- Configure image domains for future CDN integration
- Set up environment variable validation using Zod schemas
- Enable React Strict Mode for development safety

**TypeScript Configuration:**
- Set `strict: true` for maximum type safety
- Enable `noUncheckedIndexedAccess` to catch array access errors
- Configure paths: `@/*` maps to `./src/*`
- Use `moduleResolution: "bundler"` for modern resolution

**Tailwind CSS v4:**
- Use `@theme inline` directive in globals.css instead of config file
- Define custom theme tokens for colors, spacing, typography
- Configure content paths to include all component directories
- Set up CSS variable-based theming for dark mode support

**Folder Structure:**
```
src/
├── app/              # Next.js App Router pages and layouts
│   ├── (shop)/      # Shop routes group
│   ├── (admin)/     # Admin routes group
│   └── api/         # API routes
├── components/       # React components
│   ├── ui/          # Base UI components (shadcn)
│   ├── products/    # Product-specific components
│   ├── cart/        # Cart components
│   └── layout/      # Layout components
├── lib/             # Shared utilities and configurations
├── services/        # Business logic services
├── types/           # TypeScript type definitions
└── styles/          # Global styles
```

**Edge Cases:**
- Ensure Node.js version compatibility (18.17+)
- Handle Windows vs Unix path differences in scripts
- Verify Tailwind intellisense works in VSCode
- Test build output size is within reasonable limits (<500KB initial bundle)

## Tasks

- [x] Initialize Next.js 14 project with create-next-app using TypeScript template
- [x] Install and configure Tailwind CSS v4 with @theme inline approach
- [x] Set up ESLint with Next.js, TypeScript, and accessibility plugins
- [x] Configure Prettier with integration for Tailwind class sorting
- [x] Create folder structure following domain-driven design pattern
- [x] Configure absolute imports with @/ prefix in tsconfig.json
- [x] Create root layout with HTML metadata, fonts (Inter), and global styles
- [x] Set up environment variable schema validation with Zod
- [x] Create basic homepage with SEO metadata
- [x] Configure next.config.js with image domains and performance settings
- [x] Set up .gitignore with proper exclusions for Next.js, Node, and IDE files
- [x] Create package.json scripts for dev, build, lint, type-check, and format
- [x] Test development server startup and hot reload functionality
- [x] Run production build and verify output optimization
- [x] Document setup instructions in README.md

## Dependencies

- None (this is the foundation story)

## Estimation

**Story Points:** 7

**Rationale:** While project initialization is straightforward, proper configuration of TypeScript, ESLint, and Tailwind CSS v4 requires careful setup. The folder structure decisions impact the entire project, so thoughtful organization is critical. Includes time for documentation and verification of all tooling working correctly together.
