# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

S-ACC-Books - A shared family accounting book system for tracking personal and shared expenses and incomes. Family members log daily expenses/incomes via the web app (MVP) or Android app (Phase 2), and the backend aggregates data for both personal and ledger-wide financial views. Users can access multiple ledgers (e.g., personal, family, roommates).

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
- `users` - User accounts (no direct ledger FK; a user may belong to multiple ledgers; soft-delete via `deleted_at`)
- `ledgers` - Shared accounting books
- `ledger_members` - Join table linking users to ledgers with per-ledger role (`owner` | `admin` | `editor` | `viewer`) and `joined_at`
- `refresh_tokens` - Revocable, DB-backed refresh tokens (rotated on every refresh)
- `transactions` - Income/expense records with categories
- `budgets` - Spending limits (optional)

**Key Design Considerations:**
- JWT-based authentication with refresh tokens
- Role-based authorization is per-ledger via `ledger_members.role` (`owner` | `admin` | `editor` | `viewer`), not a global user role
- Data isolation: users only see data for ledgers they are members of
- Multi-currency support for international families (ledger-level currency in MVP)
- Users can belong to multiple ledgers simultaneously

**Technology Stack:**
- Spring Boot 3.x (Java 21) with Spring Data JPA
- MySQL 8.x
- Flyway for database migrations
- Spring Security for authentication (JWT)
- **Password hashing:** bcrypt (`BCryptPasswordEncoder`)
- **Refresh tokens:** DB-backed `refresh_tokens` table (revocable on logout via `revoked_at`)
- **Money columns:** `DECIMAL(15,2)` (up to 9,999,999,999,999.99 with cent precision; `BigDecimal` in Java)
- **Email/SMTP:** none in MVP — password-reset emails and ledger-invite emails are both Phase 3

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
- Authentication (login/register, create-or-join ledger flow after registration)
- Dashboard (balance summary, recent transactions, ledger selector)
- Transaction management (CRUD operations)
- Reports/analytics (charts, breakdowns)
- Ledger management (members, roles, invites)
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

**Phase 1 (MVP — Web):** Backend + responsive web app — auth, single-user ledger creation, transaction CRUD, basic filtering, personal dashboard
**Phase 2 (Android):** Android app reaching feature parity with web MVP
**Phase 3 (Ledger features):** Multi-member ledgers, invitations, role-based permissions (owner/admin/editor/viewer), shared/personal visibility, ledger dashboard
**Phase 4 (Analytics & Reports):** Charts, category breakdowns, custom date ranges, CSV/PDF export
**Phase 5 (Advanced):** Budget limits + notifications, multi-currency conversion, recurring transactions, receipt uploads, account self-delete
**Phase 6 (Polish & Launch):** Hosting target chosen; performance, security audit, accessibility review, production deploy, Play Store release
**Future:** iOS app, bank integrations, AI categorization

## Open Design Questions

- **Ledger Invitations:** Email, invite codes, or QR codes?
- **Data Ownership:** What happens to user data when leaving a ledger?
- **Privacy Model:** Individual transactions visible to ledger members or only aggregated data?
- **Categories:** Predefined list vs fully customizable?
- **Recurring Transactions:** Support for subscriptions, salary, rent?
