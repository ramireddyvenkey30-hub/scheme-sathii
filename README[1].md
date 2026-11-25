# ClubConnect Hive

Club management and cross-club collaboration portal: searchable club directory, membership, live resource
booking with **interval-based conflict detection**, joint event planner, campus calendar and club scorecard.

## Run it (no `npm install` needed)
Requires **Node.js 22.18+** (uses the built-in `node:sqlite` and TypeScript type-stripping, so there are zero runtime dependencies).
```bash
npm start            # http://localhost:3000  (first run creates data/hive.db and seeds demo data)
npm test             # 14 tests: conflict engine + full REST API
npm install && npm run typecheck   # optional: strict tsc check (installs typescript + @types/node)
```
`PORT=4000 npm start` changes the port. "Reset demo data" in the page footer restores the seed data.

## 3-minute demo script (matches the brief)
1. Top right, log in as **Aarav Reddy - President, Robotics Club**. Browse the **Directory** (search "coding", filter by category).
2. Open **Coding Society** -> *Propose joint event* -> title **AI Robo-Hack 2026** -> Send. The partner is invited; it shows on the **Calendar**.
   (Log in as Rohan Mehta, Coding president, to accept it: the event turns *confirmed*, and he can edit it too.)
3. **Book Resources** is pre-filled with **Main Audi, next Friday 2-4 PM** -> *Reserve* -> approved.
4. Switch to **Priya Nair - President, Dramatics Society**, same slot -> red **Facility Already Reserved** alert showing who
   holds it, plus a one-click *next free slot* suggestion. The live timeline highlights the clash.

## Architecture
```
src/conflict.ts   overlap test (StartA < EndB && EndA > StartB) + next-free-slot search
src/services.ts   business rules: clubs, membership, bookings, events, scorecard (SQL + transactions)
src/db.ts         SQLite schema: clubs, members, memberships, resources, bookings, events, event_clubs
src/seed.ts       8 clubs, 30 members, 8 resources (Main Audi, Seminar Hall A, Project Lab 3...), 12 events
src/app.ts        small REST router + static file server (node:http)
public/           responsive single-page UI (vanilla JS, dark-mode aware)
tests/            node:test suites
```
Key behaviours: back-to-back bookings are allowed (half-open intervals); multi-resource bookings are atomic
(a clash on the projector rolls back the hall too); joint events store co-host clubs in `event_clubs`, and officers of any
non-declined co-host club may edit; joining a club updates the roster instantly.

## REST API (JSON; `Authorization: Bearer <token>` from `/api/login`)
`GET /api/clubs?category=&q=` | `GET /api/clubs/:id` | `POST /api/clubs/:id/join|leave` | `GET /api/resources` |
`GET /api/availability?from=&to=` | `POST /api/bookings` (409 on conflict) | `DELETE /api/bookings/:id` |
`GET|POST /api/events` | `POST /api/events/:id/respond` | `PATCH /api/events/:id` | `GET /api/scorecard`

## Deviations from the suggested stack (and why)
- **Node `http` + `node:sqlite` instead of Express + a SQLite driver**: runs with no network or `npm install`, ideal for a hackathon demo machine. Routes are plain functions, so moving to Express is mechanical.
- **Plain CSS + vanilla JS front end** instead of TypeScript + Tailwind: no build step. The server and tests are TypeScript.
- Demo auth is a user picker (no passwords). Not for production.

## Roadmap (stretch features not yet built)
Faculty approval for auditorium bookings, QR attendance, inter-club chat, recurring bookings.
