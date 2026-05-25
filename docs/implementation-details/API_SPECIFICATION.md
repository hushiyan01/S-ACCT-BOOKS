# API Specification - S-ACCT-BOOKS

**Version:** 1.0.0-draft
**Base URL:** `https://api.s-acct-books.com/v1`
**Protocol:** REST
**Auth:** JWT Bearer Token

## Overview

This document defines the RESTful API for S-ACCT-BOOKS backend services. The API follows REST principles with JSON payloads and standard HTTP methods.

### Data Model Note: Users ↔ Groups

A user can belong to multiple groups. Membership is modeled by a `group_memberships` join table (`user_id`, `group_id`, `role`, `joined_at`) with a unique constraint on `(user_id, group_id)`. **All group-scoped endpoints** (transactions, analytics, group reads, member admin actions) must verify the requester has a matching `group_memberships` row before responding. Role (`admin` vs `member`) is per-group, not a global property of the user.

### Data Model Note: Money

All monetary fields (`amount`, `limit_amount`, `value`, analytics totals) are persisted as `DECIMAL(15,2)` in MySQL and exposed in JSON as numbers with up to two decimal places. Server-side they are `BigDecimal`. Range: up to 9,999,999,999,999.99 with cent precision. Currency is a separate `CHAR(3)` ISO 4217 column; mixed currencies within one group are not auto-converted in MVP (single currency per group enforced at write time).

### Tech Stack

- **Backend:** Spring Boot 3.x (Java 21) + Spring Data JPA + Spring Security
- **Database:** MySQL 8.x with Flyway migrations
- **Auth:** JWT (access) + DB-backed refresh tokens (`refresh_tokens` table); passwords hashed with bcrypt
- **Validation:** see [`VALIDATION.md`](./VALIDATION.md) for the canonical per-field rules used by both BE and FE

### Note: Password Reset & Email

There is no email service in MVP. Endpoints for password reset (`/auth/password-reset-request`, `/auth/password-reset-confirm`) and email-based group invites are deferred to **Phase 3**, when an email provider is selected. Until then, password recovery is a manual operator action against the DB.

## Authentication

### Authentication Flow

1. User registers or logs in → receives access token + refresh token
2. Access token included in `Authorization: Bearer <token>` header for protected routes
3. Access token expires after 15 minutes
4. Refresh token used to get new access token (expires after 7 days)

### Token Refresh Flow

When access token expires (401 response), client should:
1. Call `/auth/refresh` with refresh token
2. Receive new access token
3. Retry original request with new token

If refresh token is invalid/expired → redirect to login

## Common Response Formats

### Success Response
```json
{
  "success": true,
  "data": { /* response data */ },
  "message": "Operation successful"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": { /* optional error details */ }
  }
}
```

### Pagination Response
```json
{
  "success": true,
  "data": [ /* array of items */ ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 150,
    "totalPages": 8
  }
}
```

## API Endpoints

---

## 1. Authentication & Authorization

### POST `/auth/register`

Register a new user account.

**Request Body:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "SecurePassword123"
}
```

**Response:** `201 Created`
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user_123",
      "name": "John Doe",
      "email": "john@example.com",
      "createdAt": "2026-01-20T10:00:00Z"
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expiresIn": 900
    }
  }
}
```

**Errors:**
- `400` - Validation error (weak password, invalid email)
- `409` - Email already registered

---

### POST `/auth/login`

Authenticate existing user.

**Request Body:**
```json
{
  "email": "john@example.com",
  "password": "SecurePassword123"
}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user_123",
      "name": "John Doe",
      "email": "john@example.com",
      "groups": [
        {
          "id": "group_456",
          "name": "Family Budget",
          "role": "admin"
        }
      ]
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expiresIn": 900
    }
  }
}
```

**Errors:**
- `401 AUTH_INVALID_CREDENTIALS` — returned for both unknown email and wrong password (do not leak which one failed)

---

### POST `/auth/refresh`

Refresh access token using refresh token.

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 900
  }
}
```

**Errors:**
- `401` - Invalid or expired refresh token

---

### POST `/auth/logout`

Revoke the refresh token. Server-side, this sets `revoked_at = now()` on the matching row in `refresh_tokens` (the row is looked up by SHA-256 hash of the supplied token). The corresponding access token is left to expire naturally (≤ 15 min).

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

**Notes:**
- Idempotent: revoking an already-revoked or unknown token still returns 200 (no info-leak).
- The refresh endpoint validates `revoked_at IS NULL AND expires_at > now()`. On every successful refresh, the old refresh token is rotated (its `revoked_at` is set) and a new one is issued.

---

## 2. Groups

### POST `/groups`

Create a new group. Inserts a new row into `groups` and a `group_memberships` row for the creator with `role=admin`.

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "name": "Family Budget",
  "currency": "USD"
}
```

**Response:** `201 Created`
```json
{
  "success": true,
  "data": {
    "id": "group_456",
    "name": "Family Budget",
    "currency": "USD",
    "createdBy": "user_123",
    "members": [
      {
        "userId": "user_123",
        "name": "John Doe",
        "email": "john@example.com",
        "role": "admin",
        "joinedAt": "2026-01-20T10:00:00Z"
      }
    ],
    "createdAt": "2026-01-20T10:00:00Z"
  }
}
```

---

### GET `/groups/:groupId`

Get group details.

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "group_456",
    "name": "Family Budget",
    "currency": "USD",
    "createdBy": "user_123",
    "memberCount": 4,
    "totalBalance": 5420.50,
    "members": [
      {
        "userId": "user_123",
        "name": "John Doe",
        "email": "john@example.com",
        "role": "admin",
        "joinedAt": "2026-01-20T10:00:00Z"
      }
    ],
    "createdAt": "2026-01-20T10:00:00Z"
  }
}
```

**Errors:**
- `403` - User not member of this group
- `404` - Group not found

---

### PATCH `/groups/:groupId`

Update group details (admin only).

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "name": "Updated Family Budget",
  "currency": "EUR"
}
```

**Response:** `200 OK`

**Errors:**
- `403` - User not admin of this group

---

### DELETE `/groups/:groupId`

Delete group (admin only). All transactions are deleted.

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`

**Errors:**
- `403` - User not admin of this group

---

### GET `/groups/:groupId/members`

List all group members. The list is read from `group_memberships` joined with `users`. `transactionCount` and `totalSpent` exclude other members' `visibility=personal` transactions — i.e., the viewer sees their own personal totals fully, but only `shared` totals for other members.

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "userId": "user_123",
      "name": "John Doe",
      "email": "john@example.com",
      "role": "admin",
      "transactionCount": 45,
      "totalSpent": 1250.50,
      "joinedAt": "2026-01-20T10:00:00Z"
    },
    {
      "userId": "user_456",
      "name": "Jane Doe",
      "email": "jane@example.com",
      "role": "member",
      "transactionCount": 32,
      "totalSpent": 890.25,
      "joinedAt": "2026-01-21T14:30:00Z"
    }
  ]
}
```

---

### POST `/groups/:groupId/invite`

Invite a member to group (admin only).

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "email": "newmember@example.com",
  "role": "member"
}
```

**Response:** `201 Created`
```json
{
  "success": true,
  "data": {
    "invitationId": "inv_789",
    "email": "newmember@example.com",
    "inviteCode": "ABC123XYZ",
    "expiresAt": "2026-01-27T10:00:00Z",
    "status": "pending"
  }
}
```

---

### POST `/groups/join`

Join a group using invite code.

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "inviteCode": "ABC123XYZ"
}
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "group": {
      "id": "group_456",
      "name": "Family Budget",
      "role": "member"
    }
  }
}
```

**Errors:**
- `400` - Invalid or expired invite code
- `409` - User already member of group

---

### PATCH `/groups/:groupId/members/:userId`

Change a member's role within the group (admin only). Updates the `role` column on the corresponding `group_memberships` row. Cannot demote the last admin.

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "role": "admin"
}
```

`role`: `"admin"` | `"member"`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "userId": "user_456",
    "role": "admin"
  }
}
```

**Errors:**
- `403 AUTH_INSUFFICIENT_PERMISSIONS` - Requester is not admin of this group
- `404 NOT_FOUND` - Target user is not a member of this group
- `409 LAST_ADMIN` - Refused: would leave the group with zero admins

---

### DELETE `/groups/:groupId/members/:userId`

Remove a member from a group (admin only, cannot remove self). Deletes the matching `group_memberships` row. Refuses if it would leave the group without an admin.

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`

**Errors:**
- `403 AUTH_INSUFFICIENT_PERMISSIONS` - Not authorized or trying to remove self (`CANNOT_REMOVE_SELF`)
- `404 NOT_FOUND` - Member not found
- `409 LAST_ADMIN` - Refused: would leave the group with zero admins

---

## 3. Transactions

### POST `/transactions`

Create a new transaction.

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "groupId": "group_456",
  "type": "expense",
  "amount": 45.99,
  "category": "food",
  "description": "Grocery shopping at Whole Foods",
  "date": "2026-01-20",
  "visibility": "shared"
}
```

**Fields:**
- `type`: `"expense"` | `"income"`
- `amount`: positive decimal with at most 2 fractional digits; stored as `DECIMAL(15,2)`
- `category`: one of the 10 predefined category slugs (see `GET /categories`)
- `visibility`: `"personal"` | `"shared"`
- `date`: ISO 8601 **date** (`YYYY-MM-DD`) — no time-of-day. Defaults to today (server UTC date).

**Response:** `201 Created`
```json
{
  "success": true,
  "data": {
    "id": "txn_789",
    "groupId": "group_456",
    "userId": "user_123",
    "type": "expense",
    "amount": 45.99,
    "category": "food",
    "description": "Grocery shopping at Whole Foods",
    "date": "2026-01-20",
    "visibility": "shared",
    "createdAt": "2026-01-20T18:35:00Z",
    "updatedAt": "2026-01-20T18:35:00Z"
  }
}
```

**Errors:**
- `400` - Validation error (invalid amount, category, etc.)
- `403` - User not member of group

---

### GET `/transactions`

List transactions with filtering and pagination.

**Headers:** `Authorization: Bearer <accessToken>`

**Query Parameters:**
- `groupId` (required): Filter by group
- `type`: "expense" | "income" | "all" (default: "all")
- `category`: Filter by category (comma-separated for multiple)
- `visibility`: "personal" | "shared" | "all" (default: "all")
- `startDate`: ISO 8601 date (inclusive)
- `endDate`: ISO 8601 date (inclusive)
- `userId`: Filter by specific user (only if admin or same user)
- `search`: Search in description
- `page`: Page number (default: 1)
- `pageSize`: Items per page (default: 20, max: 100)
- `sortBy`: "date" | "amount" | "category" (default: "date")
- `sortOrder`: "asc" | "desc" (default: "desc")

**Example Request:**
```
GET /transactions?groupId=group_456&type=expense&startDate=2026-01-01&page=1&pageSize=20
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "id": "txn_789",
      "groupId": "group_456",
      "userId": "user_123",
      "userName": "John Doe",
      "type": "expense",
      "amount": 45.99,
      "category": "food",
      "description": "Grocery shopping",
      "date": "2026-01-20",
      "visibility": "shared",
      "createdAt": "2026-01-20T18:35:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 150,
    "totalPages": 8
  }
}
```

---

### GET `/transactions/:transactionId`

Get single transaction details.

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "txn_789",
    "groupId": "group_456",
    "userId": "user_123",
    "userName": "John Doe",
    "type": "expense",
    "amount": 45.99,
    "category": "food",
    "description": "Grocery shopping at Whole Foods",
    "date": "2026-01-20",
    "visibility": "shared",
    "createdAt": "2026-01-20T18:35:00Z",
    "updatedAt": "2026-01-20T18:35:00Z"
  }
}
```

**Errors:**
- `403` - Transaction not accessible (personal transaction of another user)
- `404` - Transaction not found

---

### PATCH `/transactions/:transactionId`

Update transaction (only own transactions).

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:** (all fields optional)
```json
{
  "type": "expense",
  "amount": 50.00,
  "category": "food",
  "description": "Updated description",
  "date": "2026-01-20",
  "visibility": "personal"
}
```

**Response:** `200 OK` (returns updated transaction)

**Errors:**
- `403` - Not the owner of this transaction
- `404` - Transaction not found

---

### DELETE `/transactions/:transactionId`

Delete transaction (only own transactions).

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`
```json
{
  "success": true,
  "message": "Transaction deleted successfully"
}
```

**Errors:**
- `403` - Not the owner of this transaction
- `404` - Transaction not found

---

## 4. Categories

### GET `/categories`

Get list of available transaction categories.

**Headers:** `Authorization: Bearer <accessToken>` (optional for public categories)

**Response:** `200 OK`

The list is static (same for all users in MVP) — backed by a Java/Kotlin enum, not a `categories` table.

```json
{
  "success": true,
  "data": [
    { "id": "food",          "name": "Food & Dining",     "icon": "🍔", "color": "#FF6B6B", "type": "expense" },
    { "id": "transport",     "name": "Transportation",    "icon": "🚗", "color": "#4ECDC4", "type": "expense" },
    { "id": "shopping",      "name": "Shopping",          "icon": "🛍️", "color": "#FFA94D", "type": "expense" },
    { "id": "entertainment", "name": "Entertainment",     "icon": "🎬", "color": "#A29BFE", "type": "expense" },
    { "id": "healthcare",    "name": "Healthcare",        "icon": "⚕️", "color": "#FF7675", "type": "expense" },
    { "id": "education",     "name": "Education",         "icon": "📚", "color": "#74B9FF", "type": "expense" },
    { "id": "bills",         "name": "Bills & Utilities", "icon": "💡", "color": "#FDCB6E", "type": "expense" },
    { "id": "salary",        "name": "Salary",            "icon": "💼", "color": "#4CAF50", "type": "income"  },
    { "id": "investment",    "name": "Investment",        "icon": "📈", "color": "#00B894", "type": "both"    },
    { "id": "other",         "name": "Other",             "icon": "•••", "color": "#95A5A6", "type": "both"    }
  ]
}
```

---

## 5. Analytics & Reports

### GET `/analytics/summary`

Get financial summary for a period.

**Opening balance computation:**
`opening` is the sum of all the in-scope transactions whose `transaction_date < startDate` (income contributes positively, expense negatively). `closing = opening + change`. `changePercent = (change / opening) * 100`, **omitted from the response when `opening` is 0** to avoid divide-by-zero — clients should treat its absence as "no comparable prior balance."

**Headers:** `Authorization: Bearer <accessToken>`

**Query Parameters:**
- `groupId` (required): Group ID. Requester must have a `group_memberships` row for this group, otherwise `403`.
- `scope`: "personal" | "group" (default: "personal")
- `startDate`: ISO 8601 date
- `endDate`: ISO 8601 date
- `userId`: Specific user (if scope=personal)

**Visibility rules:**
- `scope=personal`, viewer = subject: all transactions included.
- `scope=personal`, viewer ≠ subject: only `visibility=shared` transactions of the subject are included.
- `scope=group`: all `visibility=shared` transactions in the group are included, plus the viewer's own personal ones.

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "period": {
      "startDate": "2026-01-01T00:00:00Z",
      "endDate": "2026-01-31T23:59:59Z"
    },
    "balance": {
      "opening": 1000.00,
      "closing": 2450.50,
      "change": 1450.50,
      "changePercent": 145.05
    },
    "income": {
      "total": 3000.00,
      "count": 2,
      "average": 1500.00
    },
    "expenses": {
      "total": 1549.50,
      "count": 45,
      "average": 34.43
    },
    "topCategories": [
      {
        "category": "food",
        "amount": 450.00,
        "count": 15,
        "percentage": 29.03
      },
      {
        "category": "transport",
        "amount": 320.50,
        "count": 10,
        "percentage": 20.68
      }
    ]
  }
}
```

---

### GET `/analytics/trends`

Get spending/income trends over time.

**Headers:** `Authorization: Bearer <accessToken>`

**Query Parameters:**
- `groupId` (required)
- `scope`: "personal" | "group"
- `startDate`: ISO 8601 date
- `endDate`: ISO 8601 date
- `interval`: "day" | "week" | "month" (default: "day")
- `category`: Filter by category (optional)

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "interval": "day",
    "dataPoints": [
      {
        "date": "2026-01-01",
        "income": 0,
        "expenses": 45.50,
        "balance": -45.50
      },
      {
        "date": "2026-01-02",
        "income": 0,
        "expenses": 120.00,
        "balance": -165.50
      }
    ]
  }
}
```

---

### GET `/analytics/category-breakdown`

Get breakdown of spending by category.

**Headers:** `Authorization: Bearer <accessToken>`

**Query Parameters:**
- `groupId` (required)
- `scope`: "personal" | "group"
- `startDate`: ISO 8601 date
- `endDate`: ISO 8601 date
- `type`: "expense" | "income" | "all"

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "total": 1549.50,
    "categories": [
      {
        "category": "food",
        "categoryName": "Food & Dining",
        "amount": 450.00,
        "count": 15,
        "percentage": 29.03,
        "color": "#FF6B6B"
      },
      {
        "category": "transport",
        "categoryName": "Transportation",
        "amount": 320.50,
        "count": 10,
        "percentage": 20.68,
        "color": "#4ECDC4"
      }
    ]
  }
}
```

---

## 6. User Profile

### GET `/users/me`

Get current user profile.

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "user_123",
    "name": "John Doe",
    "email": "john@example.com",
    "groups": [
      {
        "id": "group_456",
        "name": "Family Budget",
        "role": "admin",
        "memberCount": 4
      }
    ],
    "createdAt": "2026-01-01T10:00:00Z"
  }
}
```

---

### PATCH `/users/me`

Update current user profile.

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "name": "John Smith",
  "email": "john.smith@example.com"
}
```

**Response:** `200 OK` (returns updated user)

**Errors:**
- `409` - Email already in use

---

### POST `/users/me/change-password`

Change user password.

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "currentPassword": "OldPassword123",
  "newPassword": "NewSecurePassword456"
}
```

**Response:** `200 OK`

**Errors:**
- `401` - Current password incorrect
- `400` - New password doesn't meet requirements

---

### DELETE `/users/me`

> **Phase 5+ — not in MVP.** Until shipped, account closure is a manual operator action that sets `users.deleted_at` directly. The contract below is documented now so the eventual implementation is consistent.

Soft-delete the current user's account. Sets `users.deleted_at = now()`. The user's `group_memberships`, `transactions`, and `assets` rows are preserved with attribution — in the UI, their name is shown as **"(deleted user)"** alongside their preserved `name` value (clients may choose to display either). Login is blocked while `deleted_at IS NOT NULL`. All issued refresh tokens are revoked.

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "password": "Password123",
  "confirmation": "DELETE MY ACCOUNT"
}
```

**Response:** `200 OK`

---

## Error Codes

### Authentication & Authorization
- `AUTH_REQUIRED` - Authentication required
- `AUTH_INVALID_TOKEN` - Invalid or malformed token
- `AUTH_TOKEN_EXPIRED` - Access token expired (use refresh)
- `AUTH_INVALID_CREDENTIALS` - Wrong email/password
- `AUTH_INSUFFICIENT_PERMISSIONS` - User lacks required permissions

### Validation
- `VALIDATION_ERROR` - Input validation failed
- `INVALID_EMAIL` - Email format invalid
- `WEAK_PASSWORD` - Password doesn't meet requirements
- `INVALID_AMOUNT` - Transaction amount invalid
- `INVALID_DATE` - Date format or value invalid
- `INVALID_CATEGORY` - Category doesn't exist

### Resources
- `NOT_FOUND` - Resource not found
- `ALREADY_EXISTS` - Resource already exists (e.g., duplicate email)
- `NOT_MEMBER` - User not member of group
- `NOT_OWNER` - User doesn't own this resource

### Business Logic
- `GROUP_LIMIT_REACHED` - User has reached max groups
- `INVITATION_EXPIRED` - Invite code expired
- `CANNOT_REMOVE_SELF` - Admin cannot remove themselves
- `LAST_ADMIN` - Cannot remove last admin from group

### Server
- `INTERNAL_ERROR` - Unexpected server error
- `SERVICE_UNAVAILABLE` - Service temporarily unavailable

---

## Rate Limiting

- **Authentication endpoints:** 5 requests per minute per IP
- **Write operations (POST, PATCH, DELETE):** 60 requests per minute per user
- **Read operations (GET):** 120 requests per minute per user

**Rate limit headers included in response:**
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1642687200
```

When rate limit exceeded: `429 Too Many Requests`

---

## Versioning

API version is included in the URL: `/v1/...`

Breaking changes will increment major version: `/v2/...`

---

## Webhooks (Future - Phase 5)

Allow clients to subscribe to events:
- `transaction.created`
- `transaction.updated`
- `transaction.deleted`
- `group.member_added`
- `group.member_removed`
- `budget.limit_reached`

---

## Testing the API

### Postman Collection
A Postman collection will be provided with example requests for all endpoints.

### Authentication Flow for Testing
1. Register a user: `POST /auth/register`
2. Save the `accessToken` from response
3. Use token in `Authorization: Bearer <token>` header for all subsequent requests

### Sample Test Data
- Email: `test@example.com`
- Password: `TestPassword123`
- Group name: `Test Family`

---

**Next Steps:**
1. Review and finalize API specification
2. Set up OpenAPI/Swagger documentation
3. Implement backend with this spec
4. Generate API client libraries for web/mobile
