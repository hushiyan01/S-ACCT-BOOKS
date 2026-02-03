# API Specification - S-ACCT-BOOKS

**Version:** 1.0.0-draft
**Base URL:** `https://api.s-acct-books.com/v1`
**Protocol:** REST
**Auth:** JWT Bearer Token

## Overview

This document defines the RESTful API for S-ACCT-BOOKS backend services. The API follows REST principles with JSON payloads and standard HTTP methods.

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
      "ledgers": [
        {
          "id": "ledger_456",
          "name": "Family Budget",
          "role": "owner"
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
- `401` - Invalid credentials
- `404` - User not found

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

Invalidate refresh token (optional - can also handle client-side only).

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

---

## 2. Ledgers

### POST `/ledgers`

Create a new ledger.

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
    "id": "ledger_456",
    "name": "Family Budget",
    "currency": "USD",
    "createdBy": "user_123",
    "members": [
      {
        "userId": "user_123",
        "name": "John Doe",
        "email": "john@example.com",
        "role": "owner",
        "joinedAt": "2026-01-20T10:00:00Z"
      }
    ],
    "createdAt": "2026-01-20T10:00:00Z"
  }
}
```

---

### GET `/ledgers`

Get all ledgers the user has access to.

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "id": "ledger_456",
      "name": "Family Budget",
      "currency": "USD",
      "role": "owner",
      "memberCount": 4
    },
    {
      "id": "ledger_789",
      "name": "Roommates",
      "currency": "USD",
      "role": "editor",
      "memberCount": 3
    }
  ]
}
```

---

### GET `/ledgers/:ledgerId`

Get ledger details.

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`
```json
{
  "success": true,
  "data": {
    "id": "ledger_456",
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
        "role": "owner",
        "joinedAt": "2026-01-20T10:00:00Z"
      }
    ],
    "createdAt": "2026-01-20T10:00:00Z"
  }
}
```

**Errors:**
- `403` - User not member of this ledger
- `404` - Ledger not found

---

### PATCH `/ledgers/:ledgerId`

Update ledger details (admin/owner only).

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
- `403` - User not admin/owner of this ledger

---

### DELETE `/ledgers/:ledgerId`

Delete ledger (owner only). All transactions are deleted.

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`

**Errors:**
- `403` - User not owner of this ledger

---

### GET `/ledgers/:ledgerId/members`

List all ledger members.

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
      "role": "owner",
      "transactionCount": 45,
      "totalSpent": 1250.50,
      "joinedAt": "2026-01-20T10:00:00Z"
    },
    {
      "userId": "user_456",
      "name": "Jane Doe",
      "email": "jane@example.com",
      "role": "editor",
      "transactionCount": 32,
      "totalSpent": 890.25,
      "joinedAt": "2026-01-21T14:30:00Z"
    }
  ]
}
```

**Role values:** `owner`, `admin`, `editor`, `viewer`

---

### POST `/ledgers/:ledgerId/invite`

Invite a member to ledger (admin/owner only).

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "email": "newmember@example.com",
  "role": "editor"
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

### POST `/ledgers/join`

Join a ledger using invite code.

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
    "ledger": {
      "id": "ledger_456",
      "name": "Family Budget",
      "role": "editor"
    }
  }
}
```

**Errors:**
- `400` - Invalid or expired invite code
- `409` - User already member of ledger

---

### PATCH `/ledgers/:ledgerId/members/:userId`

Update member role (admin/owner only).

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "role": "admin"
}
```

**Response:** `200 OK`

**Errors:**
- `403` - Not authorized or trying to change owner role
- `404` - Member not found

---

### DELETE `/ledgers/:ledgerId/members/:userId`

Remove member from ledger (admin/owner only, cannot remove owner).

**Headers:** `Authorization: Bearer <accessToken>`

**Response:** `200 OK`

**Errors:**
- `403` - Not authorized or trying to remove owner
- `404` - Member not found

---

## 3. Transactions

### POST `/transactions`

Create a new transaction.

**Headers:** `Authorization: Bearer <accessToken>`

**Request Body:**
```json
{
  "ledgerId": "ledger_456",
  "type": "expense",
  "amount": 45.99,
  "category": "food",
  "description": "Grocery shopping at Whole Foods",
  "date": "2026-01-20T18:30:00Z",
  "visibility": "shared"
}
```

**Fields:**
- `type`: "expense" | "income"
- `amount`: positive number (decimal)
- `category`: string (predefined categories)
- `visibility`: "personal" | "shared"
- `date`: ISO 8601 timestamp (defaults to now)

**Response:** `201 Created`
```json
{
  "success": true,
  "data": {
    "id": "txn_789",
    "ledgerId": "ledger_456",
    "userId": "user_123",
    "type": "expense",
    "amount": 45.99,
    "category": "food",
    "description": "Grocery shopping at Whole Foods",
    "date": "2026-01-20T18:30:00Z",
    "visibility": "shared",
    "createdAt": "2026-01-20T18:35:00Z",
    "updatedAt": "2026-01-20T18:35:00Z"
  }
}
```

**Errors:**
- `400` - Validation error (invalid amount, category, etc.)
- `403` - User not member of ledger

---

### GET `/transactions`

List transactions with filtering and pagination.

**Headers:** `Authorization: Bearer <accessToken>`

**Query Parameters:**
- `ledgerId` (required): Filter by ledger
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
GET /transactions?ledgerId=ledger_456&type=expense&startDate=2026-01-01&page=1&pageSize=20
```

**Response:** `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "id": "txn_789",
      "ledgerId": "ledger_456",
      "userId": "user_123",
      "userName": "John Doe",
      "type": "expense",
      "amount": 45.99,
      "category": "food",
      "description": "Grocery shopping",
      "date": "2026-01-20T18:30:00Z",
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
    "ledgerId": "ledger_456",
    "userId": "user_123",
    "userName": "John Doe",
    "type": "expense",
    "amount": 45.99,
    "category": "food",
    "description": "Grocery shopping at Whole Foods",
    "date": "2026-01-20T18:30:00Z",
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
  "date": "2026-01-20T18:30:00Z",
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
```json
{
  "success": true,
  "data": [
    {
      "id": "cat_food",
      "name": "Food & Dining",
      "icon": "🍔",
      "color": "#FF6B6B",
      "type": "expense"
    },
    {
      "id": "cat_transport",
      "name": "Transportation",
      "icon": "🚗",
      "color": "#4ECDC4",
      "type": "expense"
    },
    {
      "id": "cat_salary",
      "name": "Salary",
      "icon": "💼",
      "color": "#4CAF50",
      "type": "income"
    }
  ]
}
```

---

## 5. Analytics & Reports

### GET `/analytics/summary`

Get financial summary for a period.

**Headers:** `Authorization: Bearer <accessToken>`

**Query Parameters:**
- `ledgerId` (required): Ledger ID
- `scope`: "personal" | "ledger" (default: "personal")
- `startDate`: ISO 8601 date
- `endDate`: ISO 8601 date
- `userId`: Specific user (if scope=personal)

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
- `ledgerId` (required)
- `scope`: "personal" | "ledger"
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
- `ledgerId` (required)
- `scope`: "personal" | "ledger"
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
    "ledgers": [
      {
        "id": "ledger_456",
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

Delete user account (soft delete - mark as deleted).

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
- `NOT_MEMBER` - User not member of ledger
- `NOT_OWNER` - User doesn't own this resource

### Business Logic
- `LEDGER_LIMIT_REACHED` - User has reached max ledgers
- `INVITATION_EXPIRED` - Invite code expired
- `CANNOT_REMOVE_SELF` - Admin cannot remove themselves
- `LAST_OWNER` - Cannot remove owner from ledger

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
- `ledger.member_added`
- `ledger.member_removed`
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
- Ledger name: `Test Family`

---

**Next Steps:**
1. Review and finalize API specification
2. Set up OpenAPI/Swagger documentation
3. Implement backend with this spec
4. Generate API client libraries for web/mobile
