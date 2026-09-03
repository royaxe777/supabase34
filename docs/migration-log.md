# Migration Log — QR-ATT Supabase Migration

> This log tracks every phase of the migration from SQLite to Supabase.
> Each entry records what changed, why, and the result.

---

## Entry Template

```markdown
### Phase X — Module Name

- **Date:** YYYY-MM-DD
- **Phase:** X
- **Module:** Module Name
- **Files changed:** list
- **Files created:** list
- **Files removed:** list
- **What changed:** description
- **Why it changed:** reason
- **Testing performed:** what was tested
- **Result:** pass/fail
- **Next step:** what comes next
```

---

## Log Entries

### Phase 0 — Baseline Documentation

- **Date:** 2026-09-03
- **Phase:** 0
- **Module:** Baseline and Safety
- **Files changed:** None
- **Files created:**
  - `docs/00-current-architecture.md`
  - `docs/migration-log.md`
- **Files removed:** None
- **What changed:** Created comprehensive documentation of the current application architecture, including file structure, database schema, navigation, components, screens, QR system, dependencies, limitations, and migration roadmap.
- **Why it changed:** Establishes a known-good baseline before any code modification. The student must understand the current state before seeing changes.
- **Testing performed:** Visual inspection of all files. No code execution changes.
- **Result:** Pass — All current files inspected and documented.
- **Next step:** Phase 1 — Supabase Project Setup (install dependencies, create client, introduce environment variables).

---

### Phase 1 — Supabase Project Setup

- **Date:** 2026-09-03
- **Phase:** 1
- **Module:** Supabase Project Setup
- **Files changed:**
  - `package.json` (added @supabase/supabase-js, expo-secure-store)
  - `.gitignore` (added .env)
- **Files created:**
  - `.env.example` (environment variable template)
  - `.env` (placeholder credentials)
  - `lib/supabase.ts` (Supabase client initialization)
  - `docs/01-project-setup.md` (updated with 18-step execution guide and Supabase dashboard walkthrough)
- **Files removed:** None
- **What changed:** Installed Supabase JavaScript client library and expo-secure-store. Created environment variable files (.env, .env.example). Updated .gitignore to prevent committing secrets. Created lib/supabase.ts with Supabase client initialization using secure token storage. Added detailed step-by-step execution guide (18 steps) including Supabase account creation, project setup, and credential retrieval with visual dashboard walkthrough. Added Expo SDK version compatibility section to prevent common version mismatch errors.
- **Why it changed:** Establishes the connection foundation for Supabase cloud services. Environment variables keep credentials separate from source code. The client module will be imported by all future service modules. Detailed steps ensure students new to Supabase can follow along without external help.
- **Testing performed:** TypeScript type check (npx tsc --noEmit) — passed with no errors. Verified package.json contains both new dependencies. Verified .gitignore includes .env.
- **Result:** Pass — Supabase client installed and configured. App still uses SQLite for data. No functional changes to the application.
- **Next step:** Phase 2 — Authentication (sign up, sign in, sign out, session management, replace hardcoded STUDENT_ID).

---

### Phase 2 — Authentication

- **Date:** 2026-09-03
- **Phase:** 2
- **Module:** Authentication
- **Files changed:**
  - `app/_layout.tsx` (simple Stack with all screens always registered)
  - `app/(tabs)/scan.tsx` (replaced STUDENT_ID with user.id from useAuth)
  - `app/(tabs)/history.tsx` (replaced STUDENT_ID with user.id from useAuth)
  - `lib/supabase.ts` (removed custom storage adapter, plain config)
- **Files created:**
  - `lib/auth.ts` (authentication service: useAuth hook, signUp, signIn, signOut with pub/sub global state)
  - `app/login.tsx` (login screen with email/password form, router.replace navigation)
  - `app/register.tsx` (registration screen with email/password/confirm form)
  - `app/(tabs)/profile.tsx` (shows user email/ID, Sign Out button with router.replace navigation)
  - `docs/02-authentication.md` (comprehensive documentation with 14 sections including known limitations)
- **Files removed:** None
- **What changed:** Replaced hardcoded STUDENT_ID with Supabase Authentication. Built a global auth state store using a pub/sub pattern instead of Supabase's `getSession()`/`onAuthStateChange` (which hang on mobile). Login and register screens navigate using `router.replace()` rather than conditional stack rendering. Sign out updates state immediately and fires Supabase server call in background (fire-and-forget).
- **Why it changed:** Enables real user accounts and multi-user support. The initial approach (getSession + onAuthStateChange + conditional Stack) failed because (1) `getSession()` hangs indefinitely due to stale session refresh attempts, (2) `onAuthStateChange` never fires in Expo Go, and (3) expo-router cannot cleanly switch between two different `<Stack>` elements. The pub/sub + explicit navigation approach reliably works.
- **Testing performed:** TypeScript type check (npx tsc --noEmit). Verified login screen appears on app start. Verified login with correct credentials navigates to tabs via router.replace. Verified wrong credentials shows error. Verified Sign Out from Profile navigates to login via router.replace.
- **Result:** Pass — Login and Sign Out work reliably. Users can register, log in, and log out. Known limitation: session does not restore on app refresh (users must log in again this phase).
- **Next step:** Phase 3 — Database Design (PostgreSQL schema, tables, relationships).

---

### Phase 3 — Database Design

- **Date:** 2026-09-03
- **Phase:** 3
- **Module:** Database Design
- **Files changed:** None (no code changes)
- **Files created:**
  - `supabase/schema.sql` (PostgreSQL schema: profiles, events, attendance + RLS + trigger)
  - `docs/03-database-design.md` (comprehensive 14-section documentation)
- **Files removed:** None
- **What changed:** Designed and wrote the PostgreSQL cloud schema that mirrors the original SQLite `events` and `attendance` tables. Added a `profiles` table that links to Supabase Auth (`auth.users`). Integrated `student_id`/`created_by` with `auth.uid()`. Added Row Level Security policies so each user can only read/write their own data. Added an auto-profile trigger that creates a profile on signup. Schema is written in `supabase/schema.sql` and is now live in the Supabase SQL Editor.
- **Why it changed:** Establishes the cloud data layer that will replace local SQLite. RLS ensures data security in a multi-user app. `auth.users` integration means the runtime user comes from the auth token (from Phase 2), not a hardcoded STUDENT_ID.
- **Testing performed:** Reviewed schema for correctness against the original SQLite schema. All tables/constraints/relationships documented. Schema executed in Supabase SQL Editor. Fixed reserved-word issue (`end` is a PostgreSQL keyword → renamed to `end_time`). Added `drop policy if exists` to make script idempotent/re-runnable. Confirmed the `profiles`, `events`, and `attendance` tables exist under the `public` schema in Table Editor (required a browser refresh).
- **Result:** Pass — Cloud database schema is live in Supabase. All three tables created with RLS enabled and the auto-profile trigger installed. App code still uses SQLite (migration happens in Phase 4+).
- **Next step:** Phase 4 — Supabase database service layer (`lib/database.ts` refactor).

---

### Phase 4 — Supabase Database Service

- **Date:** 2026-09-03
- **Phase:** 4
- **Module:** Database Service Layer
- **Files changed:**
  - `lib/database.ts` (refactored from SQLite to Supabase)
- **Files created:**
  - `docs/04-database-service.md` (comprehensive student-centered documentation with line-by-line code walkthrough)
- **Files removed:** None
- **What changed:** Rewrote the three data functions in `lib/database.ts` to use Supabase instead of SQLite. `registerAttendance(payload, studentId)` now validates the QR, checks the time window, finds-or-creates the cloud `events` row by `event_code`, then inserts into cloud `attendance`; duplicate scans are caught by PostgreSQL unique-constraint error code `23505`. `getAttendanceHistory(studentId)` now SELECTs from cloud `attendance` joined to `events` (nested select) and re-maps `scanned_at`/`event_code`/`title` back to the screen's expected `AttendanceRecord` shape. `createEvent(event)` now uses `.upsert(..., { onConflict: 'event_code' })` and records `created_by` from the logged-in user. The public function signatures and return types (`AttendanceRecord`, `RegisterResult`, `Event`) were preserved so no screen code needed changes.
- **Why it changed:** Replaces the local file storage with the secure, user-scoped, cloud PostgreSQL schema built in Phase 3. The query builder removes manual SQL strings, RLS enforces per-user access, and the database's unique constraint handles duplicate scans instead of fragile app-side checks. Keeping the same function contract means the Scan/History/Teacher screens are unchanged (encapsulation).
- **Testing performed:** TypeScript type check (`npx tsc --noEmit`) — passed with no errors, confirming the refactored module and all screens still compile. Verified each screen's usage matches the preserved signatures: `registerAttendance(data, studentId)` returns `{success, message}` (scan.tsx), `getAttendanceHistory(studentId)` returns `AttendanceRecord[]` (history.tsx), `createEvent({eventId,title,start,end})` (teacher.tsx).
- **Result:** Pass — `lib/database.ts` now reads/writes cloud Supabase data. Registration flow, history, and event creation all mapped to the Phase 3 schema. Function contract preserved so no screen edits were required.
- **Next step:** Phase 5 — Profile management (link `profiles` table to the Profile screen, per-user display data).
