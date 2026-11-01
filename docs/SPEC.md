# CareX — practice app spec (Codex-first Playwright course)

Draft spec · Nov 1, 2026

## What it is
CareX is a fictional US healthcare platform for a network of clinics and hospitals, built to production-grade delivery standards:
- **Patient portal**: find care (clinic search), book visits, telehealth, lab results, health summary, secure messages, copay billing, documents, intake and consent.
- **Provider portal** (doctors, nurses, care coordinators): live waiting room, schedule, schedule board, charting with a streaming visit-summary assistant, lab orders, e-prescribing.
- **Admin console**: clinics and staff, patient directory, claims, audit log.
It is deliberately difficult: every hard real-world Playwright problem testers ask about is built in, and a **Chaos panel** switches problems on and off so lessons are repeatable.

Look and feel: calm, corporate healthcare UI (white/navy with one accent), desktop first at 1280×800. Real build stack: React + Lit web components, Node API, WebSocket gateway, payment sandbox on a second origin, mailbox service, Postgres; one `docker compose up`.

## Naming, legal and data rules (must ship with the app)
- Name "CareX" is Ramya's choice. A quick search (Nov 1) found existing healthcare brands with the same name: Carex® home health care products (Carex Health Brands, carex.com), CareX AI (Crunchbase), CareX apps on the app stores. Run a trademark search (USPTO + target markets) and get legal advice before publishing; a distinct variant may be safer. Not legal advice; no trademark search was run.
- Footer and login disclaimer: "CareX is a fictional training application. All people, clinics and records are made up. Not medical software. Not HIPAA-certified. Not affiliated with any healthcare provider."
- Fictional data only. No real PHI, no real doctor or clinic names. Do not copy Crossover Health's (or anyone's) name, logo, colours, copy or layouts.

## Priorities: features ranked by real tester pain
Evidence: top 25 Stack Overflow [playwright] questions by views (Stack Exchange API, Nov 1, 2026), most-upvoted Playwright GitHub issues, and the top-5 course review gaps. Build in this order.

| Rank | Feature in CareX | Where | Pain evidence | Lessons |
|---|---|---|---|---|
| 1 | Late/async UI: "Confirm booking" enables after 1–4 s; skeletons on lab results; slow insurance-eligibility check; filtered clinic search | Patient: Book visit, Lab results | SO #1, #2, #3, #6: 545,638 views | L11, L10, L13, L28 |
| 2 | Animations and interruptions: drawers, toasts, flu-shot campaign modal (~30% of loads), privacy-notice banner, click-blocking overlay | All portals | SO #4, #12, #18: 190,298 views | L12 |
| 3 | Navigation chain: login → MFA → redirect to the right portal (source of "Target closed") | Login, MFA | SO #5: 127,769 views | L09 |
| 4 | Auth and 5 roles (patient, provider, coordinator, admin, auditor); storageState; auth token in IndexedDB; auto-logoff | All portals | SO #7, #20: 95,828 views; GitHub #11164 (125 upvotes) | L32, L33, L35, L36 |
| 5 | New tab (visit summary), OAuth popup (connect pharmacy), telehealth popup window | Patient: Visits, Settings | SO #10: 55,628 views | L23 |
| 6 | Infinite-scroll audit log; virtualised directory of 10,000 fictional patients | Admin: Audit log, Patients | SO #9: 54,694 views | L16 |
| 7 | Unstable selectors: random IDs, hashed classes, a "Book" button on every clinic card (strict-mode trap), sparse data-testids | Patient: Find care | SO #11: 51,731 views; top-5 complaint about messy locators | L10, L15 |
| 8 | Flaky clicks under chaos: random 500s, latency, slow animation | Chaos panel | SO #21: 47,226 views | L51, L73 |
| 9 | Files: insurance card / photo ID drop zone (no visible input), progress, "scan" delay, wrong-type error; visit-summary PDF; claims CSV export | Patient: Documents; Admin: Claims | SO #14, #15: 44,373 views | L25, L26 |
| 10 | Hover menus and pointer-event drag and drop (move an appointment to a new slot) | Coordinator: Schedule board | SO #24: 42,331 views | L21 |
| 11 | Nested cross-origin iframes: copay payment sandbox → 3-D Secure challenge; embedded telehealth room | Patient: Billing, Telehealth | SO #13: 21,076 views | L19 |
| 12 | Time: auto-logoff warning after 15 idle min, OTP 30 s countdown, next-day slots released at midnight clinic time, US time zones, date/time picker with booked slots | Patient: Book visit; all portals | SO #22: 19,618 views; GitHub #6347 (240 upvotes) | L27, L22 |
| 13 | WebSocket: provider's live waiting room (patient checked in), live slot availability, "reconnecting…" banner | Provider: Waiting room | GitHub #4488 (171 upvotes) | L14 |
| 14 | Flaky/rate-limited APIs: insurance eligibility 429 + Retry-After; pharmacy outage; random 500s | Chaos panel | SO #23: 7,043 views; GitHub #8622 (94 upvotes) | L29, L30, L31 |
| 15 | Open shadow DOM design system: cx-button, cx-select, cx-dialog | All portals | SO #16: 7,673 views | L17, L48 |
| 16 | Canvas chart: vitals and lab trends (blood pressure, A1C) with tooltips | Patient: Health summary; Provider: Chart | SO #17: 7,596 views | L20, L44 |
| 17 | Closed shadow DOM: third-party e-consent signature widget | Patient: Intake and consent | Ramya's list; Playwright docs (closed roots not supported) | L18 |
| 18 | Streaming (SSE): visit-summary assistant streams its draft | Provider: Charting | Ramya's list | L13 |
| 19 | A/B flag booking_v2 | Patient: Book visit | Ramya's list | L15 |
| 20 | i18n: en-US, es-US, Arabic (RTL) | All portals | Ramya's list | L47, L45 |
| 21 | MFA: TOTP for providers, email OTP for patients (Mailbox) | Login, Mailbox | Ramya's list; Butch Mayhew's 2FA article | L34 |

Repo-level pains (config/.env SO #8, #25; Docker/CI SO #19) are taught in the test repo, not the app.

## Screens, routes and the Playwright challenge on each
1. **Login** `/login` → **MFA** `/mfa` — providers TOTP, patients email OTP (code in Mailbox); 30 s countdown; lockout after 5 wrong codes; redirect to the role's portal; auto-logoff warning after 15 idle minutes.
2. **Patient home** `/patient` — skeleton cards, flu-shot modal, privacy banner, toasts, upcoming visits, health summary canvas chart.
3. **Find care** `/patient/find-care` — clinic search with chaos latency; clinic cards (open-shadow cx- components) each with a "Book" button; "nearest clinic" uses geolocation.
4. **Book visit** `/patient/book` — date/time picker (booked/disabled slots, clinic time zone), live slot availability (WebSocket), insurance eligibility check (slow, 429 under chaos), "Confirm booking" enables late, A/B flag booking_v2.
5. **Visits** `/patient/visits` — cancel with confirm dialog; "View visit summary" opens a new tab (PDF download); "Join telehealth" opens a popup with camera/mic permission.
6. **Lab results** `/patient/labs` — delayed results, empty and error states (route mocking), trend chart.
7. **Billing** `/patient/billing` — copay payment iframe (second origin) → nested 3-D Secure iframe (approve/fail).
8. **Documents** `/patient/documents` — drop zone with no visible input, multi-file, progress, scan delay, wrong-type error.
9. **Intake and consent** `/patient/intake` — long form with unsaved-changes dialog; e-consent signature widget in a CLOSED shadow root.
10. **Settings** `/patient/settings` — language en-US / es-US / ar (RTL); "Connect pharmacy" OAuth popup; dismissed-tour flag in localStorage; token in IndexedDB.
11. **Provider waiting room** `/provider/waiting-room` — live WebSocket queue; table re-sorts as patients check in; random row IDs.
12. **Provider chart** `/provider/patients/:mrn` — vitals canvas chart, lab order (patient sees result appear in their portal — two-role test), streaming visit-summary assistant (SSE).
13. **Schedule board** `/coordinator/schedule` — drag appointments between slots (pointer-event library), hover menus, persists after reload.
14. **Admin patients** `/admin/patients` — virtualised grid of 10,000 fictional patients (MRN CX-000001…).
15. **Admin claims** `/admin/claims` — filters, CSV export download.
16. **Audit log** `/admin/audit` — infinite scroll, read-only for auditor.
17. **Chaos panel** `/chaos` (+ `POST /api/chaos`) — latency 0–8 s, error rate 0–50%, eligibility 429 with Retry-After, random IDs, booking_v2 forced/random, animation off/normal/slow, campaign-modal frequency, WebSocket drop.
18. **Mailbox** `/mailbox` + `GET /api/mailbox/:user/latest` — email OTPs and appointment emails.

## Test hooks (taught as testability contracts)
- `POST /api/test/seed`, `POST /api/test/reset` (one clinic namespace per worker for parallel runs)
- `POST /api/chaos`, `GET /api/flags`; seeded TOTP secrets for test providers documented in the repo
- data-testid only where roles/labels can't work (sparse on purpose)

## Sources
- Stack Overflow: Stack Exchange API, Nov 1 2026 (question links in the PDF, section 1).
- Playwright GitHub issues: github.com/microsoft/playwright/issues/6347, /4488, /11164, /8622.
- Playwright docs: playwright.dev/docs/locators (closed shadow roots not supported), /clock, /api/class-websocketroute.
- Naming search: carex.com; crunchbase.com/organization/carex-ai; play.google.com (CareX app) — search results, Nov 1 2026.
