# Validation Rules — S-ACCT-BOOKS

Canonical source for input validation rules across the backend (Spring + Jakarta Bean Validation) and the web frontend (Zod schemas). Both layers must enforce these — FE enforcement is for UX, BE enforcement is for security.

> **Status:** Locked (2026-05-25). Source of truth for both layers.

## Conventions

- Strings are **trimmed** before length checks (no leading/trailing whitespace allowed in stored values).
- Numbers reject `NaN`, `Infinity`, and negative zero unless explicitly noted.
- Empty optional fields are JSON `null` (not empty string).
- All `*_at` timestamps are server-assigned (`now()`); clients cannot supply them.

---

## Auth & User

### `users.name`
- Required, string
- Length: 1–100 (trimmed)
- Pattern: any Unicode; spaces, hyphens, apostrophes OK; reject control characters

### `users.email`
- Required, string
- Max length: 254 (RFC 5321)
- Pattern: standard email — Jakarta `@Email` is sufficient
- Stored lowercased; lookups case-insensitive
- Globally unique

### `users.password` (register, change-password)
- Required, string
- Length: 8–100
- Complexity: must contain at least one letter and one digit
- Never logged, never echoed in any response

### `confirmation` (`DELETE /users/me`, Phase 5+)
- Required, string
- Must equal the exact literal `DELETE MY ACCOUNT`

### Refresh tokens (server-internal)
- Token format: 32 random bytes → base64url (43 chars)
- `token_hash`: SHA-256 hex (64 chars)
- `expires_at`: now + 7 days
- Rotated on every successful refresh

---

## Groups

### `groups.name`
- Required, string
- Length: 1–100 (trimmed)

### `groups.currency` and `transactions.currency`
- Required, string
- Pattern: exactly 3 uppercase ASCII letters (`^[A-Z]{3}$`)
- Validated against a static ISO 4217 set — reject unknown codes
- A transaction's `currency` defaults to the group's `currency` if omitted

### `group_memberships.role`
- Required, string (enum)
- Values: `admin` | `member`

### Invite codes (Phase 3)
- Format: 12 chars, `^[A-Z0-9]{12}$`
- `expires_at`: now + 7 days
- One-time use (cleared / marked consumed once accepted)

---

## Transactions

### `transactions.amount`
- Required, decimal (server: `BigDecimal`; JSON: number)
- Range: `> 0` and `≤ 1,000,000.00` (one-million per-transaction cap; higher-value assets like property belong in the `assets` table, not as a single transaction)
- Precision: at most 2 fractional digits — **reject** (do not silently round) if more are supplied
- Stored as `DECIMAL(15,2)`
- Error code on overflow: `OUT_OF_RANGE` (with `max = 1000000.00` in `details`)

### `transactions.type`
- Required, string (enum)
- Values: `expense` | `income`

### `transactions.category`
- Required, string (enum)
- Values: one of the 10 predefined slugs (`food`, `transport`, `shopping`, `entertainment`, `healthcare`, `education`, `bills`, `salary`, `investment`, `other`)
- Server validates against `TransactionCategory` enum
- Soft rule (FE-only): if `type=expense`, hide `salary` from the picker; if `type=income`, hide `food`/`transport`/`shopping`/`entertainment`/`healthcare`/`education`/`bills`. `investment` and `other` always offered.

### `transactions.description`
- Optional, string (`null` allowed)
- Max length: 255

### `transactions.visibility`
- Optional, string (enum)
- Values: `personal` | `shared`
- Default if omitted (server-side): `shared`

### `transactions.date`
- Optional, string (`YYYY-MM-DD`)
- Default if omitted (server-side): today's date in UTC
- Range: `>= 1970-01-01` and `<= today + 1 day` (1-day grace for client/server TZ skew)

---

## Pagination & Sorting (query parameters)

- `page`: integer, ≥ 1, default 1
- `pageSize`: integer, 1–100, default 20
- `sortBy`: per-endpoint enum (e.g., transactions: `date | amount | category`)
- `sortOrder`: `asc` | `desc`, default `desc`

---

## Request-level limits

- Max request body: 1 MB (reject `413 Payload Too Large` if larger)
- Max query string: 2048 chars
- Reject any request body that doesn't parse as JSON (`400 INVALID_JSON`)
- All endpoints require `Content-Type: application/json` on write requests

---

## Error Response Format

All validation failures return `400 VALIDATION_ERROR` with a `details` array of `{ field, code, message }`:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more fields failed validation",
    "details": [
      { "field": "amount",   "code": "MUST_BE_POSITIVE", "message": "Amount must be greater than 0" },
      { "field": "category", "code": "INVALID_ENUM",     "message": "Category must be one of: food, transport, shopping, ..." }
    ]
  }
}
```

### Per-field error codes used here

| Code | Meaning |
|---|---|
| `REQUIRED` | Field is required but missing or null |
| `TOO_SHORT` / `TOO_LONG` | String/array outside length bounds |
| `INVALID_FORMAT` | Doesn't match the expected pattern (regex / email / date) |
| `INVALID_ENUM` | Value not in the allowed set |
| `MUST_BE_POSITIVE` | Numeric value must be > 0 |
| `OUT_OF_RANGE` | Numeric/date outside the allowed range |
| `TOO_MANY_DECIMALS` | Decimal value has more than 2 fractional digits |
| `UNKNOWN_CURRENCY` | Currency code not in the ISO 4217 set |
| `MISMATCH` | Value doesn't equal a required literal (e.g., delete-account confirmation) |

---

## Decision Log

Resolved on 2026-05-25:

| Item | Decision |
|---|---|
| Password complexity | At least one letter + one digit; **no special character required** |
| `transactions.amount` cap | `≤ 1,000,000.00` per transaction (asset-scale values go in `assets`) |
| Phase 3 invite codes | **One-time use** — consumed on accept; cannot be reused even before `expires_at` |
| `transactions.description` max | **255 chars** (no change) |
