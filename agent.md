
# agent.md — Garden Growth Companion

This document briefs AI coding assistants (e.g., Codex) and contributors on the product vision, scope, tech stack, and working conventions for the **Garden Growth Companion** app.

---

## 1) Product Overview
**Goal:** Help gardeners grow plants from seed to maturity with step-by-step guides, personalised logs, reminders, and a friendly community.  
**Audience:** Gardeners of all levels, with emphasis on accessibility for elderly, disabled, and low‑tech users.  
**Initial Platform:** Web‑first **PWA** with a clean path to native via Capacitor/React Native later.

---

## 2) MVP Scope (what to build first)
- **Auth & Profiles:** Email + Google login via Supabase Auth.
- **Plant Library:** Searchable list; plant detail pages with images, growth stages, care tips.
- **Personal Logs:** Create/view/edit/delete log entries (date, notes, photo). Attach to a plant.
- **Reminders:** User-set watering/fertilising schedules with push notifications (Web Push/OneSignal).
- **Offline Mode:** Installable PWA; cache essential assets + plant library for read-only offline.
- **Basic Admin (owner-only):** CRUD on plant records via a protected route.
- **Monetisation (phase-ready):** Free with ads; membership tier toggles ad-free (stubbed in MVP).

**Non‑Goals for MVP:** Smart-device integrations, full community feed, gamification, multilingual, advanced analytics.

---

## 3) Future Features (design for expansion)
- **Weather-based adjustments** to reminders per user location.
- **Community area** (Reddit‑style threads with images + comments; moderation & reporting).
- **Gamification** (badges for milestones).
- **Data portability** (CSV/JSON export/import).
- **Premium tier**: Ad-free, advanced guides, power-user features.
- **Smart devices**: Optional integration with irrigation/greenhouse sensors.
- **Multilingual** and localisation.

---

## 4) Tech Stack & Architecture
- **Frontend:** Next.js (App Router), React, TypeScript, Tailwind CSS, shadcn/ui (Radix).
- **Backend/Data:** Supabase (Postgres, Auth, Storage, Realtime). Next.js API routes for server tasks.
- **Notifications:** Web Push (VAPID) or OneSignal SDK (abstract via a `notifications` service).
- **Images:** Next/Image with remote loader; store originals in Supabase Storage.
- **Deployment:** Vercel (frontend + serverless functions). Supabase hosted.
- **Analytics:** Plausible (GDPR-friendly).

**High-level modules**
```
/app
  /(public) plant pages, home, auth
  /dashboard (authed)
  /plants [list/detail]
  /logs [mine/detail]
  /admin [owner-only]
/lib (supabase, notifications, validation, utils)
/components (ui, forms, cards, shells)
/styles, /public (manifest.json, icons), /workers (service worker)
```
**State:** Server Components + React Query (or SWR) for client fetching where needed.

---

## 5) Data Model (initial)
```sql
-- auth handled by Supabase (auth.users)

create table profiles (
  id uuid primary key references auth.users (id),
  display_name text,
  created_at timestamptz default now()
);

create table plants (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  image_url text,
  description text,
  care_guide jsonb,        -- { stages: [...], watering: "...", soil: "...", season: "..." }
  created_by uuid,         -- owner admin
  created_at timestamptz default now()
);

create table logs (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users (id) on delete cascade,
  plant_id uuid references plants (id) on delete cascade,
  log_date date not null,
  notes text,
  image_url text,
  created_at timestamptz default now()
);

create table reminders (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users (id) on delete cascade,
  plant_id uuid references plants (id) on delete cascade,
  type text check (type in ('water','fertilise','custom')),
  frequency_days int not null,
  next_due timestamptz,
  created_at timestamptz default now()
);
```

**RLS policy sketch (enable RLS on all tables):**
- `profiles`: users can select/update only their row.
- `logs`, `reminders`: user can CRUD only their rows.
- `plants`: public read; only owner/admin can insert/update/delete.

---

## 6) APIs & Services (Next.js API routes)
- `POST /api/logs` — create log (auth required)
- `GET /api/logs?plant_id=...` — list my logs
- `PATCH /api/logs/:id` — edit log
- `DELETE /api/logs/:id` — delete log
- `POST /api/reminders` — create/update reminder
- `POST /api/notifications/dispatch` — server job to send due reminders
- `GET /api/plants` — list plants (public)
- `POST /api/plants` — admin only (guard by role)

*Note:* Prefer direct Supabase client where possible; API routes used for privileged ops (e.g., notifications, admin, server-only keys).

---

## 7) UX & Accessibility Principles
- Large tap targets (44px+), high-contrast theme, readable typography (16–18px base).
- Simple navigation with 3–5 primary actions: **Home**, **Library**, **My Plants**, **Reminders**, **Profile**.
- Clear, step-by-step language; avoid jargon; show progress indicators.
- Works well with keyboard and screen readers; labels + aria attributes.
- Elderly-friendly defaults: can increase font size; avoid infinite scroll; obvious buttons.

---

## 8) Notifications & Weather (later phase)
- Store user’s approximate location (with consent).
- Nightly cron: fetch weather (e.g., OpenWeather) and adjust reminders (e.g., hot/dry => earlier watering).
- Respect browser permission states; fallback to email if push is blocked (optional).

---

## 9) Community & Moderation (future)
- Threads (title, body, images), comments, votes.
- Report content, rate limits, basic profanity filter.
- Owner/admin tools for hide/ban/remove.
- Image scanning (size/type) and safe-content heuristics.

---

## 10) Security, Privacy, Compliance
- GDPR: clear consent, data export/delete, privacy policy, cookie banner.
- Store only necessary personal data. No sensitive categories.
- Validate and sanitise all inputs. Use Zod or Yup on forms and APIs.
- Use Supabase RLS + row-level policies; never expose service keys to client.

---

## 11) Dev Workflow & Conventions
- **Branching:** `main` (protected), `dev`, `feature/*`.
- **Commits:** Conventional Commits (e.g., `feat: add plant log form`).
- **Code style:** TypeScript, ESLint, Prettier, strict mode.
- **CI/CD:** Vercel preview deployments on PRs. Lint & typecheck on push.
- **Testing:** Vitest/React Testing Library for critical flows (auth, logs, reminders).

---

## 12) Acceptance Criteria (MVP)
- User can sign up, sign in, and view their profile.
- User can browse a plant library (≥3 seed items) and open plant details.
- User can create, edit, and delete their own plant logs with an image.
- User can set a watering reminder and receive a push notification on schedule.
- App passes Lighthouse PWA installability and basic accessibility checks (score ≥90).
- Owner can add/edit plants through an admin route.

---

## 13) Local Setup (developer quickstart)
1. Clone repo; install deps:
   ```bash
   pnpm i  # or npm i
   ```
2. Create `.env.local` with Supabase keys:
   ```env
   NEXT_PUBLIC_SUPABASE_URL=...
   NEXT_PUBLIC_SUPABASE_ANON_KEY=...
   SUPABASE_SERVICE_ROLE_KEY=... # server-only
   NEXT_PUBLIC_ONESIGNAL_APP_ID=... # or VAPID keys for Web Push
   ```
3. Run dev server:
   ```bash
   pnpm dev
   ```
4. Seed plants via script or SQL; enable RLS and policies as above.
5. Test PWA (Lighthouse) and push to GitHub; deploy to Vercel.

---

## 14) Example Prompts for Codex / AI Pairing
- *“Create a Next.js server component that lists plants from Supabase with search and pagination; show image, name, and a link to details.”*
- *“Build a controlled form with Zod validation for creating a plant log (date, notes, photo upload to Supabase Storage).”*
- *“Implement a notifications service that schedules and dispatches Web Push reminders from `/api/notifications/dispatch`.”*
- *“Write Supabase RLS policies for logs so only the owner can read/write.”*
- *“Add a manifest.json and service worker to enable PWA install and offline caching of `/plants`.”*

---

## 15) Roadmap (suggested milestones)
- **M1 (Week 1–2):** Auth, Plant Library (read-only), PWA shell.
- **M2 (Week 3–4):** Logs CRUD + images, basic reminders.
- **M3 (Week 5):** Admin CRUD for plants, accessibility pass, beta.
- **M4 (Week 6):** Payments (membership toggle), ads integration, telemetry, launch.

---

## 16) Ownership & Roles
- **Product Owner:** Maintains private plant DB and editorial standards.
- **Maintainers:** Review PRs, enforce policies.
- **Contributors:** Good-first-issues labelled; follow this `agent.md`.
