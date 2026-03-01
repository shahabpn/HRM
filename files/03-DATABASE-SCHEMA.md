# HRM Database Schema (Supabase / PostgreSQL)

## Entity Relationships

```
auth.users (Supabase managed)
  │
  ├── agent_profiles (1:1)
  │
  ├── contacts (1:many)
  │     ├── contact_mackay_profiles (1:1)
  │     ├── contact_touches (1:many)
  │     └── contact_referrals (1:many)
  │
  ├── touch_types (1:many, system defaults + agent custom)
  │
  ├── touch_plans (1:many)
  │     └── touch_plan_items (1:many)
  │
  ├── transactions (1:many)
  │
  └── badge_definitions (system-wide, no agent FK)
```

## Tables

### agent_profiles

```sql
CREATE TABLE agent_profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name TEXT NOT NULL,
  email TEXT NOT NULL,
  phone TEXT,
  brokerage_name TEXT,
  city TEXT,
  province_state TEXT,
  timezone TEXT DEFAULT 'America/Vancouver',

  -- Onboarding assessment
  referral_rating_current INTEGER CHECK (referral_rating_current BETWEEN 1 AND 10),
  referral_runway TEXT CHECK (referral_runway IN ('1_day', '1_week', '1_month', '1_year', '5_years')),
  referral_rating_goal INTEGER CHECK (referral_rating_goal BETWEEN 1 AND 10),
  referral_goal_timeline TEXT CHECK (referral_goal_timeline IN ('3_months', '6_months', '12_months')),

  -- Settings
  average_commission NUMERIC(10,2),
  daily_touch_goal INTEGER DEFAULT 5,
  annual_touch_target INTEGER DEFAULT 36,
  preferred_touch_channels TEXT[],

  -- Subscription
  tier TEXT DEFAULT 'free' CHECK (tier IN ('free', 'paid')),
  contact_limit INTEGER DEFAULT 50,

  -- Future multi-tenancy
  organization_id UUID,
  team_id UUID,

  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  onboarding_completed_at TIMESTAMPTZ
);
```

### contacts

```sql
CREATE TABLE contacts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,

  -- Required (minimum to create)
  first_name TEXT NOT NULL,
  last_name TEXT,

  -- Contact info
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

  -- Relationship tier
  relationship_tier TEXT DEFAULT 'satisfied'
    CHECK (relationship_tier IN ('satisfied', 'loyal', 'advocacy')),
  tier_override BOOLEAN DEFAULT FALSE,

  -- Communication
  preferred_channel TEXT CHECK (preferred_channel IN ('email', 'phone', 'text', 'messenger', 'in_person', 'social')),
  communication_permission BOOLEAN DEFAULT FALSE,

  -- Calculated scores (updated by triggers/app logic)
  profile_completeness_score NUMERIC(5,2) DEFAULT 0,
  touch_consistency_score NUMERIC(5,2) DEFAULT 0,
  relationship_health_score NUMERIC(5,2) DEFAULT 0,

  -- Touch tracking
  last_touch_date DATE,
  next_touch_due DATE,
  total_touches_ytd INTEGER DEFAULT 0,
  touch_target_annual INTEGER, -- NULL = use agent default

  -- Referrals
  total_referrals_given INTEGER DEFAULT 0,
  last_referral_date DATE,

  -- Badges
  badges_earned JSONB DEFAULT '[]',

  -- Notes
  notes TEXT,

  -- Future multi-tenancy
  organization_id UUID,
  team_id UUID,

  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  archived_at TIMESTAMPTZ,

  UNIQUE(agent_id, email)
);

CREATE INDEX idx_contacts_agent_touch ON contacts(agent_id, next_touch_due);
CREATE INDEX idx_contacts_agent_segment ON contacts(agent_id, primary_segment);
CREATE INDEX idx_contacts_agent_tier ON contacts(agent_id, relationship_tier);
CREATE INDEX idx_contacts_tags ON contacts USING GIN(tags);
CREATE INDEX idx_contacts_archived ON contacts(agent_id) WHERE archived_at IS NULL;
```

### contact_mackay_profiles

Stores all 66 Mackay fields as structured JSONB by category. One row per contact.

```sql
CREATE TABLE contact_mackay_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID NOT NULL UNIQUE REFERENCES contacts(id) ON DELETE CASCADE,

  personal_info JSONB DEFAULT '{}',
  -- Keys: nickname, birthdate, height, weight, hometown

  education JSONB DEFAULT '{}',
  -- Keys: high_school, high_school_year, college, college_year, degrees,
  --   college_honors, fraternity_sorority, extracurricular_activities,
  --   sensitive_about_no_college (boolean)

  family JSONB DEFAULT '{}',
  -- Keys: spouse_name, spouse_education, spouse_interests, anniversary,
  --   children (array of {name, age, education, interests})

  military JSONB DEFAULT '{}',
  -- Keys: service, discharge_rank, attitude_toward_service

  business_background JSONB DEFAULT '{}',
  -- Keys: current_occupation, previous_employers (array of {company, location, title, dates}),
  --   previous_positions_current_company (array), status_symbols_in_office,
  --   professional_associations, offices_held_honors, long_range_business_objective,
  --   immediate_business_objective, greatest_concern, time_orientation

  special_interests JSONB DEFAULT '{}',
  -- Keys: clubs_associations, politically_active (bool), political_party,
  --   political_importance, community_involvement, religion,
  --   confidential_sensitive_items, strong_feelings_topics,
  --   hobbies_recreational, vacation_habits

  lifestyle JSONB DEFAULT '{}',
  -- Keys: medical_current_health, drinks_alcohol (bool), alcohol_details,
  --   offended_by_drinking (bool), smokes (bool), objects_to_smoking (bool),
  --   favorite_lunch_spots, favorite_dinner_spots, favorite_menu_items,
  --   objects_to_meal_being_bought (bool)

  customer_and_you JSONB DEFAULT '{}',
  -- Keys: moral_ethical_considerations, feels_obligation_to_you, obligation_details,
  --   requires_habit_change, concerned_about_others_opinions (bool),
  --   self_centered (bool), highly_ethical (bool), key_problems_their_view,
  --   management_priorities, can_you_help, competitor_advantages,
  --   spectator_sports_teams, cars, conversational_interests,
  --   who_they_want_to_impress, how_they_want_to_be_seen,
  --   adjectives_to_describe, proudest_achievement,
  --   long_range_personal_objective, immediate_personal_goal

  fields_completed INTEGER DEFAULT 0,
  total_fields INTEGER DEFAULT 66,
  last_updated_at TIMESTAMPTZ DEFAULT NOW(),
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### touch_types

System defaults (agent_id = NULL) + agent custom types.

```sql
CREATE TABLE touch_types (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID REFERENCES agent_profiles(id) ON DELETE CASCADE,

  name TEXT NOT NULL,
  category TEXT NOT NULL CHECK (category IN (
    'mail', 'email', 'personal', 'social_media', 'event', 'gift', 'cma', 'other'
  )),
  description TEXT,
  icon TEXT,
  is_system_default BOOLEAN DEFAULT FALSE,
  is_active BOOLEAN DEFAULT TRUE,
  suggested_annual_frequency INTEGER,

  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**System default seed data:**

| Name | Category | Frequency/Year | Icon |
|---|---|---|---|
| Phone Call | personal | 4 | phone |
| In-Person Meeting | personal | 4 | users |
| Personal Lunch/Coffee | personal | 4 | coffee |
| Home Visit | personal | 4 | home |
| Email - Market Update | email | 12 | mail |
| Email - Newsletter | email | 8 | newspaper |
| Email - Hot New Listing | email | 4 | trending-up |
| Personalized Mail | mail | 8 | send |
| Birthday Card/Gift | gift | 1 | gift |
| Holiday Card/Gift | gift | 2 | heart |
| Special Occasion Card | gift | 2 | star |
| Unexpected Gift/Surprise | gift | 1 | sparkles |
| CMA / Real Estate Health Check | cma | 2 | bar-chart |
| Social Media Engagement | social_media | 10 | thumbs-up |
| Client Event (Micro) | event | 2 | calendar |
| Client Event (Macro) | event | 2 | users |
| Trades & Services Directory | mail | 1 | book |
| Post-Sale Follow-Up | personal | 4 | check-circle |

### contact_touches

```sql
CREATE TABLE contact_touches (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  contact_id UUID NOT NULL REFERENCES contacts(id) ON DELETE CASCADE,
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,
  touch_type_id UUID NOT NULL REFERENCES touch_types(id),

  touch_date DATE NOT NULL DEFAULT CURRENT_DATE,

  -- V2 fields (nullable for MVP)
  channel TEXT,
  outcome TEXT,
  next_action TEXT,
  note TEXT,

  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_touches_contact ON contact_touches(contact_id, touch_date DESC);
CREATE INDEX idx_touches_agent_date ON contact_touches(agent_id, touch_date DESC);
```

### contact_referrals

```sql
CREATE TABLE contact_referrals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,
  referrer_contact_id UUID NOT NULL REFERENCES contacts(id) ON DELETE CASCADE,

  referred_name TEXT NOT NULL,
  referred_contact_id UUID REFERENCES contacts(id),

  referral_date DATE NOT NULL DEFAULT CURRENT_DATE,
  status TEXT DEFAULT 'received' CHECK (status IN ('received', 'contacted', 'converted', 'lost')),
  became_transaction BOOLEAN DEFAULT FALSE,

  note TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### transactions

```sql
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,
  contact_id UUID REFERENCES contacts(id),
  referral_id UUID REFERENCES contact_referrals(id),

  transaction_date DATE NOT NULL,
  transaction_type TEXT CHECK (transaction_type IN ('buy', 'sell', 'both')),
  source TEXT CHECK (source IN ('database_referral', 'database_repeat', 'external', 'other')),

  note TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### touch_plans + touch_plan_items

```sql
CREATE TABLE touch_plans (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,

  name TEXT NOT NULL DEFAULT 'My Touch Plan',
  year INTEGER NOT NULL DEFAULT EXTRACT(YEAR FROM NOW()),
  is_active BOOLEAN DEFAULT TRUE,

  cadence_rules JSONB DEFAULT '{}',
  -- Structure:
  -- {
  --   "buyer_seller": {
  --     "advocacy": { "annual_target": 36, "channels": [...] },
  --     "loyal": { "annual_target": 30, "channels": [...] },
  --     "satisfied": { "annual_target": 24, "channels": [...] }
  --   },
  --   "referring_agent": { ... },
  --   "strategic_alliance": { ... }
  -- }

  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE touch_plan_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  touch_plan_id UUID NOT NULL REFERENCES touch_plans(id) ON DELETE CASCADE,
  agent_id UUID NOT NULL REFERENCES agent_profiles(id) ON DELETE CASCADE,

  touch_type_id UUID NOT NULL REFERENCES touch_types(id),
  title TEXT NOT NULL,
  description TEXT,

  month INTEGER NOT NULL CHECK (month BETWEEN 1 AND 12),
  day_of_month INTEGER,

  applies_to_segments TEXT[] DEFAULT '{buyer_seller,referring_agent,strategic_alliance}',
  applies_to_tiers TEXT[] DEFAULT '{satisfied,loyal,advocacy}',
  is_recurring BOOLEAN DEFAULT TRUE,

  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### badge_definitions

```sql
CREATE TABLE badge_definitions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  name TEXT NOT NULL,
  description TEXT NOT NULL,
  icon TEXT NOT NULL,
  category TEXT NOT NULL, -- 'profile', 'touch', 'referral', 'milestone'

  criteria_type TEXT NOT NULL,
  criteria_config JSONB NOT NULL,
  -- Examples:
  -- { "type": "mackay_category_complete", "category": "family", "threshold": 80 }
  -- { "type": "touch_streak", "days": 30 }
  -- { "type": "referrals_received", "count": 5 }
  -- { "type": "profile_completeness", "threshold": 50 }

  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## Row Level Security

**Pattern applied to ALL agent-owned tables:**

```sql
ALTER TABLE [table_name] ENABLE ROW LEVEL SECURITY;

CREATE POLICY "agents_own_data" ON [table_name]
  FOR ALL
  USING (agent_id = auth.uid())
  WITH CHECK (agent_id = auth.uid());
```

Apply to: `contacts`, `contact_mackay_profiles` (via join), `contact_touches`, `contact_referrals`, `transactions`, `touch_plans`, `touch_plan_items`, agent-specific `touch_types`.

For `touch_types` (has system defaults where agent_id IS NULL):

```sql
CREATE POLICY "system_and_own_types" ON touch_types
  FOR SELECT
  USING (agent_id IS NULL OR agent_id = auth.uid());

CREATE POLICY "manage_own_types" ON touch_types
  FOR INSERT
  WITH CHECK (agent_id = auth.uid());

CREATE POLICY "modify_own_types" ON touch_types
  FOR UPDATE
  USING (agent_id = auth.uid());

CREATE POLICY "delete_own_types" ON touch_types
  FOR DELETE
  USING (agent_id = auth.uid());
```

## Kanban Column Assignment

```sql
CASE
  WHEN next_touch_due IS NULL THEN 'no_plan'
  WHEN next_touch_due < CURRENT_DATE THEN 'overdue'
  WHEN next_touch_due = CURRENT_DATE THEN 'due_today'
  WHEN next_touch_due <= date_trunc('week', CURRENT_DATE) + INTERVAL '6 days' THEN 'due_this_week'
  ELSE 'up_to_date'
END AS kanban_column
```
