---
status: done
completedAt: "2026-01-19"
---
# Story 1-2: CI/CD Pipeline Setup

## Description

This story establishes automated continuous integration and deployment pipelines using GitHub Actions and Vercel. The CI pipeline ensures code quality by running linting, type-checking, and automated tests on every pull request, preventing broken code from reaching production. The CD pipeline automates deployments to preview environments for each PR and to production when changes merge to the main branch.

We're implementing a comprehensive quality gate system that includes ESLint checks for code style, TypeScript compilation for type safety, and automated test suites with coverage reporting. Each pull request triggers a preview deployment on Vercel, allowing stakeholders to review changes in a production-like environment before merging. This reduces the risk of deployment failures and enables faster iteration cycles.

Branch protection rules enforce that all checks must pass before merging, ensuring the main branch remains stable. Code coverage reports are generated and tracked over time to maintain testing standards. This automation reduces manual review burden and catches issues early in the development cycle when they're cheapest to fix.

## Acceptance Criteria

- GitHub Actions workflow file created at `.github/workflows/ci.yml` running on all PRs
- Lint check (ESLint) runs successfully and fails the build on errors
- Type-check (TypeScript) runs successfully and fails the build on type errors
- Automated tests run with coverage reporting to Codecov or similar service
- Vercel integration configured for automatic preview deployments on PRs
- Production deployment triggers automatically on merge to main branch
- Branch protection rules enabled on main requiring all checks to pass
- Status badges added to README.md showing build and coverage status
- Workflow runs complete in under 3 minutes for typical changes
- Failed builds provide clear error messages for debugging

## Technical Notes

**GitHub Actions Workflow:**
- Use matrix strategy to test on Node 18.x and 20.x
- Cache node_modules and Next.js build cache for faster runs
- Run jobs in parallel: lint, typecheck, and test
- Use concurrency groups to cancel outdated workflow runs on new pushes

**Vercel Configuration:**
- Install Vercel GitHub App for automatic deployments
- Configure build command: `npm run build`
- Set environment variables in Vercel dashboard (DATABASE_URL, etc.)
- Enable automatic comment on PRs with preview URL
- Configure production deployment from main branch only

**Branch Protection:**
- Require pull request before merging to main
- Require status checks: lint, typecheck, test
- Require up-to-date branches before merging
- Restrict push access to main branch (only via PR)

**Code Coverage:**
- Use Jest with coverage reporters: lcov, json, text
- Upload coverage to Codecov using GitHub Action
- Set minimum coverage threshold: 70% for new code
- Generate coverage badges for README

**Edge Cases:**
- Handle flakey tests with retry logic (max 2 retries)
- Skip CI on documentation-only changes using path filters
- Provide manual workflow dispatch for emergency deployments
- Handle secrets securely using GitHub Secrets, never commit credentials

## Tasks

- [x] Create GitHub Actions workflow file with lint, typecheck, test jobs
- [x] Configure ESLint job to run on all TypeScript/JavaScript files
- [x] Configure TypeScript compilation check job
- [x] Set up Jest test runner with coverage collection
- [x] Integrate Codecov for coverage reporting and tracking
- [x] Install and configure Vercel GitHub App integration
- [x] Configure Vercel environment variables for preview and production
- [x] Set up automatic preview deployments for pull requests
- [x] Configure production deployment on main branch merge
- [x] Enable branch protection rules on main branch
- [x] Add status badges to README.md for build and coverage
- [x] Test full workflow with a sample pull request
- [x] Document CI/CD process in CONTRIBUTING.md
- [x] Set up Slack/Discord notifications for deployment status (optional)
- [x] Optimize workflow caching to reduce build time under 3 minutes

## Dependencies

- Story 1-1: Project Setup (requires package.json scripts and configuration)

## Estimation

**Story Points:** 5

**Rationale:** Setting up CI/CD pipelines is well-understood but requires careful configuration of multiple services (GitHub Actions, Vercel, Codecov). The integration testing to ensure all checks work correctly adds complexity. Branch protection setup and documentation are straightforward but time-consuming.
