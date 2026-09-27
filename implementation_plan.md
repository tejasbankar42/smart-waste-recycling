# Hackathon Implementation Plan - Smart Waste Collection & Recycling Platform (EcoCollect)

## Objective & Hackathon Strategy
Build a production-grade, highly presentable MVP of **EcoCollect** designed for college hackathon judging:
- **Core Value**: Bridges the gap between households disposing waste and haulers managing collection logistics.
- **Demo Appeal**: 1-click demo logins, live visual status progression, interactive Recharts analytics, waste disposal cheat-sheet, and unique tracking IDs.
- **Execution Speed**: High-velocity architecture using FastAPI + SQLite on the backend and React + Tailwind CSS + Recharts on the frontend.

---

## 1. System Architecture

```
                    +--------------------------------------------+
                    |          React 18 + Tailwind CSS           |
                    |  - Home & Quick Tracking Search            |
                    |  - Interactive Waste Disposal Guide        |
                    |  - User Portal (Booking & Status Timeline) |
                    |  - Admin Command Center (Recharts & Queue) |
                    +---------------------+----------------------+
                                          |
                                          | REST API (JSON)
                                          | Bearer JWT Auth
                                          v
                    +--------------------------------------------+
                    |            FastAPI Backend Engine          |
                    |  - /api/auth (JWT, Roles: User/Admin)      |
                    |  - /api/requests (Booking & Tracking)      |
                    |  - /api/guide (Disposal knowledge base)    |
                    |  - /api/admin (Stats, Search, Filters)     |
                    +---------------------+----------------------+
                                          |
                                          | SQLAlchemy ORM
                                          v
                    +--------------------------------------------+
                    |           SQLite Database Layer            |
                    |  - users                                   |
                    |  - collection_requests                     |
                    |  - Auto-seeder for rich demo state         |
                    +--------------------------------------------+
```

---

## 2. Database Schema

### `users` Table
| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | Integer | PK, Auto | Unique user ID |
| `email` | String(255) | Unique, Indexed | User login email |
| `hashed_password` | String(255) | Not Null | Salted PBKDF2 hash |
| `full_name` | String(255) | Not Null | User's full name |
| `role` | String(50) | Default: `'user'` | Access role: `'user'` or `'admin'` |
| `phone` | String(50) | Nullable | Contact phone for pickups |
| `address` | String(255) | Nullable | Default street address |
| `created_at` | DateTime | Default: `now()` | Registration timestamp |

### `collection_requests` Table
| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | Integer | PK, Auto | Internal primary key |
| `tracking_id` | String(30) | Unique, Indexed | Formatted public tracking ID (e.g., `REQ-8F2A1C`) |
| `user_id` | Integer | FK(`users.id`) | Submitting user ID |
| `waste_category` | String(50) | Not Null | `Organic`, `Plastic`, `Paper`, `Glass`, `Metal`, `E-Waste`, `Hazardous` |
| `quantity_desc` | String(255) | Not Null | Quantity / packaging description (e.g. `3 bags / ~15 kg`) |
| `pickup_address` | String(255) | Not Null | Doorstep collection address |
| `pickup_date` | String(20) | Not Null | Scheduled collection date (`YYYY-MM-DD`) |
| `pickup_time_slot` | String(50) | Not Null | Slot (e.g., `09:00 - 12:00`, `13:00 - 16:00`, `16:00 - 19:00`) |
| `special_notes` | Text | Nullable | Gate codes, landmark, packaging notes |
| `status` | String(50) | Default: `'Pending'` | `Pending` → `Accepted` → `Scheduled` → `Picked Up` → `Completed` (or `Cancelled`) |
| `cancellation_reason` | String(255) | Nullable | Reason if cancelled |
| `created_at` | DateTime | Default: `now()` | Submission timestamp |
| `updated_at` | DateTime | Default: `now()` | Last status transition timestamp |

---

## 3. API Endpoints

### 🔐 Authentication (`/api/auth`)
- `POST /api/auth/register`: Create user account, returns JWT token & user object.
- `POST /api/auth/login`: Authenticate email & password, returns JWT token & user object.
- `GET /api/auth/me`: Current session profile check via Bearer token.

### 📦 Collection Requests (`/api/requests`)
- `POST /api/requests/`: Create pickup request; generates human-readable unique tracking ID (`REQ-XXXXXX`).
- `GET /api/requests/`: Fetch all pickup requests belonging to the authenticated user.
- `GET /api/requests/track/{tracking_id}`: Public or authenticated tracking endpoint returning request status & timeline.
- `PATCH /api/requests/{id}/cancel`: User cancellation endpoint (allowed for `Pending` / `Accepted` stages).

### 📘 Waste Disposal Guide (`/api/guide`)
- `GET /api/guide`: Complete directory of the 7 waste categories with accepted items, prohibited items, recycling instructions, and ecological impact score.

### 🛡️ Admin Management (`/api/admin`)
- `GET /api/admin/requests`: Query all platform requests with combined query filters:
  - `q`: Search by Tracking ID, user name, email, or address
  - `category`: Filter by waste stream
  - `status`: Filter by status (`Pending`, `Accepted`, `Scheduled`, `Picked Up`, `Completed`, `Cancelled`)
  - `date`: Filter by scheduled pickup date
- `GET /api/admin/requests/{id}`: Detailed view of a single request.
- `PATCH /api/admin/requests/{id}/status`: Advance status (`Accepted`, `Scheduled`, `Picked Up`, `Completed`, `Cancelled`).
- `GET /api/admin/stats`: Aggregated metrics for Recharts visualizations:
  - Total requests, Pending, Scheduled, Completed, Cancelled
  - Category breakdown distribution (e.g. Plastic: 35%, E-Waste: 20%, etc.)
  - 7-day pickup volume timeline

---

## 4. Frontend Pages

1. **Home / Landing Page (`/`)**:
   - Hero header with value proposition.
   - **Live Tracking Search Box**: Enter any tracking code (e.g. `REQ-8F2A1C`) to view live status on the spot without signing in.
   - Quick waste categories preview with direct link to Guide.
   - Quick "How It Works" 3-step diagram.

2. **Waste Disposal Guide (`/guide`)**:
   - Category selector tabs (Organic, Plastic, Paper, Glass, Metal, E-Waste, Hazardous).
   - "What Goes In" (Do's) vs "What Stays Out" (Don'ts).
   - Preparation rules (clean/dry plastics, tape battery terminals, fold cardboard).

3. **Authentication Pages (`/login`, `/register`)**:
   - Clean login with **1-Click Demo Buttons** (`Demo Resident` and `Demo Admin`) to streamline hackathon demonstration.

4. **User Dashboard (`/dashboard`)**:
   - **Active Request Spotlight**: Visual 5-step stepper tracker showing progress from *Pending* to *Completed*.
   - **Schedule New Pickup Button** (triggers modal).
   - **Pickup History Table**: Search, status badge, tracking ID copy button, and cancellation action.

5. **Admin Operations Portal (`/admin`)**:
   - **KPI Summary Cards**: Total Pickups, Pending Dispatch, Completed Today, Active Recyclers.
   - **Interactive Analytics (Recharts)**:
     - Donut chart: Waste stream breakdown.
     - Bar / Area chart: Pickup requests volume and completion trends.
   - **Operations Queue Table**:
     - Real-time search bar (by Tracking ID, User, or Location).
     - Filter dropdowns: Category, Status, Date.
     - Inline status dropdown selector to transition requests in 1 click.

---

## 5. Component Structure

```
src/
├── components/
│   ├── layout/
│   │   ├── Navbar.jsx            # Dynamic links, Demo switcher, Tracking search link
│   │   └── Footer.jsx            # Hackathon credits & eco mission
│   ├── ui/
│   │   ├── StatusBadge.jsx       # Color-coded pill for all 6 statuses
│   │   ├── StatusStepper.jsx     # 5-stage visual progress bar with checkmarks
│   │   └── StatCard.jsx          # Reusable KPI counter with trend indicator
│   ├── requests/
│   │   ├── RequestModal.jsx      # Multi-step booking form (Category -> Details -> Date/Time -> Confirm)
│   │   ├── RequestCard.jsx       # Compact card representation for history
│   │   └── RequestDetailModal.jsx# Full details modal for Admin inspect
│   ├── analytics/
│   │   ├── CategoryPieChart.jsx  # Recharts Pie/Donut for waste breakdown
│   │   └── PickupTrendChart.jsx  # Recharts Bar/Area for collection volume
│   └── common/
│       └── ProtectedRoute.jsx    # Guard for User & Admin views
├── pages/
│   ├── Home.jsx
│   ├── Guide.jsx
│   ├── Login.jsx
│   ├── Register.jsx
│   ├── Dashboard.jsx
│   └── AdminDashboard.jsx
├── services/
│   └── api.js                    # Fetch / Axios wrapper with token handling
├── App.jsx                       # Routing & Auth state
└── main.jsx
```

---

## 6. Development Phases

- **Phase 1: Backend API & Auto-Seeding (FastAPI + SQLite)**
  - Implement models with unique formatted `tracking_id` (`REQ-XXXXXX`).
  - Implement full status lifecycle: `Pending` → `Accepted` → `Scheduled` → `Picked Up` → `Completed` (+ `Cancelled`).
  - Create database seeder with 6+ realistic sample requests across all stages so the demo looks active immediately upon launch.
- **Phase 2: Frontend Design System & Disposal Guide (React + Tailwind)**
  - Set up Tailwind CSS with emerald/teal eco palette.
  - Build the comprehensive Waste Disposal Guide page with category filters and practical tips.
- **Phase 3: User Booking & Tracking Flow**
  - Implement Schedule Pickup Modal with category selector and time slot choices.
  - Build `StatusStepper` component showing visual milestone progression for any request.
  - Add public Quick Tracking bar on the landing page.
- **Phase 4: Admin Portal with Search, Filters & Recharts**
  - Implement Admin table with multi-filter queries (Search text, Category, Status, Date).
  - Integrate **Recharts** charts for Category distribution and Pickup statistics.
  - Add instant status-transition controls.
- **Phase 5: Hackathon Demo Hardening & Verification**
  - Verify all 10 user features and 10 admin features end-to-end.
  - Test responsiveness and zero-friction demo login.

---

## 7. Potential Risks / Bugs & Mitigations

| Risk / Bug | Why It Happens | Mitigation |
|---|---|---|
| **Date / Timezone desync** | Browsers sending local datetime vs UTC parsing in Python | Use discrete date strings (`YYYY-MM-DD`) and discrete time slot enums (`09:00 - 12:00`, `13:00 - 16:00`). |
| **Empty Chart crashes** | Recharts can break or look empty if no data exists | Pre-seed database with realistic multi-category records on server startup; add empty state fallback UI. |
| **Judges forgetting demo logins** | Manual typing mistakes during live judge demo | Provide prominent "1-Click Demo User" and "1-Click Demo Admin" buttons on the login screen. |
| **Invalid status progression** | Admin accidentally skipping stages or cancelling completed jobs | Server-side validation enforcing status flow logic; UI disables invalid transitions. |
| **SQLite database concurrency** | Multi-threaded requests locking SQLite file | Configure SQLAlchemy engine with `connect_args={"check_same_thread": False}` and scoped sessions. |

---

## 8. Recommended MVP Scope vs Optional Features

### 🌟 Core MVP Features (Must-Have for Demo - Recommended)
1. **User Auth & Role Management** (Resident vs Administrator, with 1-click demo logins).
2. **7 Waste Categories** (Organic, Plastic, Paper, Glass, Metal, E-Waste, Hazardous) with category badges and icons.
3. **Pickup Booking Modal** (Location, Date, Time Slot, Quantity description, Special instructions).
4. **Unique Human-Friendly Tracking ID** (`REQ-XXXXXX` generated per request).
5. **Interactive 5-Stage Status Stepper** (`Pending` → `Accepted` → `Scheduled` → `Picked Up` → `Completed`, plus `Cancelled`).
6. **User Pickup History & Cancellation** (Cancel active requests).
7. **Waste Disposal Guide Page** (Searchable rules, Do's & Don'ts per category).
8. **Admin Operations Center** (View all requests, search by text, 3 filters: category, status, date).
9. **Admin Inline Status Updater** (Advance request from queue with instant UI feedback).
10. **Admin Recharts Dashboard** (Waste stream donut chart, volume trends, total kg/volume metrics).
11. **Automated Seeder** (Pre-fills sample data so charts and history look populated out of the box).

### 🔮 Optional / Post-Hackathon Bonus Features (Skip for MVP)
- Real-time GPS truck routing simulation on Leaflet/Google Maps.
- Live SMS / WhatsApp notification via Twilio.
- Camera image upload with AI waste recognition (keep image upload mock or optional text description for MVP to save time).
- Gamified loyalty reward points system.

---

## Verification Plan

### Automated Verification
- Backend unit tests testing:
  - Auth registration and login.
  - Creation of request with generated `tracking_id`.
  - Status transition validation (Pending -> Accepted -> Scheduled -> Picked Up -> Completed).
  - Admin filtering by category, status, date, and search keyword.
- Frontend build verification:
  - `npm run build` cleanly compiling React + Tailwind + Recharts with zero lint or JSX errors.

### Demonstration Walkthrough
- **Flow 1: User Experience**:
  1. Click "Demo Resident" on login.
  2. Explore the Waste Disposal Guide to learn how to prepare E-Waste.
  3. Schedule an E-Waste collection with quantity, location, date, and time slot.
  4. Obtain tracking code `REQ-XXXXXX` and observe request in "Pending" status with visual milestone stepper.
- **Flow 2: Admin Experience**:
  1. Switch to "Demo Admin" in 1 click.
  2. Inspect Recharts charts showing category distribution and weekly pickup volume.
  3. Use filters to narrow down requests by category ("E-Waste") and status ("Pending").
  4. Advance status to "Accepted" → "Scheduled" → "Picked Up" → "Completed".
  5. Verify user dashboard reflects the live updated stage immediately.
