# HRM — Technical Specification

**Version:** 1.0
**Date:** February 19, 2026
**Authors:** Shahab & Braden
**Status:** Pre-Development / Documentation Phase
**Target MVP Timeline:** 6–8 weeks

---

## 1. Technical Architecture Overview

### Stack

| Layer | Technology | Notes |
|---|---|---|
| Frontend | Next.js 14+ (App Router) | React + SSR + API routes |
| UI Components | Chakra UI | Opinionated, accessible component library |
| Database | Supabase (hosted cloud) | PostgreSQL + Auth + Realtime + Edge Functions |
| Hosting | Vercel | Auto-deploy from GitHub |
| Version Control | GitHub | Monorepo |
| Development | Claude Code | AI-assisted development |
| Auth | Supabase Auth (email/password) | Google OAuth can be added post-MVP |
| Email (digest) | Supabase Edge Functions + Resend or similar | Cron-triggered daily/weekly digest |

### Architecture Pattern

```
┌─────────────────────────────────────────────┐
│                   Vercel                      │
│  ┌─────────────────────────────────────────┐ │
│  │         Next.js App Router               │ │
│  │  ┌──────────┐  ┌──────────────────────┐ │ │
│  │  │  Pages/   │  │  API Routes          │ │ │
│  │  │  Components│  │  /api/digest-cron   │ │ │
│  │  │  (SSR+CSR)│  │  /api/csv-import    │ │ │
│  │  └──────────┘  └──────────────────────┘ │ │
│  └─────────────────────────────────────────┘ │
└──────────────────┬──────────────────────────┘
                   │
          Supabase Client SDK
                   │
┌──────────────────▼──────────────────────────┐
│              Supabase Cloud                   │
│  ┌────────────┐  ┌────────┐  ┌────────────┐ │
│  │ PostgreSQL  │  │  Auth  │  │   Edge     │ │
│  │ + RLS       │  │        │  │ Functions  │ │
│  └────────────┘  └────────┘  └────────────┘ │
└─────────────────────────────────────────────┘
```

### Key Architectural Decisions

1. **Next.js App Router** — SSR for initial page loads (SEO for landing/marketing pages), client-side interactivity for the Kanban and dashboard. API routes handle server-side operations like CSV parsing, email digest cron triggers, and any logic that shouldn't run client-side.

2. **Supabase RLS (Row Level Security)** — All data access controlled at the database level. Every table has RLS policies ensuring agents can only access their own data. This is critical for multi-tenancy even in the single-agent MVP.

3. **No custom backend server** — Supabase handles auth, database, and realtime. Next.js API routes handle the few server-side operations needed. No Express, no separate API.

4. **Schema designed for future teams** — `organization_id` and `team_id` columns exist in the schema from day one but are nullable and unused in MVP. This prevents painful migrations later.

---

## 2. Database Schema (Supabase / PostgreSQL)

### Entity Relationship Overview

```
users (Supabase Auth)
  │
  ├── agent_profiles (1:1)
  │     ├── onboarding_assessment
  │     └── settings/preferences
  │
  ├── contacts (1:many)
  │     ├── contact_mackay_profile (1:1, JSONB)
  │     ├── contact_segments (primary segment + tags)
  │     ├── contact_touches (1:many)
  │     └── contact_referrals (1:many)
  │
  ├── touch_plans (1:many)
  │     └── touch_plan_items (1:many)
  │
  ├── touch_types (1:many, includes defaults + custom)
  │
  └── transactions (1:many)
```

### Core Tables

#### `agent_profiles`

Extends Supabase auth.users with HRM-specific data.

```sql
CREATE TABLE agent_profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  
  -- Basic info
  full_name TEXT NOT NULL,
  email TEXT NOT NULL,
  phone TEXT,
  brokerage_name TEXT,
  city TEXT,
  province_state TEXT,
  
  -- Onboarding assessment (Referral Worthiness)
  referral_rating_current INTEGER CHECK (referral_rating_current BETWEEN 1 AND 10),
  referral_runway TEXT CHECK (referral_runway IN ('1_day', '1_week', '1_month', '1_year', '5_years')),
  referral_rating_goal INTEGER CHECK (referral_rating_goal BETWEEN 1 AND 10),
  referral_goal_timeline TEXT CHECK (referral_goal_timeline IN ('3_months', '6_months', '12_months')),
  
  -- Settings
  average_commission NUMERIC(10,2), -- for future ROI calculations
  daily_touch_goal INTEGER DEFAULT 5,
  annual_touch_target INTEGER DEFAULT 36,
  preferred_touch_channels TEXT[], -- e.g. {'email', 'phone', 'text'}
  
  -- Subscription
  tier TEXT DEFAULT 'free' CHECK (tier IN ('free', 'paid')),
  contact_limit INTEGER DEFAULT 50,
  
  -- Future multi-tenancy (nullable for MVP)
  organization_id UUID,
  team_id UUID,
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  onboarding_completed_at TIMESTAMPTZ
);
```

#### `contacts`

Core contact record. Minimal required fields + segment + relationship tier.

```sql
CREATE TABLE contacts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,
  
  -- Required fields (minimum to create a contact)
  first_name TEXT NOT NULL,
  last_name TEXT,
  
  -- Core contact info
  email TEXT,
  phone TEXT,
  phone_secondary TEXT,
  address_street TEXT,
  address_city TEXT,
  address_province_state TEXT,
  address_postal_code TEXT,
  
  -- Segmentation
  primary_segment TEXT NOT NULL DEFAULT 'buyer_seller' 
    CHECK (primary_segment IN ('buyer_seller', 'referring_agent', 'strategic_alliance')),
  tags TEXT[] DEFAULT '{}',
  
  -- Relationship tier (system-suggested, agent can override)
  relationship_tier TEXT DEFAULT 'satisfied' 
    CHECK (relationship_tier IN ('satisfied', 'loyal', 'advocacy')),
  tier_override BOOLEAN DEFAULT FALSE, -- TRUE if agent manually set the tier
  
  -- Communication preferences
  preferred_channel TEXT CHECK (preferred_channel IN ('email', 'phone', 'text', 'messenger', 'in_person', 'social')),
  communication_permission BOOLEAN DEFAULT FALSE,
  
  -- Calculated scores (updated by triggers/functions)
  profile_completeness_score NUMERIC(5,2) DEFAULT 0, -- 0-100 weighted
  touch_consistency_score NUMERIC(5,2) DEFAULT 0, -- 0-100
  relationship_health_score NUMERIC(5,2) DEFAULT 0, -- composite of above two
  
  -- Touch tracking
  last_touch_date DATE,
  next_touch_due DATE,
  total_touches_ytd INTEGER DEFAULT 0,
  touch_target_annual INTEGER, -- NULL = use agent default, overridable per contact
  
  -- Referral tracking
  total_referrals_given INTEGER DEFAULT 0,
  last_referral_date DATE,
  
  -- Badges (JSONB for flexibility)
  badges_earned JSONB DEFAULT '[]',
  
  -- Notes
  notes TEXT,
  
  -- Future multi-tenancy
  organization_id UUID,
  team_id UUID,
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  archived_at TIMESTAMPTZ, -- soft delete / purge
  
  -- Constraints
  UNIQUE(agent_id, email) -- prevent duplicates per agent
);

-- Indexes for Kanban view performance
CREATE INDEX idx_contacts_agent_touch_status ON contacts(agent_id, next_touch_due);
CREATE INDEX idx_contacts_agent_segment ON contacts(agent_id, primary_segment);
CREATE INDEX idx_contacts_agent_tier ON contacts(agent_id, relationship_tier);
CREATE INDEX idx_contacts_tags ON contacts USING GIN(tags);
```

#### `contact_mackay_profiles`

The full Mackay 66 profile stored as structured JSONB. All fields visible in UI from day one, but only populated progressively.

```sql
CREATE TABLE contact_mackay_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID NOT NULL UNIQUE REFERENCES contacts(id) ON DELETE CASCADE,
  
  -- Mackay 66 categories as JSONB objects
  -- Each category is a JSON object with named fields
  -- This allows flexible querying while maintaining structure
  
  personal_info JSONB DEFAULT '{}',
  /*
    {
      "nickname": "",
      "birthdate": "",
      "height": "",
      "weight": "",
      "hometown": ""
    }
  */
  
  education JSONB DEFAULT '{}',
  /*
    {
      "high_school": "",
      "high_school_year": "",
      "college": "",
      "college_year": "",
      "degrees": "",
      "college_honors": "",
      "fraternity_sorority": "",
      "extracurricular_activities": "",
      "sensitive_about_no_college": null  // boolean
    }
  */
  
  family JSONB DEFAULT '{}',
  /*
    {
      "spouse_name": "",
      "spouse_education": "",
      "spouse_interests": "",
      "anniversary": "",
      "children": [
        { "name": "", "age": "", "education": "", "interests": "" }
      ]
    }
  */
  
  military JSONB DEFAULT '{}',
  /*
    {
      "service": "",
      "discharge_rank": "",
      "attitude_toward_service": ""
    }
  */
  
  business_background JSONB DEFAULT '{}',
  /*
    {
      "current_occupation": "",
      "previous_employers": [
        { "company": "", "location": "", "title": "", "dates": "" }
      ],
      "previous_positions_current_company": [],
      "status_symbols_in_office": "",
      "professional_associations": "",
      "offices_held_honors": "",
      "long_range_business_objective": "",
      "immediate_business_objective": "",
      "greatest_concern": "", // "company_welfare" | "personal_welfare"
      "time_orientation": "" // "present" | "future"
    }
  */
  
  special_interests JSONB DEFAULT '{}',
  /*
    {
      "clubs_associations": "",
      "politically_active": null, // boolean
      "political_party": "",
      "political_importance": "",
      "community_involvement": "",
      "religion": "",
      "confidential_sensitive_items": "",
      "strong_feelings_topics": "",
      "hobbies_recreational": "",
      "vacation_habits": ""
    }
  */
  
  lifestyle JSONB DEFAULT '{}',
  /*
    {
      "medical_current_health": "",
      "drinks_alcohol": null, // boolean
      "alcohol_details": "",
      "offended_by_drinking": null,
      "smokes": null, // boolean
      "objects_to_smoking": null,
      "favorite_lunch_spots": "",
      "favorite_dinner_spots": "",
      "favorite_menu_items": "",
      "objects_to_meal_being_bought": null
    }
  */
  
  customer_and_you JSONB DEFAULT '{}',
  /*
    {
      "moral_ethical_considerations": "",
      "feels_obligation_to_you": "",
      "obligation_details": "",
      "requires_habit_change": "",
      "concerned_about_others_opinions": null, // boolean
      "self_centered": null,
      "highly_ethical": null,
      "key_problems_their_view": "",
      "management_priorities": "",
      "can_you_help": "",
      "competitor_advantages": "",
      "spectator_sports_teams": "",
      "cars": "",
      "conversational_interests": "",
      "who_they_want_to_impress": "",
      "how_they_want_to_be_seen": "",
      "adjectives_to_describe": "",
      "proudest_achievement": "",
      "long_range_personal_objective": "",
      "immediate_personal_goal": ""
    }
  */
  
  -- Metadata
  fields_completed INTEGER DEFAULT 0, -- count of non-empty fields
  total_fields INTEGER DEFAULT 66,
  last_updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### `touch_types`

Default + custom touch types per agent.

```sql
CREATE TABLE touch_types (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID REFERENCES agent_profiles(id) ON DELETE CASCADE, -- NULL = system default
  
  name TEXT NOT NULL,
  category TEXT NOT NULL CHECK (category IN (
    'mail', 'email', 'personal', 'social_media', 'event', 'gift', 'cma', 'other'
  )),
  description TEXT,
  icon TEXT, -- icon name for UI
  is_system_default BOOLEAN DEFAULT FALSE,
  is_active BOOLEAN DEFAULT TRUE,
  
  -- Suggested frequency
  suggested_annual_frequency INTEGER,
  
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Seed system defaults
INSERT INTO touch_types (name, category, is_system_default, suggested_annual_frequency, icon) VALUES
  ('Phone Call', 'personal', TRUE, 4, 'phone'),
  ('In-Person Meeting', 'personal', TRUE, 4, 'users'),
  ('Personal Lunch/Coffee', 'personal', TRUE, 4, 'coffee'),
  ('Home Visit', 'personal', TRUE, 4, 'home'),
  ('Email - Market Update', 'email', TRUE, 12, 'mail'),
  ('Email - Newsletter', 'email', TRUE, 8, 'newspaper'),
  ('Email - Hot New Listing', 'email', TRUE, 4, 'trending-up'),
  ('Personalized Mail', 'mail', TRUE, 8, 'send'),
  ('Birthday Card/Gift', 'gift', TRUE, 1, 'gift'),
  ('Holiday Card/Gift', 'gift', TRUE, 2, 'heart'),
  ('Special Occasion Card', 'gift', TRUE, 2, 'star'),
  ('Unexpected Gift/Surprise', 'gift', TRUE, 1, 'sparkles'),
  ('CMA / Real Estate Health Check', 'cma', TRUE, 2, 'bar-chart'),
  ('Social Media Engagement', 'social_media', TRUE, 10, 'thumbs-up'),
  ('Client Event (Micro)', 'event', TRUE, 2, 'calendar'),
  ('Client Event (Macro)', 'event', TRUE, 2, 'users'),
  ('Trades & Services Directory', 'mail', TRUE, 1, 'book'),
  ('Post-Sale Follow-Up', 'personal', TRUE, 4, 'check-circle');
```

#### `contact_touches`

Log of every touchpoint between agent and contact.

```sql
CREATE TABLE contact_touches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID NOT NULL REFERENCES contacts(id) ON DELETE CASCADE,
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,
  touch_type_id UUID NOT NULL REFERENCES touch_types(id),
  
  -- MVP: minimal logging (just type + timestamp)
  touch_date DATE NOT NULL DEFAULT CURRENT_DATE,
  
  -- V2: structured logging fields (nullable for MVP)
  channel TEXT, -- how the touch was delivered
  outcome TEXT, -- what happened
  next_action TEXT, -- follow-up planned
  note TEXT,
  
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_touches_contact ON contact_touches(contact_id, touch_date DESC);
CREATE INDEX idx_touches_agent_date ON contact_touches(agent_id, touch_date DESC);
```

#### `contact_referrals`

Simple referral logging.

```sql
CREATE TABLE contact_referrals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,
  referrer_contact_id UUID NOT NULL REFERENCES contacts(id) ON DELETE CASCADE,
  
  -- Who was referred
  referred_name TEXT NOT NULL,
  referred_contact_id UUID REFERENCES contacts(id), -- NULL if not yet added as contact
  
  -- Outcome
  referral_date DATE NOT NULL DEFAULT CURRENT_DATE,
  status TEXT DEFAULT 'received' CHECK (status IN ('received', 'contacted', 'converted', 'lost')),
  became_transaction BOOLEAN DEFAULT FALSE,
  
  note TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### `transactions`

Minimal transaction tracking for ROI calculation.

```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,
  contact_id UUID REFERENCES contacts(id), -- the client
  referral_id UUID REFERENCES contact_referrals(id), -- if from referral
  
  transaction_date DATE NOT NULL,
  transaction_type TEXT CHECK (transaction_type IN ('buy', 'sell', 'both')),
  source TEXT CHECK (source IN ('database_referral', 'database_repeat', 'external', 'other')),
  
  note TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### `touch_plans`

Agent's annual touch calendar configuration.

```sql
CREATE TABLE touch_plans (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,
  
  name TEXT NOT NULL DEFAULT 'My Touch Plan',
  year INTEGER NOT NULL DEFAULT EXTRACT(YEAR FROM NOW()),
  is_active BOOLEAN DEFAULT TRUE,
  
  -- Cadence rules (generated by wizard)
  -- Different cadences per segment + tier combination
  cadence_rules JSONB DEFAULT '{}',
  /*
    {
      "buyer_seller": {
        "advocacy": { "annual_target": 36, "channels": ["personal", "email", "mail", "social_media", "event"] },
        "loyal": { "annual_target": 30, "channels": ["email", "personal", "mail", "social_media"] },
        "satisfied": { "annual_target": 24, "channels": ["email", "mail", "social_media"] }
      },
      "referring_agent": {
        "advocacy": { "annual_target": 24, "channels": ["personal", "email", "event"] },
        ...
      },
      "strategic_alliance": {
        ...
      }
    }
  */
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Specific planned touch items on the calendar
CREATE TABLE touch_plan_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  touch_plan_id UUID NOT NULL REFERENCES touch_plans(id) ON DELETE CASCADE,
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,
  
  -- What
  touch_type_id UUID NOT NULL REFERENCES touch_types(id),
  title TEXT NOT NULL, -- e.g. "Mother's Day Card", "Q2 Market Update Video"
  description TEXT,
  
  -- When
  month INTEGER NOT NULL CHECK (month BETWEEN 1 AND 12),
  day_of_month INTEGER, -- optional specific day
  
  -- Who (which segments/tiers this applies to)
  applies_to_segments TEXT[] DEFAULT '{buyer_seller,referring_agent,strategic_alliance}',
  applies_to_tiers TEXT[] DEFAULT '{satisfied,loyal,advocacy}',
  
  -- Recurrence
  is_recurring BOOLEAN DEFAULT TRUE, -- repeats annually
  
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### `badges`

Badge definitions for gamification.

```sql
CREATE TABLE badge_definitions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  name TEXT NOT NULL,
  description TEXT NOT NULL,
  icon TEXT NOT NULL,
  category TEXT NOT NULL, -- 'profile', 'touch', 'referral', 'milestone'
  
  -- Unlock criteria (evaluated by application logic)
  criteria_type TEXT NOT NULL,
  criteria_config JSONB NOT NULL,
  /*
    Examples:
    { "type": "mackay_category_complete", "category": "family" }
    { "type": "touch_streak", "days": 30 }
    { "type": "referrals_received", "count": 5 }
    { "type": "profile_completeness", "threshold": 50 }
    { "type": "contacts_at_advocacy", "count": 10 }
  */
  
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Row Level Security (RLS) Policies

Every table must have RLS enabled. Pattern for all agent-owned tables:

```sql
-- Enable RLS
ALTER TABLE contacts ENABLE ROW LEVEL SECURITY;

-- Agents can only see their own data
CREATE POLICY "agents_own_data" ON contacts
  FOR ALL
  USING (agent_id = auth.uid())
  WITH CHECK (agent_id = auth.uid());
```

Apply this pattern to: `contacts`, `contact_mackay_profiles`, `contact_touches`, `contact_referrals`, `transactions`, `touch_plans`, `touch_plan_items`, and agent-specific `touch_types`.

---

## 3. Application Structure

### Route Map (Next.js App Router)

```
/                           → Landing page (marketing, public)
/login                      → Email/password auth
/signup                     → Registration
/onboarding                 → Multi-step onboarding flow
  /onboarding/assessment    → Referral Worthiness self-assessment
  /onboarding/first-five    → Add 5 contacts with Memory Jogger
  /onboarding/calendar      → Guided touch calendar wizard
  /onboarding/complete      → Success + redirect to dashboard

/dashboard                  → Main app (protected)
  /dashboard/today          → Kanban board (default/home)
  /dashboard/contacts       → All contacts list/grid view
  /dashboard/contacts/[id]  → Contact detail + Mackay 66 profile
  /dashboard/contacts/import → CSV import flow
  /dashboard/calendar       → Annual touch calendar view
  /dashboard/insights       → ROI dashboard + metrics
  /dashboard/settings       → Profile, preferences, touch types, subscription
```

### Navigation (Tab-Based)

Primary navigation as bottom tabs (mobile) / sidebar tabs (desktop):

| Tab | Icon | View |
|---|---|---|
| **Today** | ☀️ | Kanban board — the daily action view |
| **Contacts** | 👥 | Full contact list with search, filter, sort |
| **Calendar** | 📅 | Annual touch calendar with monthly detail |
| **Insights** | 📊 | ROI dashboard, metrics, progress tracking |

---

## 4. Feature Specifications

### 4.1 Onboarding Flow

**Step 1: Account Creation**
- Email + password signup
- Basic profile: name, brokerage, city

**Step 2: Referral Worthiness Assessment**
- Current rating (1–10 slider)
- Runway question (1 day / 1 week / 1 month / 1 year / 5 years)
- Goal rating (1–10 slider)
- Goal timeline (3 / 6 / 12 months)

**Step 3: Add Your First 5 Contacts (Memory Jogger)**
- Show the Memory Jogger category list (Accountant, Attorney, Barber, Close Friends, etc.)
- Simple form: first name, last name, phone, email (minimum)
- Encourage: "Think of 5 people you know by first name who you haven't talked to in a while"
- Quick-add UI — no Mackay fields, just the essentials to get started

**Step 4: Touch Calendar Wizard**
- "How many times per year do you want to connect with your best contacts?" (slider: 24–36)
- "Which channels do you use most?" (multi-select: phone, email, mail, social, events, gifts)
- "Do you host client events?" (yes/no, how many per year)
- System generates a recommended annual calendar based on answers
- Agent can review and adjust before saving

**Step 5: You're Ready**
- Show their Kanban board with 5 contacts in "No Plan" column
- Prompt: "Move a contact to get started — or add more from your database"

### 4.2 Kanban Board (Today View / Home Screen)

The primary daily interface. Cards represent contacts. Columns represent touch urgency.

**Columns:**

| Column | Logic | Color |
|---|---|---|
| **Overdue** | `next_touch_due < today` | Red/warm accent |
| **Due Today** | `next_touch_due = today` | Orange/amber |
| **Due This Week** | `next_touch_due BETWEEN tomorrow AND end_of_week` | Blue |
| **Up to Date** | `next_touch_due > end_of_week` AND has a plan | Green |
| **No Plan** | `next_touch_due IS NULL` OR no touch plan assigned | Gray |

**Contact Card (on Kanban):**
- Name + avatar/initials
- Primary segment badge (buyer/seller, agent, alliance)
- Relationship tier indicator (★ / ★★★ / ★★★★★)
- Relationship health score (small ring/progress indicator)
- Last touch date + type
- Days since last touch
- Next suggested touch type (from calendar plan)

**Interactions:**
- Drag contact between columns (updates `next_touch_due`)
- Click card → opens quick-touch logging modal
- Click name → navigates to full contact detail
- Filter by: segment, tier, tags
- Sort by: days since last touch, name, health score

**Kanban Summary Bar (top of board):**
- Total contacts per column
- "You've touched X contacts today" counter
- Daily goal progress (e.g., "3/5 today")

### 4.3 Contact Detail Page

Full profile view for a single contact.

**Header:**
- Name, photo/avatar, segment badge, tier badge
- Relationship health score (large, prominent)
- Profile completeness % (with visual progress ring)
- Quick actions: Log Touch, Edit, Archive

**Tabs within contact detail:**
1. **Overview** — Key info, recent touches timeline, next touch due, referrals given
2. **Profile (Mackay 66)** — All 66 fields organized by category, expandable sections, badge indicators per category showing completion
3. **Touch History** — Complete log of all touches, filterable by type/channel
4. **Referrals** — Referrals this contact has given, status of each

**Mackay 66 Profile Categories (expandable sections):**
Each section shows a category badge (locked/unlocked) and field count.

- Personal Info (5 fields)
- Education (9 fields)
- Family (6 fields)
- Military (3 fields)
- Business Background (~15 fields)
- Special Interests & Lifestyle (~16 fields)
- The Customer & You (~14 fields)

**Badge System:**
Badges appear on the contact card and detail page. Examples:

| Badge | Criteria |
|---|---|
| 🏠 Knows Their Family | Family section ≥ 80% complete |
| 💼 Business Insider | Business Background ≥ 80% complete |
| 🎯 Fully Profiled | Overall Mackay profile ≥ 75% complete |
| 🔥 Hot Streak | Touched consistently for 3+ months |
| ⭐ Advocate | Relationship tier = Advocacy |
| 🎁 Referral Champion | 3+ referrals given |
| 📅 Never Missed | All planned touches completed for 6+ months |

### 4.4 Touch Logging

**MVP (Minimal friction):**
1. Select contact (or trigger from Kanban card)
2. Select touch type (from fixed list + custom)
3. Date (defaults to today)
4. Done — logs the touch, updates `last_touch_date`, recalculates `next_touch_due`

**V2 (Structured):**
Add optional fields:
- Channel used
- Outcome / what was discussed
- Next action planned
- Free-text note

### 4.5 Annual Touch Calendar

**Calendar Wizard Output:**
Generates a 12-month grid showing planned touch types per month, personalized based on the agent's preferences and the cadence rules per segment/tier.

**Calendar View:**
- Month-by-month view (similar to Keith Roy's calendar)
- Columns: Touch type categories (Mail, Email, Personal, Social, Events)
- Rows: Months (January–December)
- Cells: Specific planned touches (e.g., "Mother's Day Card", "Q2 Market Update")
- Color coding by completion status: planned / completed / missed

**Calendar → Kanban Integration:**
The calendar generates `next_touch_due` dates for contacts, which feed the Kanban columns. When a planned touch month arrives, contacts who are due for that touch type appear in the appropriate Kanban column.

### 4.6 CSV Import

**MVP Import Flow:**
1. Upload CSV file
2. Column mapping UI — agent maps their CSV columns to HRM fields
3. Preview first 10 rows with mapped data
4. Import with duplicate detection (match on email)
5. Show results: X imported, Y duplicates skipped, Z errors

**Minimum mapped fields:** first_name (required), last_name, email, phone

**Optional mapped fields:** address fields, segment, tags, notes

### 4.7 Insights Dashboard

**Key Metrics (always visible):**

- **Database Size:** Total active contacts (with trend arrow)
- **Database Return Rate:** Transactions from database ÷ total contacts (with target line)
- **Touch Consistency:** % of contacts on track with their touch plan
- **Overdue Contacts:** Count + % of database that's overdue
- **Referrals This Year:** Total referrals received
- **Relationship Tier Distribution:** Pie/donut chart — Satisfied vs Loyal vs Advocacy

**Secondary Metrics:**
- Average relationship health score across database
- Touches logged this week/month/quarter
- Top 10 most-touched contacts (are you over-indexing on a few?)
- Top 10 most-neglected contacts (who are you forgetting?)
- Referral Worthiness Score progress (from onboarding assessment)
- Monthly touch activity chart (line/bar over time)

### 4.8 Email Digest (Notifications)

**Daily Digest (paid tier):**
Sent each morning. Contains:
- Contacts due today (names + suggested touch type)
- Overdue count
- Yesterday's touch count vs goal

**Weekly Digest (all tiers):**
Sent Monday morning. Contains:
- Week summary: touches completed, contacts reached
- Coming week: who's due, any special dates (birthdays, anniversaries)
- Database health snapshot

**Implementation:**
- Vercel Cron Job → calls Next.js API route → queries Supabase → sends via email provider (Resend)
- Cron schedule: daily at 7:00 AM agent's local time (store timezone in profile)

---

## 5. Relationship Health Score Algorithm

The relationship health score is the composite metric displayed on every contact card. It combines two dimensions:

### Profile Depth Score (0–100)

Weighted calculation based on Mackay 66 category completion:

| Category | Weight | Rationale |
|---|---|---|
| Family | 20% | High-value relationship knowledge |
| Special Interests & Lifestyle | 20% | Enables personalized touches |
| Customer & You | 15% | Competitive advantage knowledge |
| Business Background | 15% | Professional context |
| Personal Info | 15% | Basic knowledge foundation |
| Education | 10% | Background context |
| Military | 5% | Niche relevance |

Formula: `profile_depth = Σ (category_completion_% × category_weight)`

### Touch Consistency Score (0–100)

Based on how well the agent is hitting their touch targets for this contact:

```
annual_target = cadence_rules[contact.segment][contact.tier].annual_target
expected_touches_ytd = annual_target × (days_elapsed_in_year / 365)
actual_touches_ytd = COUNT(touches WHERE year = current_year)

touch_ratio = actual_touches_ytd / expected_touches_ytd
touch_consistency = MIN(touch_ratio × 100, 100)  -- cap at 100

// Recency bonus: +10 if touched within last 14 days, +5 if within 30 days
// Recency penalty: -10 if last touch > 60 days, -20 if > 90 days
```

### Composite Health Score

```
relationship_health = (profile_depth × 0.35) + (touch_consistency × 0.65)
```

Touch consistency is weighted higher because the primary value proposition is staying in touch — profile depth enhances the quality of touches.

### Tier Suggestion Logic

Based on health score + referral history:

| Suggested Tier | Criteria |
|---|---|
| **Advocacy** | Health score ≥ 75 AND has given 1+ referrals in last 12 months |
| **Loyal** | Health score ≥ 50 OR (health score ≥ 40 AND has referred ever) |
| **Satisfied** | Default / health score < 50 |

Agent can always override with `tier_override = TRUE`.

---

## 6. MVP Scope Definition

### In Scope (MVP — Weeks 1–8)

| Feature | Priority |
|---|---|
| Email/password auth (Supabase) | P0 |
| Agent profile + onboarding flow | P0 |
| Contact CRUD (create, read, update, archive) | P0 |
| Mackay 66 profile view (all fields, JSONB) | P0 |
| Contact segmentation (primary + tags) | P0 |
| Kanban board (5 columns, drag, filter) | P0 |
| Touch logging (minimal: type + date) | P0 |
| Touch types (system defaults + custom) | P0 |
| Relationship health score calculation | P0 |
| Badge system (profile + touch badges) | P1 |
| CSV import with column mapping | P1 |
| Annual touch calendar (wizard + view) | P1 |
| Insights dashboard (core metrics) | P1 |
| Referral logging (simple) | P1 |
| Transaction logging (count only) | P1 |
| Weekly email digest | P2 |
| Responsive design (desktop + mobile) | P0 |
| Supabase RLS on all tables | P0 |
| Free tier enforcement (50 contact limit) | P1 |
| Landing/marketing page | P2 |

### Out of Scope (Post-MVP)

- AI-assisted profiling (extract Mackay fields from notes)
- AI touch suggestions
- Built-in email/SMS sending
- CRM integrations (Follow Up Boss, kvCORE, etc.)
- Google Contacts / phone import
- Team/brokerage dashboards
- Push notifications (browser)
- Daily email digest (weekly only for MVP)
- Native mobile app
- Stripe payment integration (manual upgrade for MVP)
- Social media integration
- Referral network visualization

---

## 7. Development Phases

### Phase 1: Foundation (Weeks 1–2)
- Project setup: Next.js + Chakra UI + Supabase + GitHub + Vercel
- Supabase schema creation (all tables, RLS, indexes, seed data)
- Auth flow: signup, login, logout, protected routes
- Basic layout: shell, navigation, responsive sidebar/tabs
- Agent profile page + settings

### Phase 2: Core Contact Engine (Weeks 3–4)
- Contact CRUD
- Contact list view (search, filter, sort)
- Contact detail page with Mackay 66 sections
- CSV import with column mapping
- Segmentation (primary segment + tags)
- Contact limit enforcement (free tier)

### Phase 3: Touch System + Kanban (Weeks 5–6)
- Touch types (defaults + custom)
- Touch logging (minimal MVP)
- Kanban board with 5 columns
- Drag-and-drop interactions
- `next_touch_due` calculation logic
- Relationship health score calculation
- Badge system

### Phase 4: Calendar + Insights (Weeks 7–8)
- Onboarding flow (assessment + first-five + wizard)
- Touch calendar wizard
- Calendar view (12-month grid)
- Insights dashboard (core metrics)
- Referral + transaction logging
- Weekly email digest
- Polish, testing, responsive QA

---

## 8. Environment & Configuration

### Environment Variables

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# App
NEXT_PUBLIC_APP_URL=
NEXT_PUBLIC_FREE_TIER_CONTACT_LIMIT=50

# Email (for digest)
RESEND_API_KEY=

# Cron (Vercel)
CRON_SECRET=
```

### Supabase Project Setup Checklist

1. Create Supabase project
2. Enable email auth provider
3. Run schema migration (all tables)
4. Enable RLS on all tables
5. Create RLS policies
6. Seed default touch types
7. Seed default badge definitions
8. Set up Edge Functions (if needed)

### Vercel Deployment Checklist

1. Connect GitHub repo
2. Set environment variables
3. Configure build: `next build`
4. Set up cron job for email digest: `/api/cron/weekly-digest`
5. Custom domain (once product name is decided)

---

## 9. Data Flow Diagrams

### Touch Logging → Score Update Flow

```
Agent logs touch
    │
    ▼
Insert into contact_touches
    │
    ▼
Update contacts.last_touch_date = today
    │
    ▼
Recalculate contacts.total_touches_ytd
    │
    ▼
Recalculate contacts.touch_consistency_score
    │
    ▼
Recalculate contacts.relationship_health_score
    │
    ▼
Evaluate tier suggestion (suggest upgrade/downgrade?)
    │
    ▼
Evaluate badge criteria (any new badges earned?)
    │
    ▼
Recalculate contacts.next_touch_due based on cadence rules
    │
    ▼
Contact card updates in Kanban (moves to new column if needed)
```

### Kanban Column Assignment Logic

```sql
-- Pseudocode for Kanban column assignment
CASE
  WHEN next_touch_due IS NULL THEN 'no_plan'
  WHEN next_touch_due < CURRENT_DATE THEN 'overdue'
  WHEN next_touch_due = CURRENT_DATE THEN 'due_today'
  WHEN next_touch_due <= (CURRENT_DATE + INTERVAL '7 days' - 
       (EXTRACT(DOW FROM CURRENT_DATE) || ' days')::INTERVAL) THEN 'due_this_week'
  ELSE 'up_to_date'
END AS kanban_column
```

---

## 10. API Routes (Next.js)

### Protected API Routes

All `/api/*` routes require Supabase auth session validation.

```
POST   /api/contacts              → Create contact
GET    /api/contacts              → List contacts (with filters)
GET    /api/contacts/[id]         → Get contact detail
PATCH  /api/contacts/[id]         → Update contact
DELETE /api/contacts/[id]         → Archive (soft delete) contact
POST   /api/contacts/import       → CSV import (multipart form)

POST   /api/touches               → Log a touch
GET    /api/touches/[contactId]   → Get touch history for contact

POST   /api/referrals             → Log a referral
POST   /api/transactions          → Log a transaction

GET    /api/insights              → Aggregate metrics for dashboard
GET    /api/kanban                → Contacts grouped by Kanban column

POST   /api/touch-plans           → Create/update touch plan
GET    /api/touch-plans/active    → Get active touch plan

GET    /api/cron/weekly-digest    → Cron endpoint (Vercel cron, auth by CRON_SECRET)
```

---

## 11. Non-Functional Requirements

| Requirement | Target |
|---|---|
| Page load (initial) | < 2 seconds |
| Kanban interaction (drag) | < 200ms response |
| CSV import (500 contacts) | < 30 seconds |
| Database query (contact list) | < 500ms |
| Uptime | 99.5% (Vercel + Supabase SLA) |
| Data backup | Supabase automatic daily backups |
| SSL/HTTPS | Enforced (Vercel default) |
| Responsive breakpoints | Mobile (< 768px), Tablet (768–1024px), Desktop (> 1024px) |
| Browser support | Chrome, Safari, Firefox, Edge (latest 2 versions) |
| Accessibility | WCAG 2.1 AA (Chakra UI baseline) |
