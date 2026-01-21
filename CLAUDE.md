# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

S-ACC-Books - A shared family accounting book system for tracking personal and group expenses, incomes, and asset changes. Family members log daily expenses/incomes via mobile app, and the backend aggregates data for both personal and family-wide financial views.

**Components:**
<<<<<<< HEAD
- Frontend: Android app using Jetpack Compose (initial release)
- Backend: REST API services
- Database: SQL-based persistence
=======
- Frontend:
  - Web app using React (primary interface)
  - Android app using Jetpack Compose
- Backend: REST API services
- Database: SQL-based persistence
- Shared Design System: Design tokens for consistent UI across platforms
>>>>>>> e9265ab (1. Add project docs folder)

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

<<<<<<< HEAD
=======
### Shared Design System

**Design Tokens:**
- Colors (primary, secondary, semantic colors for income/expense/warning)
- Typography (font families, sizes, weights)
- Spacing scale (margins, padding)
- Border radius values
- Shadow/elevation levels

**Implementation:**
- Define tokens in JSON format at repository root
- Generate platform-specific code (CSS variables for web, Kotlin constants for Android)
- Use Material Design 3 as foundation for consistency

### Frontend (Web)

**Technology Stack:**
- Framework: React 18+ with TypeScript
- UI Library: Material UI (MUI) v5 for Material Design 3 components
- State Management: React Context API + useReducer (or Zustand for complex state)
- HTTP Client: Axios with interceptors for auth
- Routing: React Router v6
- Forms: React Hook Form + Zod validation
- Charts: Recharts or Chart.js
- Build Tool: Vite
- Testing: Vitest + React Testing Library

**Architecture Pattern:**
```
UI Components (React) → Custom Hooks → Services Layer → API Client
                     ↓
                State Management (Context/Zustand)
```

**Project Structure:**
```
src/
├── components/        # Reusable UI components
├── pages/            # Route-level components
├── hooks/            # Custom React hooks
├── services/         # Business logic and API calls
├── context/          # Global state management
├── utils/            # Helper functions
├── types/            # TypeScript interfaces
├── styles/           # Global styles and theme
└── config/           # App configuration
```

**Core Features:**
- Responsive design (mobile-first approach)
- PWA capabilities (service workers, offline support)
- Real-time updates via WebSocket or polling
- Lazy loading and code splitting
- Accessibility compliance (WCAG 2.1)

>>>>>>> e9265ab (1. Add project docs folder)
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

<<<<<<< HEAD
**Core Screens:**
=======
**Core Screens (shared across Web and Android):**
>>>>>>> e9265ab (1. Add project docs folder)
- Authentication (login/register with group selection)
- Dashboard (balance summary, recent transactions)
- Transaction management (CRUD operations)
- Reports/analytics (charts, breakdowns)
- Group management (members, invites)
- Settings/profile

<<<<<<< HEAD
### Development Phases

**Phase 1 (MVP):** Basic auth, transaction CRUD, personal history
**Phase 2:** Group features and aggregation
**Phase 3:** Analytics, reports, and visualizations
**Phase 4:** Budget limits, push notifications, offline support
**Phase 5:** Multi-currency, recurring transactions, iOS app
=======
**Android-Specific Features:**
- Home screen widget for quick expense entry
- Biometric authentication (fingerprint/face)
- Camera integration for receipt scanning
- Location tagging for transactions

### Development Phases

**Phase 1 (MVP - Backend + Web):**
- Backend API with auth and transaction CRUD
- Web app with login, dashboard, and transaction management
- Basic personal history and filtering

**Phase 2 (Android App):**
- Android app with feature parity to web
- Offline-first architecture with Room database
- Push notifications for transaction reminders

**Phase 3 (Group Features):**
- Group creation and member invitations
- Multi-user data aggregation
- Role-based permissions (admin vs member)
- Shared group dashboard

**Phase 4 (Analytics & Reports):**
- Visual charts and spending breakdowns
- Custom date range reports
- Category-based insights
- Export functionality (PDF, CSV)

**Phase 5 (Advanced Features):**
- Budget limits with notifications
- Multi-currency support
- Recurring transactions
- Receipt photo attachments
- iOS app (optional)
>>>>>>> e9265ab (1. Add project docs folder)

## Open Design Questions

- **Group Invitations:** Email, invite codes, or QR codes?
- **Data Ownership:** What happens to user data when leaving a group?
- **Privacy Model:** Individual transactions visible to group or only aggregated data?
- **Categories:** Predefined list vs fully customizable?
- **Recurring Transactions:** Support for subscriptions, salary, rent?
