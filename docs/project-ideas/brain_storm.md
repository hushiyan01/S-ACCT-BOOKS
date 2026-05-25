# S-ACC-Books - Family Accounting System

## Project Overview

A shared family accounting book system for tracking personal and group expenses, incomes, and asset changes.

**Components:**
- Frontend: Responsive web app (initial MVP release)
- Mobile: Android app using Jetpack Compose (Phase 2)
- Backend: REST API services
- Database: SQL-based persistence

**Core Use Case:** Family members log their daily expenses/incomes via the web app (MVP) or, later, the Android app. Backend aggregates data to provide both personal and family-wide financial views.

---

## Backend Architecture

### Database Schema

**Proposed Tables:**

1. **`groups`**
   - `id` (PK)
   - `name`
   - `currency` (group default, ISO 4217, e.g. "USD")
   - `created_by` (FK to users — group creator)
   - `created_at`
   - `updated_at`

2. **`users`**
   - `id` (PK)
   - `name` (display name; not unique; shown in UI)
   - `email` (unique, login identifier)
   - `password_hash` (bcrypt)
   - `deleted_at` (nullable `TIMESTAMP` — soft-delete marker; login blocked when set; all issued refresh tokens are revoked at the same time)
   - `created_at`
   - `updated_at`
   - Note: users have NO direct `group_id`. A user can belong to zero or more groups; membership is modeled via `group_memberships`. Login is by `email` only — there is no `username` field. Soft-deleted users retain their `name` so historic transactions stay attributable.

3. **`refresh_tokens`** (auth — issued refresh tokens, revocable)
   - `id` (PK)
   - `user_id` (FK to users)
   - `token_hash` (SHA-256 of the issued token; never store raw tokens)
   - `issued_at`
   - `expires_at`
   - `revoked_at` (nullable — set on logout or password change)
   - `last_used_at` (nullable — updated on each refresh; useful for rotation)
   - **Indexes:** `(user_id)`, `(token_hash)` for refresh validation
   - **Refresh rule:** token is valid iff `revoked_at IS NULL AND expires_at > now()`. Rotation: on every successful refresh, mark the old token's `revoked_at` and issue a fresh one.

4. **`group_memberships`** (join table — users ↔ groups, with per-group role)
   - `id` (PK)
   - `user_id` (FK to users)
   - `group_id` (FK to groups)
   - `role` (`admin` | `member`)
   - `joined_at`
   - **Unique constraint:** `(user_id, group_id)` — a user can only be in a given group once
   - **Indexes:** `(user_id)`, `(group_id)` for membership lookups in both directions

5. **`transactions`**
   - `id` (PK)
   - `user_id` (FK to users — the creator)
   - `group_id` (FK to groups — the group this transaction is recorded under)
   - `type` (`income` | `expense`)
   - `category` (ENUM `VARCHAR(20)` — one of the 10 predefined slugs; see "Predefined Categories" below)
   - `amount` (`DECIMAL(15,2)`, non-null, must be > 0)
   - `currency` (`CHAR(3)`, ISO 4217; defaults to the group's currency; explicit multi-currency conversion is Phase 5)
   - `description` (`VARCHAR(255)`, nullable)
   - `visibility` (`personal` | `shared`) — controls whether other group members see this transaction
   - `transaction_date` (`DATE`, non-null — no time-of-day; one row per day's spend)
   - `created_at` (`TIMESTAMP`)
   - `updated_at` (`TIMESTAMP`)
   - **Authorization rule:** before insert/select, verify `(user_id, group_id)` exists in `group_memberships`.

6. **`budgets`** (optional for expense limits)
   - `id` (PK)
   - `user_id` (FK to users, nullable for group-wide budgets)
   - `group_id` (FK to groups)
   - `category`
   - `limit_amount` (`DECIMAL(15,2)`, non-null, must be > 0)
   - `period` (daily, weekly, monthly)
   - `created_at`

7. **`assets`** (for tracking savings, investments, etc.)
   - `id` (PK)
   - `user_id` (FK to users)
   - `group_id` (FK to groups, nullable — null means personal asset not tied to any group)
   - `asset_type` (savings, investment, property)
   - `name`
   - `value` (`DECIMAL(15,2)`, non-null)
   - `currency` (CHAR(3), ISO 4217)
   - `updated_at`

### Predefined Categories (MVP)

The MVP ships with a fixed enum of 10 categories — no `categories` table, no user-defined categories. Full customization is deferred to Phase 5.

| Slug | Display | Icon | Type | Color |
|---|---|---|---|---|
| `food` | Food & Dining | 🍔 | expense | `#FF6B6B` |
| `transport` | Transportation | 🚗 | expense | `#4ECDC4` |
| `shopping` | Shopping | 🛍️ | expense | `#FFA94D` |
| `entertainment` | Entertainment | 🎬 | expense | `#A29BFE` |
| `healthcare` | Healthcare | ⚕️ | expense | `#FF7675` |
| `education` | Education | 📚 | expense | `#74B9FF` |
| `bills` | Bills & Utilities | 💡 | expense | `#FDCB6E` |
| `salary` | Salary | 💼 | income | `#4CAF50` |
| `investment` | Investment | 📈 | both | `#00B894` |
| `other` | Other | • • • | both | `#95A5A6` |

In Spring Boot this is a Java `enum` persisted with `@Enumerated(EnumType.STRING)`. `GET /categories` returns the list as JSON for the client.

### API Endpoints (Ideas)

**Authentication:**
- `POST /api/auth/register` - User registration (user is created with no group; a separate create-or-join step inserts the first `group_memberships` row)
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout

**Transactions:**
- `POST /api/transactions` - Create new transaction
- `GET /api/transactions?user_id={id}` - Get user's transactions
- `GET /api/transactions?group_id={id}` - Get group's transactions
- `PUT /api/transactions/{id}` - Update transaction
- `DELETE /api/transactions/{id}` - Delete transaction

**Reports/Analytics:**
- `GET /api/reports/user/{user_id}?period={month/year}` - User financial summary
- `GET /api/reports/group/{group_id}?period={month/year}` - Group financial summary
- `GET /api/reports/category-breakdown?user_id={id}` - Expense by category

**Budgets:**
- `POST /api/budgets` - Create budget/limit
- `GET /api/budgets/check/{user_id}` - Check if approaching limit (for notifications)

**Groups:**
- `POST /api/groups` - Create new group (creator inserted into `group_memberships` as `admin`)
- `GET /api/groups/{id}/members` - Get group members (reads from `group_memberships` joined to `users`)
- `POST /api/groups/{id}/invite` - Invite member to group
- `POST /api/groups/join` - Accept an invite code (inserts a `group_memberships` row)
- `PATCH /api/groups/{id}/members/{userId}` - Change role (admin only)
- `DELETE /api/groups/{id}/members/{userId}` - Remove member (admin only; cannot remove last admin)

### Technology Stack (Backend — locked in)

- **Framework:** Spring Boot 3.x (Java 21)
- **ORM:** Spring Data JPA + Hibernate
- **Database:** MySQL 8.x
- **Migrations:** Flyway
- **Auth:** Spring Security with JWT (access + refresh tokens)
- **Password hashing:** bcrypt via `BCryptPasswordEncoder` (Spring Security default)
- **Refresh-token revocation:** DB-backed `refresh_tokens` table — logout sets `revoked_at`; refresh checks both expiry and revocation
- **Money:** all monetary columns are `DECIMAL(15,2)` — range up to 9,999,999,999,999.99 with cent precision; `BigDecimal` in Java
- **API docs:** springdoc-openapi (auto-generates OpenAPI 3 from controllers)
- **Validation:** Jakarta Bean Validation (`@Valid`, `@NotNull`, `@Size`, etc.)
- **Testing:** JUnit 5 + Testcontainers (MySQL container) for integration tests
- **Email:** no email service in MVP — password reset and group-invite emails are deferred to Phase 3 when an email provider is selected

### Additional Backend Considerations

- **Authentication:** JWT-based with refresh tokens
- **Authorization:** Per-group role via `group_memberships.role` (admin vs member). A user's privileges are scoped to each group independently.
- **Validation:** Input validation on all endpoints
- **Error Handling:** Standardized error responses
- **Logging:** Request/response logging for debugging
- **Rate Limiting:** Prevent abuse
- **Data Privacy:** Users can only see data for groups they are members of; `visibility=personal` transactions are hidden from other members and excluded from group aggregates.

---

## Frontend Architecture

The MVP ships as a responsive web app. An Android app (described below) follows in Phase 2.

### Web (MVP) — Technology Stack

- **Framework:** React 18 + Vite with TypeScript
- **UI Library:** Material UI (Material Design 3)
- **Routing:** React Router
- **HTTP Client:** Axios (with JWT auth interceptors and refresh-token retry)
- **Forms & Validation:** React Hook Form + Zod
- **Server State:** React Query (TanStack Query) — caching, pagination, invalidation
- **Charts (Phase 4):** Recharts
- **PWA:** Web App Manifest + service worker for installability and basic offline shell

### Android (Phase 2) — Technology Stack

- **UI:** Jetpack Compose
- **Concurrency:** Kotlin Coroutines + Flow
- **Navigation:** Jetpack Navigation Component with NavGraph
- **Architecture:** MVVM (Model-View-ViewModel)
- **DI:** Hilt or Koin
- **Network:** Retrofit + OkHttp
- **Local Storage:** Room database (for offline support)
- **Notifications:** Firebase Cloud Messaging (FCM)

### Screen Structure (Initial Ideas)

1. **Authentication Screens**
   - Login
   - Register (with group selection/creation)

2. **Main Dashboard**
   - Personal balance summary
   - Recent transactions list
   - Quick add transaction button
   - Budget alerts/warnings

3. **Transaction Management**
   - Add transaction (income/expense)
   - Transaction history (filterable by date, category)
   - Edit/delete transactions

4. **Reports/Analytics**
   - Personal expense breakdown (pie/bar charts)
   - Group expense breakdown
   - Monthly/yearly comparisons
   - Category-wise spending trends

5. **Group Management**
   - View group members
   - Invite new members
   - Group expense overview

6. **Settings/Profile**
   - User profile
   - Budget limit settings
   - Notification preferences
   - Currency settings

### Key Features

- **Offline Support:** Cache transactions locally, sync when online
- **Push Notifications:**
  - Budget limit warnings (80%, 90%, 100%)
  - Daily expense reminders
  - Group activity updates (optional)
- **Categories:** Predefined + custom categories
- **Multi-currency Support:** For international families
- **Export Data:** CSV/PDF export for personal records
- **Search & Filter:** Advanced transaction search

### State Management Pattern

**Web (MVP):**
```
UI Layer (React Components)
    ↓
Custom Hooks (encapsulate domain operations)
    ↓
React Query (server-state cache)
    ↓
API Client (Axios) → REST Backend
```

**Android (Phase 2):**
```
UI Layer (Compose)
    ↓
ViewModel (holds UI state)
    ↓
Repository (business logic + data coordination)
    ↓ ↓
Remote Data Source    Local Data Source
(Retrofit API)        (Room DB)
```

---

## Development Phases (Proposal)

### Phase 1: MVP — Backend + Web (Minimum Viable Product)
- Backend: auth (JWT + refresh), schema with `group_memberships`, transaction CRUD endpoints
- Web: React + Vite + Material UI app
- User registration / login / logout
- Single-user group creation on first login (multi-user invites are Phase 3)
- Transaction CRUD (create, read, update, delete) with predefined categories
- Personal transaction history with basic filtering (date / type / category)
- Dashboard with total income / expense and recent transactions

### Phase 2: Android App
- Native Android app reaching feature parity with the web MVP
- Compose UI, Retrofit networking, Room for offline cache
- Internal testing track

### Phase 3: Group Features (Web + Android)
- Group invitations (email + invite code)
- Multi-member groups with per-group roles (`admin` / `member`) backed by `group_memberships`
- Personal vs shared transaction visibility
- Group dashboard / aggregated views (excluding personal transactions)

### Phase 4: Analytics & Reporting
- Category-based breakdown
- Monthly/yearly reports
- Charts and visualizations (Recharts on web, MPAndroidChart on Android)
- CSV/PDF export

### Phase 5: Advanced Features
- Budget limits and alerts
- Push notifications (mobile) / email notifications (web)
- Multi-currency support with conversion
- Recurring transactions
- Receipt photo attachments

### Phase 6: Polish & Launch
- Performance optimization, security audit, accessibility review
- Production deploy, Play Store release

### Future
- iOS app
- Bank integrations, AI categorization

---

## Questions to Consider

1. **Group Management:** How should group invitations work? Via email, invite code, or QR code?
   _Answered (Phase 3):_ invite code as the primary mechanism, optionally emailed once an email service is selected. QR is just a render of the same code.
2. **Data Ownership:** Can users leave a group? What happens to their transaction history?
   _Partially answered:_ account deletion is a soft-delete (`users.deleted_at`) and preserves transactions/memberships/assets with attribution to the now-deleted user (UI label "(deleted user)"). "Leave a single group while keeping account" is still open — likely a `DELETE /groups/:id/members/me` in Phase 3.
3. **Privacy:** Should group members see each other's individual transactions or just aggregated data?
   _Answered:_ per-transaction `visibility` (`personal` | `shared`). Personal transactions are hidden from other members and excluded from group aggregates.
4. **Currency:** Support multiple currencies? Auto-conversion?
   Single currency per group in MVP; multi-currency + conversion in Phase 5.
5. **Recurring Transactions:** Support for subscriptions, salary, rent?
   Phase 5.
6. **Categories:** Predefined list vs fully customizable?
   _Answered:_ curated 10-category enum in MVP (see "Predefined Categories"); user-defined customization in Phase 5.
7. **Deployment:** Cloud hosting (AWS, GCP, Azure) or self-hosted option?
   _Deferred:_ decide when Phase 1 nears deploy. Develop locally with Docker Compose for MySQL until then.

---

## Next Steps

1. Finalize database schema
2. Choose backend technology stack
3. Set up development environment
4. Create API documentation (OpenAPI/Swagger)
5. Design UI mockups/wireframes
6. Set up Git workflow and project structure