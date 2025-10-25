# User Stories — Airbnb Clone Backend
_Updated: 2025-10-26_

These user stories are derived from the use case diagram and drive the MVP scope. Each story includes acceptance criteria and notes for API linkage (see `requirements.md`).

---

## Authentication & Profiles

### 1) Register an account (Guest/Host) — **Must Have**
**As a** user (guest or host)  
**I want** to register with my email and password  
**So that** I can access the platform.

**Acceptance Criteria**
- Email must be unique and valid; password ≥ 8 chars and hashed (bcrypt).
- On success: 201 Created with user id; verification email is queued.
- Role defaults to `guest` if not provided; may be set to `host`.

**API Link**: `POST /auth/register`

---

### 2) Login and stay authenticated — **Must Have**
**As a** user  
**I want** to log in and receive a secure token  
**So that** I can remain authenticated across requests.

**Acceptance Criteria**
- Valid credentials return `accessToken` + `refreshToken`.
- Token expiry/refresh flow works; invalid creds return 401.
- Rate limiting on repeated failures.

**API Link**: `POST /auth/login`, `POST /auth/refresh`

---

## Listings

### 3) Create a property listing — **Must Have**
**As a** host  
**I want** to create a property listing with photos, price, and availability  
**So that** guests can find and book it.

**Acceptance Criteria**
- Required: title, description, address or lat/lng, basePrice, maxGuests, at least one photo.
- Only authenticated `host` can create; returns 201 with listing id.
- Photos validated by mime type/size; stored via configured adapter.

**API Link**: `POST /properties`, `POST /properties/:id/photos`

---

### 4) Update/delete my listing — **Should Have**
**As a** host  
**I want** to update or delete my listing  
**So that** I can keep information accurate.

**Acceptance Criteria**
- Only the listing owner can update/delete.
- Changes reflected in search results within one request cycle.
- Deleting a listing with future confirmed bookings is blocked with a clear error.

**API Link**: `PATCH /properties/:id`, `DELETE /properties/:id`

---

## Search & Booking

### 5) Search for available properties — **Must Have**
**As a** guest  
**I want** to search by location, dates, and guest count with filters  
**So that** I can find suitable places.

**Acceptance Criteria**
- Query parameters for location (text/geo), date range, guests, price range, amenities.
- Results paginated; sort by price/rating/relevance.
- Availability filter excludes dates overlapping confirmed or pending bookings.

**API Link**: `GET /properties`

---

### 6) Book a property — **Must Have**
**As a** guest  
**I want** to book a property for specific dates  
**So that** I can secure my stay.

**Acceptance Criteria**
- Validates no overlaps; guests ≤ maxGuests.
- Shows final price (nights × rate + fees/taxes) before payment.
- On successful payment intent + confirmation → booking `confirmed`.

**API Link**: `POST /bookings`, `POST /payments/intent`

---

### 7) Cancel a booking under policy — **Should Have**
**As a** guest or host  
**I want** to cancel a booking according to the cancellation policy  
**So that** refunds (if any) are applied fairly.

**Acceptance Criteria**
- State machine updates to `canceled`; policy matrix computes refund.
- Stripe/PayPal webhook events reconcile final state.
- Audit log entry is created.

**API Link**: `POST /bookings/:id/cancel`

---

## Payments & Payouts

### 8) Pay securely — **Must Have**
**As a** guest  
**I want** to pay securely for my booking  
**So that** my reservation is confirmed.

**Acceptance Criteria**
- Stripe/PayPal supported; 3-D Secure when required.
- Idempotency keys prevent duplicate charges.
- Webhook `payment_intent.succeeded` marks booking `confirmed`.

**API Link**: `POST /payments/intent`, `POST /payments/webhook`

---

### 9) Receive host payout — **Should Have**
**As a** host  
**I want** automatic payouts after check-out  
**So that** I receive my earnings.

**Acceptance Criteria**
- Payout scheduled T+1 after checkout; platform fees deducted.
- Payout status visible in dashboard (upcoming/paid).
- Ledger entries are immutable.

**API Link**: `GET /payouts`

---

## Reviews & Notifications

### 10) Leave a review — **Should Have**
**As a** guest  
**I want** to leave a review after my stay  
**So that** I can share feedback.

**Acceptance Criteria**
- Only for completed bookings; one review per booking.
- Rating 1–5; basic profanity filter.

**API Link**: `POST /reviews`, `GET /properties/:id/reviews`

---

### 11) Receive booking/payment notifications — **Should Have**
**As a** user  
**I want** email/in-app notifications for booking and payment events  
**So that** I stay informed.

**Acceptance Criteria**
- Email sent on confirmation/cancellation/payouts.
- In-app notifications retrievable with unread/seen flags.
- User preferences can disable specific categories.

**API Link**: Notifications endpoints (to be finalized)

---

## Admin

### 12) Moderate marketplace — **Should Have**
**As an** admin  
**I want** to manage users, listings, bookings, and payments  
**So that** the marketplace remains healthy.

**Acceptance Criteria**
- RBAC-protected endpoints visible only to `admin`.
- Can suspend users/listings; view disputes and refunds.
- Actions are audited.

**API Link**: Admin endpoints (see `requirements.md`)
