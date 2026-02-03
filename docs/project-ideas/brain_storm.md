# S-ACC-Books - Family Accounting System

## Project Overview

A shared family accounting book system for tracking personal and group expenses and incomes.

**Components:**
- Frontend: Android app (initial release)
- Backend: REST API services
- Database: SQL-based persistence

**Core Use Case:** Family members log their daily expenses/incomes via mobile app. Backend aggregates data to provide both personal and family-wide financial views.

---

## Backend Architecture

### Database Schema

**Proposed Tables:**

1. **`users`**
   - `id` (PK)
   - `username`
   - `email`
   - `password_hash`
   - `created_at`

2. **`ledgers`** (shared accounting books)
   - `id` (PK)
   - `name`
   - `currency` (default: USD)
   - `created_by` (FK to users)
   - `created_at`
   - `updated_at`

3. **`ledger_members`** (many-to-many: users ↔ ledgers)
   - `id` (PK)
   - `ledger_id` (FK to ledgers)
   - `user_id` (FK to users)
   - `role` (owner, admin, editor, viewer)
   - `joined_at`

4. **`transactions`**
   - `id` (PK)
   - `user_id` (FK to users)
   - `ledger_id` (FK to ledgers)
   - `type` (income, expense)
   - `category` (food, transport, salary, etc.)
   - `amount`
   - `currency` (default: USD)
   - `description`
   - `transaction_date`
   - `created_at`
   - `updated_at`

5. **`budgets`** (optional for expense limits)
   - `id` (PK)
   - `user_id` (FK to users, nullable for ledger-wide budgets)
   - `ledger_id` (FK to ledgers)
   - `category`
   - `limit_amount`
   - `period` (daily, weekly, monthly)
   - `created_at`

**Role Permissions:**
- **owner**: Full control, can delete ledger, manage all members
- **admin**: Can invite/remove members (except owner), edit ledger settings
- **editor**: Can add/edit/delete transactions
- **viewer**: Read-only access to ledger data

### API Endpoints (Ideas)

**Authentication:**
- `POST /api/auth/register` - User registration
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

### Technology Stack (Backend Ideas)

**Option 1 - Spring Boot (Java/Kotlin):**
- Spring Boot 3.x
- Spring Data JPA
- PostgreSQL or MySQL
- Spring Security for auth
- JWT for token-based authentication

**Option 2 - Node.js + Express:**
- Express.js
- Sequelize ORM or TypeORM
- PostgreSQL
- Passport.js for auth

**Option 3 - FastAPI (Python):**
- FastAPI
- SQLAlchemy ORM
- PostgreSQL
- JWT authentication

### Additional Backend Considerations

- **Authentication:** JWT-based with refresh tokens
- **Authorization:** Role-based (owner/admin/editor/viewer per ledger)
- **Validation:** Input validation on all endpoints
- **Error Handling:** Standardized error responses
- **Logging:** Request/response logging for debugging
- **Rate Limiting:** Prevent abuse
- **Data Privacy:** Users can only see ledgers they have access to

---

## Frontend Architecture (Android)

### Technology Stack

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

### Phase 1: MVP (Minimum Viable Product)
- User registration/login
- Basic transaction CRUD (create, read, update, delete)
- Personal transaction history
- Simple dashboard with total income/expense

### Phase 2: Ledger Features
- Ledger creation and joining
- Multi-user ledger access with roles
- Ledger transaction aggregation
- Ledger dashboard/reports

### Phase 3: Analytics & Reporting
- Category-based breakdown
- Monthly/yearly reports
- Charts and visualizations

### Phase 4: Advanced Features
- Budget limits and alerts
- Push notifications
- Offline support
- Export functionality

### Phase 5: Enhancements
- Multi-currency support
- Recurring transactions
- Receipt photo attachments
- iOS app version

---

## Questions to Consider

1. **Ledger Invitations:** How should ledger invitations work? Via email, invite code, or QR code?
2. **Data Ownership:** Can users leave a ledger? What happens to their transaction history?
3. **Privacy:** Should ledger members see each other's individual transactions or just aggregated data?
4. **Currency:** Support multiple currencies? Auto-conversion?
5. **Recurring Transactions:** Support for subscriptions, salary, rent?
6. **Categories:** Predefined list vs fully customizable?
7. **Deployment:** Cloud hosting (AWS, GCP, Azure) or self-hosted option?

---

## Next Steps

1. Finalize database schema
2. Choose backend technology stack
3. Set up development environment
4. Create API documentation (OpenAPI/Swagger)
5. Design UI mockups/wireframes
6. Set up Git workflow and project structure