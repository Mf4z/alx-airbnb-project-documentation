# Requirements Specification — Airbnb Clone Backend
_Updated: 2025-10-26_

This document defines **functional** and **technical** requirements for three core backend features:
1. Authentication & Authorization  
2. Property Listings  
3. Booking and Payment

It links directly to earlier artifacts:
- **Use Case Diagram:** maps interactions
- **User Stories:** defines intent
- **Data Flow Diagram:** shows data movement
- **Flowchart:** illustrates booking logic

---

## 1️⃣ Authentication & Authorization

### Overview
Secure login, registration, and role-based access (Guest, Host, Admin).

### Endpoints

| Method | Endpoint | Description |
|--------|-----------|-------------|
| `POST` | `/api/v1/auth/register` | Register user |
| `POST` | `/api/v1/auth/login` | Login and issue tokens |
| `POST` | `/api/v1/auth/refresh` | Refresh access token |
| `GET` | `/api/v1/users/me` | Get current profile |
| `PATCH` | `/api/v1/users/me` | Update profile |

### Input / Output
- **Register:** `{ "email": "...", "password": "...", "role": "guest|host" }`  
  → `201 Created` `{ "id": 1, "email": "..." }`
- **Login:** `{ "email": "...", "password": "..." }`  
  → `200 OK` `{ "accessToken": "...", "refreshToken": "...", "user": {...} }`

### Validation
- Email must be unique and valid.
- Password ≥ 8 characters; hashed with bcrypt.
- Role defaults to `guest` if not supplied.

### Security
- JWT (RS256) for access/refresh tokens.  
- Token rotation + blacklist on logout.  
- RBAC middleware for `admin`-only endpoints.  
- Rate-limit login to 5/min/IP.

### Performance
- P99 login < 300 ms.  
- Cache `/users/me` responses for 60 s.

---

## 2️⃣ Property Listings

### Overview
Hosts can create, edit, delete, and view properties.  
Guests can browse and filter.

### Endpoints

| Method | Endpoint | Description |
|--------|-----------|-------------|
| `POST` | `/api/v1/properties` | Create property (Host only) |
| `GET` | `/api/v1/properties` | Search & filter listings |
| `GET` | `/api/v1/properties/:id` | Get property details |
| `PATCH` | `/api/v1/properties/:id` | Update listing (Host only) |
| `DELETE` | `/api/v1/properties/:id` | Delete listing (Host only) |

### Input / Output
- **Create:**  
  `{ "title": "...", "description": "...", "price": 120, "location": "Paris", "maxGuests": 4, "amenities": ["wifi","pool"], "photos": [...] }`  
  → `201 Created` `{ "id": 42 }`
- **Search:**  
  Query params: `?location=Paris&minPrice=50&maxPrice=200&guests=2&amenities=wifi`  
  → Paginated JSON array of listings.

### Validation
- Title ≤ 120 chars; price > 0; at least 1 photo.
- Only owner (host) may modify/delete.
- Prevent delete if confirmed bookings exist.

### Database
- Tables: `properties`, `property_photos`, `amenities`, `property_amenities`.
- Index: `btree(location)`, `GIN(amenities)`.

### Performance
- Pagination (limit = 20 default).  
- Redis cache for top search queries (60 s).  
- Use query joins to avoid N+1 issues.

---

## 3️⃣ Booking & Payment

### Overview
Guests reserve stays and pay securely.  
Hosts receive payouts post-checkout.

### Endpoints

| Method | Endpoint | Description |
|--------|-----------|-------------|
| `POST` | `/api/v1/bookings` | Create booking |
| `GET` | `/api/v1/bookings` | List user bookings |
| `POST` | `/api/v1/bookings/:id/cancel` | Cancel booking |
| `POST` | `/api/v1/payments/intent` | Create payment intent (Stripe/PayPal) |
| `POST` | `/api/v1/payments/webhook` | Handle payment webhook |
| `GET` | `/api/v1/payouts` | View host payouts |

### Input / Output
- **Create Booking:**  
  `{ "propertyId": 42, "checkIn": "2025-11-01", "checkOut": "2025-11-05", "guests": 2 }`  
  → `201 Created` `{ "id": 101, "status": "confirmed" }`

### Business Rules
- Prevent double-booking (no overlapping confirmed/pending ranges).
- Cancellations allowed only before check-in.
- Refunds follow policy (flexible, moderate, strict).

### Payment Logic
1. Create payment intent (Stripe API).
2. Receive webhook on success → mark booking confirmed.
3. Payout scheduled T+1 after checkout.

### Validation
- `guests ≤ property.maxGuests`
- `checkIn < checkOut`
- `paymentIntent` verified via signature.

### Performance / Reliability
- Idempotency key on booking creation.
- Retry logic for webhooks.
- P99 booking confirmation < 800 ms.

### Security
- HTTPS everywhere.
- Signed webhooks.
- Mask payment data in logs.

---

## 📈 Non-Functional Requirements
- **Availability:** ≥ 99.9 % uptime.  
- **Testing:** Unit + integration coverage ≥ 70 %.  
- **Monitoring:** Structured JSON logs + CloudWatch metrics.  
- **Scalability:** Stateless API, load-balanced.  
- **Error Handling:** Consistent REST error schema:  
  `{ "error": { "code": "...", "message": "..." } }`

---

## ✅ Summary
| Area | Focus | Tools |
|------|--------|-------|
| Security | JWT, HTTPS, RBAC | bcrypt, jsonwebtoken |
| API | REST (OpenAPI Spec) | Express.js / FastAPI |
| Data | Relational DB | PostgreSQL |
| Payments | Stripe / PayPal | Webhooks |
| Email | SendGrid / Mailgun | Async Jobs |
| Caching | Redis | Query Caching |
| Testing | Jest / Pytest | CI/CD |

---

