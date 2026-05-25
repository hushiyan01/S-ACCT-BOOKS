# S-ACC-Books - Family Accounting System

## Project Overview

A shared family accounting book system for tracking personal and shared expenses and incomes across one or more ledgers.

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

1. **`users`**
   - `id` (PK)
   - `name` (display name; not unique; shown in UI)
   - `email` (unique, login identifier)
   - `password_hash` (bcrypt)
   - `deleted_at` (nullable `TIMESTAMP` — soft-delete marker; login blocked when set; all issued refresh tokens are revoked at the same time)
   - `created_at`
   - `updated_at`
   - Note: users have NO direct `ledger_id`. A user can belong to zero or more ledgers; membership is modeled via `ledger_members`. Login is by `email` only — there is no `username` field. Soft-deleted users retain their `name` so historic transactions stay attributable.

2. **`ledgers`** (shared accounting books)
   - `id` (PK)
   - `name`
   - `currency` (ledger default, `CHAR(3)` ISO 4217, e.g. `"USD"`)
   - `created_by` (FK to users — ledger creator; becomes the initial `owner` member)
   - `created_at`
   - `updated_at`

3. **`ledger_members`** (join table — users ↔ ledgers, with per-ledger role)
   - `id` (PK)
   - `ledger_id` (FK to ledgers)
   - `user_id` (FK to users)
   - `role` (`owner` | `admin` | `editor` | `viewer`)
   - `joined_at`
   - **Unique constraint:** `(user_id, ledger_id)` — a user can only be in a given ledger once
   - **Indexes:** `(user_id)`, `(ledger_id)` for membership lookups in both directions

4. **`refresh_tokens`** (auth — issued refresh tokens, revocable)
   - `id` (PK)
   - `user_id` (FK to users)
   - `token_hash` (SHA-256 of the issued token; never store raw tokens)
   - `issued_at`
   - `expires_at`
   - `revoked_at` (nullable — set on logout or password change)
   - `last_used_at` (nullable — updated on each refresh; useful for rotation)
   - **Indexes:** `(user_id)`, `(token_hash)` for refresh validation
   - **Refresh rule:** token is valid iff `revoked_at IS NULL AND expires_at > now()`. Rotation: on every successful refresh, mark the old token's `revoked_at` and issue a fresh one.

5. **`transactions`**
   - `id` (PK)
   - `user_id` (FK to users — the creator)
   - `ledger_id` (FK to ledgers — the ledger this transaction is recorded under)
   - `type` (`income` | `expense`)
   - `category` (ENUM `VARCHAR(20)` — one of the 10 predefined slugs; see "Predefined Categories" below)
   - `amount` (`DECIMAL(15,2)`, non-null, must be > 0)
   - `currency` (`CHAR(3)`, ISO 4217; defaults to the ledger's currency; explicit multi-currency conversion is Phase 5)
   - `description` (`VARCHAR(255)`, nullable)
   - `visibility` (`personal` | `shared`) — controls whether other ledger members see this transaction
   - `transaction_date` (`DATE`, non-null — no time-of-day; one row per day's spend)
   - `created_at` (`TIMESTAMP`)
   - `updated_at` (`TIMESTAMP`)
   - **Authorization rule:** before insert/select, verify `(user_id, ledger_id)` exists in `ledger_members` and that the user's role permits the operation (see Role Permissions below).

6. **`budgets`** (optional for expense limits)
   - `id` (PK)
   - `user_id` (FK to users, nullable for ledger-wide budgets)
   - `ledger_id` (FK to ledgers)
   - `category`
   - `limit_amount` (`DECIMAL(15,2)`, non-null, must be > 0)
   - `period` (daily, weekly, monthly)
   - `created_at`

**Role Permissions** (`ledger_members.role`):
- **`owner`** — Full control. Can delete the ledger, change anyone's role, manage all members. The ledger creator is inserted as `owner` automatically. **Last-owner guard:** role-change and remove-member endpoints refuse if the operation would leave the ledger with zero `owner`s.
- **`admin`** — Can invite members and remove non-owners; can change `editor`/`viewer` roles; can edit ledger settings (name, currency).
- **`editor`** — Can add/edit/delete their own transactions; reads other members' transactions subject to visibility rules.
- **`viewer`** — Read-only access to ledger data (subject to visibility rules); cannot create transactions.

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
- `POST /api/auth/register` - User registration (user row is created with no ledger; a separate create-or-join step inserts the first `ledger_members` row, with `role=owner` on create or `role=editor`/etc. on join)
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout

**Ledgers:**
- `POST /api/ledgers` - Create new ledger
- `GET /api/ledgers` - Get user's ledgers (all ledgers user has access to)
- `GET /api/ledgers/{id}` - Get ledger details
- `PUT /api/ledgers/{id}` - Update ledger (admin/owner only)
- `DELETE /api/ledgers/{id}` - Delete ledger (owner only)

**Ledger Members:**
- `GET /api/ledgers/{id}/members` - Get ledger members
- `POST /api/ledgers/{id}/members` - Add member to ledger (admin/owner only)
- `PUT /api/ledgers/{id}/members/{user_id}` - Update member role (admin/owner only)
- `DELETE /api/ledgers/{id}/members/{user_id}` - Remove member (admin/owner only)
- `POST /api/ledgers/{id}/invite` - Generate invite link/code

**Transactions:**
- `POST /api/transactions` - Create new transaction
- `GET /api/transactions?user_id={id}` - Get user's transactions
- `GET /api/transactions?ledger_id={id}` - Get ledger's transactions
- `PUT /api/transactions/{id}` - Update transaction
- `DELETE /api/transactions/{id}` - Delete transaction

**Reports/Analytics:**
- `GET /api/reports/user/{user_id}?period={month/year}` - User financial summary
- `GET /api/reports/ledger/{ledger_id}?period={month/year}` - Ledger financial summary
- `GET /api/reports/category-breakdown?ledger_id={id}` - Expense by category

**Budgets:**
- `POST /api/budgets` - Create budget/limit
- `GET /api/budgets/check/{user_id}` - Check if approaching limit (for notifications)

> Ledger creation inserts the creator into `ledger_members` as `owner`. `POST /api/ledgers/{id}/join` accepts an invite code and inserts a `ledger_members` row (role determined by the invite). Role change and member removal are subject to the **last-owner guard** — operations refusing to leave the ledger with zero owners.

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
- **Email:** no email service in MVP — password reset and ledger-invite emails are deferred to Phase 3 when an email provider is selected

### Additional Backend Considerations

- **Authentication:** JWT-based with refresh tokens
- **Authorization:** Per-ledger role via `ledger_members.role` (`owner` / `admin` / `editor` / `viewer`). A user's privileges are scoped to each ledger independently.
- **Validation:** Input validation on all endpoints
- **Error Handling:** Standardized error responses
- **Logging:** Request/response logging for debugging
- **Rate Limiting:** Prevent abuse
- **Data Privacy:** Users can only see ledgers they are members of; `visibility=personal` transactions are hidden from other members and excluded from ledger aggregates.

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
   - Register (with ledger creation/joining)

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
   - Ledger expense breakdown
   - Monthly/yearly comparisons
   - Category-wise spending trends

5. **Ledger Management**
   - View ledger members and roles
   - Invite new members
   - Ledger expense overview
   - Switch between ledgers

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
  - Ledger activity updates (optional)
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
- Backend: auth (JWT + refresh), schema with `ledger_members`, transaction CRUD endpoints
- Web: React + Vite + Material UI app
- User registration / login / logout
- Single-user ledger creation on first login (multi-user invites are Phase 3)
- Transaction CRUD (create, read, update, delete) with predefined categories
- Personal transaction history with basic filtering (date / type / category)
- Dashboard with total income / expense and recent transactions

### Phase 2: Android App
- Native Android app reaching feature parity with the web MVP
- Compose UI, Retrofit networking, Room for offline cache
- Internal testing track

### Phase 3: Ledger Features (Web + Android)
- Ledger invitations (email + invite code)
- Multi-member ledgers with per-ledger roles (`owner` / `admin` / `editor` / `viewer`) backed by `ledger_members`
- Personal vs shared transaction visibility
- Ledger dashboard / aggregated views (excluding personal transactions)

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

1. **Ledger Invitations:** How should ledger invitations work? Via email, invite code, or QR code?
   _Answered (Phase 3):_ invite code as the primary mechanism, optionally emailed once an email service is selected. QR is just a render of the same code.
2. **Data Ownership:** Can users leave a ledger? What happens to their transaction history?
   _Partially answered:_ account deletion is a soft-delete (`users.deleted_at`) and preserves transactions/memberships with attribution to the now-deleted user (UI label "(deleted user)"). "Leave a single ledger while keeping account" is still open — likely a `DELETE /ledgers/:id/members/me` in Phase 3.
3. **Privacy:** Should ledger members see each other's individual transactions or just aggregated data?
   _Answered:_ per-transaction `visibility` (`personal` | `shared`). Personal transactions are hidden from other members and excluded from ledger aggregates.
4. **Currency:** Support multiple currencies? Auto-conversion?
   Single currency per ledger in MVP; multi-currency + conversion in Phase 5.
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