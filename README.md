# Bliss Courier — Complete Project Documentation
### Spring Boot + Angular | JWT Security | Presentation Guide

---

## 1. PROJECT OVERVIEW

**Bliss Courier** is a full-stack Courier Management System built with:

| Layer | Technology |
|---|---|
| Backend | Spring Boot 3, Spring Security, Spring Data JPA |
| Frontend | Angular 15+, Angular Material, Bootstrap 5 |
| Database | MySQL |
| Auth | JWT (JSON Web Tokens) with HMAC-SHA256 |
| Build | Maven (backend), npm (frontend) |

**What it does:**
- Register and manage courier companies
- Add parcels and link them to companies
- Track parcels by city, status, tracking number, date range, weight
- Auto-calculate delivery charges based on weight and service type
- Role-based access: ADMIN can write data, USER can only read

---

## 2. PROJECT STRUCTURE

```
Bliss Courier
├── Backend  (Spring Boot — port 4004)
│   ├── controller/
│   │   ├── CourierCompanyController.java
│   │   └── ParcelController.java
│   ├── service/
│   │   ├── ICourierCompanyService.java
│   │   ├── CourierCompanyService.java
│   │   ├── IParcelService.java
│   │   └── ParcelService.java
│   ├── model/
│   │   ├── CourierCompany.java
│   │   └── Parcel.java
│   ├── repository/
│   │   ├── CourierRepository.java
│   │   └── ParcelRepository.java
│   ├── exception/
│   │   ├── GlobalExceptionHandler.java
│   │   ├── InvalidCourierCompanyException.java
│   │   └── InvalidParcelException.java
│   └── security/
│       ├── config/SecurityConfig.java        ← Spring Security rules
│       ├── jwt/JwtUtil.java                  ← Token generation & validation
│       ├── jwt/JwtFilter.java                ← Intercepts every HTTP request
│       ├── model/AppUser.java                ← User entity
│       ├── repository/UserRepository.java
│       ├── service/CustomUserDetailsService.java
│       └── controller/AuthController.java    ← /auth/register, /auth/login
│
└── Frontend (Angular — port 4200)
    ├── landing/          ← Public landing page (no login needed)
    ├── login/            ← Login form
    ├── register/         ← Register form
    ├── home/             ← Post-login dashboard
    ├── add-company/      ← ADMIN only
    ├── add-parcel/       ← ADMIN only
    ├── view-companies/   ← All users
    ├── view-parcels/     ← All users
    ├── guard/
    │   ├── auth.guard.ts     ← Blocks unauthenticated users
    │   └── role.guard.ts     ← Blocks non-admin users
    ├── interceptor/
    │   └── auth.interceptor.ts  ← Attaches JWT to every HTTP request
    └── service/
        ├── auth.service.ts      ← Login, register, token storage
        └── parcel.service.ts    ← Parcel API calls
```

---

## 3. COMPLETE APPLICATION FLOW

```
User opens browser
        │
        ▼
  [ / ] Landing Page  ──────────────────────────────────────────────
        │                                                           │
   Click Login                                               Click Register
        │                                                           │
        ▼                                                           ▼
  [ /login ]                                               [ /register ]
  POST /auth/login                                       POST /auth/register
        │                                                           │
   JWT returned                                           User saved to DB
   Stored in localStorage                                           │
        │                                                           │
        ▼                                                           │
  AuthGuard checks token ◄──────────────────────────────────────────
        │
   Token valid?
   YES → navigate to /home
   NO  → redirect to /login
        │
        ▼
  [ /home ] Dashboard
        │
   ┌────┴────────────────────────────────┐
   │                                     │
ADMIN role                           USER role
   │                                     │
   ├── /add-company  (POST)              ├── /view-companies (GET)
   ├── /add-parcel   (POST)              └── /view-parcels   (GET)
   ├── /view-companies (GET)
   └── /view-parcels   (GET)
```

---

## 4. SECURITY ARCHITECTURE — THE FULL PICTURE

This is the most important section for your presentation.

### 4.1 Security Components Map

```
┌─────────────────────────────────────────────────────────────────┐
│                        SPRING SECURITY CHAIN                    │
│                                                                 │
│  HTTP Request                                                   │
│      │                                                          │
│      ▼                                                          │
│  ┌─────────────┐                                                │
│  │  JwtFilter  │  ← Runs BEFORE every request                  │
│  │             │    Reads "Authorization: Bearer <token>"       │
│  │             │    Validates token with JwtUtil                │
│  │             │    Loads user from CustomUserDetailsService    │
│  │             │    Sets Authentication in SecurityContext      │
│  └──────┬──────┘                                                │
│         │                                                       │
│         ▼                                                       │
│  ┌──────────────────┐                                           │
│  │  SecurityConfig  │  ← Checks rules:                         │
│  │  (FilterChain)   │    /auth/** → permitAll                   │
│  │                  │    GET /**  → ADMIN or USER               │
│  │                  │    POST /** → ADMIN only                  │
│  │                  │    PUT /**  → ADMIN only                  │
│  └──────┬───────────┘                                           │
│         │                                                       │
│         ▼                                                       │
│    Controller (if allowed)                                      │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 JWT Token Flow — Step by Step

```
REGISTRATION
─────────────────────────────────────────────────────
Angular          →  POST /auth/register
                    { email, password, role }
                         │
                    AuthController
                         │
                    BCryptPasswordEncoder.encode(password)
                         │
                    AppUser saved to MySQL
                         │
                    Returns saved AppUser


LOGIN
─────────────────────────────────────────────────────
Angular          →  POST /auth/login
                    { email, password }
                         │
                    AuthController
                         │
                    AuthenticationManager.authenticate()
                         │
                    CustomUserDetailsService.loadUserByUsername()
                         │
                    BCrypt compares hashed password
                         │
                    JwtUtil.generateToken(email, role)
                         │
                    Returns { email, token, role }
                         │
Angular          ←  Stores token in localStorage


AUTHENTICATED REQUEST
─────────────────────────────────────────────────────
Angular          →  GET /parcels/123
                    Header: Authorization: Bearer eyJhbGci...
                         │
                    JwtFilter intercepts
                         │
                    JwtUtil.isTokenValid()  → checks signature
                    JwtUtil.isTokenExpired() → checks expiry
                    JwtUtil.extractUsername() → gets email
                         │
                    CustomUserDetailsService.loadUserByUsername(email)
                         │
                    SecurityContext.setAuthentication(user)
                         │
                    SecurityConfig checks role
                         │
                    Controller executes
                         │
Angular          ←  Response data
```

---

## 5. EACH SECURITY CLASS EXPLAINED

### 5.1 `AppUser.java` — The User Entity
```java
@Entity
public class AppUser {
    @Id @GeneratedValue
    private Long id;

    @Column(unique = true, nullable = false)
    private String email;       // used as username

    private String password;    // BCrypt hashed — never plain text
    private String role;        // "ROLE_ADMIN" or "ROLE_USER"
}
```
**Key point:** Password is NEVER stored as plain text. BCrypt adds a random salt and hashes it.

---

### 5.2 `JwtUtil.java` — Token Factory
Responsible for creating and reading JWT tokens.

```
Token Structure (3 parts separated by dots):
eyJhbGciOiJIUzI1NiJ9   ← Header  (algorithm: HS256)
.
eyJzdWIiOiJhZG1pbkBibGlzcy5jb20iLCJyb2xlIjoiUk9MRV9BRE1JTiJ9
                        ← Payload (subject=email, role, iat, exp)
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
                        ← Signature (HMAC-SHA256 with secret key)
```

**What it does:**
- `generateToken(email, role)` → builds JWT, signs with HMAC-SHA256, sets 1hr expiry
- `isTokenValid(token)` → verifies signature hasn't been tampered
- `isTokenExpired(token)` → checks expiry date
- `extractUsername(token)` → reads email from payload
- `extractRole(token)` → reads role from payload

**Secret key** is in `application.properties`:
```properties
jwt.secret=bliss-courier-secret-key-please-change-to-32-bytes-min
jwt.expiration.ms=3600000   # 1 hour
```

---

### 5.3 `JwtFilter.java` — The Gatekeeper
Extends `OncePerRequestFilter` — runs exactly once per HTTP request.

```
Every request hits this filter FIRST:

1. Read header:  Authorization: Bearer <token>
2. Extract token (remove "Bearer " prefix)
3. Validate token with JwtUtil
4. Extract email from token
5. Load full UserDetails from DB (CustomUserDetailsService)
6. Create UsernamePasswordAuthenticationToken
7. Put it in SecurityContextHolder
8. Pass request to next filter/controller

If no token or invalid token:
→ SecurityContext stays empty
→ SecurityConfig will reject the request (401 Unauthorized)

Skip filter for /auth/** routes (login/register don't need a token)
```

---

### 5.4 `CustomUserDetailsService.java` — User Loader
Spring Security calls this to load user details during authentication.

```java
loadUserByUsername(email)
    → queries UserRepository.findByEmail(email)
    → returns Spring Security User object with:
       - email (as username)
       - hashed password
       - List of GrantedAuthority ["ROLE_ADMIN"] or ["ROLE_USER"]
```

**Why it matters:** Spring Security uses this during login to compare the submitted password against the stored BCrypt hash.

---

### 5.5 `SecurityConfig.java` — The Rule Book
Defines what is allowed and what is not.

```java
// CSRF disabled — safe because we use stateless JWT, not cookies
csrf.disable()

// Sessions disabled — JWT is stateless, no server-side session
SessionCreationPolicy.STATELESS

// URL rules:
/auth/**        → anyone (no token needed)
GET  /**        → ADMIN or USER (needs valid token)
POST /**        → ADMIN only
PUT  /**        → ADMIN only
DELETE /**      → ADMIN only

// CORS — allows Angular (localhost:4200) to call the API
allowedOrigins: ["http://localhost:4200"]
allowedMethods: GET, POST, PUT, DELETE, OPTIONS

// Custom error responses:
401 Unauthorized → "Missing or invalid JWT"
403 Forbidden    → "You do not have permission"
```

---

### 5.6 `AuthController.java` — Register & Login Endpoints

**Register flow:**
```
POST /auth/register { email, password, role }
    │
    ├── Check if email already exists → throw RuntimeException if yes
    ├── BCryptPasswordEncoder.encode(password)
    ├── Prefix role: "ADMIN" → "ROLE_ADMIN"
    └── Save AppUser to DB → return saved user
```

**Login flow:**
```
POST /auth/login { email, password }
    │
    ├── AuthenticationManager.authenticate()
    │       └── calls CustomUserDetailsService.loadUserByUsername()
    │               └── BCrypt.matches(rawPassword, hashedPassword)
    │
    ├── If wrong password → AuthenticationException → 401
    │
    ├── JwtUtil.generateToken(email, role)
    └── Return { email, token, role }
```

---

## 6. FRONTEND SECURITY — ANGULAR SIDE

### 6.1 `AuthService` — Token Manager
```
login(email, password)
    → POST /auth/login
    → on success: store token in localStorage["bliss_token"]
    → store user info in localStorage["bliss_current_user"]

logout()
    → localStorage.clear()

isAuthenticated()
    → checks if token exists in localStorage

isAdmin()
    → reads role from localStorage, checks if "ADMIN"

getToken()
    → returns raw JWT string for the interceptor
```

### 6.2 `AuthInterceptor` — Auto Token Attachment
```
Every HTTP request Angular makes:
    │
    ├── AuthInterceptor.intercept() runs
    ├── Gets token from AuthService
    ├── Clones the request and adds header:
    │       Authorization: Bearer eyJhbGci...
    └── Passes modified request to backend
```
This means you never manually add the token in any service call — it's automatic.

### 6.3 `AuthGuard` — Route Protection
```
User tries to navigate to /home, /view-parcels, etc.
    │
    ├── AuthGuard.canActivate() runs
    ├── Checks AuthService.isAuthenticated()
    │       → checks localStorage for token
    ├── Token exists? → allow navigation
    └── No token?    → redirect to /login
```

### 6.4 `RoleGuard` — Admin Route Protection
```
User tries to navigate to /add-parcel or /add-company
    │
    ├── RoleGuard.canActivate() runs
    ├── Reads route.data["roles"] → ["ADMIN"]
    ├── Gets user role from localStorage
    ├── Role matches? → allow navigation
    └── Role doesn't match? → redirect to /home
```

### 6.5 Route Configuration
```typescript
{ path: '',            component: LandingComponent }           // public
{ path: 'login',       component: LoginComponent }             // public
{ path: 'register',    component: RegisterComponent }          // public
{ path: 'home',        canActivate: [AuthGuard] }              // logged in
{ path: 'view-parcels',canActivate: [AuthGuard] }              // logged in
{ path: 'add-parcel',  canActivate: [AuthGuard, RoleGuard],
                       data: { roles: ['ADMIN'] } }            // admin only
{ path: 'add-company', canActivate: [AuthGuard, RoleGuard],
                       data: { roles: ['ADMIN'] } }            // admin only
```

---

## 7. COMPLETE REQUEST LIFECYCLE (Frontend → Backend)

```
Example: Admin adds a parcel

1. ANGULAR FORM
   User fills Add Parcel form → clicks Submit
   AddParcelComponent.onSubmit()

2. ANGULAR SERVICE
   ParcelService.addParcel(companyId, payload)
   → HttpClient.post("http://localhost:4004/parcels/1", payload)

3. HTTP INTERCEPTOR
   AuthInterceptor intercepts the request
   → Adds header: Authorization: Bearer eyJhbGci...

4. BACKEND — JwtFilter
   Reads Authorization header
   → Strips "Bearer " prefix
   → JwtUtil.isTokenValid(token) → verifies HMAC signature
   → JwtUtil.isTokenExpired(token) → checks exp claim
   → JwtUtil.extractUsername(token) → gets "admin@bliss.com"
   → CustomUserDetailsService.loadUserByUsername("admin@bliss.com")
   → Sets Authentication in SecurityContextHolder

5. BACKEND — SecurityConfig
   Checks: POST /parcels/** → requires ROLE_ADMIN
   User has ROLE_ADMIN → ALLOWED

6. BACKEND — Controller
   ParcelController.addParcel(companyId, parcel)

7. BACKEND — Service
   ParcelService.addParcel()
   → Check if parcelId already exists (409 if yes)
   → Find company by companyId (404 if not found)
   → Set bookingDate = today
   → Calculate deliveryCharge
   → parcelRepository.save(parcel)

8. BACKEND — Response
   Returns 201 Created with saved Parcel JSON

9. ANGULAR
   AddParcelComponent receives success
   → Shows "Parcel added successfully!" inline message
   → Resets form
```

---

## 8. DATA MODELS

### CourierCompany
```
companyId (PK, String)
companyName
serviceType          → "Domestic" or "International"
headquartersLocation
contactNumber
rating
```

### Parcel
```
parcelId (PK, String)
trackingNumber
senderName
receiverName
destinationCity
weight (Double)
serviceType          → "Domestic" or "International"
deliveryStatus       → "Pending" | "In Transit" | "Delivered" | "Cancelled"
bookingDate          → auto-set to today on creation
deliveredDate        → optional
deliveryCharge       → auto-calculated:
                        Domestic:      50 + (weight × 10)
                        International: 200 + (weight × 25)
courierCompany (FK)  → ManyToOne → CourierCompany
```

### AppUser
```
id (PK, auto-increment)
email (unique)
password (BCrypt hashed)
role → "ROLE_ADMIN" or "ROLE_USER"
```

---

## 9. API ENDPOINTS

### Auth (Public)
| Method | URL | Description |
|--------|-----|-------------|
| POST | /auth/register | Register new user |
| POST | /auth/login | Login, returns JWT |

### Parcels (Protected)
| Method | URL | Role | Description |
|--------|-----|------|-------------|
| POST | /parcels/{companyId} | ADMIN | Add parcel |
| GET | /parcels/{parcelId} | ALL | Get by ID |
| GET | /parcels/tracking/{trackingNumber} | ALL | Get by tracking |
| PUT | /parcels/{parcelId}/{status} | ADMIN | Update status |
| GET | /parcels/destination/{city} | ALL | Filter by city |
| GET | /parcels/status/{status} | ALL | Filter by status |
| GET | /parcels/company/{companyName} | ALL | Filter by company |
| GET | /parcels/serviceType/{type}?fromDate=&toDate= | ALL | Filter by type + date |
| GET | /parcels/bookedAfter/{date} | ALL | Booked after date |
| GET | /parcels/weight/{weight} | ALL | Weight greater than |
| GET | /parcels/filter?destinationCity=&deliveryStatus= | ALL | Combined filter |

### Companies (Protected)
| Method | URL | Role | Description |
|--------|-----|------|-------------|
| POST | /companies | ADMIN | Add company |
| GET | /companies | ALL | View all |
| GET | /companies/{id} | ALL | View by ID |

---

## 10. EXPECTED INTERVIEW / PRESENTATION QUESTIONS

### Q1: What is JWT and why did you use it?
**Answer:** JWT (JSON Web Token) is a compact, self-contained token for securely transmitting information between parties. I used it because:
- It is **stateless** — the server doesn't store sessions, the token carries all info
- It is **signed** with HMAC-SHA256, so the server can verify it wasn't tampered with
- It works perfectly with REST APIs and Angular SPAs
- It contains the user's email and role, so the server knows who is making the request without a DB lookup every time

---

### Q2: How does the JWT token get verified on every request?
**Answer:** Through `JwtFilter` which extends `OncePerRequestFilter`:
1. Reads the `Authorization` header
2. Strips the `Bearer ` prefix
3. Calls `JwtUtil.isTokenValid()` — this re-signs the token with the secret key and compares signatures
4. Calls `JwtUtil.isTokenExpired()` — checks the `exp` claim
5. Extracts the username and loads the user from DB
6. Sets the `Authentication` object in `SecurityContextHolder`
7. Spring Security then checks the role against the URL rules in `SecurityConfig`

---

### Q3: What is BCrypt and why not store plain text passwords?
**Answer:** BCrypt is a password hashing function. It:
- Adds a random **salt** before hashing, so two identical passwords produce different hashes
- Is intentionally **slow** (configurable rounds), making brute-force attacks expensive
- Is **one-way** — you can never reverse a BCrypt hash back to the original password
- Spring's `BCryptPasswordEncoder.matches(raw, hashed)` is used during login to compare

Plain text passwords are never stored because if the database is compromised, all user passwords would be exposed immediately.

---

### Q4: What is the difference between Authentication and Authorization?
**Answer:**
- **Authentication** = Who are you? → Verified at login by checking email + BCrypt password
- **Authorization** = What can you do? → Checked on every request by `SecurityConfig` rules

In this project:
- Authentication happens in `AuthController.login()` via `AuthenticationManager`
- Authorization happens in `SecurityConfig` — GET is allowed for USER and ADMIN, POST/PUT/DELETE only for ADMIN

---

### Q5: What is CSRF and why is it disabled?
**Answer:** CSRF (Cross-Site Request Forgery) is an attack where a malicious site tricks a logged-in user's browser into making unwanted requests. It is a risk when using **cookie-based sessions** because browsers automatically send cookies.

In this project, CSRF is disabled because:
- We use **JWT in the Authorization header**, not cookies
- Browsers do NOT automatically send custom headers to other sites
- So CSRF attacks are not possible with this setup

---

### Q6: What is CORS and why is it configured?
**Answer:** CORS (Cross-Origin Resource Sharing) is a browser security policy that blocks requests from a different origin (domain/port). Angular runs on `localhost:4200` and Spring Boot on `localhost:4004` — different ports = different origins.

Without CORS config, the browser would block all API calls. The `CorsConfigurationSource` bean in `SecurityConfig` explicitly allows:
- Origin: `http://localhost:4200`
- Methods: GET, POST, PUT, DELETE, OPTIONS
- Headers: all

---

### Q7: What is SecurityContextHolder?
**Answer:** It is Spring Security's in-memory store for the current request's authentication. When `JwtFilter` validates a token, it creates a `UsernamePasswordAuthenticationToken` and stores it in `SecurityContextHolder`. For the rest of that request's lifecycle, Spring Security reads from here to know who the user is and what roles they have. It is cleared after each request because sessions are stateless.

---

### Q8: How does role-based access work end to end?
**Answer:**

**Backend:**
- `SecurityConfig` maps HTTP methods to roles: `POST → ROLE_ADMIN`
- When a request comes in, Spring reads the `Authentication` from `SecurityContextHolder`
- It checks if the user's `GrantedAuthority` list contains the required role
- If not → 403 Forbidden with a custom JSON error response

**Frontend:**
- `RoleGuard` checks `localStorage` for the user's role before allowing navigation
- If a USER tries to go to `/add-parcel`, `RoleGuard` redirects them to `/home`
- Even if they bypass the guard, the backend will reject the API call with 403

---

### Q9: What happens if the JWT token expires?
**Answer:**
- `JwtUtil.isTokenExpired()` checks the `exp` claim in the token payload
- `JwtFilter` will not set the `Authentication` in `SecurityContextHolder`
- The request reaches `SecurityConfig` with no authentication
- Spring returns 401 Unauthorized with the message "Missing or invalid JWT"
- On the Angular side, the user would need to log in again (token is 1 hour by default)

---

### Q10: Why is the session set to STATELESS?
**Answer:** `SessionCreationPolicy.STATELESS` tells Spring Security to never create or use an HTTP session. This is correct for JWT-based APIs because:
- The token itself carries all authentication info
- No server memory is used for sessions
- The API can scale horizontally (multiple servers) without session sharing
- Each request is fully self-contained

---

### Q11: What is OncePerRequestFilter?
**Answer:** It is a Spring base class that guarantees the filter runs exactly once per HTTP request, even in complex filter chains or request forwarding scenarios. `JwtFilter` extends it so the token validation logic runs once and only once per incoming request.

---

### Q12: How does the Angular interceptor work?
**Answer:** `AuthInterceptor` implements `HttpInterceptor`. Angular's `HttpClient` passes every outgoing request through all registered interceptors. The interceptor:
1. Gets the JWT from `AuthService.getToken()` (reads localStorage)
2. Clones the request (HTTP requests are immutable)
3. Adds the `Authorization: Bearer <token>` header to the clone
4. Passes the cloned request to `next.handle()`

This means every API call automatically has the token — no manual header management needed anywhere.

---

## 11. TECHNOLOGY DECISIONS — WHY THESE CHOICES

| Decision | Why |
|---|---|
| JWT over Sessions | Stateless, scalable, works with SPAs |
| BCrypt over MD5/SHA | Salted, slow by design, industry standard |
| Spring Security | Battle-tested, integrates with Spring Boot seamlessly |
| Stateless sessions | No server memory for sessions, horizontally scalable |
| CORS config | Required for Angular (different port) to call Spring API |
| Role prefix ROLE_ | Spring Security convention — `hasRole("ADMIN")` internally checks for `ROLE_ADMIN` |
| OncePerRequestFilter | Prevents double-execution of JWT validation |
| Custom error handler | Returns clean JSON instead of Spring's default HTML error page |

---

## 12. QUICK REFERENCE — KEY CLASSES

| Class | Package | Purpose |
|---|---|---|
| `SecurityConfig` | security.config | Defines all security rules, CORS, session policy |
| `JwtUtil` | security.jwt | Creates and validates JWT tokens |
| `JwtFilter` | security.jwt | Intercepts requests, validates token, sets auth context |
| `CustomUserDetailsService` | security.service | Loads user from DB for Spring Security |
| `AuthController` | security.controller | /auth/register and /auth/login endpoints |
| `AppUser` | security.model | User entity stored in DB |
| `UserRepository` | security.repository | DB queries for AppUser |
| `AuthService` | Angular service | Manages token in localStorage |
| `AuthInterceptor` | Angular interceptor | Attaches token to every HTTP request |
| `AuthGuard` | Angular guard | Blocks unauthenticated route access |
| `RoleGuard` | Angular guard | Blocks non-admin route access |

---

*Good luck with your presentation!*
