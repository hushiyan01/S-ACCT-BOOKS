# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

S-ACC-Books - A shared family accounting book system for tracking personal and group expenses, incomes, and asset changes. Family members log daily expenses/incomes via the web app (MVP) or Android app (Phase 2), and the backend aggregates data for both personal and family-wide financial views.

**Components:**
- Frontend: Responsive web app (initial MVP release)
- Mobile: Android app using Jetpack Compose (Phase 2)
- Backend: REST API services
- Database: SQL-based persistence

## Repository Status

This is a greenfield project in planning phase. No code has been implemented yet. Technology stack decisions are still being finalized.

## Planned Architecture

### Backend

**Proposed Database Schema:**
- `groups` - Family/group accounts
- `users` - User accounts (no direct group FK; a user may belong to multiple groups)
- `group_memberships` - Join table linking users to groups with per-group role (admin, member) and `joined_at`
- `transactions` - Income/expense records with categories
- `budgets` - Spending limits (optional)
- `assets` - Savings, investments tracking (optional)

**Key Design Considerations:**
- JWT-based authentication with refresh tokens
- Role-based authorization is per-group via `group_memberships.role`, not a global user role
- Data isolation: users only see data for groups they are members of
- Multi-currency support for international families (group-level currency in MVP)

**Technology Stack:**
- Spring Boot 3.x (Java 21) with Spring Data JPA
- MySQL 8.x
- Flyway for database migrations
- Spring Security for authentication (JWT)
- **Password hashing:** bcrypt (`BCryptPasswordEncoder`)
- **Refresh tokens:** DB-backed `refresh_tokens` table (revocable on logout via `revoked_at`)
- **Money columns:** `DECIMAL(15,2)` (up to 9,999,999,999,999.99 with cent precision; `BigDecimal` in Java)
- **Email/SMTP:** none in MVP — password-reset emails and group-invite emails are both Phase 3

### Frontend — Web (MVP)

**Planned Technology Stack:**
- Framework: React + Vite with TypeScript
- UI: Material UI (Material Design 3)
- Routing: React Router
- Network: Axios with JWT auth interceptors
- Forms/Validation: React Hook Form + Zod
- State: React Query for server state, local component state for UI
- PWA: manifest + service worker for installability

**State Management Pattern:**
```
UI Layer (React components) → Hook / React Query → API Client (Axios) → REST Backend
```

**Core Screens:**
- Authentication (login/register, create-or-join group flow after registration)
- Dashboard (balance summary, recent transactions)
- Transaction management (CRUD operations)
- Reports/analytics (charts, breakdowns)
- Group management (members, invites)
- Settings/profile

### Frontend — Android (Phase 2)

**Planned Technology Stack:**
- UI: Jetpack Compose
- Architecture: MVVM pattern
- DI: Hilt or Koin
- Network: Retrofit + OkHttp
- Local Storage: Room database for offline support
- Concurrency: Kotlin Coroutines + Flow
- Navigation: Jetpack Navigation Component

### Development Phases

**Phase 1 (MVP — Web):** Backend + responsive web app — auth, single-user group creation, transaction CRUD, basic filtering, personal dashboard
**Phase 2 (Android):** Android app reaching feature parity with web MVP
**Phase 3 (Group features):** Multi-member groups, invitations, shared/personal visibility, group dashboard
**Phase 4 (Analytics & Reports):** Charts, category breakdowns, custom date ranges, CSV/PDF export
**Phase 5 (Advanced):** Budget limits + notifications, multi-currency conversion, recurring transactions, receipt uploads
**Phase 6 (Polish & Launch):** Performance, security audit, accessibility review, production deploy, Play Store release
**Future:** iOS app, bank integrations, AI categorization

## Open Design Questions

- **Group Invitations:** Email, invite codes, or QR codes?
- **Data Ownership:** What happens to user data when leaving a group?
- **Privacy Model:** Individual transactions visible to group or only aggregated data?
- **Categories:** Predefined list vs fully customizable?
- **Recurring Transactions:** Support for subscriptions, salary, rent?
