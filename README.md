<![CDATA[# ⚡ Sats-Check

> A hyper-verticalized Lightning Network attendance application for universities.
> Students check in to events and receive instant sats rewards via Lightning Address.

[![Next.js](https://img.shields.io/badge/Next.js-15-black?logo=next.js)](https://nextjs.org/)
[![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-3ECF8E?logo=supabase)](https://supabase.com/)
[![Lightning](https://img.shields.io/badge/Lightning-LNbits-F7931A?logo=bitcoin)](https://lnbits.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Folder Structure](#folder-structure)
- [Architecture](#architecture)
- [Database Schema](#database-schema)
- [API Endpoints](#api-endpoints)
- [Anti-Fraud System](#anti-fraud-system)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Development Workflow](#development-workflow)
- [Contributing](#contributing)

---

## Overview

**Sats-Check** lets university organizers create time- and location-bound events with a Bitcoin (sats) reward. Students scan a QR code, provide their Lightning Address, and receive **10–50 sats instantly** if they pass three validation gates:

1. **Time Gate** — Check-in must fall within the event's active window.
2. **Location Gate** — Student's GPS coordinates must be within the event's geofence radius.
3. **Device Gate** — Each physical device can only claim once per event (FingerprintJS).

---

## Key Features

| Category | Feature |
|---|---|
| **Organizer** | Create events with time window, geofence, and sats-reward |
| **Organizer** | Generate shareable QR code per event |
| **Organizer** | Real-time dashboard with claim feed & stats |
| **Student** | Mobile-first, high-contrast check-in page |
| **Student** | Lightning Address input with instant payout |
| **Student** | Live payment status feedback (Pending → Success / Failed) |
| **Security** | FingerprintJS device fingerprinting |
| **Security** | Browser Geolocation API with haversine distance check |
| **Security** | Server-side time-window enforcement |
| **Payments** | LNbits API for automated outbound micro-payments |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | **Next.js 15** (App Router, JavaScript) |
| Styling | **Tailwind CSS** — Black `#000000`, Bitcoin Orange `#F7931A`, Yellow `#FFD700` |
| Database | **Supabase** (PostgreSQL + Row Level Security) |
| Auth | **Supabase Auth** (Email/Password for organizers) |
| Payments | **LNbits API** (Lightning Network micro-payments) |
| Device Fingerprint | **FingerprintJS** (visitor identification) |
| Geolocation | **Browser Geolocation API** + server-side haversine |
| QR Codes | **qrcode.react** |
| Deployment | **Vercel** |

---

## Folder Structure

```
sats-check/
├── public/
│   ├── fonts/                        # Custom font files
│   ├── images/
│   │   ├── logo.svg                  # App logo
│   │   ├── bitcoin-icon.svg          # Bitcoin/Lightning icons
│   │   └── og-image.png              # Open Graph social preview
│   └── favicon.ico
│
├── src/
│   ├── app/                          # Next.js App Router
│   │   ├── layout.js                 # Root layout (providers, global styles)
│   │   ├── page.js                   # Landing / marketing page
│   │   ├── globals.css               # Tailwind directives + custom properties
│   │   │
│   │   ├── (auth)/                   # Auth route group (no layout nesting)
│   │   │   ├── login/
│   │   │   │   └── page.js           # Organizer login
│   │   │   └── register/
│   │   │       └── page.js           # Organizer registration
│   │   │
│   │   ├── dashboard/                # Organizer dashboard (protected)
│   │   │   ├── layout.js             # Dashboard shell (sidebar, auth guard)
│   │   │   ├── page.js               # Overview: active events + stats
│   │   │   ├── events/
│   │   │   │   ├── page.js           # List all events
│   │   │   │   ├── new/
│   │   │   │   │   └── page.js       # Create event form
│   │   │   │   └── [eventId]/
│   │   │   │       ├── page.js       # Event detail + live claim feed
│   │   │   │       ├── edit/
│   │   │   │       │   └── page.js   # Edit event
│   │   │   │       └── qr/
│   │   │   │           └── page.js   # Full-screen QR display
│   │   │   └── settings/
│   │   │       └── page.js           # Organizer profile + LNbits config
│   │   │
│   │   ├── checkin/                   # Student check-in (public)
│   │   │   └── [eventId]/
│   │   │       └── page.js           # Mobile-first check-in page
│   │   │
│   │   └── api/                      # API Route Handlers
│   │       ├── events/
│   │       │   ├── route.js           # GET (list) / POST (create)
│   │       │   └── [eventId]/
│   │       │       └── route.js       # GET / PATCH / DELETE single event
│   │       ├── claims/
│   │       │   ├── route.js           # POST (submit claim)
│   │       │   └── [eventId]/
│   │       │       └── route.js       # GET claims for an event
│   │       ├── payments/
│   │       │   ├── send/
│   │       │   │   └── route.js       # POST trigger LNbits payout
│   │       │   └── status/
│   │       │       └── [paymentId]/
│   │       │           └── route.js   # GET payment status from LNbits
│   │       └── validate/
│   │           ├── location/
│   │           │   └── route.js       # POST validate geolocation
│   │           ├── device/
│   │           │   └── route.js       # POST validate device fingerprint
│   │           └── time/
│   │               └── route.js       # POST validate time window
│   │
│   ├── components/                    # Reusable UI components
│   │   ├── ui/                        # Primitives (design system)
│   │   │   ├── Button.js
│   │   │   ├── Input.js
│   │   │   ├── Card.js
│   │   │   ├── Badge.js
│   │   │   ├── Modal.js
│   │   │   ├── Spinner.js
│   │   │   ├── Toast.js
│   │   │   └── Skeleton.js
│   │   ├── layout/
│   │   │   ├── Navbar.js              # Top navigation
│   │   │   ├── Sidebar.js             # Dashboard sidebar
│   │   │   ├── Footer.js
│   │   │   └── MobileNav.js           # Bottom nav for mobile
│   │   ├── events/
│   │   │   ├── EventCard.js           # Event summary card
│   │   │   ├── EventForm.js           # Create/Edit event form
│   │   │   ├── EventList.js           # Paginated event list
│   │   │   └── QRCodeDisplay.js       # QR code renderer
│   │   ├── checkin/
│   │   │   ├── CheckinForm.js         # Lightning Address input + submit
│   │   │   ├── LocationGate.js        # Geolocation permission + status
│   │   │   ├── PaymentStatus.js       # Pending → Success/Failed animation
│   │   │   └── EventBanner.js         # Event info header on check-in page
│   │   ├── dashboard/
│   │   │   ├── StatsGrid.js           # Key metrics (claims, sats sent, etc.)
│   │   │   ├── ClaimFeed.js           # Real-time claim activity
│   │   │   └── ClaimTable.js          # Tabular claim history
│   │   └── auth/
│   │       ├── LoginForm.js
│   │       └── RegisterForm.js
│   │
│   ├── hooks/                         # Custom React hooks
│   │   ├── useAuth.js                 # Supabase auth state
│   │   ├── useEvents.js               # CRUD operations for events
│   │   ├── useClaims.js               # Claim submission + listing
│   │   ├── useGeolocation.js          # Browser Geolocation API wrapper
│   │   ├── useFingerprint.js          # FingerprintJS integration
│   │   ├── usePayment.js             # LNbits payment trigger + polling
│   │   └── useRealtimeClaims.js       # Supabase Realtime subscription
│   │
│   ├── lib/                           # Core utilities & service clients
│   │   ├── supabase/
│   │   │   ├── client.js             # Browser Supabase client
│   │   │   ├── server.js             # Server-side Supabase client
│   │   │   └── middleware.js          # Auth middleware for protected routes
│   │   ├── lnbits/
│   │   │   ├── client.js             # LNbits API client (axios wrapper)
│   │   │   └── utils.js              # Lightning Address validation + helpers
│   │   ├── fingerprint/
│   │   │   └── client.js             # FingerprintJS initialization
│   │   ├── geo/
│   │   │   └── haversine.js          # Haversine distance calculation
│   │   └── validators/
│   │       ├── event.js              # Event form validation (Zod schemas)
│   │       ├── claim.js              # Claim payload validation
│   │       └── lightning.js          # Lightning Address format validation
│   │
│   ├── constants/
│   │   ├── config.js                 # App-wide constants (geofence defaults, etc.)
│   │   └── routes.js                 # Route path constants
│   │
│   ├── styles/
│   │   └── themes.js                 # Extended Tailwind theme tokens
│   │
│   └── utils/
│       ├── formatters.js             # Date, sats, address formatters
│       ├── errors.js                 # Custom error classes + handler
│       └── api.js                    # Fetch wrapper with error handling
│
├── supabase/
│   ├── migrations/                    # SQL migration files
│   │   ├── 001_create_events.sql
│   │   ├── 002_create_claims.sql
│   │   └── 003_create_rls_policies.sql
│   └── seed.sql                       # Development seed data
│
├── .env.local.example                 # Template for environment variables
├── .eslintrc.json
├── .prettierrc
├── .gitignore
├── jsconfig.json                      # Path aliases (@/ → src/)
├── next.config.js
├── tailwind.config.js
├── postcss.config.js
├── package.json
└── LICENSE
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SATS-CHECK ARCHITECTURE                      │
└─────────────────────────────────────────────────────────────────────┘

  ┌──────────────┐         ┌──────────────┐
  │   Organizer  │         │   Student    │
  │   (Desktop)  │         │   (Mobile)   │
  └──────┬───────┘         └──────┬───────┘
         │                        │
         │  Dashboard UI          │  Check-in UI
         │  (Protected)           │  (Public)
         ▼                        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    NEXT.JS APP ROUTER (Vercel)                      │
│                                                                     │
│  ┌─────────────────┐  ┌──────────────────┐  ┌───────────────────┐  │
│  │  /dashboard/*   │  │  /checkin/[id]   │  │   /api/*          │  │
│  │  Event CRUD     │  │  Mobile-first    │  │   Route Handlers  │  │
│  │  QR Generation  │  │  High contrast   │  │   Validation      │  │
│  │  Live Feed      │  │  Lightning input │  │   Payment trigger │  │
│  └────────┬────────┘  └────────┬─────────┘  └─────────┬─────────┘  │
│           │                    │                       │            │
│           └────────────────────┼───────────────────────┘            │
│                                │                                    │
│  ┌─────────────────────────────┼──────────────────────────────────┐ │
│  │               VALIDATION PIPELINE (Server-side)               │ │
│  │                                                                │ │
│  │   ① Time Gate ──→ ② Location Gate ──→ ③ Device Gate          │ │
│  │   (event window)   (haversine ≤ r)    (fingerprint unique)   │ │
│  │                                                                │ │
│  │   All three must pass ──→ Claim approved ──→ Trigger payout  │ │
│  └────────────────────────────────────────────────────────────────┘ │
└─────────────────────┬───────────────────────────┬──────────────────┘
                      │                           │
           ┌──────────▼──────────┐     ┌──────────▼──────────┐
           │     SUPABASE        │     │      LNBITS         │
           │                     │     │                     │
           │  ┌───────────────┐  │     │  Lightning Network  │
           │  │    events     │  │     │  Micro-payments     │
           │  ├───────────────┤  │     │                     │
           │  │    claims     │  │     │  POST /api/v1/      │
           │  ├───────────────┤  │     │    payments         │
           │  │  profiles     │  │     │                     │
           │  └───────────────┘  │     │  Auto-payout to     │
           │                     │     │  Lightning Address  │
           │  Auth (JWT)         │     │                     │
           │  Realtime (WS)      │     └─────────────────────┘
           │  RLS Policies       │
           └─────────────────────┘
```

### Request Flow — Student Check-in

```
Student scans QR → /checkin/[eventId]
    │
    ├─ 1. FingerprintJS generates visitorId (client-side)
    ├─ 2. Browser Geolocation API captures lat/lng
    ├─ 3. Student enters Lightning Address
    ├─ 4. POST /api/claims
    │       │
    │       ├─ Validate time:   now ∈ [starts_at, ends_at]?
    │       ├─ Validate location: haversine(student, event) ≤ radius?
    │       ├─ Validate device:  visitorId not in claims for this event?
    │       ├─ Validate Lightning Address format (user@domain.com)
    │       │
    │       ├─ ✅ All pass → Insert claim (status: "pending")
    │       │               → POST LNbits payout
    │       │               → Update claim (status: "success" | "failed")
    │       │
    │       └─ ❌ Any fail → Return specific error message
    │
    └─ 5. UI shows PaymentStatus: Pending ⏳ → Success ⚡ / Failed ❌
```

---

## Database Schema

### `events` Table

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `uuid` | `PK, DEFAULT gen_random_uuid()` | Unique event identifier |
| `organizer_id` | `uuid` | `FK → auth.users(id), NOT NULL` | Creator of the event |
| `title` | `text` | `NOT NULL` | Event name |
| `description` | `text` | | Optional description |
| `sats_reward` | `integer` | `NOT NULL, CHECK (10 ≤ val ≤ 50)` | Sats per successful check-in |
| `total_budget_sats` | `integer` | `NOT NULL, CHECK (val > 0)` | Total sats allocated to the event |
| `sats_spent` | `integer` | `DEFAULT 0` | Running total of sats paid out |
| `max_claims` | `integer` | | Optional cap on number of claims |
| `starts_at` | `timestamptz` | `NOT NULL` | Event check-in window start |
| `ends_at` | `timestamptz` | `NOT NULL, CHECK (ends_at > starts_at)` | Event check-in window end |
| `latitude` | `double precision` | `NOT NULL` | Event venue latitude |
| `longitude` | `double precision` | `NOT NULL` | Event venue longitude |
| `radius_meters` | `integer` | `NOT NULL, DEFAULT 100` | Geofence radius in meters |
| `is_active` | `boolean` | `DEFAULT true` | Whether event accepts check-ins |
| `created_at` | `timestamptz` | `DEFAULT now()` | Record creation timestamp |
| `updated_at` | `timestamptz` | `DEFAULT now()` | Last update timestamp |

### `claims` Table

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `uuid` | `PK, DEFAULT gen_random_uuid()` | Unique claim identifier |
| `event_id` | `uuid` | `FK → events(id), NOT NULL` | Associated event |
| `lightning_address` | `text` | `NOT NULL` | Student's Lightning Address |
| `device_fingerprint` | `text` | `NOT NULL` | FingerprintJS visitorId |
| `latitude` | `double precision` | `NOT NULL` | Student's lat at check-in |
| `longitude` | `double precision` | `NOT NULL` | Student's lng at check-in |
| `distance_meters` | `double precision` | | Calculated distance from event |
| `sats_amount` | `integer` | `NOT NULL` | Sats awarded for this claim |
| `payment_status` | `text` | `NOT NULL, DEFAULT 'pending'` | `pending` · `success` · `failed` |
| `payment_hash` | `text` | | LNbits payment hash |
| `failure_reason` | `text` | | Reason if validation/payment failed |
| `claimed_at` | `timestamptz` | `DEFAULT now()` | Timestamp of claim submission |

#### Indexes

```sql
-- Fast lookups for duplicate-device checks
CREATE UNIQUE INDEX idx_claims_event_device
  ON claims (event_id, device_fingerprint);

-- Dashboard queries: claims per event ordered by time
CREATE INDEX idx_claims_event_time
  ON claims (event_id, claimed_at DESC);

-- Payment status monitoring
CREATE INDEX idx_claims_payment_status
  ON claims (payment_status)
  WHERE payment_status = 'pending';
```

### `profiles` Table (extends Supabase Auth)

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `uuid` | `PK, FK → auth.users(id)` | Maps to Supabase Auth user |
| `display_name` | `text` | | Organizer's display name |
| `university` | `text` | | University affiliation |
| `lnbits_url` | `text` | | LNbits instance URL |
| `lnbits_admin_key` | `text` | | Encrypted LNbits admin API key |
| `created_at` | `timestamptz` | `DEFAULT now()` | Record creation timestamp |

### Row Level Security (RLS) Policies

```sql
-- Events: organizers can CRUD their own events; anyone can read active events
ALTER TABLE events ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can view active events"
  ON events FOR SELECT
  USING (is_active = true);

CREATE POLICY "Organizers manage own events"
  ON events FOR ALL
  USING (auth.uid() = organizer_id);

-- Claims: public can insert (check-in); organizers can read their event claims
ALTER TABLE claims ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Anyone can submit a claim"
  ON claims FOR INSERT
  WITH CHECK (true);  -- Server-side validation handles security

CREATE POLICY "Organizers view their event claims"
  ON claims FOR SELECT
  USING (
    event_id IN (
      SELECT id FROM events WHERE organizer_id = auth.uid()
    )
  );

-- Profiles: users manage their own profile
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users manage own profile"
  ON profiles FOR ALL
  USING (auth.uid() = id);
```

### Entity Relationship Diagram

```
┌──────────────┐       ┌──────────────────┐       ┌──────────────┐
│  auth.users  │       │     events       │       │    claims    │
│──────────────│       │──────────────────│       │──────────────│
│ id (PK)      │◄──┐   │ id (PK)          │◄──┐   │ id (PK)      │
│ email        │   │   │ organizer_id (FK)─┘   │   │ event_id (FK)┘
│ ...          │   │   │ title             │   │   │ lightning_   │
└──────────────┘   │   │ sats_reward       │   │   │   address    │
                   │   │ starts_at         │   └───│ device_      │
┌──────────────┐   │   │ ends_at           │       │   fingerprint│
│  profiles    │   │   │ latitude          │       │ payment_     │
│──────────────│   │   │ longitude         │       │   status     │
│ id (PK/FK)───┘   │   │ radius_meters     │       │ claimed_at   │
│ display_name │   │   │ is_active         │       └──────────────┘
│ university   │   │   │ ...               │
│ lnbits_url   │   │   └──────────────────┘
│ lnbits_      │   │
│  admin_key   │   │
└──────────────┘   │
       │           │
       └───────────┘
       1:1
```

---

## API Endpoints

### Events

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `GET` | `/api/events` | Organizer | List organizer's events |
| `POST` | `/api/events` | Organizer | Create a new event |
| `GET` | `/api/events/[eventId]` | Public* | Get event details (*public fields only for students) |
| `PATCH` | `/api/events/[eventId]` | Organizer | Update event |
| `DELETE` | `/api/events/[eventId]` | Organizer | Soft-delete event |

### Claims

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/api/claims` | Public | Submit a check-in claim |
| `GET` | `/api/claims/[eventId]` | Organizer | Get all claims for an event |

**POST `/api/claims` — Request Body:**

```json
{
  "event_id": "uuid",
  "lightning_address": "student@walletofsatoshi.com",
  "device_fingerprint": "fp_abc123xyz",
  "latitude": 12.9716,
  "longitude": 77.5946
}
```

**POST `/api/claims` — Response (Success):**

```json
{
  "success": true,
  "claim": {
    "id": "uuid",
    "sats_amount": 21,
    "payment_status": "pending",
    "payment_hash": "lnbc..."
  }
}
```

**POST `/api/claims` — Response (Failure):**

```json
{
  "success": false,
  "error": {
    "code": "DEVICE_ALREADY_CLAIMED",
    "message": "This device has already checked in to this event."
  }
}
```

**Error Codes:**

| Code | HTTP | Description |
|---|---|---|
| `EVENT_NOT_FOUND` | 404 | Event doesn't exist |
| `EVENT_INACTIVE` | 403 | Event is deactivated |
| `EVENT_NOT_STARTED` | 403 | Current time is before `starts_at` |
| `EVENT_ENDED` | 403 | Current time is after `ends_at` |
| `OUTSIDE_GEOFENCE` | 403 | Student is outside allowed radius |
| `DEVICE_ALREADY_CLAIMED` | 409 | Device fingerprint already used |
| `INVALID_LIGHTNING_ADDRESS` | 422 | Malformed Lightning Address |
| `BUDGET_EXHAUSTED` | 403 | Event's sats budget is fully spent |
| `PAYMENT_FAILED` | 502 | LNbits payout failed |

### Payments

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/api/payments/send` | Internal | Trigger LNbits payout (server-only) |
| `GET` | `/api/payments/status/[paymentId]` | Internal | Poll LNbits payment status |

### Validation (Internal)

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/validate/time` | Check if current time is within event window |
| `POST` | `/api/validate/location` | Haversine distance check |
| `POST` | `/api/validate/device` | Fingerprint uniqueness check |

---

## Anti-Fraud System

### Three-Gate Validation Pipeline

```
Request ──→ ① TIME GATE ──→ ② LOCATION GATE ──→ ③ DEVICE GATE ──→ ✅ Approved
               │                   │                    │
               ❌                  ❌                   ❌
          "Event not active"  "Outside geofence"  "Already claimed"
```

| Gate | Method | Details |
|---|---|---|
| **Time** | Server-side `Date` comparison | `starts_at ≤ now() ≤ ends_at` |
| **Location** | Haversine formula | Distance ≤ `radius_meters` |
| **Device** | FingerprintJS `visitorId` | Unique index on `(event_id, device_fingerprint)` |

### Why FingerprintJS?

- Browser-level device identification without user accounts
- Persists across incognito/private browsing modes
- 99.5% identification accuracy
- Students don't need to create accounts — frictionless UX

---

## Getting Started

### Prerequisites

- **Node.js** ≥ 18.x
- **npm** or **pnpm**
- **Supabase** account ([supabase.com](https://supabase.com))
- **LNbits** instance ([lnbits.com](https://lnbits.com)) or self-hosted
- **FingerprintJS** API key ([fingerprint.com](https://fingerprint.com))

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/sats-check.git
cd sats-check

# Install dependencies
npm install

# Copy environment variables
cp .env.local.example .env.local
# → Fill in your Supabase, LNbits, and FingerprintJS credentials

# Run Supabase migrations (if using Supabase CLI)
npx supabase db push

# Start the development server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## Environment Variables

```bash
# .env.local.example

# ── Supabase ──────────────────────────────────────
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# ── LNbits ────────────────────────────────────────
LNBITS_URL=https://your-lnbits-instance.com
LNBITS_ADMIN_KEY=your-admin-api-key

# ── FingerprintJS ─────────────────────────────────
NEXT_PUBLIC_FINGERPRINT_API_KEY=your-fingerprint-api-key

# ── App Config ────────────────────────────────────
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_DEFAULT_GEOFENCE_RADIUS=100
```

---

## Development Workflow

This project follows a **modular, 100-commit strategy** — every logical unit of work gets its own commit.

### Branch Naming

```
feature/  → feature/event-creation-form
fix/      → fix/geofence-validation-edge-case
chore/    → chore/update-tailwind-config
```

### Commit Convention

```
feat(events): add event creation API route
feat(checkin): implement geolocation permission flow
fix(payments): handle LNbits timeout gracefully
style(ui): update button hover states for orange theme
```

---

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes following the commit convention above
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

<p align="center">
  Built with ⚡ for the Bitcoin community
</p>
]]>