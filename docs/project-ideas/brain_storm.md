# S-ACC-Books - Family Accounting System

## Project Overview

A shared family accounting book system for tracking personal and group expenses, incomes, and asset changes.

**Components:**
- Frontend: Android app (initial release)
- Backend: REST API services
- Database: SQL-based persistence

**Core Use Case:** Family members log their daily expenses/incomes via mobile app. Backend aggregates data to provide both personal and family-wide financial views.

---

## Backend Architecture

### Database Schema

**Proposed Tables:**

1. **`groups`**
   - `id` (PK)
   - `name`
   - `created_at`
   - `updated_at`

2. **`users`**
   - `id` (PK)
   - `group_id` (FK to groups)
   - `username`
   - `email`
   - `password_hash`
   - `role` (admin, member)
   - `created_at`

3. **`transactions`**
   - `id` (PK)
   - `user_id` (FK to users)
   - `group_id` (FK to groups)
   - `type` (income, expense)
   - `category` (food, transport, salary, etc.)
   - `amount`
   - `currency` (default: USD)
   - `description`
   - `transaction_date`
   - `created_at`
   - `updated_at`

4. **`budgets`** (optional for expense limits)
   - `id` (PK)
   - `user_id` (FK to users, nullable for group budgets)
   - `group_id` (FK to groups)
   - `category`
   - `limit_amount`
   - `period` (daily, weekly, monthly)
   - `created_at`

5. **`assets`** (for tracking savings, investments, etc.)
   - `id` (PK)
   - `user_id` (FK to users)
   - `asset_type` (savings, investment, property)
   - `name`
   - `value`
   - `currency`
   - `updated_at`

### API Endpoints (Ideas)

**Authentication:**
- `POST /api/auth/register` - User registration (requires group_id or creates new group)
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
- `POST /api/groups` - Create new group
- `GET /api/groups/{id}/members` - Get group members
- `POST /api/groups/{id}/invite` - Invite member to group

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
- **Authorization:** Role-based (group admin vs member)
- **Validation:** Input validation on all endpoints
- **Error Handling:** Standardized error responses
- **Logging:** Request/response logging for debugging
- **Rate Limiting:** Prevent abuse
- **Data Privacy:** Users can only see their group's data

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

### Phase 2: Group Features
- Group creation and joining
- Group transaction aggregation
- Group dashboard/reports

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

1. **Group Management:** How should group invitations work? Via email, invite code, or QR code?
2. **Data Ownership:** Can users leave a group? What happens to their transaction history?
3. **Privacy:** Should group members see each other's individual transactions or just aggregated data?
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