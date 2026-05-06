# Bliss Courier — Full Stack Courier Management System

> **Stack:** Spring Boot 3 · Angular 16 · MySQL · Spring Security · JWT

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Project Structure](#3-project-structure)
4. [Getting Started](#4-getting-started)
5. [Security Architecture](#5-security-architecture)
6. [Core Features](#6-core-features)
7. [Invoice & Billing System](#7-invoice--billing-system) ← main topic
8. [API Reference](#8-api-reference)
9. [Database Schema](#9-database-schema)
10. [Role-Based Access Control](#10-role-based-access-control)
11. [Frontend Architecture](#11-frontend-architecture)

---

## 1. Project Overview

Bliss Courier is a full-stack **Courier Management System** that allows businesses to:

- Register and manage courier companies
- Create and track parcels end-to-end
- Auto-generate invoices with itemised pricing
- Manage billing status (Paid / Unpaid)
- Secure every operation with JWT-based role access

The system has two user roles:

| Role | What they can do |
|------|-----------------|
| **ADMIN** | Full access — create companies, parcels, generate invoices, mark payments |
| **USER** | Read-only — view companies and parcels |

---

## 2. Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Backend framework | Spring Boot | 3.5.x |
| Security | Spring Security + JWT (jjwt) | 0.11.5 |
| ORM | Spring Data JPA + Hibernate | 6.x |
| Database | MySQL | 8.x |
| Frontend framework | Angular | 16+ |
| UI components | Angular Material + Bootstrap | 5.x |
| HTTP client | Angular HttpClient | — |
| Build tool (backend) | Maven | 3.x |
| Build tool (frontend) | npm + Angular CLI | — |

---

## 3. Project Structure

```
workspace/
├── Bliss-Courier-Level1_212/blisscourier/     ← Spring Boot backend
│   └── src/main/java/com/example/courier/
│       ├── controller/
│       │   ├── CourierCompanyController.java
│       │   ├── ParcelController.java
│       │   └── InvoiceController.java          ← Invoice API
│       ├── service/
│       │   ├── CourierCompanyService.java
│       │   ├── ParcelService.java
│       │   └── InvoiceService.java             ← Pricing logic
│       ├── model/
│       │   ├── CourierCompany.java
│       │   ├── Parcel.java
│       │   └── Invoice.java                    ← Invoice entity
│       ├── repository/
│       │   ├── CourierRepository.java
│       │   ├── ParcelRepository.java
│       │   └── InvoiceRepository.java
│       ├── exception/
│       │   ├── GlobalExceptionHandler.java
│       │   ├── InvalidCourierCompanyException.java
│       │   └── InvalidParcelException.java
│       └── security/
│           ├── config/SecurityConfig.java
│           ├── jwt/JwtUtil.java
│           ├── jwt/JwtFilter.java
│           ├── model/AppUser.java
│           ├── repository/UserRepository.java
│           ├── service/CustomUserDetailsService.java
│           └── controller/AuthController.java
│
└── BlissCourier/                               ← Angular frontend
    └── src/app/
        ├── landing/          ← Public landing page
        ├── login/
        ├── register/
        ├── home/             ← Post-login dashboard
        ├── add-company/      ← ADMIN only
        ├── add-parcel/       ← ADMIN only
        ├── view-companies/
        ├── view-parcels/     ← Has "View Invoice" button
        ├── invoice/          ← Single invoice view + generate
        ├── all-invoices/     ← Admin invoice dashboard
        ├── model/
        │   ├── parcel.model.ts
        │   └── invoice.model.ts
        ├── service/
        │   ├── auth.service.ts
        │   ├── parcel.service.ts
        │   └── invoice.service.ts
        ├── guard/
        │   ├── auth.guard.ts
        │   └── role.guard.ts
        └── interceptor/
            └── auth.interceptor.ts
```

---

## 4. Getting Started

### Prerequisites
- Java 17+
- Node.js 18+
- MySQL 8 running locally
- Maven (or use the included `mvnw` wrapper)

### Backend

```bash
cd Bliss-Courier-Level1_212/blisscourier
./mvnw spring-boot:run
```

Runs on **http://localhost:4004**

Database config in `src/main/resources/application.properties`:
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/blisscourier?createDatabaseIfNotExist=true
spring.datasource.username=root
spring.datasource.password=<your_password>
spring.jpa.hibernate.ddl-auto=create
server.port=4004
jwt.secret=bliss-courier-secret-key-please-change-to-32-bytes-min
jwt.expiration.ms=3600000
```

### Frontend

```bash
cd BlissCourier
npm install
npm start
```

Runs on **http://localhost:4200**

### First-time Admin Setup

Register an admin account via API (the UI register form creates USER by default):

```bash
curl -X POST http://localhost:4004/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@bliss.com","password":"Admin@123","role":"ADMIN"}'
```

---

## 5. Security Architecture

### How JWT Security Works

```
1. User logs in → POST /auth/login
2. Server validates credentials with BCrypt
3. Server generates JWT token (signed with HMAC-SHA256)
4. Token returned to Angular → stored in localStorage
5. Every subsequent request:
   Angular AuthInterceptor adds:  Authorization: Bearer <token>
6. JwtFilter on backend:
   - Reads the header
   - Validates signature + expiry
   - Loads user from DB
   - Sets Authentication in SecurityContextHolder
7. SecurityConfig checks role against the URL
8. Controller executes if allowed
```

### Key Security Classes

| Class | Role |
|-------|------|
| `JwtUtil` | Generates and validates JWT tokens using HMAC-SHA256 |
| `JwtFilter` | Intercepts every HTTP request, validates token, sets auth context |
| `SecurityConfig` | Defines URL-level access rules, CORS, stateless session policy |
| `CustomUserDetailsService` | Loads user from DB for Spring Security |
| `AuthController` | Handles `/auth/register` and `/auth/login` |
| `AuthInterceptor` (Angular) | Automatically attaches JWT to every HTTP request |
| `AuthGuard` (Angular) | Blocks unauthenticated route navigation |
| `RoleGuard` (Angular) | Blocks non-admin route navigation |

### URL Security Rules

```java
/auth/**              → Public (no token needed)
/api/invoice/**       → ADMIN only
GET  /**              → ADMIN or USER
POST /**              → ADMIN only
PUT  /**              → ADMIN only
DELETE /**            → ADMIN only
```

---

## 6. Core Features

### Courier Company Management
- Add companies with name, location, contact, rating, service type
- View all companies in a searchable table

### Parcel Management
- Add parcels linked to a company
- Fields: parcel ID, tracking number, sender, receiver, city, weight, service type, status, dates
- Auto-calculates delivery charge on creation
- Duplicate parcel ID check — returns 409 Conflict if ID already exists
- Update delivery status (Pending → In Transit → Delivered → Cancelled)
- Search by: parcel ID, tracking number, city, status, date range

---

## 7. Invoice & Billing System

This is the most recently added feature. It provides complete billing management for parcels, accessible only to ADMIN users.

---

### 7.1 What is the Invoice System?

Every parcel can have one invoice. The invoice captures:
- The parcel it belongs to (one-to-one relationship)
- Distance of delivery in km
- Delivery type (STANDARD or EXPRESS)
- An itemised price breakdown
- Payment status (PAID / UNPAID)
- Payment method (CASH / ONLINE)
- Timestamp of when it was generated

Invoices are displayed entirely on the UI — no PDF generation.

---

### 7.2 Pricing Formula

```
base_price       = 50
weight_charge    = weight (kg)  × 10
distance_charge  = distance (km) × 5
delivery_charge  = EXPRESS → 100  |  STANDARD → 0

total_amount = base_price + weight_charge + distance_charge + delivery_charge
```

**Example — 5 kg parcel, 100 km, EXPRESS:**

| Component | Calculation | Amount |
|-----------|------------|--------|
| Base Price | flat | ₹50.00 |
| Weight Charge | 5 kg × ₹10 | ₹50.00 |
| Distance Charge | 100 km × ₹5 | ₹500.00 |
| Delivery Surcharge | EXPRESS | ₹100.00 |
| **Total** | | **₹700.00** |

---

### 7.3 Invoice Entity (Database)

```java
@Entity
public class Invoice {
    @Id @GeneratedValue
    private Long id;

    @OneToOne
    @JoinColumn(name = "parcel_id", unique = true)
    private Parcel parcel;           // FK to parcel table

    private Double weight;           // copied from parcel at generation time
    private Double distance;         // provided at invoice generation
    private DeliveryType deliveryType;   // STANDARD or EXPRESS

    private Double basePrice;        // always 50
    private Double weightCharge;     // weight × 10
    private Double distanceCharge;   // distance × 5
    private Double deliveryCharge;   // 100 if EXPRESS, else 0
    private Double totalAmount;      // sum of all above

    private PaymentStatus paymentStatus;  // PAID or UNPAID
    private PaymentMethod paymentMethod;  // CASH or ONLINE
    private LocalDateTime createdAt;
}
```

**Relationship:** `Invoice` has a `@OneToOne` with `Parcel`. The `unique = true` constraint on `parcel_id` ensures one parcel can only ever have one invoice.

---

### 7.4 Backend — Service Layer (InvoiceService)

The service is the brain of the invoice system. It handles:

**`generateInvoice(parcelId, distance, deliveryType, paymentMethod)`**

```
1. Check if invoice already exists for this parcel
   → If yes, return existing invoice (idempotent — safe to call twice)
2. Fetch parcel from DB by parcelId
   → Throw InvalidParcelException if not found (404)
3. Read weight from the parcel
4. Apply pricing formula:
   weightCharge   = weight × 10
   distanceCharge = distance × 5
   deliveryCharge = EXPRESS ? 100 : 0
   totalAmount    = 50 + weightCharge + distanceCharge + deliveryCharge
5. Build Invoice object with all fields
6. Set paymentStatus = UNPAID (default)
7. Set createdAt = now
8. Save to DB and return
```

**`getInvoiceByParcelId(parcelId)`**
- Queries `InvoiceRepository.findByParcel_ParcelId(parcelId)`
- Throws `InvalidParcelException` (404) if not found

**`getAllInvoices()`**
- Returns all invoices — admin dashboard use

**`markAsPaid(invoiceId, paymentMethod)`**
- Finds invoice by ID
- Sets `paymentStatus = PAID`
- Updates payment method if provided
- Saves and returns updated invoice

---

### 7.5 Backend — Controller (InvoiceController)

Base path: `/api/invoice`

```
POST   /api/invoice/generate/{parcelId}   → Generate invoice
GET    /api/invoice/{parcelId}            → Get invoice by parcel ID
GET    /api/invoice/all                   → Get all invoices (admin)
PUT    /api/invoice/pay/{invoiceId}       → Mark invoice as paid
```

All endpoints are secured — require `ROLE_ADMIN` (enforced in `SecurityConfig`).

**Generate Invoice — Request Body:**
```json
{
  "distance": "120",
  "deliveryType": "EXPRESS",
  "paymentMethod": "ONLINE"
}
```

**Generate Invoice — Response:**
```json
{
  "id": 1,
  "parcel": {
    "parcelId": "1001",
    "senderName": "Rahul",
    "receiverName": "Priya",
    "destinationCity": "Mumbai",
    "weight": 5.0,
    "courierCompany": { "companyName": "SpeedEx" }
  },
  "weight": 5.0,
  "distance": 120.0,
  "deliveryType": "EXPRESS",
  "basePrice": 50.0,
  "weightCharge": 50.0,
  "distanceCharge": 600.0,
  "deliveryCharge": 100.0,
  "totalAmount": 800.0,
  "paymentStatus": "UNPAID",
  "paymentMethod": "ONLINE",
  "createdAt": "2025-05-06T10:30:00"
}
```

---

### 7.6 Repository — InvoiceRepository

```java
public interface InvoiceRepository extends JpaRepository<Invoice, Long> {
    Optional<Invoice> findByParcel_ParcelId(String parcelId);
    boolean existsByParcel_ParcelId(String parcelId);
}
```

Spring Data JPA auto-generates the SQL for these methods using the `_` notation to traverse the `parcel` relationship and query by `parcelId`.

---

### 7.7 Frontend — Invoice Component (`/invoice/:parcelId`)

**Flow when admin clicks "View Invoice" on a parcel:**

```
1. Navigate to /invoice/:parcelId
2. InvoiceComponent.ngOnInit() fires
3. Calls InvoiceService.getByParcelId(parcelId)
   → GET /api/invoice/{parcelId}

   If 404 (no invoice yet):
     → Show "Generate Invoice" form
     → Admin enters: distance, delivery type, payment method
     → Click "Generate Invoice"
     → POST /api/invoice/generate/{parcelId}
     → Invoice displayed

   If 200 (invoice exists):
     → Display invoice card immediately

4. Invoice card shows:
   - Parcel details (ID, company, sender, receiver, city, status)
   - Price breakdown table (base, weight, distance, surcharge, total)
   - Payment status badge (PAID / UNPAID)
   - "Mark as Paid" button if UNPAID
   - "Print" button (browser print, hides UI chrome)
```

---

### 7.8 Frontend — All Invoices Component (`/invoices`)

Admin dashboard showing all invoices with:

**Analytics cards (live, updates with filters):**
- Total Invoices count
- Total Revenue (sum of all totalAmount)
- Paid count
- Unpaid count
- Express count

**Filters:**
- Search by Parcel ID
- Filter by Payment Status (All / Paid / Unpaid)
- Filter by Delivery Type (All / Standard / Express)
- Filter by Payment Method (All / Cash / Online)

**Table columns:**
Invoice ID · Parcel ID · Company · Sender→Receiver · Weight · Distance · Type · Total · Method · Status · Date · View button

---

### 7.9 Frontend — Invoice Service

```typescript
// Generate invoice for a parcel
generate(parcelId, distance, deliveryType, paymentMethod)
  → POST /api/invoice/generate/{parcelId}

// Fetch existing invoice
getByParcelId(parcelId)
  → GET /api/invoice/{parcelId}

// Admin: all invoices
getAll()
  → GET /api/invoice/all

// Mark as paid
markAsPaid(invoiceId, paymentMethod)
  → PUT /api/invoice/pay/{invoiceId}
```

All calls go through `AuthInterceptor` which automatically attaches the JWT token.

---

### 7.10 How Invoice Fits Into the Full Flow

```
Admin logs in
      │
      ▼
View Parcels → Search for a parcel
      │
      ▼
Click "Invoice" button on any row
      │
      ▼
/invoice/:parcelId
      │
      ├── Invoice exists? → Show invoice card
      │
      └── No invoice? → Show generate form
                │
                ▼
          Enter distance + delivery type + payment method
                │
                ▼
          POST /api/invoice/generate/{parcelId}
                │
                ▼
          InvoiceService calculates price
                │
                ▼
          Invoice saved to DB
                │
                ▼
          Invoice card displayed with full breakdown
                │
                ▼
          Admin clicks "Mark as Paid"
                │
                ▼
          PUT /api/invoice/pay/{invoiceId}
                │
                ▼
          Status updates to PAID ✅
```

---

### 7.11 Key Design Decisions

| Decision | Reason |
|----------|--------|
| `@OneToOne` between Invoice and Parcel | Each parcel has exactly one invoice — enforced at DB level with `unique = true` |
| Idempotent generate endpoint | Calling generate twice returns the existing invoice instead of creating a duplicate |
| Pricing calculated server-side | Client cannot manipulate prices — all calculation happens in `InvoiceService` |
| No PDF generation | Invoice is rendered entirely in Angular — clean, fast, printable via browser |
| ADMIN-only access | Invoice data is sensitive billing info — `SecurityConfig` enforces `ROLE_ADMIN` on all `/api/invoice/**` routes |
| Enums for DeliveryType, PaymentStatus, PaymentMethod | Type-safe, stored as strings in DB, validated at compile time |
| Weight copied to Invoice at generation time | Parcel weight could change later — invoice captures the value at billing time |

---

### 7.12 Questions Your Team Lead May Ask

**Q: Why is the invoice endpoint separate from the parcel endpoint?**
Separation of concerns. Parcel management and billing are different domains. Keeping them separate makes each controller focused and easier to test or extend independently.

**Q: What happens if someone calls generate twice for the same parcel?**
The service checks `invoiceRepository.existsByParcel_ParcelId(parcelId)` first. If an invoice already exists, it returns the existing one without creating a new record. This makes the endpoint idempotent.

**Q: How is the pricing protected from client-side manipulation?**
The client only sends `distance`, `deliveryType`, and `paymentMethod`. All price calculations happen exclusively in `InvoiceService` on the server. The client never sends any price values.

**Q: How does the `@OneToOne` relationship work in JPA?**
`Invoice` has a `@OneToOne` annotation pointing to `Parcel` with `@JoinColumn(name = "parcel_id", unique = true)`. JPA creates a `parcel_id` foreign key column in the `invoice` table. The `unique = true` constraint at the DB level ensures no two invoices can reference the same parcel.

**Q: Why use enums for DeliveryType and PaymentStatus?**
Enums prevent invalid values from being stored. `@Enumerated(EnumType.STRING)` stores them as readable strings (`"EXPRESS"`, `"PAID"`) in MySQL rather than ordinal integers, making the DB data human-readable and safe from reordering bugs.

**Q: How does the frontend know whether to show the generate form or the invoice?**
`InvoiceComponent` calls `getByParcelId()` on load. If the backend returns 404, it means no invoice exists yet and the generate form is shown. If 200, the invoice card is shown directly.

**Q: How is the "Mark as Paid" secured?**
The `PUT /api/invoice/pay/{invoiceId}` endpoint is under `/api/invoice/**` which is mapped to `hasRole("ADMIN")` in `SecurityConfig`. Even if a USER somehow calls this URL, Spring Security rejects it with 403 Forbidden before the controller is reached.

---

## 8. API Reference

### Auth
| Method | URL | Auth | Body |
|--------|-----|------|------|
| POST | `/auth/register` | None | `{email, password, role}` |
| POST | `/auth/login` | None | `{email, password}` |

### Parcels
| Method | URL | Auth | Description |
|--------|-----|------|-------------|
| POST | `/parcels/{companyId}` | ADMIN | Add parcel |
| GET | `/parcels/{parcelId}` | ALL | Get by ID |
| GET | `/parcels/tracking/{trackingNumber}` | ALL | Get by tracking |
| PUT | `/parcels/{parcelId}/{status}` | ADMIN | Update status |
| GET | `/parcels/destination/{city}` | ALL | Filter by city |
| GET | `/parcels/status/{status}` | ALL | Filter by status |
| GET | `/parcels/company/{companyName}` | ALL | Filter by company |
| GET | `/parcels/serviceType/{type}?fromDate=&toDate=` | ALL | Filter by type + date |
| GET | `/parcels/bookedAfter/{date}` | ALL | Booked after date |
| GET | `/parcels/weight/{weight}` | ALL | Weight greater than |
| GET | `/parcels/filter?destinationCity=&deliveryStatus=` | ALL | Combined filter |

### Invoice
| Method | URL | Auth | Description |
|--------|-----|------|-------------|
| POST | `/api/invoice/generate/{parcelId}` | ADMIN | Generate invoice |
| GET | `/api/invoice/{parcelId}` | ADMIN | Get invoice by parcel |
| GET | `/api/invoice/all` | ADMIN | All invoices |
| PUT | `/api/invoice/pay/{invoiceId}` | ADMIN | Mark as paid |

---

## 9. Database Schema

```
app_user
  id (PK, auto)
  email (unique)
  password (BCrypt)
  role

courier_company
  company_id (PK)
  company_name
  headquarters_location
  contact_number
  rating
  service_type

parcel
  parcel_id (PK)
  tracking_number
  sender_name
  receiver_name
  destination_city
  weight
  service_type
  delivery_status
  booking_date
  delivered_date
  delivery_charge
  courier_company_company_id (FK → courier_company)

invoice
  id (PK, auto)
  parcel_id (FK → parcel, UNIQUE)   ← one invoice per parcel
  weight
  distance
  delivery_type
  base_price
  weight_charge
  distance_charge
  delivery_charge
  total_amount
  payment_status
  payment_method
  created_at
```

---

## 10. Role-Based Access Control

### Backend (Spring Security)

```java
// SecurityConfig.java
.requestMatchers("/auth/**").permitAll()
.requestMatchers("/api/invoice/**").hasRole("ADMIN")
.requestMatchers(HttpMethod.GET, "/**").hasAnyRole("ADMIN", "USER")
.requestMatchers(HttpMethod.POST, "/**").hasRole("ADMIN")
.requestMatchers(HttpMethod.PUT, "/**").hasRole("ADMIN")
.requestMatchers(HttpMethod.DELETE, "/**").hasRole("ADMIN")
```

### Frontend (Angular Guards)

```
Route: /invoice/:parcelId  → canActivate: [AuthGuard, RoleGuard]  data: { roles: ['ADMIN'] }
Route: /invoices           → canActivate: [AuthGuard, RoleGuard]  data: { roles: ['ADMIN'] }
Route: /add-parcel         → canActivate: [AuthGuard, RoleGuard]  data: { roles: ['ADMIN'] }
Route: /add-company        → canActivate: [AuthGuard, RoleGuard]  data: { roles: ['ADMIN'] }
Route: /home               → canActivate: [AuthGuard]
Route: /view-parcels       → canActivate: [AuthGuard]
```

**Double protection:** Even if a user bypasses the Angular guard (e.g. by directly calling the API), Spring Security on the backend will reject the request with `403 Forbidden`.

---

## 11. Frontend Architecture

### Component Flow

```
AppComponent (shell — navbar + router-outlet)
│
├── LandingComponent      /              public
├── LoginComponent        /login         public
├── RegisterComponent     /register      public
│
├── HomeComponent         /home          AuthGuard
├── ViewCompaniesComponent /view-companies AuthGuard
├── ViewParcelsComponent  /view-parcels  AuthGuard
│
├── AddCompanyComponent   /add-company   AuthGuard + RoleGuard(ADMIN)
├── AddParcelComponent    /add-parcel    AuthGuard + RoleGuard(ADMIN)
├── InvoiceComponent      /invoice/:id   AuthGuard + RoleGuard(ADMIN)
└── AllInvoicesComponent  /invoices      AuthGuard + RoleGuard(ADMIN)
```

### HTTP Request Lifecycle

```
Component calls Service
    → Service calls HttpClient
        → AuthInterceptor adds Authorization: Bearer <token>
            → Request hits Spring Boot
                → JwtFilter validates token
                    → SecurityConfig checks role
                        → Controller handles request
                            → Service layer executes business logic
                                → Repository queries MySQL
                                    → Response flows back to Angular
```

---

*Bliss Courier — Built with Spring Boot & Angular*
