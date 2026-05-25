# Implementation Roadmap

This document outlines the step-by-step implementation plan for S-ACCT-BOOKS.

## Pre-Implementation (Current Phase)

- [x] Initial project concept and architecture planning
- [x] MVP platform decided: **Web first** (React + Vite + Material UI). Android in Phase 2.
- [x] Shared design system approach defined (Material Design 3 tokens shared across web and Android)
- [x] **Product definition and user flows** — see `docs/project-ideas/PRODUCT_SPEC.md`
- [x] API specification document — see `docs/implementation-details/API_SPECIFICATION.md`
- [x] Database schema finalized (includes `group_memberships` join table)
- [x] Backend stack chosen: **Spring Boot 3.x + MySQL 8.x + Flyway**. Money columns: `DECIMAL(15,2)`.
- [x] Design approach decided: **Claude-driven** — UI is built directly in React + MUI code (no Figma deliverable). See `docs/implementation-details/CLAUDE_DESIGN_GUIDE.md`.

## Phase 1: MVP - Backend + Web (Est. 4-6 weeks)

### Setup & Infrastructure
- [ ] Set up monorepo structure (consider: Nx, Turborepo, or separate repos)
- [ ] Initialize design tokens repository/package
- [ ] Set up CI/CD pipeline (GitHub Actions)
- [ ] Configure development environments

### Design Tokens
- [ ] Define color palette (primary, secondary, semantic colors)
- [ ] Define typography scale
- [ ] Define spacing system
- [ ] Define border radius and elevation levels
- [ ] Generate CSS variables for web
- [ ] Generate Kotlin constants for Android

### Backend Development
- [ ] Initialize Spring Boot 3.x project (Java 21) with Spring Web, Spring Data JPA, Spring Security, Validation, Flyway, MySQL Connector
- [ ] Set up MySQL 8.x (local via Docker; staging on chosen host)
- [ ] Configure Testcontainers for integration tests against a real MySQL instance
- [ ] Implement database schema with Flyway migrations, including:
  - [ ] `users` table (`name` + `email` + `password_hash` + `deleted_at`; no `username` column, no direct group FK)
  - [ ] `groups` table
  - [ ] `refresh_tokens` table (`token_hash`, `expires_at`, `revoked_at`, `last_used_at`)
  - [ ] `group_memberships` join table with `(user_id, group_id)` unique constraint and indexes on both FKs
  - [ ] `transactions` table with `visibility`, `transaction_date` as `DATE`, `amount` as `DECIMAL(15,2)`, `currency` as `CHAR(3)`, and `category` as an enum (10 predefined values)
- [ ] Define `TransactionCategory` enum (10 values) and `GET /categories` controller — no `categories` table needed in MVP
- [ ] Create authentication system: bcrypt password hashing, JWT access tokens (15 min), DB-backed refresh tokens with rotation (logout sets `revoked_at`)
- [ ] Login blocked when `users.deleted_at IS NOT NULL`
- [ ] Implement user registration and login endpoints (login returns the same generic 401 for invalid email or wrong password)
- [ ] Implement "create group" endpoint that also inserts the creator into `group_memberships` as `admin`
- [ ] Implement transaction CRUD endpoints with membership-based authorization (verify `(user_id, group_id)` in `group_memberships` before any read/write)
- [ ] Serve predefined categories from a static list (full category management is Phase 5)
- [ ] Add input validation and error handling
- [ ] Write unit and integration tests
- [ ] Set up API documentation (Swagger/OpenAPI)
- [ ] Deploy to staging environment

### Web Frontend Development
- [ ] Initialize React + Vite project with TypeScript
- [ ] Set up Material UI and theme configuration
- [ ] Configure routing (React Router)
- [ ] Set up Axios with auth interceptors
- [ ] Implement authentication flow (login/register/logout)
- [ ] Create protected route wrapper
- [ ] Build Dashboard page (balance summary, recent transactions)
- [ ] Build Transaction List page with filtering
- [ ] Build Transaction Create/Edit forms
- [ ] Implement form validation with Zod
- [ ] Build Settings/Profile page
- [ ] Add loading states and error handling
- [ ] Implement responsive design (mobile, tablet, desktop)
- [ ] Write component tests
- [ ] Set up PWA manifest and service workers
- [ ] Deploy to hosting platform (Vercel/Netlify)

## Phase 2: Android App (Est. 3-4 weeks)

### Setup
- [ ] Initialize Android project with Kotlin
- [ ] Set up Jetpack Compose
- [ ] Configure Hilt for dependency injection
- [ ] Set up Retrofit and OkHttp
- [ ] Configure Room database

### Core Features
- [ ] Implement authentication screens (login/register)
- [ ] Set up navigation component
- [ ] Build Dashboard screen
- [ ] Build Transaction List screen
- [ ] Build Transaction Create/Edit screens
- [ ] Implement offline-first architecture
- [ ] Add data synchronization logic
- [ ] Build Settings screen
- [ ] Implement biometric authentication
- [ ] Add home screen widget
- [ ] Write UI and unit tests
- [ ] Deploy to internal testing track

## Phase 3: Group Features (Est. 3-4 weeks)

### Backend
- [ ] **Choose and integrate an email provider** (deferred from MVP — required before invitations + password reset can ship)
- [ ] Implement password reset flow:
  - [ ] `POST /auth/password-reset-request` (issues a token, emails it)
  - [ ] `POST /auth/password-reset-confirm` (validates token, updates `password_hash`)
- [ ] Implement multi-member group management (group creation already exists from MVP; extend with member-level endpoints)
- [ ] Implement group invitation system (email + invite code; QR is just a render of the same code)
  - [ ] `POST /groups/:groupId/invite` — issues invite code, optionally emails it
  - [ ] `POST /groups/join` — accepts code, inserts `group_memberships` row
- [ ] Role management endpoints:
  - [ ] `PATCH /groups/:groupId/members/:userId` — change role (admin only, refuse if it would leave zero admins)
  - [ ] `DELETE /groups/:groupId/members/:userId` — remove member (admin only, same last-admin guard)
- [ ] Add aggregated group data endpoints (exclude transactions where `visibility=personal` and `user_id != viewer`)
- [ ] Update transaction visibility logic
- [ ] Update `/users/me` and login responses to include the user's full membership list

### Frontend (Web + Android)
- [ ] Build group creation flow
- [ ] Build group invitation system
- [ ] Build group member management UI
- [ ] Build shared group dashboard
- [ ] Add group selector in navigation
- [ ] Implement group-specific transaction filtering
- [ ] Add permission checks in UI
- [ ] Update both web and Android apps

## Phase 4: Analytics & Reports (Est. 2-3 weeks)

- [ ] Design analytics data models
- [ ] Implement backend analytics endpoints
- [ ] Integrate chart library (Recharts for web, MPAndroidChart for Android)
- [ ] Build spending breakdown charts (pie, bar, line)
- [ ] Build category-based insights
- [ ] Build custom date range reports
- [ ] Implement export functionality (PDF, CSV)
- [ ] Add trend analysis and predictions
- [ ] Update both platforms

## Phase 5: Advanced Features (Est. 3-4 weeks)

- [ ] Implement budget limits with notifications
- [ ] Add multi-currency support
- [ ] Build recurring transactions system
- [ ] Add receipt photo upload and storage (S3/Cloud Storage)
- [ ] Implement OCR for receipt scanning (optional)
- [ ] Add notification system (push for mobile, email for web)
- [ ] Build transaction search with advanced filters
- [ ] Add data export and backup features

## Phase 6: Polish & Launch (Est. 2 weeks)

- [ ] **Choose hosting target** (deferred earlier — pick from Fly.io / Render / AWS / GCP / self-hosted)
- [ ] Performance optimization
- [ ] Security audit
- [ ] Accessibility compliance review
- [ ] User acceptance testing
- [ ] Bug fixes and refinements
- [ ] Prepare marketing materials
- [ ] Write user documentation
- [ ] Deploy to production
- [ ] Publish Android app to Play Store
- [ ] Set up monitoring and analytics

## Future Enhancements

- [ ] iOS app development
- [ ] Split transaction feature (shared expenses)
- [ ] Bill reminders and due dates
- [ ] Integration with banks/financial services
- [ ] AI-powered categorization
- [ ] Custom tags and labels
- [ ] Shared notes on transactions
- [ ] Financial goals tracking
- [ ] Investment portfolio tracking
