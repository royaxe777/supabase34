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

*Future entries will be appended below as each phase is completed.*
