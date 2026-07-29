# Cozy English — Teaching Studio

A cozy, minimalist web app for an online English teacher to manage students, plan
lessons, track progress automatically, run assessments, and communicate with
students — built with a Pinterest × Notion × Apple aesthetic (soft neutrals,
beige, warm grey, sage green, dusty blue, rounded cards, gentle motion).

## Tech stack

- **Next.js 16** (App Router) + **TypeScript**
- **Tailwind CSS v4** for the cozy design system (see `src/app/globals.css`)
- **Framer Motion** for animated progress bars, tab transitions, cards, modals
- **Recharts** for statistics (line/area/bar/radar/pie charts)
- **Zustand** (`persist` middleware) as the client data layer

## Two dashboards, one app

- **Teacher dashboard** (`/teacher/*`) — full CRUD over unlimited student
  profiles: add / archive / delete / duplicate, search, sort, and filter by
  level, progress, attendance, and goals. Lesson planning, lesson history,
  vocabulary/homework/attendance trackers, tests, learning goals, feedback,
  private teacher notes vs. student-visible notes, achievements, timeline,
  AI insights, statistics, calendar, and printable/CSV reports.
- **Student dashboard** (`/student/*`) — read-only view scoped to the logged-in
  student: progress, homework, lesson notes, vocabulary, feedback,
  achievements, upcoming lessons, learning goals, and statistics. Students
  cannot edit grades or scores.

Demo login (`/login`) lets you pick the teacher account or any seeded student
account — no password required, since this build ships with local demo data
instead of a live backend.

## Automatic progress tracking

`src/lib/scoring.ts` computes every score (`overall`, grammar, vocabulary,
listening, speaking, writing, reading, confidence, homework, attendance)
directly from logged activity — attendance records, homework status, lesson
assessments (star ratings), and tests. `computeScoreTimeline` replays a
student's history lesson-by-lesson to produce the growth charts on the
Statistics pages, so nothing is hand-entered or hardcoded.

`src/lib/insights.ts` generates the "AI Insights" (trend detection, weakest /
strongest skills, recommended revision topics, estimated CEFR level and
completion time) using rule-based analysis of the same stored data — no
external API key required, and it's structured so a real LLM call could be
swapped in later without touching the UI.

## Connecting a real backend (Supabase / PostgreSQL)

This build persists data to `localStorage` via Zustand so it runs anywhere
with zero configuration. To move to the production architecture described in
the spec (Supabase/Firebase auth + PostgreSQL, unlimited student profiles,
role-based access):

1. Create a Supabase project and mirror the shapes in `src/lib/types.ts` as
   Postgres tables (`students`, `lessons`, `vocabulary`, `homework`,
   `attendance`, `tests`, `learning_goals`, `achievements`, `feedback`,
   `teacher_notes`, `student_notes`, `lesson_plans`), each keyed by
   `student_id`, with row-level security so a `student` role can only `select`
   rows matching their own `student_id` and a `teacher` role has full access.
2. Replace `src/lib/auth.ts` with Supabase Auth (`supabase.auth.signInWithPassword`,
   magic links, etc.), mapping `auth.users` to a `role` claim.
3. Replace the Zustand store's in-memory state in `src/lib/store.ts` with
   Supabase queries/mutations (or keep Zustand as a cache layer backed by
   Supabase's realtime subscriptions for live updates across devices).
4. Swap `computeScores`/`computeScoreTimeline` to run as a Postgres function
   or Edge Function if you want scoring to happen server-side.

## Getting started

```bash
npm install
npm run dev
```

Open http://localhost:3000, choose **Teacher** or **Student** on the login
screen, and explore. Use **Settings → Export all data (JSON)** to inspect the
current local dataset, or **Reset demo data** to start fresh.
