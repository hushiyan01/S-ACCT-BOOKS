# S-ACCT-BOOKS

A shared family accounting book system for tracking personal and shared expenses and incomes across multiple ledgers.

## 📋 Project Status

**Current Phase:** Planning & Design
**Last Updated:** 2026-01-20

This is a greenfield project currently in the planning phase. No code has been implemented yet.

## 📖 Documentation Index

This repository contains comprehensive planning documentation to help you understand and implement S-ACCT-BOOKS.

### Core Documentation

1. **[CLAUDE.md](./CLAUDE.md)** - Project overview and architecture
   - High-level project description
   - Technology stack decisions (React web + Android native)
   - Architecture patterns for backend, web, and mobile
   - Development phases and open questions

#### Project Ideas & Specifications

2. **[PRODUCT_SPEC.md](./docs/project-ideas/PRODUCT_SPEC.md)** - Detailed product specification
   - User personas and stories
   - Feature prioritization (Must/Should/Could/Won't have)
   - Complete user flows with diagrams
   - Screen-by-screen specifications
   - Success metrics and KPIs

3. **[brain_storm.md](./docs/project-ideas/brain_storm.md)** - Initial brainstorming notes
   - Original project ideas and architecture concepts
   - Database schema proposals
   - Technology stack considerations

#### Implementation Details

4. **[API_SPECIFICATION.md](./docs/implementation-details/API_SPECIFICATION.md)** - RESTful API documentation
   - Complete endpoint definitions
   - Request/response formats
   - Authentication flow
   - Error codes and handling
   - Rate limiting and versioning

5. **[FIGMA_DESIGN_GUIDE.md](./docs/implementation-details/FIGMA_DESIGN_GUIDE.md)** - Design system and Figma instructions
   - Design system setup (colors, typography, spacing)
   - Component library specifications
   - Screen design guidelines
   - Figma file structure and organization
   - Recommended plugins and resources

#### Roadmaps

6. **[IMPLEMENTATION_ROADMAP.md](./docs/roadmaps/IMPLEMENTATION_ROADMAP.md)** - Step-by-step implementation plan
   - Pre-implementation checklist
   - Phase-by-phase breakdown with tasks
   - Estimated timelines for each phase
   - Future enhancement ideas

## 🎯 Project Vision

S-ACCT-BOOKS enables families and small groups to collaboratively track finances while maintaining individual privacy. Users can log daily expenses and income via web or mobile, view personal financial history, and access aggregated ledger insights. Users can belong to multiple ledgers (e.g., personal, family, roommates).

**Target Users:** Families, roommates, couples, or any small group sharing expenses

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────┐
│          Frontend Applications          │
├──────────────────┬──────────────────────┤
│   Web (React)    │  Android (Compose)   │
│  - TypeScript    │  - Kotlin           │
│  - Material UI   │  - Jetpack Compose  │
│  - React Router  │  - MVVM Pattern     │
│  - PWA Support   │  - Offline-first    │
└──────────┬───────┴──────────┬───────────┘
           │                  │
           └────────┬─────────┘
                    │
                    ↓
         ┌──────────────────────┐
         │     REST API         │
         │  (Backend Service)   │
         │  - JWT Auth          │
         │  - Spring Boot /     │
         │    Node.js / FastAPI │
         └──────────┬───────────┘
                    │
                    ↓
         ┌──────────────────────┐
         │    PostgreSQL DB     │
         │  - Users             │
         │  - Ledgers           │
         │  - Ledger_Members    │
         │  - Transactions      │
         │  - Budgets           │
         └──────────────────────┘

         ┌──────────────────────┐
         │  Shared Design System│
         │  - Design Tokens     │
         │  - JSON Format       │
         │  - Generated Code    │
         └──────────────────────┘
```

## 🚀 Getting Started

### For New Contributors

1. **Read the documentation in this order:**
   - Start with [CLAUDE.md](./CLAUDE.md) for project overview
   - Read [PRODUCT_SPEC.md](./docs/project-ideas/PRODUCT_SPEC.md) to understand features and user flows
   - Review [API_SPECIFICATION.md](./docs/implementation-details/API_SPECIFICATION.md) if working on backend
   - Check [IMPLEMENTATION_ROADMAP.md](./docs/roadmaps/IMPLEMENTATION_ROADMAP.md) for current progress

2. **For Designers:**
   - Read [PRODUCT_SPEC.md](./docs/project-ideas/PRODUCT_SPEC.md) - Screen Specifications section
   - Follow [FIGMA_DESIGN_GUIDE.md](./docs/implementation-details/FIGMA_DESIGN_GUIDE.md) to create designs
   - Use the design system specifications provided

3. **For Backend Developers:**
   - Review [API_SPECIFICATION.md](./docs/implementation-details/API_SPECIFICATION.md)
   - Choose technology stack from options in [CLAUDE.md](./CLAUDE.md)
   - Follow Phase 1 backend tasks in [IMPLEMENTATION_ROADMAP.md](./docs/roadmaps/IMPLEMENTATION_ROADMAP.md)

4. **For Frontend Developers (Web):**
   - Review React architecture in [CLAUDE.md](./CLAUDE.md)
   - Check API endpoints in [API_SPECIFICATION.md](./docs/implementation-details/API_SPECIFICATION.md)
   - Follow Phase 1 web tasks in [IMPLEMENTATION_ROADMAP.md](./docs/roadmaps/IMPLEMENTATION_ROADMAP.md)

5. **For Frontend Developers (Android):**
   - Review Android architecture in [CLAUDE.md](./CLAUDE.md)
   - Follow Phase 2 Android tasks in [IMPLEMENTATION_ROADMAP.md](./docs/roadmaps/IMPLEMENTATION_ROADMAP.md)

### Prerequisites (Once Implementation Starts)

- **Backend:** JDK 17+ / Node.js 18+ / Python 3.10+ (depending on choice)
- **Web:** Node.js 18+, npm/yarn/pnpm
- **Android:** Android Studio, JDK 17+
- **Database:** PostgreSQL 14+
- **Design:** Figma account

## 📁 Planned Repository Structure

Once implementation begins, the repository will be organized as:

```
S-ACCT-BOOKS/
├── docs/                    # Documentation (current files)
├── design-tokens/           # Shared design system
│   ├── tokens.json
│   └── generators/
├── backend/                 # Backend service
│   ├── src/
│   ├── tests/
│   └── README.md
├── web/                     # React web app
│   ├── src/
│   ├── public/
│   └── package.json
├── android/                 # Android app
│   ├── app/
│   ├── gradle/
│   └── build.gradle
└── README.md               # This file
```

## 🎨 Design Resources

**Design will be created using Figma following the guidelines in [FIGMA_DESIGN_GUIDE.md](./docs/implementation-details/FIGMA_DESIGN_GUIDE.md)**

Once designs are ready, links will be added here:
- [ ] Figma Design System (TBD)
- [ ] Figma Wireframes (TBD)
- [ ] Figma High-Fidelity Designs (TBD)
- [ ] Interactive Prototype (TBD)

## 🔑 Key Features

### Phase 1 (MVP)
- User authentication (JWT-based)
- Ledger creation and management
- Transaction logging (expenses & income)
- Personal transaction history with filtering
- Dashboard with balance and recent transactions
- Responsive web interface

### Phase 2
- Android mobile app
- Offline-first architecture
- Ledger invitation system with roles
- Multi-user data aggregation

### Phase 3+
- Analytics and visual reports
- Budget limits with alerts
- Multi-currency support
- Receipt photo uploads
- Recurring transactions

## 🤝 Contributing

Once implementation begins:

1. Choose a task from [IMPLEMENTATION_ROADMAP.md](./docs/roadmaps/IMPLEMENTATION_ROADMAP.md)
2. Create a feature branch: `git checkout -b feature/your-feature-name`
3. Follow coding standards (TBD)
4. Write tests
5. Submit a pull request

## 📜 License

TBD - To be decided by project owner

## 📞 Contact & Support

TBD - Add project owner contact information

---

## Quick Reference

### Important Decisions Made

✅ **Frontend:** React (web) + Android Native (mobile) with full feature parity
✅ **Design System:** Shared design tokens across platforms, Material Design 3 foundation
✅ **Backend Options:** Spring Boot, Node.js, or FastAPI (choice pending)
✅ **Database:** PostgreSQL
✅ **Authentication:** JWT with refresh tokens
✅ **Development Approach:** Web MVP first, then Android, then advanced features

### Open Questions

❓ **Privacy Model:** Ledger admins see all transactions or only "shared" ones?
❓ **Ledger Invitations:** Email, invite codes, or QR codes? (or all three?)
❓ **Categories:** Fixed list or user-customizable?
❓ **Backend Technology:** Which framework to use?
❓ **Monetization:** Free tier only or freemium model?

See [PRODUCT_SPEC.md](./docs/project-ideas/PRODUCT_SPEC.md) for complete list of open questions.

---

**Next Steps:**
1. ✅ Complete planning documentation
2. ⏳ Create Figma designs (use [FIGMA_DESIGN_GUIDE.md](./docs/implementation-details/FIGMA_DESIGN_GUIDE.md))
3. ⏳ Finalize open product questions
4. ⏳ Choose backend technology stack
5. ⏳ Set up development environment
6. ⏳ Begin Phase 1 implementation
