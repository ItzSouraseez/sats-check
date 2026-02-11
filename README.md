# Sats-Check

A Lightning Network attendance application for universities. Organizers create time-bound, location-bound events with a sats reward. Students scan a QR code, provide their Lightning Address, and receive 10-50 sats instantly after passing validation.

[![Next.js](https://img.shields.io/badge/Next.js-15-black?logo=next.js)](https://nextjs.org/)
[![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-3ECF8E?logo=supabase)](https://supabase.com/)
[![Lightning](https://img.shields.io/badge/Lightning-LNbits-F7931A?logo=bitcoin)](https://lnbits.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
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

Sats-Check solves a simple problem: **proving attendance and rewarding it instantly with Bitcoin.**

An organizer creates an event with:
- A **time window** (start and end timestamps)
- A **geofence** (latitude, longitude, and radius)
- A **sats reward** (10-50 sats per check-in)

Students check in by:
1. Scanning the event's QR code on their phone
2. Granting location access
3. Entering their Lightning Address
4. Receiving sats within seconds

Every claim passes through a **three-gate validation pipeline** before payout:

| Gate | What it checks |
|---|---|
| **Time** | Is the current time within the event window? |
| **Location** | Is the student within the geofence radius? (Haversine distance) |
| **Device** | Has this physical device already claimed? (FingerprintJS) |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 15 (App Router, JavaScript) |
| Styling | Tailwind CSS v4 |
| Database | Supabase (PostgreSQL + Row Level Security) |
| Auth | Supabase Auth (Email/Password for organizers) |
| Payments | LNbits API (Lightning Network micro-payments) |
| Device Fingerprint | FingerprintJS (visitor identification) |
| Geolocation | Browser Geolocation API + server-side haversine |
| QR Generation | qrcode.react |
| Deployment | Vercel |

**Design Palette:** Black `#000000` / Bitcoin Orange `#F7931A` / Yellow `#FFD700`

---

## Project Structure

```
sats-check/
├── public/
│   ├── fonts/                         # Custom font files
│   └── images/                        # Logo, icons, OG image
│
├── src/
│   ├── app/                           # Next.js App Router
│   │   ├── layout.js                  # Root layout (providers, global styles)
│   │   ├── page.js                    # Landing page
│   │   ├── globals.css                # Tailwind directives + custom properties
│   │   │
│   │   ├── (auth)/                    # Auth route group
│   │   │   ├── login/page.js          # Organizer login
│   │   │   └── register/page.js       # Organizer registration
│   │   │
│   │   ├── dashboard/                 # Organizer dashboard (protected)
│   │   │   ├── layout.js              # Dashboard shell (sidebar, auth guard)
│   │   │   ├── page.js                # Overview: active events + stats
│   │   │   ├── events/
│   │   │   │   ├── page.js            # List all events
│   │   │   │   ├── new/page.js        # Create event form
│   │   │   │   └── [eventId]/
│   │   │   │       ├── page.js        # Event detail + live claim feed
│   │   │   │       ├── edit/page.js   # Edit event
│   │   │   │       └── qr/page.js     # Full-screen QR display
│   │   │   └── settings/page.js       # Organizer profile + LNbits config
│   │   │
│   │   ├── checkin/                    # Student check-in (public, mobile-first)
│   │   │   └── [eventId]/page.js      # Check-in page
│   │   │
│   │   └── api/                       # API Route Handlers
│   │       ├── events/
│   │       │   ├── route.js            # GET (list) / POST (create)
│   │       │   └── [eventId]/route.js  # GET / PATCH / DELETE
│   │       ├── claims/
│   │       │   ├── route.js            # POST (submit claim)
│   │       │   └── [eventId]/route.js  # GET claims for event
│   │       ├── payments/
│   │       │   ├── send/route.js       # POST trigger LNbits payout
│   │       │   └── status/
│   │       │       └── [paymentId]/route.js  # GET payment status
│   │       └── validate/
│   │           ├── location/route.js   # POST geolocation check
│   │           ├── device/route.js     # POST fingerprint check
│   │           └── time/route.js       # POST time-window check
│   │
│   ├── components/
│   │   ├── ui/                         # Design system primitives
│   │   │   ├── Button.js
│   │   │   ├── Input.js
│   │   │   ├── Card.js
│   │   │   ├── Badge.js
│   │   │   ├── Modal.js
│   │   │   ├── Spinner.js
│   │   │   ├── Toast.js
│   │   │   └── Skeleton.js
│   │   ├── layout/
│   │   │   ├── Navbar.js
│   │   │   ├── Sidebar.js
│   │   │   ├── Footer.js
│   │   │   └── MobileNav.js
│   │   ├── events/
│   │   │   ├── EventCard.js
│   │   │   ├── EventForm.js
│   │   │   ├── EventList.js
│   │   │   └── QRCodeDisplay.js
│   │   ├── checkin/
│   │   │   ├── CheckinForm.js          # Lightning Address input + submit
│   │   │   ├── LocationGate.js         # Geolocation permission + status
│   │   │   ├── PaymentStatus.js        # Pending / Success / Failed states
│   │   │   └── EventBanner.js          # Event info header
│   │   ├── dashboard/
│   │   │   ├── StatsGrid.js
│   │   │   ├── ClaimFeed.js            # Real-time claim activity
│   │   │   └── ClaimTable.js
│   │   └── auth/
│   │       ├── LoginForm.js
│   │       └── RegisterForm.js
│   │
│   ├── hooks/
│   │   ├── useAuth.js                  # Supabase auth state
│   │   ├── useEvents.js                # CRUD operations for events
│   │   ├── useClaims.js                # Claim submission + listing
│   │   ├── useGeolocation.js           # Browser Geolocation API wrapper
│   │   ├── useFingerprint.js           # FingerprintJS integration
│   │   ├── usePayment.js               # LNbits payment trigger + polling
│   │   └── useRealtimeClaims.js        # Supabase Realtime subscription
│   │
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.js               # Browser Supabase client
│   │   │   ├── server.js               # Server-side Supabase client
│   │   │   └── middleware.js            # Auth middleware for protected routes
│   │   ├── lnbits/
│   │   │   ├── client.js               # LNbits API client
│   │   │   └── utils.js                # Lightning Address validation helpers
│   │   ├── fingerprint/
│   │   │   └── client.js               # FingerprintJS initialization
│   │   ├── geo/
│   │   │   └── haversine.js            # Haversine distance calculation
│   │   └── validators/
│   │       ├── event.js                # Event form validation schemas
│   │       ├── claim.js                # Claim payload validation
│   │       └── lightning.js            # Lightning Address format validation
│   │
│   ├── constants/
│   │   ├── config.js                   # App-wide constants
│   │   └── routes.js                   # Route path constants
│   │
│   └── utils/
│       ├── formatters.js               # Date, sats, address formatters
│       ├── errors.js                   # Custom error classes
│       └── api.js                      # Fetch wrapper with error handling
│
├── supabase/
│   ├── migrations/
│   │   ├── 001_create_events.sql
│   │   ├── 002_create_claims.sql
│   │   └── 003_create_rls_policies.sql
│   └── seed.sql                        # Development seed data
│
├── .env.local.example
├── .gitignore
├── eslint.config.mjs
├── jsconfig.json
├── next.config.mjs
├── package.json
├── postcss.config.mjs
├── tailwind.config.js
└── LICENSE
```

---

## Architecture

### High-Level Overview

```
  Organizer (Desktop)              Student (Mobile)
        |                               |
        | Dashboard UI                  | Check-in UI
        | (Protected)                   | (Public)
        v                               v
+-------------------------------------------------------+
|              NEXT.JS APP ROUTER (Vercel)               |
|                                                        |
|  /dashboard/*        /checkin/[id]       /api/*        |
|  Event CRUD          Mobile-first        Route         |
|  QR Generation       High contrast       Handlers     |
|  Live Feed           Lightning input     Validation   |
|                                                        |
|  +--------------------------------------------------+  |
|  |          VALIDATION PIPELINE (Server-side)        |  |
|  |                                                    |  |
|  |  1. Time Gate --> 2. Location Gate --> 3. Device   |  |
|  |  (event window)   (haversine <= r)    (unique fp) |  |
|  |                                                    |  |
|  |  All pass --> Insert claim --> Trigger LNbits pay  |  |
|  +--------------------------------------------------+  |
+-------------------+-------------------------+----------+
                    |                         |
         +----------v----------+   +----------v----------+
         |      SUPABASE       |   |       LNBITS        |
         |                     |   |                     |
         |  events table       |   |  Lightning Network  |
         |  claims table       |   |  micro-payments     |
         |  profiles table     |   |                     |
         |                     |   |  POST /api/v1/      |
         |  Auth (JWT)         |   |    payments         |
         |  Realtime (WS)      |   |                     |
         |  RLS Policies       |   |  Payout to          |
         +---------------------+   |  Lightning Address  |
                                   +---------------------+
```

### Student Check-in Flow

```
Student scans QR --> /checkin/[eventId]
    |
    |-- 1. FingerprintJS generates visitorId (client-side)
    |-- 2. Browser Geolocation API captures lat/lng
    |-- 3. Student enters Lightning Address
    |-- 4. POST /api/claims
    |       |
    |       |-- Validate time:     now within [starts_at, ends_at]?
    |       |-- Validate location: haversine(student, event) <= radius?
    |       |-- Validate device:   visitorId not in claims for this event?
    |       |-- Validate address:  valid Lightning Address format?
    |       |
    |       |-- All pass --> Insert claim (status: "pending")
    |       |                --> POST LNbits payout
    |       |                --> Update claim (status: "success" | "failed")
    |       |
    |       |-- Any fail --> Return specific error code
    |
    |-- 5. UI shows payment status: Pending --> Success / Failed
```

---

## Database Schema

### `events`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `uuid` | PK, DEFAULT `gen_random_uuid()` | Unique event identifier |
| `organizer_id` | `uuid` | FK -> `auth.users(id)`, NOT NULL | Creator of the event |
| `title` | `text` | NOT NULL | Event name |
| `description` | `text` | | Optional description |
| `sats_reward` | `integer` | NOT NULL, CHECK (10 <= val <= 50) | Sats per successful check-in |
| `total_budget_sats` | `integer` | NOT NULL, CHECK (val > 0) | Total sats allocated |
| `sats_spent` | `integer` | DEFAULT 0 | Running total of sats paid out |
| `max_claims` | `integer` | | Optional cap on number of claims |
| `starts_at` | `timestamptz` | NOT NULL | Check-in window start |
| `ends_at` | `timestamptz` | NOT NULL, CHECK (ends_at > starts_at) | Check-in window end |
| `latitude` | `double precision` | NOT NULL | Event venue latitude |
| `longitude` | `double precision` | NOT NULL | Event venue longitude |
| `radius_meters` | `integer` | NOT NULL, DEFAULT 100 | Geofence radius in meters |
| `is_active` | `boolean` | DEFAULT true | Whether event accepts check-ins |
| `created_at` | `timestamptz` | DEFAULT `now()` | Record creation timestamp |
| `updated_at` | `timestamptz` | DEFAULT `now()` | Last update timestamp |

### `claims`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `uuid` | PK, DEFAULT `gen_random_uuid()` | Unique claim identifier |
| `event_id` | `uuid` | FK -> `events(id)`, NOT NULL | Associated event |
| `lightning_address` | `text` | NOT NULL | Student's Lightning Address |
| `device_fingerprint` | `text` | NOT NULL | FingerprintJS visitorId |
| `latitude` | `double precision` | NOT NULL | Student's GPS lat at check-in |
| `longitude` | `double precision` | NOT NULL | Student's GPS lng at check-in |
| `distance_meters` | `double precision` | | Calculated distance from event |
| `sats_amount` | `integer` | NOT NULL | Sats awarded for this claim |
| `payment_status` | `text` | NOT NULL, DEFAULT 'pending' | `pending` / `success` / `failed` |
| `payment_hash` | `text` | | LNbits payment hash |
| `failure_reason` | `text` | | Reason if validation or payment failed |
| `claimed_at` | `timestamptz` | DEFAULT `now()` | Timestamp of claim submission |

**Indexes:**

```sql
-- Enforce one claim per device per event
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

### `profiles` (extends Supabase Auth)

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `uuid` | PK, FK -> `auth.users(id)` | Maps to Supabase Auth user |
| `display_name` | `text` | | Organizer's display name |
| `university` | `text` | | University affiliation |
| `lnbits_url` | `text` | | LNbits instance URL |
| `lnbits_admin_key` | `text` | | Encrypted LNbits admin API key |
| `created_at` | `timestamptz` | DEFAULT `now()` | Record creation timestamp |

### Row Level Security (RLS)

```sql
-- Events: anyone can read active events, organizers manage their own
ALTER TABLE events ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Public can view active events"
  ON events FOR SELECT USING (is_active = true);

CREATE POLICY "Organizers manage own events"
  ON events FOR ALL USING (auth.uid() = organizer_id);

-- Claims: anyone can insert (server validates), organizers read their event claims
ALTER TABLE claims ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Anyone can submit a claim"
  ON claims FOR INSERT WITH CHECK (true);

CREATE POLICY "Organizers view their event claims"
  ON claims FOR SELECT
  USING (event_id IN (SELECT id FROM events WHERE organizer_id = auth.uid()));

-- Profiles: users manage their own profile
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users manage own profile"
  ON profiles FOR ALL USING (auth.uid() = id);
```

### Entity Relationships

```
auth.users 1──1 profiles
auth.users 1──* events
events     1──* claims
```

---

## API Endpoints

### Events

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `GET` | `/api/events` | Organizer | List organizer's events |
| `POST` | `/api/events` | Organizer | Create a new event |
| `GET` | `/api/events/[eventId]` | Public | Get event details (public fields only for students) |
| `PATCH` | `/api/events/[eventId]` | Organizer | Update event |
| `DELETE` | `/api/events/[eventId]` | Organizer | Soft-delete event (sets `is_active = false`) |

### Claims

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/api/claims` | Public | Submit a check-in claim |
| `GET` | `/api/claims/[eventId]` | Organizer | Get all claims for an event |

#### POST `/api/claims` Request Body

```json
{
  "event_id": "uuid",
  "lightning_address": "student@walletofsatoshi.com",
  "device_fingerprint": "fp_abc123xyz",
  "latitude": 12.9716,
  "longitude": 77.5946
}
```

#### Success Response

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

#### Error Response

```json
{
  "success": false,
  "error": {
    "code": "DEVICE_ALREADY_CLAIMED",
    "message": "This device has already checked in to this event."
  }
}
```

#### Error Codes

| Code | HTTP Status | Description |
|---|---|---|
| `EVENT_NOT_FOUND` | 404 | Event does not exist |
| `EVENT_INACTIVE` | 403 | Event is deactivated |
| `EVENT_NOT_STARTED` | 403 | Current time is before `starts_at` |
| `EVENT_ENDED` | 403 | Current time is after `ends_at` |
| `OUTSIDE_GEOFENCE` | 403 | Student is outside the allowed radius |
| `DEVICE_ALREADY_CLAIMED` | 409 | Device fingerprint already used for this event |
| `INVALID_LIGHTNING_ADDRESS` | 422 | Malformed Lightning Address |
| `BUDGET_EXHAUSTED` | 403 | Event's sats budget is fully spent |
| `PAYMENT_FAILED` | 502 | LNbits payout failed |

### Payments (Internal)

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/payments/send` | Trigger LNbits payout (server-only) |
| `GET` | `/api/payments/status/[paymentId]` | Poll LNbits payment status |

### Validation (Internal)

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/validate/time` | Check if current time is within event window |
| `POST` | `/api/validate/location` | Haversine distance check |
| `POST` | `/api/validate/device` | Fingerprint uniqueness check |

These validation endpoints are called server-side during claim processing and are not exposed to clients directly.

---

## Anti-Fraud System

### Three-Gate Validation Pipeline

```
Request --> TIME GATE --> LOCATION GATE --> DEVICE GATE --> Approved
               |               |                |
             Reject          Reject           Reject
         "Not active"   "Outside geofence"  "Already claimed"
```

| Gate | Method | Details |
|---|---|---|
| **Time** | Server-side `Date` comparison | `starts_at <= now() <= ends_at` |
| **Location** | Haversine formula | Distance must be <= `radius_meters` |
| **Device** | FingerprintJS `visitorId` | Unique constraint on `(event_id, device_fingerprint)` |

### Why FingerprintJS?

- Browser-level device identification without requiring user accounts
- Persists across incognito and private browsing sessions
- 99.5% identification accuracy
- Zero friction for students -- no signup required

---

## Getting Started

### Prerequisites

- Node.js >= 18.x
- npm or pnpm
- A [Supabase](https://supabase.com) project
- An [LNbits](https://lnbits.com) instance (hosted or self-hosted)
- A [FingerprintJS](https://fingerprint.com) API key

### Installation

```bash
# Clone the repository
git clone https://github.com/ItzSouraseez/sats-check.git
cd sats-check

# Install dependencies
npm install

# Set up environment variables
cp .env.local.example .env.local
# Fill in your credentials (see Environment Variables below)

# Run Supabase migrations
npx supabase db push

# Start the development server
npm run dev
```

The app will be running at [http://localhost:3000](http://localhost:3000).

---

## Environment Variables

Create a `.env.local` file from the provided `.env.local.example`. See the table below for details on each variable:

| Variable | Scope | Description |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Client + Server | Your Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Client + Server | Supabase anonymous/public key |
| `SUPABASE_SERVICE_ROLE_KEY` | Server only | Supabase service role key (bypasses RLS) |
| `LNBITS_URL` | Server only | LNbits instance base URL |
| `LNBITS_ADMIN_KEY` | Server only | LNbits admin API key for payouts |
| `NEXT_PUBLIC_FINGERPRINT_API_KEY` | Client | FingerprintJS browser API key |
| `NEXT_PUBLIC_APP_URL` | Client + Server | Application base URL |
| `NEXT_PUBLIC_DEFAULT_GEOFENCE_RADIUS` | Client | Default geofence radius in meters |

---

## Development Workflow

This project uses a modular commit strategy. Each logical unit of work gets its own commit.

### Branch Naming

```
feature/  -->  feature/event-creation-form
fix/      -->  fix/geofence-validation-edge-case
chore/    -->  chore/update-tailwind-config
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
2. Create your feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes following the convention above
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
