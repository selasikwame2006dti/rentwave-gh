# 🌊 RentWave GH — Product Concept Document

**Version:** 0.2 (Core decisions locked)
**Date:** 2026-09-09
**Author:** Selasi (Richmond), CTO — Lumora Tech
**Status:** Product draft — owner review. Team discussion deferred.

---

## 1. The Idea (One Paragraph)

RentWave GH is a **rental marketplace platform for Ghana** where landlords list their
houses/rooms for rent with photos, prices, and their own terms — and renters find,
tour, and pay for homes **entirely in the app**. When a property is rented, the system
automatically removes it from the search list and marks it **Occupied**, so every
listing a renter sees is actually available. No more roaming for "TO LET" signs, no
more paying agents before seeing a room, no more calling about houses that were
rented out months ago.

**Tagline ideas:** "Find your home. Skip the stress." / "Ghana's cleanest way to rent."

---

## 2. The Problem

Finding a place to rent in Ghana is broken:

1. **Discovery is manual and offline** — walking around looking for signs, asking
   friends/family, relying on word of mouth.
2. **Agents ("goro boys") charge upfront fees** just to show you a place — and often
   show you places already taken.
3. **Stale listings** — Jiji/meQasa-style listings are frequently already occupied;
   renters waste calls, time, and transport money.
4. **Trust gap** — fake photos, fake landlords, advance-fee scams. Renters are
   scared, and rightly so.
5. **Payments are messy** — cash in hand, unclear receipts, no record of what was
   agreed.

> **RentWave GH's promise: every listing you see is real and available, every
> payment has a record, and every deal has clear terms up front.**

---

## 3. The Solution — Core Pillars

| # | Pillar | What it means in the product |
|---|--------|------------------------------|
| 1 | **Always-available listings** | Auto-delist on occupancy — the occupied unit leaves search instantly. |
| 2 | **Landlord-owned terms** | Landlords write their own terms (advance months, utilities, rules) shown on every listing before contact. |
| 3 | **Book a tour, not a gamble** | In-app tour booking with date/time; chat + call stay inside the app — landlord numbers are hidden unless the landlord is premium. |
| 4 | **Pay rent in-app** | MoMo/card payment flows to the landlord's wallet; both sides get records. |
| 5 | **Verified trust layer** | Landlord ID/phone verification, GPS-pinned listings, review system. |

---

## 4. Who Pays — Business Model

**Main revenue: Landlord subscriptions.** Secondary: small service charge on rent
transactions, **baked into the listed price** so tenants never see a "fee" at checkout.

### 4.1 Landlord Subscription Tiers (pricing TBD)

| Tier | Price (GH₵) | Listings | Extras |
|------|------------|----------|--------|
| **Starter** | Free | 1 active listing | Basic listing, chat, tours |
| **Pro** | ~50/mo | Up to 10 active | Verified badge, **phone number shown on listings**, analytics, priority support |
| **Agency** | ~150/mo | Up to 50 active | Multi-agent staff accounts, bulk tools, API later |
| **Agency+** | ~400/mo | Unlimited | Everything + dedicated support + early features |

- Free tier exists to solve the chicken-and-egg problem: supply first.
- **Verified badge** is the killer upsell — renters will filter for it.

### 4.1.1 The Phone-Number Rule (LOCKED)

A landlord's phone number **never appears anywhere in the UI** — not on listings,
not in search results, not in chat headers — **unless the landlord is on a premium
plan (Pro and above)**. Free-tier contact happens entirely through in-app chat and
in-app calling.

Why this rule wins twice:
1. Contact stays inside RentWave, so we see the deal through to in-app payment.
2. "Renters can call you directly" becomes a real, tangible perk landlords pay for.

### 4.2 Transaction fee (proposal)

- Small % on rent paid in-app (proposal: **2–3%**, capped), **already included** in
  the price the tenant sees. Landlord receives wallet credit **net of the fee**.
- Example: Tenant pays GH₵1,500. Landlord listed 1,500. Platform takes 2.5% (GH₵37.50).
  Landlord wallet gets GH₵1,462.50. Tenant saw "GH₵1,500/mo" the whole time.

### 4.3 Future revenue lines (not MVP)

- Featured/boosted listings (pay to appear at top of search)
- Tenant reference/credit reports
- Rent financing / installment products (needs licence — later, with Sammy's lead)
- White-label for estate developers

---

## 5. The Three Sides of the System

### 5.1 👤 RENTER (the "buyer")

**Goal:** find a real, available home, verify it, tour it, pay safely.

| Feature | Detail |
|---------|--------|
| Search & filters | By area (Osu, Madina, Takoradi…), price range, bedrooms, property type (chamber & hall, single room, 2/3-bedroom, apartment), amenities (water, toilet type, electricity meter, parking, fenced) |
| Listing view | Photos, price & advance terms, full description, amenities list, landlord profile (verified badge, rating, response rate), **GPS location map pin**, "What you should know" section (the landlord's terms) |
| Tour booking | Pick date + time → landlord confirms/declines → in-app reminder |
| Contact | In-app chat + in-app call. **Landlord's number is never displayed** unless the landlord is premium (Pro+) — see the phone-number rule (§4.1.1). |
| Apply & pay | After a confirmed tour (or landlord override), renter pays first rent + agreed advance via MoMo/card → lands in landlord wallet |
| My Rentals | Active rental, payment history, receipts, next-due reminders, renewal in one tap |
| Reviews | Rate landlord + property after moving in (only verified renters can review) |

### 5.2 🏠 LANDLORD

**Goal:** fill vacancies with serious tenants without agent wahala, and get paid cleanly.

| Feature | Detail |
|---------|--------|
| Onboarding | Phone/ID verification (Ghana Card / voter ID photo), payout MoMo number setup |
| Listing manager | Create listing: photos, video walkthrough (optional), price, advance required, amenities, description, **GPS pin** (pick on map), availability date |
| **Terms of service** | Landlord writes their own terms template per property — advance months (landlord's choice, platform-enforced range **6–24 months**), utility billing, visitors policy, maintenance rules. Shown on listing BEFORE anyone books a tour. |
| Requests inbox | Tour requests (confirm/decline/reschedule), tenant questions, applications |
| Occupancy switch | **One tap: "Mark as Rented"** → listing leaves search instantly, moves to "Occupied" section. System can also auto-suggest this after a payment succeeds. |
| Wallet & payouts | Running balance, transaction history, withdraw to MoMo/bank. Rent money lands here net of platform fee. |
| Subscription | Choose tier, pay in-app (MoMo/card), see usage (e.g. 7/10 listings used) |
| Analytics (Pro+) | Views, tour requests, chat response rate, average days-to-rent |

### 5.3 🛡️ ADMIN (Lumora team — back office)

**Goal:** keep the marketplace clean, trusted, and running.

| Feature | Detail |
|---------|---------|
| Dashboard | Total listings, active/occupied, users, GMV (rent volume), revenue (subs + fees), churn |
| Listing moderation | Approve/flag listings (photos real? price suspicious? duplicate?) — manual review queue at first, auto-checks later |
| User management | Verify/suspend landlords & renters, handle reports |
| Dispute center | Tenant↔landlord disputes: view chat log, payment records, decide refunds |
| Finance | Wallet ledger (every credit/debit), fee report, subscription revenue, payout approvals |
| Verification queue | Review landlord ID submissions, approve Verified badge |
| Content/CRM | Push notifications, broadcast messages, promos |

---

## 6. How It Flows — Key User Journeys

### 6.1 Renter journey
```
Search (area/price/type) → Listing with photos, price, terms, map pin
  → Book Tour (date/time) → Landlord confirms → Tour happens
  → Happy? Apply → Pay in-app (MoMo/card) → Money → landlord wallet
  → Listing auto-marked OCCUPIED (leaves search) → Renter's "My Rentals" activates
```

### 6.2 Landlord journey
```
Sign up → Verify ID + MoMo payout → Choose plan (Starter free = 1 listing)
  → Post listing (photos, price, terms, GPS pin) → Renter books tour
  → Confirm tour → Meet → Renter pays in-app → Wallet credited (net of fee)
  → Mark as Rented (or auto) → Listing shows OCCUPIED → New tenant appears
```

### 6.3 Money flow
```
Tenant pays (MoMo/card)
  → Payment gateway (Paystack/Hubtel)
  → Platform account → fee retained → net credited to Landlord Wallet
  → Landlord withdraws to MoMo/bank anytime
```

---

## 7. Trust & Safety Rules (non-negotiables)

1. **Verified landlords only** can receive in-app payments (ID + phone + payout MoMo in own name).
2. **GPS pin required** on every listing — kills the "location vague" scam pattern.
3. **No listing without photos** — minimum 3 real photos.
4. **Occupied = invisible** in search. Occupied listings show "Occupied" badge on profile only.
5. **All chat in-app** — gives both sides a record and gives Admin dispute evidence. Landlord phone numbers are **hidden in the UI** (visible only for premium landlords) so contact starts inside RentWave.
6. **Reviews from verified renters only** (must have a completed payment).
7. Report button on every listing; admin moderation queue.

---

## 8. Technical Shape (early view)

> 📐 **Expanded & finalized in [`TECHNICAL-DESIGN.md`](TECHNICAL-DESIGN.md)** — full schema, state machines, wallet rules, API surface, milestones.
>
> 🧪 **Web-first testing (locked 2026-09-12):** the first live build is a **web app** (React, same API) covering renter + landlord core flows — pilot with real users via a simple URL, zero app-store friction, iterate daily. Flutter mobile apps come **after** the pilot validates the flows.

| Layer | Proposal |
|-------|----------|
| **Mobile app** | Phone-first. **Testing v1 ships as a responsive web app (PWA) first** — Flutter (one codebase → Android+iOS) follows once testing validates the loops. |
| **Backend** | Node.js (NestJS) or Laravel API — decide at build planning. REST + webhooks for payments. |
| **Database** | PostgreSQL — listings, users, leases, wallet ledger, bookings. |
| **Payments** | **Paystack — locked.** MoMo + cards. Webhook-verified transactions only. |
| **Wallet** | Double-entry ledger (every pesewa accounted: credit/debit/balance). Not a running number. |
| **Search** | Postgres full-text + geo radius search first; Elastic later at scale. |
| **Media** | Cloud storage (S3/Cloudinary) with image compression — photos are the product. |
| **Notifications** | FCM push + SMS (for tour confirmations & receipts — SMS builds trust in GH) |
| **Admin panel** | Web dashboard (React) — separate role, same API. |

### 8.1 Core data objects (first sketch)
`User (renter|landlord|admin)` · `Property` (photos, geo, price, terms, status: available/occupied/paused) · `TourBooking` · `Conversation/Message` · `Lease/Tenancy` · `Payment` · `WalletEntry` · `Subscription` · `Review` · `Report`

---

## 9. MVP Scope (Build v1)

**In:**
- Renter app: search/filter, listing detail, tour booking, in-app chat + call (landlord numbers hidden unless premium), pay rent, my rentals
- Landlord app: verify, create listing (photos + GPS + terms incl. the 6–24 month advance rule), requests inbox, mark-as-rented, wallet + payout request, subscription paywall
- Admin web: dashboard, listing moderation, user management, wallet ledger view, disputes
- Payments: one gateway (Paystack), MoMo + card, webhooks, receipts (in-app + SMS)
- Reviews, reports/blocking, push + SMS notifications

**Out (v2+):** featured/boosted listings, rent financing, web PWA, agency staff
accounts, API for partners, referencing/credit checks, ID scanning automation.

**Rough MVP milestones (team to sanity-check):**
1. Design system + screens (Vasti leads, ~2-3 wks)
2. Auth + landlord listing flow + admin moderation (~3-4 wks)
3. Renter search + tours + chat (~3 wks)
4. Payments + wallet + auto-occupied (~3-4 wks)
5. National launch — product supports all regions from day one; beta cohort of 20–50 landlords across regions → learn → scale

---

## 10. Risks & Honest Talk

| Risk | Reality | Mitigation |
|------|---------|------------|
| Chicken-and-egg (no landlords = no app) | Biggest risk | Free tier, founder-led onboarding. **Launch is nationwide from day one** — seed supply region by region (Accra, Kumasi, Takoradi, Tamale as first wave), run region-aware marketing |
| Scam/fake listings kill trust early | High | Verification gate before payouts, moderation queue, GPS pins, report system |
| Renters used to paying agents | Cultural | "Zero agent fee" messaging; renters pay nothing extra — fee is baked in |
| Cash-loving landlords | Real | Wallet + instant MoMo payout is the pitch: money lands SAME day, with records |
| Gateway downtime | Medium | Start with one gateway, design wallet ledger so a second can be added |
| Offline touring culture stays | Partial | Even if they tour offline, the LISTING + PAYMENT still runs through RentWave |

---

## 11. Decisions Locked (v0.2) & What's Still Open

**Locked by Selasi:**
1. **Contact policy:** landlord phone numbers NEVER show in the UI — in-app chat/call only. Numbers become visible only for premium (Pro+) landlords.
2. **Advance terms:** landlord sets their own terms, but the platform enforces a **6–24 month advance range** (6 months = minimum).
3. **Launch scope:** the **whole country at once** — region-based search from day one, supply seeded across regions.
4. **Payment gateway:** **Paystack** (MoMo + cards).
5. **Platform configurability (2026-09-12):** payment/SMS providers & their API keys, fee %/cap (default GH₵100), payout timing, settlement style — ALL runtime-changeable from the admin dashboard. No code deploys for ops changes. (Full design: TECHNICAL-DESIGN.md §8)

**Still open:**
- ✅ Name check DONE (2026-09-11, live RDAP + Play Store probe): rentwave.com TAKEN · rentwave.app TAKEN · **rentwavegh.com AVAILABLE** · **rentwave.africa AVAILABLE** · **Play Store: no "RentWave" app exists**. Action: register rentwavegh.com + .africa and secure social handles soon.
- Exact subscription pricing numbers (tiers/percentages) to be finalized.
- Team roles & staffing — deferred for now.

---

## 12. Success Metrics (how we know it's working)

- Landlords onboarded & % with verified badge
- Active listings per city; **listing → rented median days**
- Tour booking rate, tour → payment conversion
- GMV (total rent processed) & take rate revenue
- **Repeat/renewal rate** (renew in-app = the real SaaS proof)

---

*Prepared by Selasi (CTO, Lumora Tech) — v0.2, core decisions locked.*
