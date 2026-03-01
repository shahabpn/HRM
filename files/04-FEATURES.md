# HRM Feature Specifications

## MVP Scope (Weeks 1–8)

### Priority Definitions
- **P0** — Must have. App doesn't function without it.
- **P1** — Should have. Core value proposition.
- **P2** — Nice to have. Can ship without.

---

## F1: Authentication (P0)

**Method:** Supabase Auth, email/password only.

**Flows:**
- Signup: email + password + full name → create auth user → create agent_profile → redirect to onboarding
- Login: email + password → redirect to /dashboard/today
- Logout: clear session → redirect to /login
- Password reset: email-based reset flow (Supabase built-in)

**Rules:**
- All /dashboard/* routes are protected
- Unauthenticated users redirect to /login
- New users without completed onboarding redirect to /onboarding

---

## F2: Onboarding Flow (P1)

**4 steps, sequential, must complete all to access dashboard.**

### Step 1: Referral Worthiness Assessment
- Current rating: 1–10 slider
- Runway question: select one of (1 day, 1 week, 1 month, 1 year, 5 years)
- Goal rating: 1–10 slider
- Goal timeline: select one of (3 months, 6 months, 12 months)
- Saves to agent_profiles

### Step 2: Add First 5 Contacts (Memory Jogger)
- Display Memory Jogger categories as inspiration prompts (Accountant, Attorney, Barber, Close Friends, Coworkers, Dentist, Family Members, etc.)
- Quick-add form per contact: first_name (required), last_name, phone, email
- Must add minimum 5 contacts to proceed
- Counter shows progress: "3 of 5 contacts added"

### Step 3: Touch Calendar Wizard
- Question 1: "How many times per year do you want to connect with your best contacts?" — slider 24–36, default 30
- Question 2: "Which channels do you use most?" — multi-select: Phone, Email, Mail, Social Media, Events, Gifts
- Question 3: "Do you host client events?" — yes/no + if yes, how many per year (1–6)
- Generate cadence_rules and touch_plan_items based on answers
- Show preview of generated 12-month calendar
- Agent can accept or adjust

### Step 4: Complete
- Show Kanban board preview with their 5 contacts in "No Plan" column
- CTA: "Start building relationships" → redirect to /dashboard/today
- Set onboarding_completed_at on agent_profile

---

## F3: Contact Management (P0)

### Create Contact
- Minimum required: first_name
- Optional at creation: last_name, email, phone, primary_segment, tags, preferred_channel, notes
- Auto-creates empty contact_mackay_profiles row
- Enforces contact_limit for free tier (50). Show upgrade prompt when at limit.

### Contact List View (/dashboard/contacts)
- Table/grid view of all contacts
- Search: by name, email, phone
- Filter by: segment, tier, tags, has_touch_plan (yes/no)
- Sort by: name (A-Z), last_touch_date (oldest first), relationship_health_score, created_at
- Show per contact: name, segment badge, tier indicator, health score, last touch date, days since last touch

### Contact Detail Page (/dashboard/contacts/[id])
- **Header:** Name, avatar/initials, segment badge, tier badge, health score (large), profile completeness ring, quick action buttons (Log Touch, Edit, Archive)
- **Tab 1 — Overview:** Key info summary, recent touches timeline (last 5), next touch due, referrals given, notes
- **Tab 2 — Profile (Mackay 66):** All categories as expandable accordion sections, badge indicator per category, field count per category (e.g., "Family: 3/6"), inline editing
- **Tab 3 — Touch History:** Full touch log, filterable by type/date
- **Tab 4 — Referrals:** Referrals this contact has given, status of each

### Edit Contact
- Inline editing on detail page
- All fields editable
- Segment and tag management
- Manual tier override (sets tier_override = TRUE)

### Archive Contact
- Soft delete: sets archived_at timestamp
- Archived contacts hidden from all views by default
- Recoverable

### CSV Import (P1)
1. Upload CSV file
2. Column mapping UI: agent maps CSV headers to HRM fields
3. Preview first 10 rows with mapped data
4. Duplicate detection on email (per agent)
5. Import results: X imported, Y duplicates skipped, Z errors
6. Minimum mapping: first_name (required)
7. Optional mappings: last_name, email, phone, address fields, notes

---

## F4: Mackay 66 Profile (P0)

**All 66 fields visible from day one.** Organized into 7 expandable categories on contact detail page.

### Categories and Field Counts

| Category | Fields | Weight (for scoring) |
|---|---|---|
| Personal Info | 5 | 15% |
| Education | 9 | 10% |
| Family | 6 | 20% |
| Military | 3 | 5% |
| Business Background | ~15 | 15% |
| Special Interests & Lifestyle | ~16 | 20% |
| Customer & You | ~14 | 15% |

### Interaction Pattern
- Expandable accordion sections
- Inline editing (click field to edit, blur to save)
- Empty fields show placeholder text encouraging completion
- Category completion indicator (e.g., "3/6 fields")
- Saving a field updates fields_completed count and recalculates profile_completeness_score

---

## F5: Kanban Board — Today View (P0)

**The home screen. The daily action interface.**

### Columns

| Column | Logic | Visual |
|---|---|---|
| Overdue | next_touch_due < today | Red/warm accent |
| Due Today | next_touch_due = today | Orange/amber |
| Due This Week | next_touch_due BETWEEN tomorrow AND end_of_week | Blue |
| Up to Date | next_touch_due > end_of_week AND has plan | Green |
| No Plan | next_touch_due IS NULL | Gray |

### Contact Card (on Kanban)
- Name + avatar/initials
- Segment badge (small colored tag)
- Tier indicator (★ / ★★★ / ★★★★★)
- Relationship health score (mini progress ring)
- Last touch: "3 days ago — Phone Call" or "Never"
- Days since last touch (prominent if overdue)
- Next suggested touch type (from calendar plan)

### Interactions
- **Drag and drop** between columns → updates next_touch_due accordingly
- **Click card** → opens quick touch logging modal
- **Click name** → navigates to contact detail page
- **Filter** by: segment, tier, tags
- **Sort within column** by: days since last touch, name, health score

### Summary Bar (top of board)
- Total contacts per column (Overdue: 12 | Due Today: 5 | ...)
- Daily progress: "You've touched 3 contacts today" with goal indicator (3/5)
- Motivational nudge if overdue count is high

---

## F6: Touch Logging (P0)

### MVP Flow (3 taps)
1. Select contact (or pre-selected from Kanban card click)
2. Select touch type (dropdown of system defaults + agent custom types)
3. Touch date (defaults to today, can backdate)
4. Submit

### On Log:
- Insert contact_touches record
- Update contacts.last_touch_date
- Increment contacts.total_touches_ytd
- Recalculate touch_consistency_score
- Recalculate relationship_health_score
- Evaluate tier suggestion change
- Evaluate badge criteria
- Recalculate next_touch_due
- Contact card moves to appropriate Kanban column

### Touch Types
- 18 system defaults (see DATABASE-SCHEMA.md)
- Agent can add custom types (Settings page)
- Agent can deactivate system defaults they don't use

---

## F7: Relationship Health Score (P0)

### Composite Score (0–100)

```
relationship_health = (profile_depth × 0.35) + (touch_consistency × 0.65)
```

### Profile Depth Score (0–100)
Weighted by Mackay category:
- Family: 20%
- Special Interests & Lifestyle: 20%
- Customer & You: 15%
- Business Background: 15%
- Personal Info: 15%
- Education: 10%
- Military: 5%

### Touch Consistency Score (0–100)
```
annual_target = cadence_rules[segment][tier].annual_target
expected_ytd = annual_target × (day_of_year / 365)
actual_ytd = count of touches this year
ratio = actual_ytd / expected_ytd
base_score = min(ratio × 100, 100)

Recency bonus: +10 if touched within 14 days, +5 if within 30 days
Recency penalty: -10 if last touch > 60 days, -20 if > 90 days
```

### Tier Suggestion
| Tier | Criteria |
|---|---|
| Advocacy | health ≥ 75 AND 1+ referral in last 12 months |
| Loyal | health ≥ 50 OR (health ≥ 40 AND has ever referred) |
| Satisfied | Default / health < 50 |

Agent can override. Override flag prevents system from changing it.

---

## F8: Badge System (P1)

### Badge Definitions

| Badge | Icon | Criteria |
|---|---|---|
| Knows Their Family | 🏠 | Family section ≥ 80% |
| Business Insider | 💼 | Business Background ≥ 80% |
| Fully Profiled | 🎯 | Mackay profile ≥ 75% overall |
| Hot Streak | 🔥 | Touched consistently 3+ months |
| Advocate | ⭐ | Tier = Advocacy |
| Referral Champion | 🎁 | 3+ referrals given |
| Never Missed | 📅 | All planned touches completed 6+ months |
| First Five | 🎉 | Completed onboarding first 5 contacts |

### Display
- Badges shown on contact cards (small icons)
- Full badge list on contact detail page
- New badge earned → toast notification

---

## F9: Annual Touch Calendar (P1)

### Calendar View
- 12-month grid
- Rows: months (Jan–Dec)
- Columns: touch categories (Mail, Email, Personal, Social, Events, Gifts, CMA)
- Cells: planned touch items (clickable to edit)
- Color coding: Planned (gray), Completed (green), Missed (red)

### Calendar Wizard (Onboarding Step 3)
Generates cadence_rules and touch_plan_items based on:
- Annual touch target
- Preferred channels
- Event frequency
- Segment × tier cadence matrix

### Calendar → Kanban Integration
Touch plan items generate next_touch_due dates per contact:
- When a plan item's month begins, contacts matching that item's segment/tier filters get updated next_touch_due dates
- Completing a touch for a contact recalculates their next_touch_due to the next planned item

---

## F10: Insights Dashboard (P1)

### Primary Metrics (always visible, top cards)
- **Database Size** — total active contacts + trend
- **Database Return Rate** — transactions / contacts (with target line at 15%)
- **Touch Consistency** — % of contacts on track
- **Overdue Contacts** — count + % of database

### Charts
- **Tier Distribution** — donut: Satisfied vs Loyal vs Advocacy
- **Monthly Touch Activity** — bar chart over last 12 months
- **Referrals This Year** — count + trend

### Lists
- **Most Neglected** — top 10 contacts by days since last touch
- **Most Engaged** — top 10 by touch count this year
- **Referral Worthiness Progress** — current score vs goal (from onboarding)

---

## F11: Referral & Transaction Logging (P1)

### Log Referral
- Select referrer contact
- Enter referred person's name
- Optional: link to existing contact if already in database
- Status: received → contacted → converted → lost
- Flag: became_transaction (boolean)

### Log Transaction
- Select client contact (optional)
- Link to referral (optional)
- Transaction date
- Type: buy / sell / both
- Source: database_referral / database_repeat / external / other

---

## F12: Email Digest (P2)

### Weekly (all tiers, MVP)
- Sent Monday 7:00 AM agent's timezone
- Content: week summary, coming week contacts due, special dates, database health

### Daily (paid tier, post-MVP)
- Sent 7:00 AM
- Content: contacts due today, overdue count, yesterday's stats

---

## F13: Settings (P0)

### Profile
- Name, email, phone, brokerage, city, timezone

### Preferences
- Daily touch goal (default 5)
- Annual touch target (default 36)
- Preferred channels

### Custom Touch Types
- Add/edit/deactivate custom types

### Subscription
- Current tier display
- Contact count / limit
- Upgrade CTA (manual for MVP)

### Data
- Export contacts (CSV)
- Delete account
