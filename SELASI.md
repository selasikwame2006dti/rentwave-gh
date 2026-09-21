# SELASI.md — RentWave GH project memory

**Project:** RentWave GH — Ghana rental marketplace (SaaS, landlord-pays)
**Owner:** Selasi (Richmond), CTO — Lumora Tech
**Folder:** C:/Users/user/Desktop/RentWave-GH/

## Doc map
- `RENTWAVE-CONCEPT.md` — product concept v0.2 (business model, 3 sides, user journeys, locked decisions)
- `TECHNICAL-DESIGN.md` — technical design v0.1 (stack, schema, state machines, fee math, API surface, M0–M7 milestones, platform-settings §8)
- `brand/` — 5 SVG logo concepts + app icons + mono variants + showcase/showdown pages. **OFFICIAL MARK: B "Wave-W" — locked 2026-09-13.**
- `mockups/` — index.html hub + renter.html + landlord.html (6 phone screens each) + admin.html (desktop) + rentwave.css (shared design system). All visually QA'd 2026-09-11.
## Locked product decisions (do not relitigate)
1. Landlord pays (subscription tiers: Starter free/1 listing, Pro/10 + phone visible, Agency/50) + ~2.5% service fee BAKED into listed price (tenant never sees a fee).
2. Landlord phone numbers NEVER in any UI/API payload unless Pro+. In-app chat/call only. Enforce in server, not just UI.
3. Advance terms landlord-set, platform enforces 6–24 months (6 = hard floor) — enforced at DB level.
4. Nationwide launch day one (16 regions), not city-by-city.
5. Paystack gateway (MoMo + cards), webhook-first, idempotent via provider_reference.
6. Occupied listings auto-delist from search (core trust pillar).
7. **Platform configurability (2026-09-12):** drivers are code, config is data — payment/SMS providers + API keys, fee_pct/fee_cap (default 2.5%/GH₵100), payout_mode (default daily_batch), settlement_style (ship collect_then_wallet) all runtime-editable via admin (platform_settings + provider_configs tables, encrypted credentials, audit-logged, cached SettingsService). Nothing operational reads .env.

## Name status
- rentwave.com TAKEN, rentwave.app TAKEN. rentwavegh.com + rentwave.africa AVAILABLE (RDAP-checked 2026-09-11). No Play Store app named RentWave exists. Selasi to register domains + socials (his call — costs money).

## Stack (agreed)
Laravel 11 (PHP 8.3) API · MySQL 8 · **Web app (PWA) FIRST for testing (locked 2026-09-12, ship before Flutter)** · Flutter apps after pilot · React admin · Cloudinary media · Leaflet/OSM maps · FCM push · Arkesel SMS (driver-swappable) · Paystack.
**Why Flutter over Expo (decided 2026-09-13):** low-end Android perf (Tecno/Infinix market — Flutter compiles native + draws its own UI, no jank on budget devices) · self-contained toolchain (no npm dependency churn for a small team) · pixel-perfect brand rendering · first-class Paystack Flutter plugin (money flows ≠ community wrappers) · free OSM maps via flutter_map. Expo's OTA-update edge mostly negated by our web-first testing. Bonus: `flutter build web` can serve the web pilot from the same codebase later.

## Brand assets
`brand/` — **LOGO DECIDED (2026-09-13): Concept B "The Wave-W" is THE official mark** (Selasi's pick after the A/B/D showdown; B = crisp white tile, navy roofline, teal swells — best all-rounder, strongest on light surfaces). A/C/D/E RETAINED in brand/ as standby alternates (showcase + showdown pages still live — flip = edit one src). Exports: logo-horizontal.svg, rentwave-mark.svg, app icons, mono navy/white. Local copy for mockup pages: mockups/logo-b-wave-w.svg (preview can't reach ../brand). Palette: navy #0b2239 / teal #0ea5b7 / gold #ffb703.

## Money rules (critical)
- Integer pesewas everywhere (BIGINT). No floats.
- Wallet = append-only ledger (wallet_entries), balance_after snapshots; nightly invariant check.
- Money writes ONLY inside the payment success-handler service. Controllers never touch wallets.
- fee = min(round(amount × 2.5%), FEE_CAP) — cap TBD by Sammy.
- First successful rent payment ⇒ tenancy created + listing auto-flips to occupied.

## Gotchas / lessons during build
- Mockup QA: preview screenshots were the reliable path; scroll_preview often landed on non-scrollable phone frames — aim at page margins or use Selasi's manual screenshots.
- Windows flag emoji (🇬🇭) renders as tofu → replaced with 🌍 in admin.html.
- .screen needs min-height:0 inside flex column or bottom nav gets pushed out of the phone frame.
- Pro tier = 10 listings (doc) — mockup briefly said 5, fixed. Keep copies in sync.

## Open items (owner input needed)
1. ~~FEE_CAP~~ resolved 2026-09-12: default GH₵100, admin-changeable at runtime.
2. Paystack account creation under Lumora Tech — before M4.
3. SMS sender-ID registration lead time (Arkesel) — before M1.
4. ~~Payout timing~~ resolved 2026-09-12: admin setting, default daily_batch (cutoff configurable).
5. ~~Settlement style~~ resolved 2026-09-12: setting built; MVP ships collect_then_wallet, subaccount_split staged behind admin toggle.
6. Register rentwavegh.com + .africa + socials.

## Brand (FINAL)
- **Official mark: B "Wave-W"** — locked 2026-09-13. Wired into: mockups hub, renter + landlord topbars, admin sidebar, root portal + favicon. Local copy `mockups/logo-b-wave-w.svg` (preview can't reach `../brand` — keep this copy in sync if the mark changes).
- A/C/D/E kept in `brand/` as alternates — flip = edit one src. Showcase + showdown pages stay live for reference.

## Web-first decision (locked 2026-09-12)
- Before Flutter: lightweight responsive web test client (Blade + Tailwind, same API) to validate all flows in browser. Flutter after web flows prove out. Added to TECHNICAL-DESIGN.md §9.

## Next step
M0 scaffold (Laravel 11 + migrations for full schema + region seeds + Sanctum) — awaiting Selasi's go.
**Build order locked 2026-09-12: WEB-FIRST testing** — M2–M5 ship behind a responsive web client (Blade + Tailwind, same API) before Flutter starts. Money loop must pass go/no-go in browser (Paystack test keys) before any Flutter work.
