# CivicFix AI

Report a civic problem in Chennai, see exactly how it was triaged, and follow it
until it closes. Citizens file reports with a photo and a location; the
Corporation's operators work a prioritised queue with SLA targets, department
routing and an audited history on every issue.

## Running it

```bash
npm install
npm run dev          # http://localhost:3000
```

That is the whole setup. With no configuration the app runs in **local mode**:
sample city data is seeded into your browser, and the reports you file persist
there. Nothing is sent anywhere and no account is needed.

### Switching to real, shared storage

Create `.env.local` (see `.env.example`) and set at least:

```bash
DATABASE_URL=postgres://user:password@host:5432/civicfix
BETTER_AUTH_SECRET=$(openssl rand -base64 32)
ADMIN_PASSCODE=something-long-and-private
```

Then apply the schema and, optionally, some starter data:

```bash
npm run db:push      # idempotent; safe to re-run on every deploy
npm run db:seed      # only touches an empty database
```

With `DATABASE_URL` set the app switches to **database mode**: real accounts,
shared data, server-enforced roles. Citizens see only their own reports; the
whole city is visible only to an operator whose account holds the `admin` role.
An account is elevated by entering `ADMIN_PASSCODE` in the topbar — the check
runs on the server, is rate limited, and is compared in constant time. If
`ADMIN_PASSCODE` is unset, admin cannot be unlocked at all.

`BETTER_AUTH_SECRET` is required in production and must be at least 32
characters; the app refuses to start without it rather than fall back to a
shared default that would let anyone forge a session.

## Checks

```bash
npm run verify       # domain rules: triage, transitions, SLA, geography, seed data
npm run typecheck    # tsc --noEmit
npm run build        # type errors fail the build
```

## On a phone

The app is installable. Add it to the home screen and it opens standalone, with
shortcuts straight to "Report an issue" and "My reports". A half-written report
is kept on the device, so a call or a lock screen cannot lose it, and going
offline shows an honest message rather than a broken page.

## How triage works

Triage is deterministic and explainable, not a black box. `lib/domain.ts` scores
a report's own words against known risk vocabulary and a per-category floor, and
the resulting severity, priority, department and the reasoning behind them are
shown to the person who filed it — before they submit and afterwards. The same
report always produces the same routing.

Targets by priority: P1 six hours, P2 one day, P3 three days, P4 seven days.

## Layout

```
app/            routes, server actions (all authorisation lives here)
components/     UI; views.tsx holds the dashboard screens
lib/domain.ts   the shared vocabulary and rules — the file to read first
lib/db/         Drizzle schema and the SQL migration
lib/local-store.ts  browser persistence for local mode
scripts/        db:push, db:seed, verify
```

Times are always rendered in Chennai's clock, whatever time zone the viewer is
in, so an operator abroad reads the city's day and the server and browser never
disagree about what "today" means.
