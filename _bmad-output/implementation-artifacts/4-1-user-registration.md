---
status: done
completedAt: "2026-02-05"
---
# Story 4-1: User Registration and Authentication

## Description

This story implements comprehensive user authentication using NextAuth.js v5, providing secure, flexible authentication with multiple sign-in methods. Authentication is foundational for personalization features like saved addresses, order history, and wishlists. We support both traditional email/password registration and OAuth providers (Google, GitHub) to reduce friction - OAuth users convert 20-30% better than email/password due to reduced signup steps.

NextAuth.js handles the complexity of session management, CSRF protection, and secure token storage. JWT-based sessions provide stateless authentication that scales horizontally without session storage requirements. Password security follows best practices with bcrypt hashing (12 rounds) and minimum password requirements (8+ characters, mix of upper/lower/numbers).

Protected routes use Next.js middleware to redirect unauthenticated users to login, while preserving the intended destination for post-login redirect. Password reset flow uses time-limited tokens sent via email, expiring after 1 hour for security. All authentication forms include proper accessibility attributes and inline validation for immediate user feedback.

## Acceptance Criteria

- Registration form at `/register` with email, password, confirm password fields
- Password validation enforces minimum 8 characters, uppercase, lowercase, number
- Email validation checks format and uniqueness in database
- Confirm password must match password field with real-time validation
- Google OAuth button initiates OAuth flow and creates/logs in user
- GitHub OAuth button initiates OAuth flow and creates/logs in user
- Login form at `/login` with email and password fields
- JWT session tokens stored in httpOnly cookies for security
- Session persists across browser sessions until explicit logout
- Protected routes redirect to login with returnUrl parameter
- After successful login, redirect to returnUrl or default dashboard
- Logout button clears session and redirects to homepage
- Password reset link at `/forgot-password` sends email with reset token
- Reset password form validates token and updates password
- Reset tokens expire after 1 hour for security

## Technical Notes

**NextAuth.js v5 Configuration:**
```typescript
// auth.config.ts
import { NextAuthConfig } from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import Google from 'next-auth/providers/google';
import GitHub from 'next-auth/providers/github';

export const authConfig = {
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID,
      clientSecret: process.env.GITHUB_CLIENT_SECRET,
    }),
    Credentials({
      async authorize(credentials) {
        const user = await verifyCredentials(
          credentials.email,
          credentials.password
        );
        return user || null;
      },
    }),
  ],
  session: { strategy: 'jwt' },
  pages: {
    signIn: '/login',
    error: '/auth/error',
  },
  callbacks: {
    jwt({ token, user }) {
      if (user) {
        token.id = user.id;
        token.role = user.role;
      }
      return token;
    },
    session({ session, token }) {
      session.user.id = token.id;
      session.user.role = token.role;
      return session;
    },
  },
};
```

**Password Hashing:**
```typescript
import bcrypt from 'bcryptjs';

export async function hashPassword(password: string) {
  return bcrypt.hash(password, 12); // 12 rounds
}

export async function verifyPassword(password: string, hash: string) {
  return bcrypt.compare(password, hash);
}
```

**Middleware for Protected Routes:**
```typescript
// middleware.ts
import { withAuth } from 'next-auth/middleware';

export default withAuth({
  callbacks: {
    authorized({ token, req }) {
      const isAuthPage = req.nextUrl.pathname.startsWith('/login');
      const isProtectedRoute = req.nextUrl.pathname.startsWith('/account');

      if (isProtectedRoute && !token) {
        return false;
      }
      return true;
    },
  },
});

export const config = {
  matcher: ['/account/:path*', '/checkout'],
};
```

**Password Reset Token:**
```typescript
import crypto from 'crypto';

export function generateResetToken() {
  return crypto.randomBytes(32).toString('hex');
}

export function getTokenExpiry() {
  return new Date(Date.now() + 60 * 60 * 1000); // 1 hour from now
}
```

**Form Validation Schema:**
```typescript
import { z } from 'zod';

const registerSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Must contain uppercase letter')
    .regex(/[a-z]/, 'Must contain lowercase letter')
    .regex(/[0-9]/, 'Must contain number'),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword'],
});
```

**Edge Cases:**
- Handle OAuth account already exists with different provider
- Handle email already registered (show clear error)
- Handle expired reset tokens (show "Token expired" message)
- Handle network errors during OAuth flow (retry logic)
- Handle session expiry during active use (refresh token)
- Handle password reset for non-existent email (don't reveal if email exists)
- Handle concurrent login sessions (allow multiple devices)

## Tasks

- [x] Install NextAuth.js v5 and required dependencies
- [x] Create auth.config.ts with provider configuration
- [x] Set up Google OAuth app and configure credentials
- [x] Set up GitHub OAuth app and configure credentials
- [x] Create database schema for User, Account, Session tables
- [x] Implement Credentials provider with bcrypt password verification
- [x] Build registration form with email/password fields
- [x] Add client-side form validation with Zod schema
- [x] Create registration API endpoint with user creation
- [x] Build login form with email/password fields
- [x] Implement OAuth buttons for Google and GitHub
- [x] Configure JWT session strategy
- [x] Create auth middleware for protected routes
- [x] Implement logout functionality clearing session
- [x] Build forgot password form and email sending
- [x] Create reset password page with token validation
- [x] Add password reset token expiry logic
- [x] Test OAuth flows with Google and GitHub
- [x] Add accessibility attributes to all forms
- [x] Create error pages for auth failures

## Dependencies

- Story 1-3: Development Environment (requires database schema)

## Estimation

**Story Points:** 8

**Rationale:** NextAuth.js v5 setup is well-documented but requires careful configuration. OAuth provider setup requires external app creation. Password hashing and reset flow add complexity. Form validation and error handling increase scope. Middleware configuration for protected routes requires testing. Overall substantial feature with moderate complexity.
