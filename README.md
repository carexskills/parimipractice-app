# ParimiPractice — practice application

ParimiPractice is a **fictional US healthcare platform** (patient portal, provider portal, admin console for a network of clinics and hospitals) built as the practice application for every carexskills Playwright course.

It is deliberately hard to test. Each real-world problem testers hit with Playwright is built in on purpose, and a **Chaos panel** switches those problems on and off so every lesson is repeatable.

## How the courses use this repo
- Every course's tests run against ParimiPractice. Learners never edit this repo; they test it.
- Lessons pin a ParimiPractice release tag (for example `parimipractice-v1.0.0`), so a recorded lesson keeps working after the app evolves.
- The same app is reused by all future carexskills courses (UI, API, accessibility, visual, performance).

## Portals
| Portal | Users | Highlights |
|---|---|---|
| Patient | patients | find care, book visits, telehealth, lab results, billing, documents, intake and consent |
| Provider | doctors, nurses, care coordinators | live waiting room, schedule board, charting with a streaming visit-summary assistant |
| Admin | admins, auditors | clinics and staff, 10,000-patient directory, claims, audit log |

## Built-in testing challenges (ranked by real tester pain)
Late/async UI and skeletons · modals, toasts and overlays · login → MFA → redirect chains · 5 roles and storageState · new tabs and popups · infinite scroll and virtualised grids · unstable selectors and strict-mode traps · flaky clicks under chaos · file upload/download · drag and drop · nested cross-origin iframes (payment → 3-D Secure) · time (auto-logoff, OTP countdown, time zones) · WebSockets · rate-limited and flaky APIs · open and closed shadow DOM · canvas charts · SSE streaming · A/B flags · i18n incl. RTL · TOTP and email OTP.

The full spec, with routes and the lesson that teaches each challenge, is in [`docs/SPEC.md`](docs/SPEC.md).

## Layout
```
apps/web/        React + Lit web components (patient, provider, admin portals)
apps/api/        Node API, WebSocket gateway, SSE, mailbox and chaos endpoints
db/seed/         Fictional seed data (patients, clinics, staff) and seed scripts
docs/            Spec, architecture notes, release notes
docker-compose.yml   One `docker compose up` for web, api, payment sandbox, mailbox, Postgres
```

## Status
Placeholder structure. The app is being built; see `docs/SPEC.md` for the target.

## Running (once built)
```bash
cp .env.example .env
docker compose up
# web: http://localhost:5173   api: http://localhost:4000   payment sandbox: http://localhost:5174
```

## Test hooks (testability contract)
- `POST /api/test/seed`, `POST /api/test/reset` — one clinic namespace per worker for parallel runs
- `POST /api/chaos`, `GET /api/flags` — switch challenges on and off
- `GET /api/mailbox/:user/latest` — email OTPs and appointment emails
- `data-testid` only where roles and labels can't work (sparse on purpose)

## Legal
ParimiPractice is a fictional training application. Not a real medical service. All people, clinics and records are made up. Not medical software. Not HIPAA-certified. Not affiliated with any healthcare provider. No real patient data may ever be added to this repo.

---
**ParimiPractice is a fictional training application. Not a real medical service.** Built for carexskills courses. All people, clinics and records are made up; it is not medical software and not affiliated with any healthcare provider.

---
© 2026 carexskills. All rights reserved. Private repository, not for redistribution. See [LICENSE](LICENSE).
