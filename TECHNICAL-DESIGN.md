# 📐 RentWave GH — Technical Design Document

**Version:** 0.1 (first full pass)
**Date:** 2026-09-11
**Author:** Selasi (Richmond), CTO — Lumora Tech
**Companion to:** [`RENTWAVE-CONCEPT.md`](RENTWAVE-CONCEPT.md) (product decisions, v0.2)
**Mockups:** `mockups/` (renter · landlord · admin — visually QA'd 2026-09-11)

> **Rule of thumb for this doc:** every locked product decision must be enforced in
> **schema or server code**, never only in the UI. UI is a suggestion; the API is the law.
>
> **Configurability principle (locked 2026-09-12):** anything ops will want to change —
> payment/SMS providers, their API keys, fee %/cap, payout timing, settlement style —
> lives in the **admin dashboard, not in code**. Drivers are code; configuration is data.

---

## 1. Stack (locked unless Selasi objects)

| Layer | Choice | Why |
|---|---|---|
| API | **Laravel 11 (PHP 8.3)**, REST + Sanctum tokens | Team already runs a PHP stack (Morning Delight on XAMPP) → fastest path; mature queue/webhook tooling; excellent Paystack PHP SDK |
| DB | **MySQL 8** | XAMPP-familiar now; JSON columns + generated cols give us what we need at MVP scale |
| Renter + Landlord apps | **Web-first:** responsive **Blade + Alpine.js** client served by the same Laravel app (L1/L2) → Flutter native (Android then iOS) once flows are validated in testing (L3) | Per concept §8 + delivery order below |
| Admin panel | **React (Vite) + Tailwind** consuming the same API | Table-heavy back office; mockup already designed |
| **Test-first web app** | **Blade + Tailwind** (same Laravel API, responsive) | 🔒 Locked 2026-09-12: flows reach real testers in a browser BEFORE Flutter exists — zero app-store friction; stays forever as our staging/smoke-test harness |
| Media | **Cloudinary** (free tier) direct uploads | Photos are the product; auto compression + CDN; keeps API thin |
| Maps / geo | **OpenStreetMap + Leaflet** (Flutter: `flutter_map`) | No Google Maps billing surprise; good GH coverage; swap later if needed |
| Push | **FCM** | Standard |
| SMS (OTP + receipts) | **Arkesel** (Ghanaian provider, cheap), behind a driver interface so we can swap | SMS builds trust in GH; never hardcode a vendor |
| Payments | **Paystack GH** — MoMo + cards, webhook-first | Locked |
| Queues | Laravel queues (database driver) → webhooks, SMS, emails | Webhook work must never block a request |

**🚚 Delivery order: web-first testing (locked 2026-09-12)**

Before any Flutter work, we ship a **responsive web client** (Blade + Alpine.js, served
by the same Laravel app) that talks to the exact API the mobile apps will use. Rationale:
browser iteration is 5–10× faster (no store builds, no installs — any phone can open a
link), and every flow — auth, listing, tour, chat, payment webhook — gets proven
end-to-end by real test users first. Flutter then wraps *proven* flows, not guesses.

- Web client covers renter + landlord sides (mockups remain the design target);
  the admin React console (already web) completes the trio.
- **Flutter native (Android first) starts after pilot feedback — target post-M5**,
  building against the proven API contract.

---

## 2. High-level architecture

```
Web test client ──┐   (Blade+Alpine · renter & landlord · M0–M4)
Flutter (renter) ─┤
Flutter (landlord)─┼──► Laravel API ──► MySQL 8
React (admin) ────┘        │  │
                           │  └──► Cloudinary (photos)
                           ├────► Paystack (initialize + verify + webhooks)
                           ├────► FCM (push)   └─ Arkesel (SMS: OTP, receipts)
                           └────► Queue workers (webhooks, SMS, digests)
```

- One API, three clients (+ one temporary web test client), role-based middleware.
- **All money truth lives in our DB + Paystack webhook**, never in the app's local state.

## 2.1 Brand assets

`brand/` holds three coded SVG logo concepts — A roof+swell (`logo-a-roof-swell.svg`), B wave-W (`logo-b-wave-w.svg`), C house-wave (`logo-c-house-wave.svg`) — presented side-by-side in `brand/logo-showcase.html` with wordmark lockups; Selasi picks before app icon work begins. Palette locked to navy #0b2239 / wave teal #0ea5b7 / gold #ffb703 from `mockups/rentwave.css`.

---

## 3. Database schema (MVP core)

> Conventions: `id` = PK BIGINT UNSIGNED; timestamps + soft deletes where noted;
> money as **BIGINT pesewas** (integers only — no floats near money, ever).

### 3.1 Identity & roles
```
users
  id, role ENUM('renter','landlord','admin')
  name, phone VARCHAR(15) UNIQUE, email NULL, password
  region_id FK, city, avatar_path NULL
  phone_verified_at NULL          -- OTP verified
  deleted_at

landlord_profiles                 -- 1:1 users(role=landlord)
  user_id FK PK
  business_name NULL
  id_type ENUM('ghana_card','voter_id','passport')
  id_number_enc TEXT              -- encrypted at rest
  id_photo_path
  verification ENUM('unverified','pending','verified','rejected'), verified_at NULL
  payout_channel ENUM('momo','bank'), payout_number, payout_name
  rating_avg DECIMAL(2,1) DEFAULT 0, rating_count INT DEFAULT 0
  phone_visible BOOLEAN DEFAULT 0 -- denormalized from subscription; SINGLE SOURCE: active sub
```

### 3.2 Geography
```
regions                           -- seed all 16 Ghana regions
  id, name, slug
```

### 3.3 Listings (the heart)
```
listings
  id, landlord_id FK(users)
  title VARCHAR(120), description TEXT
  property_type ENUM('chamber_hall','single_room','one_bed','two_bed','three_bed',
                     'house','apartment','hostel')
  bedrooms TINYINT, bathrooms TINYINT
  rent_amount BIGINT              -- pesewas/month
  advance_months TINYINT          -- CHECK (advance_months BETWEEN 6 AND 24)  ← locked rule in DB
  rules JSON                      -- landlord's terms/house rules (array of strings)
  amenities JSON                  -- ["water_tank","prepaid_meter","parking",...]
  region_id FK, city VARCHAR(80), area VARCHAR(80)
  latitude DECIMAL(9,6), longitude DECIMAL(9,6)   -- required; NULL rejected at API
  available_from DATE NULL
  status ENUM('draft','pending_review','live','paused','occupied','rejected','delisted')
  review_note NULL                -- admin rejection reason
  views_count INT DEFAULT 0
  occupied_at NULL, occupied_by_tenancy_id NULL FK   -- audit trail of WHO filled it
  created_at, updated_at, deleted_at

  INDEX (status, region_id, rent_amount)      -- the search query
  INDEX (latitude, longitude)
  FULLTEXT (title, description, city, area)

listing_photos
  id, listing_id FK, cloudinary_public_id, path, sort TINYINT
  -- API rejects create/publish with < 3 photos (min 3, max 12)
```

### 3.4 Tours
```
tour_bookings
  id, listing_id FK, renter_id FK, landlord_id FK (denorm for inbox speed)
  status ENUM('requested','confirmed','declined','completed','cancelled','no_show')
  scheduled_at DATETIME           -- must be future at request time
  renter_note NULL, response_note NULL
  requested_at, responded_at NULL, completed_at NULL
  UNIQUE(listing_id, renter_id, scheduled_at)   -- no double-books same slot
```

### 3.5 Chat (in-app only — contact policy lives here)
```
conversations
  id, listing_id FK, renter_id FK, landlord_id FK, last_message_at
  UNIQUE(listing_id, renter_id)   -- one thread per renter per listing

messages
  id, conversation_id FK, sender_id FK, body TEXT NULL,
  attachment_path NULL, read_at NULL, created_at
  -- API masks any phone-number pattern in body? No — BLOCK sending: reject bodies
  -- matching phone patterns for non-Pro senders (policy enforced server-side).
```

### 3.6 Tenancy & payments
```
tenancies                         -- created on FIRST successful rent payment
  id, listing_id FK, renter_id FK, landlord_id FK
  monthly_rent BIGINT, advance_months TINYINT
  start_date DATE, status ENUM('active','notice','ended','terminated')
  notice_given_at NULL, ended_at NULL
  UNIQUE(listing_id, renter_id, start_date)

payments
  id, uuid UNIQUE                 -- idempotency key for Paystack init
  tenancy_id NULL FK, listing_id NULL FK
  payer_id FK, landlord_id FK
  type ENUM('first_month','advance','monthly','subscription')
  amount BIGINT                   -- pesewas the tenant/landlord pays
  fee BIGINT DEFAULT 0            -- platform cut (see §5)
  net BIGINT                      -- amount - fee → landlord wallet
  method ENUM('momo_mtn','momo_telecel','momo_at','card')
  provider_reference UNIQUE       -- Paystack trx ref — webhook de-dupe anchor
  status ENUM('initialized','pending','success','failed','refunded')
  paid_at NULL, receipt_no UNIQUE NULL, created_at

  INDEX (landlord_id, status), INDEX (payer_id, status)
```

### 3.7 Wallet — immutable ledger (double-entry style)
```
wallet_entries                    -- APPEND ONLY. Never UPDATE, never DELETE.
  id, landlord_id FK
  direction ENUM('credit','debit')
  type ENUM('rent','advance','subscription','fee','payout','refund','adjustment')
  amount BIGINT
  balance_after BIGINT            -- running balance snapshot
  payment_id NULL FK, payout_id NULL FK
  memo VARCHAR(160), created_at

  -- Invariant enforced by code + nightly job:
  -- SUM(credit) - SUM(debit) per landlord = latest balance_after
```

```
payouts
  id, landlord_id FK, amount BIGINT, channel ENUM('momo','bank'),
  destination VARCHAR(40), status ENUM('requested','processing','paid','failed'),
  provider_reference NULL, requested_at, processed_at NULL
```

### 3.8 Subscriptions (paywall = server truth)
```
subscriptions
  id, landlord_id FK, plan ENUM('starter','pro','agency')
  status ENUM('active','past_due','cancelled')
  started_at, renews_at NULL, cancelled_at NULL
  -- PLAN CAPS (server-enforced, from concept §4.1):
  -- starter: 1 live listing, phone hidden
  -- pro:     10 live listings, phone visible
  -- agency:  50 live listings, phone visible, sub-accounts later
```

### 3.9 Trust & ops
```
reviews     id, tenancy_id UNIQUE FK, renter_id FK, listing_id FK,
            rating TINYINT 1-5, body, created_at
            -- only creatable by a renter with a SUCCESS payment on that tenancy

reports     id, reporter_id FK, listing_id NULL FK, reported_user_id NULL FK,
            reason ENUM('fake','scam','already_rented','wrong_location','abuse','other'),
            details, status ENUM('open','reviewing','resolved'), resolution_note

audit_logs  id, actor_id, action, subject_type, subject_id, meta JSON, created_at
            -- MANDATORY for: verification decisions, listing publish/reject,
            -- wallet ops, payout ops, plan changes, dispute actions
```

---

## 4. State machines (no other transitions allowed)

### 4.1 Listing lifecycle
```
draft ──submit──► pending_review ──admin approve──► live ──► occupied
 │                     │                              │  ▲         │
 │                     └─admin reject──► rejected    │  └─auto after
 └─(edit keeps draft)                                 │     FIRST success payment
                     live ──landlord pause──► paused ─┘
                     paused ──resume──► live (re-check caps)
                     live/paused ──admin/landlord delist──► delisted
```
- **occupied** ⇒ auto-delisted from search (locked pillar #1). Landlord can also mark manually.
- Only `live` listings are searchable. `occupied` shows "Occupied" badge on profile only.

### 4.2 Tour booking
```
requested ──landlord──► confirmed ──(after time)──► completed
    │                      │
    └─landlord──► declined │
                           └─either side──► cancelled   ──landlord miss──► no_show
```

### 4.3 Payment (webhook-first, idempotent)
```
initialize (server) ──► initialized ──Paystack callback/webhook──► pending ──► success | failed
                                                        success ──► refunded (admin, full audit)
```
- **Webhook is the source of truth.** Callbacks/redirects are UX only.
- De-dupe on `provider_reference` UNIQUE + `uuid` idempotency at init.
- On `success` (single transactional handler):
  1. insert `payments` row (or skip if ref exists)
  2. wallet credit `net` → `wallet_entries` with `balance_after`
  3. create/extend `tenancy`
  4. **listing.status → occupied** (+ occupied_by_tenancy_id)
  5. receipt_no generated, SMS + FCM out via queue

---

## 5. Fee math (locked style: baked in, tenant never sees a fee)

```
fee_pct & fee_cap live in platform_settings — runtime-changeable from admin, audit-logged
DEFAULTS: fee_pct = 0.025 · fee_cap = 10_000 pesewas (GH₵100)
fee = min( round(amount * fee_pct), fee_cap )   // rent/advance only — via FeeCalculator service
net = amount - fee   // each payment row stores its own fee → changing settings never rewrites history
```
- Tenant pays exactly `amount` (the listed rent). Landlord nets `net`.
- `subscription` payments: fee = 0 (it IS revenue).
- Every fee is its own wallet row type=`fee` → finance reports = simple GROUP BY.

## 6. Contact/phone policy — enforced in API, not UI (locked)

- Landlord phone NEVER appears in any listing payload. No field. Ever.
- In-app **call** endpoint returns a masked bridge token only when caller has
  `tour_bookings.status='confirmed'` for that listing (Pro/Free alike).
- The raw number exists in exactly one API: admin-only `GET /admin/users/{id}` (audit-logged).
- Pro unlock = server checks active `subscriptions` row → flips what call/chat
  endpoints expose. UI only mirrors this.

---

## 7. API surface (v1 — grouped, role-gated)

**Auth** `POST /auth/phone-otp/request` · `/auth/phone-otp/verify` · `/auth/register` · `/auth/login` · `/auth/logout`
**Listings (public)** `GET /listings` (search: region, city, price range, type, beds, geo-radius, sort) · `GET /listings/{id}` (excludes contact, includes landlord public profile + rating)
**Renter** `POST /tours` · `GET /tours/mine` · `POST /conversations` · `GET /conversations/{id}/messages` · `POST /messages` · `POST /payments/init` · `GET /payments/{id}` · `POST /tenancies/{id}/notice` · `POST /reviews` · `POST /reports`
**Landlord (auth: landlord)** `GET /me/listings` · `POST /listings` (draft) · `POST /listings/{id}/submit` · `PATCH /listings/{id}` · `POST /listings/{id}/pause|resume|mark-occupied` · `GET /tours/inbox` · `POST /tours/{id}/confirm|decline|complete|no-show` · `GET /me/wallet` (balance + ledger) · `POST /payouts` · `GET /me/subscription` · `POST /me/subscription/subscribe|cancel`
**Admin (auth: admin)** `GET /admin/overview` (KPIs) · `GET /admin/queue/listings|verifications` · `POST /admin/listings/{id}/approve|reject` · `POST /admin/landlords/{id}/verify|reject` · `GET /admin/ledger` · `POST /admin/payouts/{id}/release|fail` · `GET /admin/disputes` · `POST /admin/payments/{id}/refund` · `POST /admin/reports/{id}/resolve`
**Webhooks** `POST /webhooks/paystack` (signature-verified, no auth token, queued)
**Admin — settings & providers** `GET/PUT /admin/settings/{group}` · `POST /admin/providers` · `POST /admin/providers/{id}/activate` · `POST /admin/providers/{id}/test` — all audit-logged; credentials write-only, masked on read

Cross-cutting: Sanctum tokens, `role:` middleware, throttle: 60/min default —
10/min on auth OTP, 30/min on search, 20/min on messages.

---

## 8. Platform Settings & Pluggable Providers (locked 2026-09-12)

**The pattern:** integrations (Paystack, Hubtel, Arkesel, Twilio, …) are hard-coded
**driver classes** behind interfaces; **which driver is active, and its keys, live in the
DB** and are managed from the admin dashboard. Adding/switching a provider or changing
fees = a settings change, never a code deploy.

### 8.1 Schema additions
```
platform_settings                 -- behavior knobs (cached, audit-logged on every change)
  id, grp ENUM('payments','sms','email','push','fees','payouts','general')
  key VARCHAR(60) UNIQUE          -- e.g. fees.fee_pct, fees.fee_cap_pesewas,
                                  -- payouts.mode, payouts.batch_cutoff, settlements.style
  value TEXT                      -- JSON-encoded scalar
  type ENUM('int','string','bool','enum','json')
  options JSON NULL               -- allowed enum values / min-max validation
  description VARCHAR(200)
  updated_by FK(users) NULL, updated_at

provider_configs                  -- one row per installed driver (+mode)
  id, kind ENUM('payment','sms','email','push')
  provider VARCHAR(30)            -- 'paystack','hubtel','arkesel','twilio','mailgun',…
  mode ENUM('test','live')
  credentials_enc TEXT            -- Crypt-encrypted JSON. NEVER returned by any API.
  is_active BOOL DEFAULT 0        -- code enforces: one active config per kind (+mode)
  last_checked_at NULL, last_check_ok NULL
  UNIQUE(kind, provider, mode)
```

### 8.2 Defaults seeded on install
`fees.fee_pct=0.025` · `fees.fee_cap_pesewas=10_000` (GH₵100) ·
`payouts.mode=daily_batch` · `payouts.batch_cutoff=18:00` · `settlements.style=collect_then_wallet`

### 8.3 How the app reads settings
- `SettingsService::get('fees.fee_cap_pesewas')` — **cached** (Laravel cache), flushed on
  every admin write. No per-request settings queries.
- **FeeCalculator**, **PayoutScheduler**, **SmsManager**, **PaymentManager** all resolve
  through this service → controllers/services never read `.env` for anything operational.

### 8.4 Admin endpoints (audit-logged)
`GET /admin/settings/{group}` · `PUT /admin/settings/{group}` ·
`POST /admin/providers` · `POST /admin/providers/{id}/activate` ·
`POST /admin/providers/{id}/test` (fires a real ping/charge-verify and stores last_check_ok)

### 8.5 Safety rails
1. Credentials **encrypted at rest** (Laravel Crypt), write-only in the admin UI (masked,
   last-4 shown), never in logs, exports, or any client payload.
2. Every settings write → `audit_logs` (old value → new value, who, when). Money-affecting
   settings (fees, payout mode) show an impact-confirm dialog in the UI.
3. Range validation server-side (e.g. `fee_pct` hard-capped at 10%, fee_cap ≥ 0) so a
   fat-finger can't bankrupt the take rate.
4. Payment success-handler reads fee config **per transaction** — historical payments keep
   the fee they were charged (already true: `payments.fee` is a stored column).
5. `settlements.style=subaccount_split` shows in admin from day one but with a
   **"requires setup"** state — MVP ships `collect_then_wallet` fully; the split path is
   a driver behind the same interface, enabled when Paystack subaccounts are configured.

---

## 9. Security rules (MVP non-negotiables)

1. Webhook signature verify (Paystack secret) → else 401. Replays de-duped by ref.
2. Money writes ONLY inside the success-handler service class. Controllers never touch wallets.
3. Landlord payout requires `verification='verified'` + active payout details.
4. Phone-number pattern blocked in message bodies unless sender is Pro (contact policy).
5. All uploads: Cloudinary unsigned-preset restricted (images only, ≤5MB, ≤12 per listing).
6. OTP: 6-digit, 10-min expiry, 5 attempts, then 15-min lockout. Hash at rest like passwords.
7. `audit_logs` written for every admin action + every wallet mutation.
8. Rate limits per route group (above). 429s logged.
9. Backup: nightly `mysqldump` to off-machine storage, 7-day retention. Restore tested once before launch.
10. Provider credentials & settings: encrypted at rest, write-only via admin (masked read), never logged; every change audit-logged with old→new values.
11. No operational value (keys, fees, modes) may ever be read from `.env` in application code — only from SettingsService/providers. `.env` holds only bootstrap config (DB connection, app key).

---

## 10. Build milestones (MVP — mirrors concept §9)

| # | Milestone | Includes | Done when |
|---|---|---|---|
| M0 | Repo + skeleton | Laravel 11 API, MySQL migrations for ALL tables (incl. platform_settings + provider_configs), SettingsService (cached), seed regions + default settings, Sanctum, CI lint | `php artisan migrate:fresh --seed` green |
| M1 | Auth + roles | OTP phone flow, register/login, role middleware, landlord profile + ID upload (pending) | A landlord can register → pending verified |
| M2 | Listings core | CRUD (draft→submit), photos via Cloudinary, admin approve/reject queue, search endpoint w/ filters | Search returns live-only listings by region/price |
| **M2.5** | **Web test client** | Blade+Alpine web app: renter + landlord flows against the live API; seeded demo data; test-user accounts | Beta testers complete search→tour→payment loop in a browser |
| M3 | Tours + chat | Tour state machine, conversations, messages w/ phone-pattern block, FCM fanout | Renter books → landlord confirms → chat works, no numbers leak |
| M3.5 | **Web pilot build** | React webapp (shares admin tooling): renter search/tour/chat/payment + landlord listing/wallet against the real API — THE pilot vehicle | 10 pilot users complete search→tour→pay on web |
| M4 | Money | Paystack init + webhook success-handler, wallet ledger, tenancy create, auto-occupied, receipts + SMS, **admin provider-config UI** (keys/test-live/test-connection) + fee settings screen | A test MoMo payment flips listing to occupied & credits wallet — configured entirely from admin, zero .env |
| M5 | Wallet ops + subs | Payout request→release, subscriptions (caps + phone visibility), **payout_mode setting** (instant \| daily_batch) with queue job + cutoff | Pro landlord number-visible path provable via API only |
| M6 | Admin console | React admin: overview KPIs, queues, ledger, disputes, refund, **full settings console** (fees, payout mode, providers, flags) | Admin can run the full lifecycle AND change fee/payout/provider config without a deploy |
| M7 | Hardening | Rate limits, audit log review, backup/restore drill, seed 30 demo listings across 5 regions | QA pass + restore drill done |

> **Build order (locked 2026-09-12 — Selasi): WEB FIRST for testing.**
> Before Flutter, ship a lightweight responsive **web test client** (Laravel Blade + Tailwind
> covering renter + landlord flows, reusing the same API + design system). Purpose: validate
> search → tour → chat → pay and the admin loop cheaply in a browser (works on phones too,
> so early testers don't need an install). Flutter starts only after the web flows prove out
> (M4 go/no-go passes on web). API-first architecture makes this a fourth client, not a fork.

**Web pilot app (React) builds from M3.5 and is the vehicle for the real-user pilot — Flutter begins after pilot feedback (target post-M5). M4 remains the go/no-go milestone.**

---

## 11. Open technical decisions

1. ~~FEE_CAP value~~ **RESOLVED (2026-09-12):** default **GH₵100 (10_000 pesewas)** in platform_settings — runtime-changeable + audited, no deploy needed.
2. **Paystack account** — new dedicated account under Lumora Tech (before M4).
3. **Arkesel vs Twilio for SMS** — Arkesel default; confirm sender-ID registration lead time (before M1).
4. ~~Payout timing~~ **RESOLVED (2026-09-12):** admin setting `payouts.mode` = instant \| daily_batch, **default daily_batch** (fraud window), cutoff hour configurable. Switchable anytime from admin.
5. ~~Settlement style~~ **RESOLVED (2026-09-12):** setting `settlements.style` = collect_then_wallet \| subaccount_split. MVP ships **collect_then_wallet** fully; subaccount_split visible in admin with "requires setup" until its Paystack path is built/verified.

---

*Next step after sign-off: M0 scaffold. Nothing in this doc blocks UI work — mockups stay the source of truth for screens.*
