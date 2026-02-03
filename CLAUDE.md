# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

S-ACC-Books - A shared family accounting book system for tracking personal and shared expenses and incomes. Family members log daily expenses/incomes via mobile app, and the backend aggregates data for both personal and ledger-wide financial views. Users can access multiple ledgers (e.g., personal, family, roommates).

**Components:**
- Frontend: Android app using Jetpack Compose (initial release)
- Backend: REST API services
- Database: SQL-based persistence

## Repository Status

This is a greenfield project in planning phase. No code has been implemented yet. Technology stack decisions are still being finalized.

## Planned Architecture

### Backend

**Proposed Database Schema:**
- `users` - User accounts (standalone, can access multiple ledgers)
- `ledgers` - Shared accounting books (replaces groups)
- `ledger_members` - Many-to-many relationship with roles (owner, admin, editor, viewer)
- `transactions` - Income/expense records with categories
- `budgets` - Spending limits (optional)

**Key Design Considerations:**
- JWT-based authentication with refresh tokens
- Role-based authorization per ledger (owner/admin/editor/viewer)
- Data isolation: users only see ledgers they have access to
- Multi-currency support for international families
- Users can belong to multiple ledgers simultaneously

**Technology Stack Options (not yet decided):**
- Spring Boot (Java/Kotlin) with Spring Data JPA and PostgreSQL
- Node.js + Express with Sequelize/TypeORM and PostgreSQL
- FastAPI (Python) with SQLAlchemy and PostgreSQL

### Frontend (Android)

**Planned Technology Stack:**
- UI: Jetpack Compose
- Architecture: MVVM pattern
- DI: Hilt or Koin
- Network: Retrofit + OkHttp
- Local Storage: Room database for offline support
- Concurrency: Kotlin Coroutines + Flow
- Navigation: Jetpack Navigation Component

**State Management Pattern:**
```
UI Layer (Compose) → ViewModel → Repository → Remote/Local Data Sources
```

**Core Screens:**
- Authentication (login/register)
- Dashboard (balance summary, recent transactions, ledger selector)
- Transaction management (CRUD operations)
- Reports/analytics (charts, breakdowns)
- Ledger management (members, roles, invites)
- Settings/profile

### Development Phases

**Phase 1 (MVP):** Basic auth, transaction CRUD, personal history
**Phase 2:** Ledger features, multi-user access, and aggregation
**Phase 3:** Analytics, reports, and visualizations
**Phase 4:** Budget limits, push notifications, offline support
**Phase 5:** Multi-currency, recurring transactions, iOS app

## Open Design Questions

- **Ledger Invitations:** Email, invite codes, or QR codes?
- **Data Ownership:** What happens to user data when leaving a ledger?
- **Privacy Model:** Individual transactions visible to ledger members or only aggregated data?
- **Categories:** Predefined list vs fully customizable?
- **Recurring Transactions:** Support for subscriptions, salary, rent?
