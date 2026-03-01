# HRM Development Conventions

## Project Structure

```
hrm/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Auth route group (no layout nesting)
│   │   ├── login/page.tsx
│   │   └── signup/page.tsx
│   ├── (marketing)/              # Public pages
│   │   └── page.tsx              # Landing page
│   ├── (app)/                    # Protected app routes
│   │   ├── layout.tsx            # App shell (sidebar/tabs + main content)
│   │   ├── dashboard/
│   │   │   ├── today/page.tsx    # Kanban board (default)
│   │   │   ├── contacts/
│   │   │   │   ├── page.tsx      # Contact list
│   │   │   │   ├── [id]/page.tsx # Contact detail
│   │   │   │   └── import/page.tsx
│   │   │   ├── calendar/page.tsx
│   │   │   ├── insights/page.tsx
│   │   │   └── settings/page.tsx
│   │   └── onboarding/
│   │       ├── assessment/page.tsx
│   │       ├── first-five/page.tsx
│   │       ├── calendar/page.tsx
│   │       └── complete/page.tsx
│   ├── api/                      # API routes
│   │   ├── contacts/
│   │   ├── touches/
│   │   ├── referrals/
│   │   ├── transactions/
│   │   ├── insights/
│   │   ├── kanban/
│   │   ├── touch-plans/
│   │   └── cron/
│   ├── layout.tsx                # Root layout (Chakra provider, Supabase provider)
│   └── globals.css
├── components/
│   ├── ui/                       # Reusable UI primitives (extending Chakra)
│   ├── contacts/                 # Contact-specific components
│   │   ├── ContactCard.tsx
│   │   ├── ContactListRow.tsx
│   │   ├── ContactDetail.tsx
│   │   ├── MackayCategorySection.tsx
│   │   └── QuickTouchModal.tsx
│   ├── kanban/                   # Kanban components
│   │   ├── KanbanBoard.tsx
│   │   ├── KanbanColumn.tsx
│   │   ├── KanbanCard.tsx
│   │   └── KanbanSummaryBar.tsx
│   ├── calendar/                 # Calendar components
│   ├── insights/                 # Dashboard components
│   ├── onboarding/               # Onboarding step components
│   └── layout/                   # Shell, nav, sidebar components
├── lib/
│   ├── supabase/
│   │   ├── client.ts             # Browser Supabase client
│   │   ├── server.ts             # Server Supabase client
│   │   └── middleware.ts         # Auth middleware
│   ├── scoring/
│   │   ├── healthScore.ts        # Relationship health score calculation
│   │   ├── profileDepth.ts       # Profile completeness calculation
│   │   └── touchConsistency.ts   # Touch consistency calculation
│   ├── badges/
│   │   └── evaluator.ts          # Badge criteria evaluation
│   ├── calendar/
│   │   └── generator.ts          # Touch plan generation from wizard answers
│   └── utils/
│       ├── kanban.ts             # Kanban column assignment logic
│       └── csv.ts                # CSV parsing and mapping
├── hooks/
│   ├── useContacts.ts
│   ├── useKanban.ts
│   ├── useTouches.ts
│   ├── useInsights.ts
│   └── useAuth.ts
├── types/
│   └── index.ts                  # TypeScript type definitions
├── theme/
│   └── index.ts                  # Chakra UI custom theme
├── middleware.ts                  # Next.js middleware (auth protection)
└── supabase/
    └── migrations/               # SQL migration files
        ├── 001_initial_schema.sql
        ├── 002_seed_touch_types.sql
        ├── 003_seed_badges.sql
        └── 004_rls_policies.sql
```

## Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Files (components) | PascalCase | `ContactCard.tsx` |
| Files (utils/hooks) | camelCase | `useContacts.ts`, `healthScore.ts` |
| Files (pages) | lowercase | `page.tsx` (Next.js convention) |
| React components | PascalCase | `ContactCard`, `KanbanBoard` |
| Functions | camelCase | `calculateHealthScore`, `logTouch` |
| Constants | UPPER_SNAKE | `FREE_TIER_CONTACT_LIMIT` |
| DB tables | snake_case | `contact_touches`, `agent_profiles` |
| DB columns | snake_case | `first_name`, `next_touch_due` |
| TypeScript types | PascalCase | `Contact`, `TouchType`, `AgentProfile` |
| API routes | kebab-case in URL | `/api/touch-plans/active` |
| CSS/theme tokens | camelCase | `primaryColor`, `cardBorderRadius` |

## TypeScript Types (Core)

```typescript
// Core entity types (match DB schema)

type Segment = 'buyer_seller' | 'referring_agent' | 'strategic_alliance';
type RelationshipTier = 'satisfied' | 'loyal' | 'advocacy';
type TouchCategory = 'mail' | 'email' | 'personal' | 'social_media' | 'event' | 'gift' | 'cma' | 'other';
type Channel = 'email' | 'phone' | 'text' | 'messenger' | 'in_person' | 'social';
type KanbanColumn = 'overdue' | 'due_today' | 'due_this_week' | 'up_to_date' | 'no_plan';
type ReferralStatus = 'received' | 'contacted' | 'converted' | 'lost';
type TransactionType = 'buy' | 'sell' | 'both';
type TransactionSource = 'database_referral' | 'database_repeat' | 'external' | 'other';
type Tier = 'free' | 'paid';
type Runway = '1_day' | '1_week' | '1_month' | '1_year' | '5_years';
type GoalTimeline = '3_months' | '6_months' | '12_months';

interface Contact {
  id: string;
  agent_id: string;
  first_name: string;
  last_name: string | null;
  email: string | null;
  phone: string | null;
  primary_segment: Segment;
  tags: string[];
  relationship_tier: RelationshipTier;
  tier_override: boolean;
  preferred_channel: Channel | null;
  communication_permission: boolean;
  profile_completeness_score: number;
  touch_consistency_score: number;
  relationship_health_score: number;
  last_touch_date: string | null;
  next_touch_due: string | null;
  total_touches_ytd: number;
  touch_target_annual: number | null;
  total_referrals_given: number;
  badges_earned: Badge[];
  notes: string | null;
  created_at: string;
  updated_at: string;
  archived_at: string | null;
}

interface TouchType {
  id: string;
  agent_id: string | null;
  name: string;
  category: TouchCategory;
  icon: string;
  is_system_default: boolean;
  is_active: boolean;
  suggested_annual_frequency: number | null;
}

interface Touch {
  id: string;
  contact_id: string;
  agent_id: string;
  touch_type_id: string;
  touch_date: string;
  note: string | null;
  // V2 fields
  channel?: string;
  outcome?: string;
  next_action?: string;
}

interface Badge {
  id: string;
  name: string;
  icon: string;
  earned_at: string;
}
```

## Supabase Patterns

### Client-Side Queries
```typescript
// Use Supabase client SDK for reads from components
import { createClient } from '@/lib/supabase/client';

const supabase = createClient();
const { data, error } = await supabase
  .from('contacts')
  .select('*')
  .eq('agent_id', userId)
  .is('archived_at', null)
  .order('next_touch_due', { ascending: true });
```

### Server-Side Queries (API Routes)
```typescript
// Use server client for writes and sensitive operations
import { createServerClient } from '@/lib/supabase/server';

export async function POST(req: Request) {
  const supabase = createServerClient();
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) return Response.json({ error: 'Unauthorized' }, { status: 401 });
  // ... operation
}
```

### RLS Reliance
- NEVER filter by agent_id in application code as the sole security measure
- RLS handles access control at DB level
- Application code can still filter for UX (e.g., filter archived contacts) but security is RLS

## Scoring Calculation Patterns

### When to Recalculate
Scores should be recalculated:
- **On touch log:** touch_consistency_score + relationship_health_score + tier suggestion
- **On Mackay field update:** profile_completeness_score + relationship_health_score
- **On referral log:** tier suggestion
- **Daily cron (future):** batch recalculate all contacts' consistency scores (for time-decay)

### Where to Calculate
- MVP: calculate in API routes (server-side) on each touch/profile update
- Future: move to Supabase database triggers for consistency

## Error Handling

```typescript
// Standard API response pattern
type ApiResponse<T> = {
  data: T | null;
  error: string | null;
};

// Standard error handling in API routes
try {
  // ... operation
  return Response.json({ data: result, error: null });
} catch (err) {
  console.error('Operation failed:', err);
  return Response.json({ data: null, error: 'Operation failed' }, { status: 500 });
}
```

## Contact Limit Enforcement

```typescript
// Check before creating a contact
const { count } = await supabase
  .from('contacts')
  .select('*', { count: 'exact', head: true })
  .eq('agent_id', userId)
  .is('archived_at', null);

if (agent.tier === 'free' && count >= FREE_TIER_CONTACT_LIMIT) {
  return Response.json({
    data: null,
    error: 'Contact limit reached. Upgrade to add more contacts.'
  }, { status: 403 });
}
```

## Git Conventions

- **Branch naming:** `feature/kanban-board`, `fix/touch-logging-date`, `chore/seed-data`
- **Commit messages:** Conventional commits — `feat:`, `fix:`, `chore:`, `docs:`
- **Main branch:** `main` (deployed to production via Vercel)
- **Development:** Feature branches → PR → merge to main

## Key Libraries to Install

```bash
# Core
npx create-next-app@latest hrm --typescript --app --src-dir=false
npm install @chakra-ui/react @emotion/react @emotion/styled framer-motion

# Supabase
npm install @supabase/supabase-js @supabase/ssr

# Kanban drag-and-drop
npm install @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities

# Charts (insights dashboard)
npm install recharts

# CSV parsing
npm install papaparse
npm install -D @types/papaparse

# Date handling
npm install date-fns

# Icons
npm install react-icons
```
