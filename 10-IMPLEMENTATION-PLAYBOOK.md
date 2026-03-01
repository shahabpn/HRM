# HRM Implementation Playbook

**Purpose:** Step-by-step implementation plan for building HRM with Claude Code. Each task is scoped to a single focused session (1–3 hours). Pick a task, paste the prompt into Claude Code, answer its questions, and build.

**How to use this document:**
1. Upload all knowledge base docs (00–09) to your Claude Code project
2. Each day, pick the next task from the current sprint
3. Copy the Claude Code prompt into your session
4. Answer Claude Code's pre-task interview questions
5. Build, test, commit
6. Check off the task and move to the next one

**Important:** Tell Claude Code at the start of each session:
> "Before building anything, read the referenced knowledge base documents. Then interview me with 3–5 clarifying questions specific to THIS task before writing any code."

---

## Sprint Overview

| Sprint | Weeks | Theme | Tasks |
|---|---|---|---|
| Sprint 0 | Pre-week 1 | Project Setup & Infrastructure | 5 tasks |
| Sprint 1 | Weeks 1–2 | Auth, Layout & Navigation | 6 tasks |
| Sprint 2 | Weeks 3–4 | Contact Engine | 8 tasks |
| Sprint 3 | Weeks 5–6 | Touch System & Kanban | 8 tasks |
| Sprint 4 | Weeks 7–8 | Calendar, Insights & Polish | 8 tasks |

**Total: 35 tasks across 5 sprints**

---

## SPRINT 0: Project Setup & Infrastructure
**Goal:** Everything is wired up. You can deploy a blank app to Vercel with Supabase connected.

---

### Task 0.1 — Initialize Next.js Project + GitHub Repo

**Time estimate:** 30–60 min
**References:** `02-ARCHITECTURE.md` (Stack, Environment Variables), `07-DEVELOPMENT-CONVENTIONS.md` (Project Structure, Key Libraries)
**Deliverable:** Next.js app running locally, pushed to GitHub, Vercel auto-deploying

**Claude Code Prompt:**
```
Read 02-ARCHITECTURE.md and 07-DEVELOPMENT-CONVENTIONS.md from the knowledge base.

I need you to initialize the HRM project. Before writing any code, interview me about:
- My preferred package manager (npm/yarn/pnpm)
- My GitHub org/username for the repo
- Any preferences on the Next.js create command options

Then:
1. Initialize Next.js 14+ with App Router, TypeScript, no src directory
2. Install all dependencies listed in 07-DEVELOPMENT-CONVENTIONS.md
3. Set up the folder structure matching the project structure spec
4. Create placeholder page.tsx files for all routes
5. Create .env.local with placeholder variables from 02-ARCHITECTURE.md
6. Create .gitignore
7. Verify it builds and runs locally
```

**Acceptance criteria:**
- [ ] `npm run dev` works
- [ ] Folder structure matches 07-DEVELOPMENT-CONVENTIONS.md
- [ ] All routes have placeholder pages
- [ ] Pushed to GitHub
- [ ] Vercel auto-deploys on push

---

### Task 0.2 — Chakra UI Theme Setup

**Time estimate:** 30–60 min
**References:** `05-UI-UX-DESIGN.md` (Design Philosophy, Color System, Typography)
**Deliverable:** Custom Chakra theme with HRM brand colors, working across all pages

**Claude Code Prompt:**
```
Read 05-UI-UX-DESIGN.md from the knowledge base.

Set up the Chakra UI theme for HRM. Before writing code, interview me about:
- My preferred primary color direction (coral/orange vs amber vs other warm tone)
- Light mode only or include dark mode for MVP?
- Any font preferences beyond the defaults?

Then:
1. Create theme/index.ts with custom Chakra theme
2. Configure ChakraProvider in the root layout
3. Set up color tokens matching the design system (primary, secondary, success, warning, error, neutral)
4. Configure Kanban column color accents
5. Set up typography scale
6. Create a simple test page that shows all theme colors and typography to verify
```

**Acceptance criteria:**
- [ ] Chakra provider wrapping the app
- [ ] Custom warm color palette applied
- [ ] All theme tokens accessible
- [ ] Test page showing color swatches and type scale

---

### Task 0.3 — Supabase Project + Schema Migration

**Time estimate:** 60–90 min
**References:** `03-DATABASE-SCHEMA.md` (all tables), `02-ARCHITECTURE.md` (Supabase config)
**Deliverable:** Supabase project created, all tables exist, RLS enabled, seed data loaded

**Claude Code Prompt:**
```
Read 03-DATABASE-SCHEMA.md completely.

I need to set up the Supabase database for HRM. Before writing SQL, interview me about:
- Have I already created a Supabase project? What's the project URL?
- Any modifications to the schema I want before we lock it in?
- Do I want to use Supabase CLI for migrations or run SQL directly in the dashboard?

Then:
1. Create a single migration file (001_initial_schema.sql) with ALL tables from 03-DATABASE-SCHEMA.md
2. Create 002_seed_touch_types.sql with all 18 default touch types
3. Create 003_seed_badges.sql with badge definitions
4. Create 004_rls_policies.sql with all RLS policies
5. Include all indexes from the schema doc
6. Add helpful comments in the SQL
```

**Acceptance criteria:**
- [ ] All tables created in Supabase
- [ ] RLS enabled on all agent-owned tables
- [ ] 18 default touch types seeded
- [ ] Badge definitions seeded
- [ ] All indexes created
- [ ] Can verify in Supabase dashboard

---

### Task 0.4 — Supabase Client Setup in Next.js

**Time estimate:** 30–45 min
**References:** `02-ARCHITECTURE.md` (Architecture Pattern), `07-DEVELOPMENT-CONVENTIONS.md` (Supabase Patterns)
**Deliverable:** Supabase client configured for both browser and server, env vars connected

**Claude Code Prompt:**
```
Read 02-ARCHITECTURE.md and the Supabase Patterns section of 07-DEVELOPMENT-CONVENTIONS.md.

Set up the Supabase client integration in our Next.js app. Interview me about:
- Do I have my Supabase URL and anon key ready?
- Any preference on using @supabase/ssr vs the basic client?

Then:
1. Create lib/supabase/client.ts (browser client)
2. Create lib/supabase/server.ts (server client for API routes and server components)
3. Create middleware.ts for auth session refresh
4. Update .env.local with actual Supabase credentials
5. Create a simple test API route that queries Supabase to verify the connection
```

**Acceptance criteria:**
- [ ] Browser client works from components
- [ ] Server client works from API routes
- [ ] Middleware refreshes auth session
- [ ] Test query to Supabase succeeds

---

### Task 0.5 — Auth Flow (Signup + Login + Logout + Protected Routes)

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F1: Authentication), `02-ARCHITECTURE.md` (Route Map)
**Deliverable:** Working signup, login, logout with route protection

**Claude Code Prompt:**
```
Read feature F1 in 04-FEATURES.md and the Route Map in 02-ARCHITECTURE.md.

Build the authentication flow for HRM. Interview me about:
- Do I want a combined login/signup page or separate pages?
- Any specific password requirements beyond Supabase defaults?
- Should the landing page (/) be the login page for now, or a separate marketing page?

Then:
1. Build /login page with email/password form
2. Build /signup page with email + password + full name
3. On signup: create auth user → create agent_profiles row → redirect to /onboarding
4. On login: redirect to /dashboard/today (or /onboarding if not completed)
5. Build logout functionality
6. Protect all /dashboard/* and /onboarding/* routes (redirect to /login if unauthenticated)
7. Redirect authenticated users away from /login and /signup
```

**Acceptance criteria:**
- [ ] Can sign up with email/password
- [ ] agent_profiles row created on signup
- [ ] Can log in
- [ ] Can log out
- [ ] /dashboard/* redirects to /login when not authenticated
- [ ] /login redirects to /dashboard when already authenticated
- [ ] New users without completed onboarding go to /onboarding

---

## SPRINT 1: App Shell, Layout & Onboarding
**Goal:** The app has a proper shell, navigation works, and new users go through the full onboarding flow.

---

### Task 1.1 — App Shell + Navigation Layout

**Time estimate:** 60–90 min
**References:** `05-UI-UX-DESIGN.md` (Layout Components, Responsive Behavior), `02-ARCHITECTURE.md` (Navigation Structure)
**Deliverable:** Working app shell with sidebar (desktop) and bottom tabs (mobile)

**Claude Code Prompt:**
```
Read the Layout Components and Responsive Behavior sections of 05-UI-UX-DESIGN.md, and the Navigation Structure in 02-ARCHITECTURE.md.

Build the app shell and navigation for HRM. Interview me about:
- Sidebar: always visible on desktop or collapsible?
- Do I want the agent's name/avatar in the sidebar header?
- Any specific icon library preference for nav icons (react-icons set)?

Then:
1. Build the AppShell layout component for /dashboard routes
2. Desktop: left sidebar with 4 nav tabs (Today, Contacts, Calendar, Insights) + Settings at bottom
3. Mobile: bottom tab bar with the same 4 tabs
4. Active tab highlighting
5. Header bar with agent name and settings gear icon
6. Responsive breakpoints matching the spec (mobile <768, tablet 768-1024, desktop >1024)
7. All tabs navigate to their respective routes
```

**Acceptance criteria:**
- [ ] Sidebar visible on desktop with 4 tabs
- [ ] Bottom tab bar on mobile
- [ ] Active tab visually distinct
- [ ] Navigation works between all 4 routes
- [ ] Responsive at all breakpoints

---

### Task 1.2 — Onboarding Step 1: Referral Worthiness Assessment

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F2: Onboarding, Step 1), `06-PRODUCT-METHODOLOGY.md` (Referral Worthiness Assessment)
**Deliverable:** Working assessment page that saves to agent_profiles

**Claude Code Prompt:**
```
Read F2 Step 1 in 04-FEATURES.md and the Referral Worthiness Assessment section in 06-PRODUCT-METHODOLOGY.md.

Build the first onboarding step. Interview me about:
- Should the sliders show labels at the extremes (e.g., "Struggling" to "Thriving")?
- Any motivational copy I want on this page?
- Should there be a progress bar showing 1 of 4 steps?

Then:
1. Build /onboarding/assessment page
2. Current rating: 1-10 slider
3. Runway question: single select (1 day / 1 week / 1 month / 1 year / 5 years)
4. Goal rating: 1-10 slider
5. Goal timeline: single select (3 months / 6 months / 12 months)
6. Save to agent_profiles on submit
7. Navigate to /onboarding/first-five on completion
8. Onboarding step progress indicator (step 1 of 4)
```

**Acceptance criteria:**
- [ ] All 4 inputs work correctly
- [ ] Data saves to agent_profiles
- [ ] Progress indicator shows step 1/4
- [ ] Navigates to step 2 on submit

---

### Task 1.3 — Onboarding Step 2: Add First 5 Contacts (Memory Jogger)

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F2: Step 2), `06-PRODUCT-METHODOLOGY.md` (Memory Jogger Categories)
**Deliverable:** Working contact quick-add with Memory Jogger prompts

**Claude Code Prompt:**
```
Read F2 Step 2 in 04-FEATURES.md and the Memory Jogger Categories in 06-PRODUCT-METHODOLOGY.md.

Build onboarding step 2. Interview me about:
- Should the Memory Jogger categories be scrollable chips, a grid, or a collapsible list?
- After adding a contact, should the form clear and stay open, or show a success message briefly?
- Should there be a "skip for now" option or must they add exactly 5?

Then:
1. Build /onboarding/first-five page
2. Display Memory Jogger categories as visual inspiration prompts
3. Quick-add form: first_name (required), last_name, phone, email
4. Counter: "3 of 5 contacts added"
5. Each submit creates a contact record + empty mackay_profiles row
6. Must add 5 to proceed (disable "Next" until 5 added)
7. Show list of added contacts below the form
8. Navigate to /onboarding/calendar on completion
```

**Acceptance criteria:**
- [ ] Memory Jogger categories displayed as prompts
- [ ] Can add contacts with minimal fields
- [ ] Counter updates correctly
- [ ] Contacts created in database with mackay_profiles
- [ ] Must add 5 before proceeding
- [ ] Navigates to step 3

---

### Task 1.4 — Onboarding Step 3: Touch Calendar Wizard

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F2: Step 3), `06-PRODUCT-METHODOLOGY.md` (Cadence Variation Rules), `03-DATABASE-SCHEMA.md` (touch_plans, touch_plan_items)
**Deliverable:** Working wizard that generates a touch plan and saves it

**Claude Code Prompt:**
```
Read F2 Step 3 in 04-FEATURES.md, the Cadence Variation Rules in 06-PRODUCT-METHODOLOGY.md, and the touch_plans tables in 03-DATABASE-SCHEMA.md.

Build the calendar wizard. Interview me about:
- Should the preview show a simplified calendar grid or just a summary of "you'll have X touches per month"?
- How much customization should the preview allow before saving?
- Should the wizard auto-select sensible defaults for each question?

Then:
1. Build /onboarding/calendar page
2. Question 1: annual touch target slider (24-36, default 30)
3. Question 2: preferred channels multi-select
4. Question 3: host client events? yes/no + frequency
5. Generate cadence_rules JSON based on answers (use the cadence matrix from methodology doc)
6. Generate touch_plan_items distributed across 12 months
7. Show calendar preview
8. Save touch_plan + touch_plan_items to database
9. Navigate to /onboarding/complete
```

**Acceptance criteria:**
- [ ] All 3 wizard questions work
- [ ] Cadence rules generated correctly per segment × tier
- [ ] Touch plan items distributed across months
- [ ] Preview displays generated plan
- [ ] Saved to database
- [ ] Navigates to completion step

---

### Task 1.5 — Onboarding Step 4: Complete + First Dashboard View

**Time estimate:** 30–45 min
**References:** `04-FEATURES.md` (F2: Step 4)
**Deliverable:** Completion page that sets onboarding flag and redirects to dashboard

**Claude Code Prompt:**
```
Read F2 Step 4 in 04-FEATURES.md.

Build the onboarding completion step. Interview me about:
- Any motivational copy or imagery for the completion screen?
- Should there be a brief animation or celebration moment?
- Direct redirect to /dashboard/today or a "You're ready" interstitial?

Then:
1. Build /onboarding/complete page
2. Show brief congratulations/success state
3. Set onboarding_completed_at on agent_profiles
4. CTA button: "Start Building Relationships" → /dashboard/today
5. Ensure login now redirects to dashboard (not onboarding) for this user
```

**Acceptance criteria:**
- [ ] Completion page displays
- [ ] onboarding_completed_at set in database
- [ ] CTA navigates to dashboard
- [ ] Subsequent logins go to dashboard, not onboarding

---

### Task 1.6 — Settings Page (Basic)

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F13: Settings)
**Deliverable:** Working settings page with profile editing

**Claude Code Prompt:**
```
Read F13 in 04-FEATURES.md.

Build the settings page. Interview me about:
- Which settings sections should exist in MVP? (Profile, Preferences, Subscription status, Data export?)
- Should settings be on a separate page or a slide-out panel?
- Do I want a "danger zone" section for account deletion?

Then:
1. Build /dashboard/settings page
2. Profile section: name, email (read-only), phone, brokerage, city, timezone
3. Preferences: daily touch goal, annual touch target, preferred channels
4. Subscription: show current tier + contact count/limit
5. Save changes to agent_profiles
6. All fields pre-populated from current data
```

**Acceptance criteria:**
- [ ] All profile fields editable and saveable
- [ ] Preferences update correctly
- [ ] Tier and contact usage displayed
- [ ] Changes persist after page reload

---

## SPRINT 2: Contact Engine
**Goal:** Full contact management — CRUD, Mackay 66 profiles, CSV import, segmentation, and the contact list view.

---

### Task 2.1 — Contact Creation Form

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F3: Create Contact), `03-DATABASE-SCHEMA.md` (contacts table)
**Deliverable:** Create contact modal/page with all core fields

**Claude Code Prompt:**
```
Read F3 Create Contact in 04-FEATURES.md and the contacts table in 03-DATABASE-SCHEMA.md.

Build the contact creation flow. Interview me about:
- Modal overlay or separate page for creating a contact?
- Should segment selection be required at creation or default to buyer_seller?
- Tag input: free-text chips or predefined options?

Then:
1. Build contact creation form (first_name required, everything else optional)
2. Fields: name, email, phone, address, primary_segment, tags, preferred_channel, notes
3. Auto-create contact_mackay_profiles row on contact creation
4. Enforce free tier contact limit (50) — show upgrade prompt if at limit
5. After creation: redirect to contact detail page OR close modal and refresh list
```

**Acceptance criteria:**
- [ ] Can create contact with just first_name
- [ ] All optional fields work
- [ ] Mackay profile row auto-created
- [ ] Free tier limit enforced
- [ ] Contact appears in database

---

### Task 2.2 — Contact List View

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F3: Contact List View), `05-UI-UX-DESIGN.md` (ContactListRow)
**Deliverable:** Searchable, filterable, sortable contact list

**Claude Code Prompt:**
```
Read the Contact List View section of F3 in 04-FEATURES.md and the ContactListRow spec in 05-UI-UX-DESIGN.md.

Build the contacts list page. Interview me about:
- Table layout or card grid layout?
- How many contacts per page before pagination kicks in?
- Should search be instant (client-side filter) or server-side?

Then:
1. Build /dashboard/contacts page
2. Contact list showing: name, segment badge, tier indicator, health score, last touch date, days since touch
3. Search bar: filter by name, email, phone
4. Filter dropdowns: segment, tier, tags
5. Sort options: name A-Z, last touch (oldest first), health score, created date
6. "Add Contact" button (opens creation form from Task 2.1)
7. Click row → navigate to /dashboard/contacts/[id]
8. Handle empty state (no contacts yet)
```

**Acceptance criteria:**
- [ ] All contacts displayed with key info
- [ ] Search works
- [ ] Filters work (segment, tier, tags)
- [ ] Sort works
- [ ] Navigation to detail page works
- [ ] Empty state shown when no contacts

---

### Task 2.3 — Contact Detail Page: Header + Overview Tab

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F3: Contact Detail Page), `05-UI-UX-DESIGN.md` (ContactDetailHeader)
**Deliverable:** Contact detail page with header, health score display, and overview tab

**Claude Code Prompt:**
```
Read the Contact Detail Page section of F3 in 04-FEATURES.md and ContactDetailHeader in 05-UI-UX-DESIGN.md.

Build the contact detail page header and overview. Interview me about:
- Should the health score be a circular progress ring, horizontal bar, or numeric display?
- For the overview tab, what's the ideal layout for the "key info" section?
- Should edit mode be inline or switch to a form view?

Then:
1. Build /dashboard/contacts/[id] page
2. Header: name, avatar/initials, segment badge, tier badge, health score, profile completeness
3. Quick action buttons: Log Touch, Edit, Archive
4. Tab navigation within the page: Overview | Profile | Touches | Referrals
5. Overview tab: key contact info, recent touches (last 5), next touch due, referrals count, notes
6. Fetch all data from Supabase on load
```

**Acceptance criteria:**
- [ ] Contact data loads and displays
- [ ] Health score and completeness visible
- [ ] Tab navigation works
- [ ] Overview shows recent touches and key info
- [ ] Quick action buttons present (functionality in later tasks)

---

### Task 2.4 — Mackay 66 Profile Tab

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F4: Mackay 66 Profile), `06-PRODUCT-METHODOLOGY.md` (The Mackay 66), `05-UI-UX-DESIGN.md` (MackayCategorySection)
**Deliverable:** All 66 fields displayed in expandable categories with inline editing

**Claude Code Prompt:**
```
Read F4 in 04-FEATURES.md, the full Mackay 66 section in 06-PRODUCT-METHODOLOGY.md, and MackayCategorySection in 05-UI-UX-DESIGN.md.

Build the Mackay 66 profile tab. Interview me about:
- Accordion (one section open at a time) or all expandable independently?
- Should empty fields show as blank inputs or as "Add..." placeholder text that becomes an input on click?
- Should children be a dynamic list (add/remove children) or fixed fields?

Then:
1. Build the Profile tab on the contact detail page
2. 7 expandable category sections matching the Mackay 66
3. Each section shows: category name, field count (e.g., "3/6"), badge indicator
4. All fields with inline editing (click to edit, blur or Enter to save)
5. Auto-save each field to contact_mackay_profiles JSONB
6. Recalculate profile_completeness_score on each save
7. Update fields_completed count
8. Children as a dynamic array (add/remove)
```

**Acceptance criteria:**
- [ ] All 66 fields present across 7 categories
- [ ] Expandable sections work
- [ ] Inline editing saves to database
- [ ] Field count updates per category
- [ ] Profile completeness score recalculates
- [ ] Children can be added/removed dynamically

---

### Task 2.5 — Profile Completeness Score Calculation

**Time estimate:** 30–45 min
**References:** `04-FEATURES.md` (F7: Relationship Health Score — Profile Depth section), `07-DEVELOPMENT-CONVENTIONS.md` (Scoring Calculation Patterns)
**Deliverable:** Working weighted profile depth calculation

**Claude Code Prompt:**
```
Read the Profile Depth Score section of F7 in 04-FEATURES.md and Scoring Calculation Patterns in 07-DEVELOPMENT-CONVENTIONS.md.

Build the profile depth scoring engine. Interview me about:
- Should scoring run client-side (instant feedback) or server-side (API call)?
- For JSONB fields, what counts as "filled"? Non-empty string? Or non-null?
- Should the score display update in real-time as fields are filled?

Then:
1. Create lib/scoring/profileDepth.ts
2. Implement weighted scoring: Family 20%, Special Interests 20%, Customer&You 15%, Business 15%, Personal 15%, Education 10%, Military 5%
3. Count non-empty fields per category
4. Calculate category completion % × weight
5. Sum for total profile_completeness_score (0-100)
6. Update contacts.profile_completeness_score after each Mackay field save
7. Wire into the contact detail header display
```

**Acceptance criteria:**
- [ ] Score calculates correctly with proper weights
- [ ] Empty vs filled detection works for all field types
- [ ] Score updates when Mackay fields are saved
- [ ] Score displays on contact header
- [ ] Score is 0 for a brand new contact

---

### Task 2.6 — CSV Import

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F3: CSV Import)
**Deliverable:** Working CSV upload with column mapping and import

**Claude Code Prompt:**
```
Read the CSV Import section of F3 in 04-FEATURES.md.

Build the CSV import flow. Interview me about:
- Maximum file size limit?
- Should mapping persist for repeat imports (save mapping template)?
- How should duplicates be handled in the UI — skip silently or show which were skipped?

Then:
1. Build /dashboard/contacts/import page
2. File upload zone (drag + click)
3. Parse CSV headers using papaparse
4. Column mapping UI: each CSV column → dropdown of HRM fields
5. Auto-detect common column names (first_name, email, phone, etc.)
6. Preview first 10 rows with mapped data
7. Import button: create contacts + mackay_profiles
8. Duplicate detection on email (per agent)
9. Results screen: X imported, Y skipped (duplicates), Z errors
10. Enforce contact limit (stop import if limit reached)
```

**Acceptance criteria:**
- [ ] Can upload CSV file
- [ ] Column mapping works with auto-detection
- [ ] Preview shows mapped data correctly
- [ ] Contacts created in database
- [ ] Duplicates detected and skipped
- [ ] Results summary shown
- [ ] Free tier limit enforced

---

### Task 2.7 — Contact Edit + Archive

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F3: Edit Contact, Archive Contact)
**Deliverable:** Edit all contact fields, archive (soft delete), recover

**Claude Code Prompt:**
```
Read the Edit Contact and Archive Contact sections of F3 in 04-FEATURES.md.

Build contact editing and archiving. Interview me about:
- Edit as a modal or inline on the detail page?
- Archive confirmation dialog needed?
- Should there be an "Archived Contacts" section to view/recover?

Then:
1. Edit mode on contact detail: all core fields editable
2. Segment change dropdown
3. Tag management (add/remove tags)
4. Manual tier override (with tier_override flag)
5. Archive button with confirmation dialog
6. Archive = set archived_at timestamp, hide from all views
7. Optional: "View Archived" toggle on contacts list to see/recover archived contacts
```

**Acceptance criteria:**
- [ ] All fields editable
- [ ] Segment and tags changeable
- [ ] Manual tier override works
- [ ] Archive soft deletes (sets archived_at)
- [ ] Archived contacts hidden from list and Kanban
- [ ] Can view and recover archived contacts

---

### Task 2.8 — Contact Detail: Touch History + Referrals Tabs

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F3: Contact Detail — Tabs 3 & 4)
**Deliverable:** Touch history timeline and referrals list on contact detail

**Claude Code Prompt:**
```
Read the contact detail Tabs 3 and 4 in F3 of 04-FEATURES.md, and the TouchTimelineItem spec in 05-UI-UX-DESIGN.md.

Build the remaining contact detail tabs. Interview me about:
- Touch history: simple list or visual timeline with icons?
- Should the touch history be paginated or infinite scroll?
- For the referrals tab, should "Log Referral" be inline or a modal?

Then:
1. Touch History tab: chronological list of all touches for this contact
2. Each touch shows: type icon, type name, date, note (if any)
3. Filter by touch type
4. Referrals tab: list of referrals this contact has given
5. Each referral shows: referred name, date, status, outcome
6. Empty states for both tabs
```

**Acceptance criteria:**
- [ ] Touch history displays all logged touches
- [ ] Filter by type works
- [ ] Referrals tab shows referral records
- [ ] Empty states display properly
- [ ] Data loads from correct database tables

---

## SPRINT 3: Touch System & Kanban
**Goal:** The core daily workflow — logging touches, Kanban board, health scoring, and badges.

---

### Task 3.1 — Touch Type Management

**Time estimate:** 30–45 min
**References:** `03-DATABASE-SCHEMA.md` (touch_types), `04-FEATURES.md` (F6: Touch Types)
**Deliverable:** Settings page section for viewing/managing touch types

**Claude Code Prompt:**
```
Read the touch_types table in 03-DATABASE-SCHEMA.md and the Touch Types section of F6 in 04-FEATURES.md.

Build touch type management. Interview me about:
- Where should custom touch type creation live — Settings page or a dedicated section?
- Should agents be able to reorder touch types?
- Can agents edit system defaults or only deactivate them?

Then:
1. Display all touch types (system + custom) on Settings page
2. Show: icon, name, category, suggested frequency, active/inactive toggle
3. System defaults: can deactivate but not edit or delete
4. Custom types: can add, edit, deactivate, delete
5. Add custom type form: name, category (dropdown), icon (picker or dropdown), frequency
```

**Acceptance criteria:**
- [ ] All 18 system defaults displayed
- [ ] Can deactivate/reactivate system types
- [ ] Can add custom touch types
- [ ] Can edit/delete custom types
- [ ] Deactivated types hidden from touch logging dropdown

---

### Task 3.2 — Quick Touch Logging Modal

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F6: Touch Logging MVP), `05-UI-UX-DESIGN.md` (QuickTouchModal)
**Deliverable:** 3-tap touch logging that updates all scores

**Claude Code Prompt:**
```
Read F6 Touch Logging in 04-FEATURES.md and QuickTouchModal in 05-UI-UX-DESIGN.md.

Build the quick touch logging system. Interview me about:
- Should the modal be accessible from both the Kanban card AND the contact detail page?
- After logging, should the modal auto-close or stay open for logging another touch?
- Should there be a success animation/toast?

Then:
1. Build QuickTouchModal component
2. Contact pre-selected (if opened from a specific card) or searchable
3. Touch type dropdown (active types only, system + custom)
4. Date picker (defaults to today)
5. On save: insert contact_touches record
6. After save: update contacts.last_touch_date, increment total_touches_ytd
7. Trigger score recalculation (touch_consistency_score, relationship_health_score)
8. Recalculate next_touch_due based on cadence rules
9. Success toast
```

**Acceptance criteria:**
- [ ] Modal opens from multiple entry points
- [ ] Can select contact, type, and date
- [ ] Touch logged to database
- [ ] last_touch_date updated
- [ ] Scores recalculated
- [ ] next_touch_due recalculated
- [ ] Success feedback shown

---

### Task 3.3 — Touch Consistency Score + Health Score Calculation

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F7: Touch Consistency Score, Composite Health Score), `07-DEVELOPMENT-CONVENTIONS.md` (Scoring Patterns)
**Deliverable:** Full health score calculation engine

**Claude Code Prompt:**
```
Read all of F7 in 04-FEATURES.md and the Scoring section in 07-DEVELOPMENT-CONVENTIONS.md.

Build the complete scoring engine. Interview me about:
- Should recency bonuses/penalties use the exact values in the spec or should I adjust?
- When a contact has no touch plan (no cadence rules), what's their consistency score?
- Should score changes be logged for historical tracking?

Then:
1. Create lib/scoring/touchConsistency.ts
2. Calculate expected touches YTD based on cadence_rules[segment][tier]
3. Calculate actual touches YTD from contact_touches
4. Compute ratio, cap at 100
5. Apply recency bonus/penalty
6. Create lib/scoring/healthScore.ts
7. Composite: (profile_depth × 0.35) + (touch_consistency × 0.65)
8. Update contacts table with all three scores
9. Implement tier suggestion logic (Advocacy/Loyal/Satisfied thresholds)
10. Wire into touch logging flow (Task 3.2)
```

**Acceptance criteria:**
- [ ] Touch consistency calculates correctly
- [ ] Recency bonus/penalty applied
- [ ] Composite health score = 0.35 × profile + 0.65 × consistency
- [ ] Tier suggestion works based on score + referral history
- [ ] Scores update after touch logging
- [ ] Scores update after profile field changes

---

### Task 3.4 — next_touch_due Calculation Engine

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F5: Calendar → Kanban Integration), `03-DATABASE-SCHEMA.md` (touch_plans, cadence_rules)
**Deliverable:** Automatic next_touch_due date calculation based on cadence rules and touch plan

**Claude Code Prompt:**
```
Read the Calendar → Kanban Integration section of F9 in 04-FEATURES.md and the touch_plans table in 03-DATABASE-SCHEMA.md.

Build the next_touch_due calculation engine. Interview me about:
- When a contact has no touch plan, should next_touch_due be null or set to a default interval?
- If a contact is overdue by 30 days and gets touched, does next_touch_due snap to the next scheduled item or calculate from today?
- Should the system distribute contacts across the week (not all due Monday)?

Then:
1. Create lib/calendar/nextTouchDue.ts
2. Given a contact's segment, tier, and the agent's active touch_plan:
   - Look up cadence_rules for that segment×tier
   - Find the next touch_plan_item that applies to this contact
   - Calculate the next due date
3. Distribute contacts across weekdays to avoid clustering
4. Update next_touch_due on contact after each touch is logged
5. Update next_touch_due when touch plans change
6. Handle edge cases: no plan, new contact, overdue contacts
```

**Acceptance criteria:**
- [ ] next_touch_due calculated correctly from cadence rules
- [ ] Updates after each touch logged
- [ ] Handles contacts with no plan (NULL or default)
- [ ] Distribution prevents all contacts being due on same day
- [ ] Handles overdue contacts correctly after being touched

---

### Task 3.5 — Kanban Board: Layout + Columns

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F5: Kanban Board), `05-UI-UX-DESIGN.md` (KanbanBoard, KanbanColumn, KanbanSummaryBar)
**Deliverable:** Kanban board with 5 columns populated from database

**Claude Code Prompt:**
```
Read F5 in 04-FEATURES.md and all Kanban components in 05-UI-UX-DESIGN.md.

Build the Kanban board layout. Interview me about:
- Should columns show a max number of cards with "show more" or scroll within column?
- Sort order within each column (most urgent first? alphabetical?)
- Should the summary bar be sticky when scrolling?

Then:
1. Build /dashboard/today page with KanbanBoard component
2. 5 columns: Overdue, Due Today, Due This Week, Up to Date, No Plan
3. Column headers with count badges and color accents
4. Query contacts grouped by Kanban column logic (use the SQL CASE from schema doc)
5. KanbanSummaryBar: contact counts per column + daily progress counter
6. Populate with real contact data
7. Handle empty board state (no contacts)
8. Responsive: 5 columns on desktop, swipeable on mobile
```

**Acceptance criteria:**
- [ ] 5 columns render with correct color accents
- [ ] Contacts sorted into correct columns based on next_touch_due
- [ ] Summary bar shows counts
- [ ] Responsive layout works
- [ ] Empty state handled

---

### Task 3.6 — Kanban Board: Contact Cards + Drag & Drop

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F5: Contact Card on Kanban, Interactions), `05-UI-UX-DESIGN.md` (ContactCard Kanban)
**Deliverable:** Draggable contact cards with all info, interactions working

**Claude Code Prompt:**
```
Read the Contact Card and Interactions sections of F5 in 04-FEATURES.md and ContactCard (Kanban) in 05-UI-UX-DESIGN.md.

Build the Kanban contact cards with drag-and-drop. Interview me about:
- Which drag-and-drop library to use? (@dnd-kit is in the spec — confirm or change)
- What happens when you drag a card to "Up to Date"? Set next_touch_due to next week? Next month?
- Should drag between columns update the database immediately or batch?

Then:
1. Build KanbanCard component matching the spec
2. Show: name, avatar, segment badge, tier stars, health score ring, last touch info, next suggested touch
3. Install and configure @dnd-kit for drag and drop
4. Drag card between columns → update next_touch_due in database
5. Click card → open QuickTouchModal (from Task 3.2)
6. Click name → navigate to contact detail
7. Filter controls: segment, tier, tags
8. Smooth drag animation
```

**Acceptance criteria:**
- [ ] Cards display all specified info
- [ ] Drag and drop works between all columns
- [ ] Database updates on drop
- [ ] Click → touch modal works
- [ ] Click name → detail page works
- [ ] Filters work
- [ ] Smooth animations

---

### Task 3.7 — Badge System

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F8: Badge System)
**Deliverable:** Badge evaluation engine + display on contact cards and detail pages

**Claude Code Prompt:**
```
Read F8 in 04-FEATURES.md.

Build the badge system. Interview me about:
- Should badge evaluation run on every touch/profile update or on a schedule?
- Should there be a notification/toast when a new badge is earned?
- How should badges display on the compact Kanban card (icons only? limit to 3?)

Then:
1. Create lib/badges/evaluator.ts
2. Evaluate all badge criteria against a contact's current data
3. Badge types: profile-based (Mackay completion), touch-based (streaks, consistency), referral-based, milestone-based
4. When a new badge is earned: add to contacts.badges_earned JSONB, show toast notification
5. Display badges on Kanban cards (small icons, max 3)
6. Display full badge list on contact detail page
7. Show locked/unearned badges with unlock criteria
```

**Acceptance criteria:**
- [ ] All badge criteria evaluate correctly
- [ ] New badges earned trigger toast notification
- [ ] Badges display on Kanban cards
- [ ] Full badge list on contact detail
- [ ] Locked badges show unlock requirements

---

### Task 3.8 — Referral + Transaction Logging

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F11: Referral & Transaction Logging)
**Deliverable:** Working referral and transaction logging from contact detail

**Claude Code Prompt:**
```
Read F11 in 04-FEATURES.md.

Build referral and transaction logging. Interview me about:
- Should referral logging be a modal or inline on the referrals tab?
- When a referral is marked as "converted" should it auto-create a transaction?
- Should transactions be loggable from the insights page too, or only from contact detail?

Then:
1. Referral logging: select referrer, enter referred name, optional link to existing contact
2. Referral status tracking: received → contacted → converted → lost
3. Update contacts.total_referrals_given when referral is logged
4. Transaction logging: date, type (buy/sell/both), source, optional link to contact and referral
5. After logging: update tier suggestion (referrals affect tier suggestion)
6. Wire referrals tab on contact detail to show referral records
```

**Acceptance criteria:**
- [ ] Can log a referral from contact detail
- [ ] Referral status can be updated
- [ ] Can log a transaction
- [ ] Referral count updates on contact
- [ ] Tier suggestion recalculates
- [ ] Referrals tab shows logged referrals

---

## SPRINT 4: Calendar, Insights & Polish
**Goal:** The remaining views, analytics dashboard, email digest, and overall polish for MVP launch.

---

### Task 4.1 — Annual Touch Calendar View

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F9: Annual Touch Calendar), `05-UI-UX-DESIGN.md` (AnnualCalendarGrid)
**Deliverable:** 12-month calendar grid showing planned touches and completion status

**Claude Code Prompt:**
```
Read F9 in 04-FEATURES.md and AnnualCalendarGrid in 05-UI-UX-DESIGN.md.

Build the calendar view. Interview me about:
- Should the calendar be read-only (view plan) or editable (add/remove items)?
- How to show completion: per touch item or per month aggregate?
- Should the calendar pull from the wizard-generated plan or allow manual additions?

Then:
1. Build /dashboard/calendar page
2. 12-month grid: rows = months, columns = touch categories
3. Populate from touch_plan_items for the active plan
4. Color coding: planned (gray), completed (green), missed (red)
5. Click cell → view details of that touch item
6. Show overall completion stats (X of Y planned touches completed this year)
7. Responsive: full grid on desktop, month-by-month on mobile
8. Allow editing the plan (add/remove/modify items)
```

**Acceptance criteria:**
- [ ] 12-month grid renders correctly
- [ ] Touch plan items displayed in correct months/categories
- [ ] Completion status color coded
- [ ] Click for details works
- [ ] Editable
- [ ] Responsive

---

### Task 4.2 — Insights Dashboard: Metric Cards

**Time estimate:** 45–60 min
**References:** `04-FEATURES.md` (F10: Insights Dashboard — Primary Metrics)
**Deliverable:** Top-level metric cards on insights page

**Claude Code Prompt:**
```
Read the Primary Metrics section of F10 in 04-FEATURES.md and MetricCard in 05-UI-UX-DESIGN.md.

Build the insights metric cards. Interview me about:
- Should trends show vs last month, last quarter, or configurable?
- How to handle "insufficient data" for new users (minimum data threshold)?
- Should cards be clickable (drill down) or display-only?

Then:
1. Build /dashboard/insights page header with metric cards
2. Card 1: Database Size (total active contacts + trend)
3. Card 2: Database Return Rate (transactions / contacts with target line at 15%)
4. Card 3: Touch Consistency (% of contacts on track)
5. Card 4: Overdue Contacts (count + %)
6. Create API route /api/insights to aggregate these metrics
7. Handle edge cases: new user with no data, user with no transactions
```

**Acceptance criteria:**
- [ ] All 4 metric cards display
- [ ] Numbers are accurate
- [ ] Trends show direction
- [ ] Handles new users gracefully
- [ ] API aggregation is performant

---

### Task 4.3 — Insights Dashboard: Charts + Lists

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F10: Charts, Lists)
**Deliverable:** Full insights dashboard with charts and ranked lists

**Claude Code Prompt:**
```
Read the Charts and Lists sections of F10 in 04-FEATURES.md.

Build the insights charts and lists. Interview me about:
- Chart library: recharts (in spec) or Chart.js? Preference?
- Should "Most Neglected" list be clickable (navigate to contact)?
- How far back should the monthly touch chart go (6 months? 12?)

Then:
1. Tier Distribution donut/pie chart (Satisfied vs Loyal vs Advocacy)
2. Monthly Touch Activity bar chart (last 12 months)
3. Referrals This Year counter with trend
4. Most Neglected list: top 10 contacts by days since last touch (clickable)
5. Most Engaged list: top 10 by touch count this year
6. Referral Worthiness Progress: current score vs goal (from onboarding)
7. Layout all components responsively on the insights page
```

**Acceptance criteria:**
- [ ] Donut chart shows tier distribution
- [ ] Bar chart shows monthly touch activity
- [ ] Both lists populated with real data
- [ ] Lists are clickable → navigate to contact
- [ ] Referral worthiness progress displays
- [ ] Responsive layout

---

### Task 4.4 — Weekly Email Digest

**Time estimate:** 60–90 min
**References:** `04-FEATURES.md` (F12: Email Digest), `02-ARCHITECTURE.md` (cron)
**Deliverable:** Vercel cron job sending weekly digest email

**Claude Code Prompt:**
```
Read F12 in 04-FEATURES.md and the cron section in 02-ARCHITECTURE.md.

Build the weekly email digest. Interview me about:
- Email service: Resend, SendGrid, or another provider?
- Should agents be able to opt out of the digest?
- HTML email or plain text for MVP?

Then:
1. Set up email service (Resend or chosen provider)
2. Create /api/cron/weekly-digest API route
3. Query all agents who should receive digest
4. For each agent: aggregate weekly stats, contacts due this week, special dates
5. Build email template (HTML)
6. Send via email service
7. Configure Vercel cron: Monday 7:00 AM UTC (adjust per timezone later)
8. Authenticate cron route with CRON_SECRET
```

**Acceptance criteria:**
- [ ] Cron job executes on schedule
- [ ] Email contains: week summary, upcoming contacts, database health
- [ ] Email renders correctly
- [ ] CRON_SECRET authentication works
- [ ] Can manually trigger for testing

---

### Task 4.5 — Landing Page

**Time estimate:** 60–90 min
**References:** `08-VISION-POSITIONING.md` (Positioning, Target User, Competitive Positioning)
**Deliverable:** Marketing landing page at / route

**Claude Code Prompt:**
```
Read 08-VISION-POSITIONING.md — specifically the Problem, Positioning, Target User, and Competitive Positioning sections.

Build the landing page. Interview me about:
- Do we have a product name yet or still using "HRM"?
- Do we have a hero image/illustration or should we use placeholder?
- What's the primary CTA — "Start Free" or "Sign Up"?
- Should there be a pricing section on the landing page?

Then:
1. Build / page (public, not authenticated)
2. Hero section: headline, subheadline, CTA
3. Problem section: the pain points established agents face
4. Solution section: the 3 systems (depth, cadence, ROI)
5. Social proof section (placeholder for testimonials)
6. CTA section: sign up / start free
7. Mobile responsive
8. Navigation: logo + Login + Sign Up buttons
```

**Acceptance criteria:**
- [ ] Professional landing page
- [ ] Clear value proposition
- [ ] CTA leads to signup
- [ ] Responsive
- [ ] Matches brand warmth/energy from design system

---

### Task 4.6 — Free Tier Enforcement + Upgrade Prompts

**Time estimate:** 30–45 min
**References:** `01-PROJECT-OVERVIEW.md` (Business Model), `04-FEATURES.md` (F3: contact limit)
**Deliverable:** Consistent contact limit enforcement and upgrade UX

**Claude Code Prompt:**
```
Read the Business Model section in 01-PROJECT-OVERVIEW.md.

Implement free tier enforcement across the app. Interview me about:
- What happens when they try to import CSV that would exceed 50? Import partial or block entirely?
- Should the upgrade prompt be a modal, banner, or inline message?
- For MVP, is "upgrade" just a contact email/form or Stripe integration?

Then:
1. Check contact count before: creating contact, CSV import
2. Show contact usage in settings (e.g., "42 / 50 contacts used")
3. Show usage indicator in contacts list header
4. Friendly upgrade prompt when limit reached (not a hard error)
5. Upgrade CTA (for MVP: link to contact/waitlist, not Stripe)
6. Block creation gracefully with helpful messaging
```

**Acceptance criteria:**
- [ ] Cannot create contacts beyond limit
- [ ] CSV import respects limit
- [ ] Usage visible in settings and contacts page
- [ ] Upgrade prompt is friendly and helpful
- [ ] Upgrade CTA works (even if just a link for MVP)

---

### Task 4.7 — Responsive QA + Mobile Polish

**Time estimate:** 60–90 min
**References:** `05-UI-UX-DESIGN.md` (Responsive Behavior)
**Deliverable:** All views working smoothly on mobile, tablet, and desktop

**Claude Code Prompt:**
```
Read the Responsive Behavior section of 05-UI-UX-DESIGN.md.

QA and polish the responsive experience. Interview me about:
- Which views feel most broken on mobile right now?
- Any specific mobile interactions that feel clunky?
- Is there a priority order for mobile polish?

Then:
1. Test every page at mobile (<768px), tablet (768-1024px), and desktop (>1024px)
2. Fix Kanban: single column view with swipe on mobile
3. Fix contact detail: stacked layout on mobile
4. Fix calendar: month-by-month swipe on mobile
5. Fix navigation: bottom tabs on mobile, sidebar on desktop
6. Fix modals: full-screen on mobile
7. Fix touch targets: minimum 44x44px tap targets
8. Test and fix any overflow, scrolling, or layout issues
```

**Acceptance criteria:**
- [ ] Kanban works on mobile (swipe between columns)
- [ ] All forms usable on mobile
- [ ] Navigation smooth on all sizes
- [ ] No horizontal overflow anywhere
- [ ] All tap targets large enough
- [ ] Modals are mobile-friendly

---

### Task 4.8 — Final QA, Empty States & Error Handling

**Time estimate:** 60–90 min
**References:** `05-UI-UX-DESIGN.md` (Empty States, Error States)
**Deliverable:** All empty states implemented, error handling polished, app ready for testing

**Claude Code Prompt:**
```
Read the Empty States and Error States sections of 05-UI-UX-DESIGN.md.

Final QA pass on the entire app. Interview me about:
- Any known bugs or rough edges?
- Are there flows I haven't tested yet?
- Any last-minute feature adjustments?

Then:
1. Implement all empty states:
   - Kanban: no contacts
   - Contact list: no contacts
   - Touch history: no touches
   - Insights: insufficient data
   - Calendar: no plan
   - Referrals: no referrals
2. Error handling:
   - Network errors: toast with retry
   - Session expired: redirect to login
   - Form validation errors: inline messages
3. Loading states: skeletons for all data-fetching views
4. Test the complete flow: signup → onboarding → add contacts → log touches → view insights
5. Check all database operations are correct
6. Verify RLS is working (can't access other users' data)
```

**Acceptance criteria:**
- [ ] All empty states have helpful guidance + CTAs
- [ ] All error states handled gracefully
- [ ] Loading skeletons on all async views
- [ ] Complete user flow works end-to-end
- [ ] RLS verified
- [ ] App ready for real agent testing

---

## Daily Workflow

1. Open this playbook
2. Find your current sprint and next task
3. Open Claude Code
4. Paste the Claude Code prompt
5. Answer Claude Code's interview questions
6. Build together
7. Test the deliverable against acceptance criteria
8. Commit to GitHub
9. Verify Vercel deployment
10. Check off the task
11. Repeat tomorrow
