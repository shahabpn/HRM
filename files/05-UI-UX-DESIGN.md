# HRM UI/UX Design System

## Design Philosophy

- **Warm and approachable** — not cold, not corporate. This is a coaching tool, not enterprise software.
- **Bold and energetic** — motivational. Every screen should inspire action.
- **Clean but not sterile** — professional enough for business, human enough to feel personal.
- **Action-oriented** — the app should make agents want to DO things, not just look at data.

## Color System (Chakra UI Theme)

### Direction
Warm palette. Avoid cool grays and blues that feel corporate. Think about colors that feel like connection, energy, and growth.

### Suggested Palette (customize in Chakra theme)

| Token | Usage | Direction |
|---|---|---|
| Primary | Buttons, links, active states | Warm coral/orange or rich amber |
| Secondary | Accents, badges, highlights | Complementary warm tone |
| Success | Up to Date column, completed touches | Warm green |
| Warning | Due Today, Due This Week | Amber/gold |
| Error | Overdue column, missed touches | Warm red (not harsh) |
| Neutral | Backgrounds, borders, disabled | Warm grays (not blue-gray) |
| Background | Page background | Warm off-white or very light cream |

### Kanban Column Colors

| Column | Color Accent |
|---|---|
| Overdue | Red/coral (urgent, warm) |
| Due Today | Amber/orange (action) |
| Due This Week | Blue-teal (planned) |
| Up to Date | Green (healthy) |
| No Plan | Warm gray (needs attention) |

## Typography

- **Headings:** Bold, modern sans-serif. Chakra UI default (Inter) or similar.
- **Body:** Clean, readable at small sizes for mobile.
- **Data/numbers:** Slightly larger weight for metrics, scores, counts.

## Component Inventory

### Layout Components

**AppShell**
- Desktop: left sidebar (collapsed by default, expandable) + main content area
- Mobile: bottom tab bar + full-width content
- Header bar with agent name, notification indicator, settings gear

**Navigation Tabs**
- 4 tabs: Today, Contacts, Calendar, Insights
- Active tab: bold text + accent underline/background
- Badge count on Today tab showing overdue count

### Contact Components

**ContactCard (Kanban)**
```
┌─────────────────────────────┐
│ [Avatar] Sarah Mitchell      │
│ 🏷️ Buyer/Seller  ★★★★★      │
│ ⬤ 78 health score           │
│ Last: 3 days ago — Phone     │
│ Next: Email - Market Update  │
│ 🏠 💼 🔥                      │
└─────────────────────────────┘
```
- Draggable
- Click → quick touch log modal
- Compact enough for 5-8 visible per column without scrolling

**ContactListRow**
```
[Avatar] Sarah Mitchell | Buyer/Seller | ★★★★★ | Score: 78 | Last: Jan 15 | 34 days ago
```
- Click → navigate to detail page
- Sortable columns

**ContactDetailHeader**
```
┌──────────────────────────────────────────────────┐
│ [Large Avatar]                                    │
│ Sarah Mitchell                                    │
│ 🏷️ Buyer/Seller  ★★★★★ Advocacy                   │
│                                                    │
│ [═══════ 78 ═══] Relationship Health               │
│ [══════ 62% ═══] Profile Completeness              │
│                                                    │
│ [Log Touch] [Edit] [Archive]                       │
└──────────────────────────────────────────────────┘
```

**MackayCategorySection (Accordion)**
```
▼ Family (3/6 fields) 🏠
  Spouse Name: [Jane Mitchell      ]
  Anniversary: [June 15, 2018      ]
  Children:    [Tommy (8), Lisa (5) ]
  Spouse Education: [_______________]  ← empty, placeholder
  Spouse Interests: [_______________]
  Children's Interests: [___________]
```

### Touch Components

**QuickTouchModal**
```
┌──────────────────────────────────┐
│ Log Touch for Sarah Mitchell     │
│                                  │
│ Type: [Phone Call        ▼]      │
│ Date: [Today, Feb 19     ▼]     │
│                                  │
│        [Cancel] [Log Touch]      │
└──────────────────────────────────┘
```
- Minimal. 3 taps: open → select type → save.
- Date defaults to today.

**TouchTimelineItem**
```
📞 Phone Call — Jan 15, 2026
   "Discussed kids' hockey season, mentioned thinking about downsizing"
```

### Kanban Components

**KanbanBoard**
- Horizontal scrolling on mobile, grid on desktop
- Column headers with count badge
- Drag and drop (use @dnd-kit or similar React DnD library)
- Summary bar above columns

**KanbanSummaryBar**
```
┌─────────────────────────────────────────────────────────┐
│ Overdue: 12 │ Today: 5 │ This Week: 8 │ ✅ 3/5 today   │
└─────────────────────────────────────────────────────────┘
```

### Dashboard Components

**MetricCard**
```
┌──────────────┐
│ Database Size │
│     127       │
│   ↑ +3 this  │
│     month     │
└──────────────┘
```

**TierDonutChart**
- Satisfied: warm gray
- Loyal: amber
- Advocacy: primary color
- Center shows total contacts

**MonthlyTouchBarChart**
- Last 12 months
- Bar height = touch count
- Target line overlay

### Calendar Components

**AnnualCalendarGrid**
```
         Mail    Email   Personal Social  Events  Gifts   CMA
Jan      ─       [x]    [x]      [x]     ─       ─       ─
Feb      ─       [x]    [x]      [x]     ─       ─       [x]
Mar      [x]     [x]    [x]      [x]     ─       ─       ─
...
```
- [x] = planned, green = completed, red = missed
- Click cell → details of that touch item

### Onboarding Components

**OnboardingProgress**
- Step indicator: ● ● ○ ○ (filled = complete, outline = remaining)
- Step labels: Assessment → First 5 → Calendar → Ready

**MemoryJoggerPrompt**
- Scrollable list of categories as inspiration chips
- "Think of your: Accountant, Attorney, Barber, Close Friends, Coworkers..."
- Tapping a category highlights it (visual aid, no logic)

**ReferralWorthinessSlider**
- 1–10 scale with emoji/label at each end
- Current vs Goal shown side by side

### Badge Components

**BadgeIcon (small)**
- 20x20px icon on contact cards
- Tooltip on hover showing badge name

**BadgeCard (detail page)**
```
┌─────────────────────────────────┐
│ 🏠 Knows Their Family           │
│ Earned Jan 15, 2026             │
│ Family profile 80%+ complete    │
└─────────────────────────────────┘
```

**BadgeLocked**
- Grayed out version showing what's needed to unlock

## Responsive Behavior

### Desktop (>1024px)
- Sidebar navigation (collapsible)
- Kanban: all 5 columns visible side by side
- Contact detail: full layout with sidebar
- Calendar: full 12-month grid visible

### Tablet (768–1024px)
- Sidebar collapsed by default, hamburger to expand
- Kanban: 3 columns visible, horizontal scroll for rest
- Contact detail: stacked layout

### Mobile (<768px)
- Bottom tab bar navigation
- Kanban: 1 column visible, swipe between columns
- Contact cards: slightly simplified (hide some secondary info)
- Touch logging: full-screen modal
- Calendar: month-by-month view (swipe between months)

## Animations & Micro-interactions

- **Kanban drag:** Smooth card lift + shadow on pickup, snap into column
- **Touch logged:** Success toast with confetti burst (subtle) + card slides to new column
- **Badge earned:** Toast notification with badge icon animation
- **Score change:** Number counter animation when health score updates
- **Onboarding progress:** Step dots animate as you advance

## Empty States

Every view needs an empty state with guidance:

- **Kanban (no contacts):** "Your board is empty. Add your first contacts to get started." + CTA
- **Contact list (no contacts):** Memory Jogger prompt + Add Contact / Import CSV buttons
- **Touch history (no touches):** "No touches logged yet. Start building this relationship." + Log Touch CTA
- **Insights (insufficient data):** "Keep going! You need at least 2 weeks of activity for meaningful insights."
- **Calendar (no plan):** "Set up your touch plan to see your annual calendar." + Calendar Wizard CTA

## Error States

- **Contact limit reached (free tier):** Friendly upgrade prompt, not a hard block on other features
- **CSV import errors:** Show specific row errors, allow partial import of valid rows
- **Network errors:** Toast with retry option
- **Session expired:** Redirect to login with "Session expired" message
