# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

S-ACC-Books - A shared family accounting book system for tracking personal and group expenses, incomes, and asset changes. Family members log daily expenses/incomes via mobile app, and the backend aggregates data for both personal and family-wide financial views.

**Components:**
- Frontend: Android app using Jetpack Compose (initial release)
- Backend: REST API services
- Database: SQL-based persistence

## Repository Status

This is a greenfield project in planning phase. No code has been implemented yet. Technology stack decisions are still being finalized.

## Planned Architecture

### Backend

**Proposed Database Schema:**
- `groups` - Family/group accounts
- `users` - Group members with roles (admin, member)
- `transactions` - Income/expense records with categories
- `budgets` - Spending limits (optional)
- `assets` - Savings, investments tracking (optional)

**Key Design Considerations:**
- JWT-based authentication with refresh tokens
- Role-based authorization (group admin vs member)
- Data isolation: users only see their group's data
- Multi-currency support for international families

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
- Authentication (login/register with group selection)
- Dashboard (balance summary, recent transactions)
- Transaction management (CRUD operations)
- Reports/analytics (charts, breakdowns)
- Group management (members, invites)
- Settings/profile

### Development Phases

**Phase 1 (MVP):** Basic auth, transaction CRUD, personal history
**Phase 2:** Group features and aggregation
**Phase 3:** Analytics, reports, and visualizations
**Phase 4:** Budget limits, push notifications, offline support
**Phase 5:** Multi-currency, recurring transactions, iOS app

## Open Design Questions

- **Group Invitations:** Email, invite codes, or QR codes?
- **Data Ownership:** What happens to user data when leaving a group?
- **Privacy Model:** Individual transactions visible to group or only aggregated data?
- **Categories:** Predefined list vs fully customizable?
- **Recurring Transactions:** Support for subscriptions, salary, rent?
