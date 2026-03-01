# HRM Technical Architecture

## Stack

| Layer | Technology | Notes |
|---|---|---|
| Framework | Next.js 14+ (App Router) | SSR + CSR + API routes in one |
| UI Library | Chakra UI | Opinionated, accessible components |
| Database | Supabase (hosted cloud) | PostgreSQL + Auth + RLS |
| Hosting | Vercel | Auto-deploy from GitHub |
| Version Control | GitHub | Monorepo |
| Auth | Supabase Auth | Email/password only for MVP |
| Email Service | Resend (or similar) | For weekly digest emails |
| Dev Tool | Claude Code | AI-assisted development |

## Architecture Diagram

```
┌──────────────────────────────────────────┐
│                 Vercel                    │
│  ┌────────────────────────────────────┐  │
│  │       Next.js App Router           │  │
│  │  ┌────────────┐ ┌───────────────┐  │  │
│  │  │ Pages/     │ │ API Routes    │  │  │
│  │  │ Components │ │ /api/cron/*   │  │  │
│  │  │ (SSR+CSR)  │ │ /api/import   │  │  │
│  │  └────────────┘ └───────────────┘  │  │
│  └────────────────────────────────────┘  │
└────────────────┬─────────────────────────┘
                 │ Supabase Client SDK
┌────────────────▼─────────────────────────┐
│            Supabase Cloud                 │
│  ┌──────────┐ ┌──────┐ ┌──────────────┐  │
│  │PostgreSQL│ │ Auth │ │Edge Functions│  │
│  │  + RLS   │ │      │ │  (future)    │  │
│  └──────────┘ └──────┘ └──────────────┘  │
└──────────────────────────────────────────┘
```

## Key Architectural Decisions

1. **Next.js App Router** — SSR for landing/marketing pages, client-side interactivity for Kanban and dashboard. API routes for server-side operations (CSV parsing, cron triggers, email sending).

2. **Supabase RLS** — All data access controlled at database level. Every table has Row Level Security policies ensuring agents only access their own data. This is the security boundary.

3. **No custom backend** — No Express, no separate API server. Supabase handles auth + DB + realtime. Next.js API routes handle server-side logic.

4. **Future-proof schema** — `organization_id` and `team_id` columns exist on key tables from day one (nullable). MVP is single-agent only but schema supports teams without migration.

5. **Chakra UI** — Chosen for opinionated design system with warm, approachable aesthetic. Not Material UI (too corporate) or Tailwind (too much custom work for MVP timeline).

6. **Responsive web** — Not mobile-first, not desktop-first. Works on both. No native app for MVP.

## Route Map

```
/                              → Landing page (public)
/login                         → Auth
/signup                        → Registration
/onboarding                    → Multi-step onboarding
  /onboarding/assessment       → Referral Worthiness self-rating
  /onboarding/first-five       → Add 5 contacts (Memory Jogger)
  /onboarding/calendar         → Touch calendar wizard
  /onboarding/complete         → Success → redirect

/dashboard                     → Main app (protected)
  /dashboard/today             → Kanban board (HOME/DEFAULT)
  /dashboard/contacts          → Contact list view
  /dashboard/contacts/[id]     → Contact detail + Mackay 66
  /dashboard/contacts/import   → CSV import
  /dashboard/calendar          → Annual touch calendar
  /dashboard/insights          → ROI dashboard
  /dashboard/settings          → Profile, preferences, touch types
```

## Navigation Structure

Tab-based: bottom tabs on mobile, sidebar on desktop.

| Tab | Route | Description |
|---|---|---|
| Today | /dashboard/today | Kanban board — daily action view |
| Contacts | /dashboard/contacts | Full contact list with search/filter |
| Calendar | /dashboard/calendar | Annual touch calendar |
| Insights | /dashboard/insights | ROI dashboard and metrics |

## API Routes

```
POST   /api/contacts              → Create contact
GET    /api/contacts              → List (with filters, pagination)
GET    /api/contacts/[id]         → Detail
PATCH  /api/contacts/[id]         → Update
DELETE /api/contacts/[id]         → Archive (soft delete)
POST   /api/contacts/import       → CSV import (multipart)

POST   /api/touches               → Log touch
GET    /api/touches/[contactId]   → Touch history

POST   /api/referrals             → Log referral
POST   /api/transactions          → Log transaction

GET    /api/insights              → Dashboard metrics
GET    /api/kanban                → Contacts by Kanban column

POST   /api/touch-plans           → Create/update plan
GET    /api/touch-plans/active    → Active plan

GET    /api/cron/weekly-digest    → Cron (Vercel, auth by CRON_SECRET)
```

## Environment Variables

```env
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
NEXT_PUBLIC_APP_URL=
NEXT_PUBLIC_FREE_TIER_CONTACT_LIMIT=50
RESEND_API_KEY=
CRON_SECRET=
```

## Development Phases

| Phase | Weeks | Focus |
|---|---|---|
| 1: Foundation | 1–2 | Project setup, schema, auth, layout, navigation |
| 2: Contact Engine | 3–4 | Contact CRUD, Mackay 66, CSV import, segmentation |
| 3: Touch System + Kanban | 5–6 | Touch types/logging, Kanban, health score, badges |
| 4: Calendar + Insights | 7–8 | Onboarding, calendar wizard, dashboard, email digest, polish |

## Non-Functional Requirements

| Requirement | Target |
|---|---|
| Initial page load | < 2s |
| Kanban drag response | < 200ms |
| CSV import (500 rows) | < 30s |
| DB query (contact list) | < 500ms |
| Uptime | 99.5% |
| Responsive breakpoints | Mobile <768px, Tablet 768–1024px, Desktop >1024px |
| Browser support | Chrome, Safari, Firefox, Edge (latest 2) |
| Accessibility | WCAG 2.1 AA (Chakra baseline) |
