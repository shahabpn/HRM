# HRM Product Methodology & Domain Knowledge

## Overview

This document captures the real estate referral methodologies that HRM operationalizes. It serves as the domain knowledge reference for anyone building or maintaining the product. All features should be traceable back to these frameworks.

---

## The Mackay 66 Contact Profiling System

Created by Harvey Mackay. The principle: the more deeply you know someone, the stronger the relationship and the more likely they are to refer you. HRM stores all 66 fields per contact.

### Category 1: Personal Info (5 fields)
nickname, birthdate, height, weight, hometown

### Category 2: Education (9 fields)
high_school, high_school_year, college, college_year, degrees, college_honors, fraternity_sorority, extracurricular_activities, sensitive_about_no_college

### Category 3: Family (6 fields)
spouse_name, spouse_education, spouse_interests, anniversary, children (name/age), children_education, children_interests

### Category 4: Military (3 fields)
service, discharge_rank, attitude_toward_service

### Category 5: Business Background (~15 fields)
current_occupation, previous_employers (company/location/title/dates — multiple), previous_positions_current_company, status_symbols_in_office, professional_associations, offices_held_honors, business_relationships_with_others, long_range_business_objective, immediate_business_objective, greatest_concern (company vs personal welfare), time_orientation (present vs future)

### Category 6: Special Interests & Lifestyle (~16 fields)
clubs_associations, politically_active (party/importance), community_involvement, religion, confidential_sensitive_items, strong_feelings_topics, medical_current_health, drinking_habits, smoking_habits, favorite_lunch_spots, favorite_dinner_spots, favorite_menu_items, objects_to_meal_bought, hobbies_recreational, vacation_habits

### Category 7: Customer & You (~14 fields)
moral_ethical_considerations, feels_obligation, requires_habit_change, concerned_about_others_opinions, self_centered, highly_ethical, key_problems_their_view, management_priorities, can_you_help, competitor_advantages, spectator_sports_teams, cars, conversational_interests, who_they_want_to_impress, how_they_want_to_be_seen, adjectives_to_describe, proudest_achievement, long_range_personal_objective, immediate_personal_goal

### Privacy Note
Some Mackay 66 fields are sensitive (health, religion, politics, drinking). The app must handle these with care — no auto-sharing, no analytics on sensitive fields, agent-only visibility.

---

## The Lifetime Referral System (LRS) — 5 Steps

From Richard Robbins International (RRI). This is the operational methodology HRM implements.

### Step 1: Build Your List
- Identify everyone you know by first name
- Use Memory Jogger categories to prompt recall
- Gather contact + profile details

**HRM feature:** Contact creation, CSV import, Memory Jogger onboarding, Mackay 66 profiling

### Step 2: Connect, Educate, Qualify, Get Permission
- Goal: 3–5 connections per day, 5 days per week
- Educate contacts on your services
- Qualify: would they work with you or refer you?
- Ask permission to stay in touch
- Record preferred communication channel

**HRM feature:** Daily touch goal (default 5), communication_permission field, preferred_channel field, Kanban daily queue

### Step 3: Segment Your Database
Three core groups:
1. **Past, present, and potential buyers/sellers** — your main sphere
2. **Referring agents** — other agents who could send you business
3. **Strategic alliances** — mortgage advisors, local businesses, service providers

**HRM feature:** primary_segment field, tags for secondary attributes

### Step 4: Grow and Nurture for 15% Return
- Target: 15–20 transactions per 100 advocates per year
- Add 1 new advocate per week
- Purge non-supporters to maintain productive database
- Track, monitor, review monthly

**HRM feature:** Database return rate metric, insights dashboard, archive (purge) contacts, database growth tracking

### Step 5: Stay in Touch 27–36 Times Per Year
- Deliver value, not noise
- Multiple channels throughout the year
- Focus on deepening relationships continuously

**HRM feature:** Touch calendar, cadence engine, touch logging, consistency scoring

---

## The 10 Touch Types (with Frequency Targets)

From RRI. These are the default touch types in HRM.

| # | Type | Touches/Year | Category |
|---|---|---|---|
| 1 | Personalized Mail (Newsletter) | 8–12 | mail |
| 2 | Email: Market updates, statistics, CMAs, video | 12 | email |
| 3 | Milestone/occasion cards and small gifts | 4–6 | gift |
| 4 | Trades & Services Directory | 1–2 | mail |
| 5 | Social media engagement (comment, like, share) | 2–10 | social_media |
| 6 | Annual/semi-annual CMAs (Real Estate Health Check) | 2 | cma |
| 7 | Face-to-face or voice-to-voice conversations | 4 | personal |
| 8 | Pre and post sale WOW (1 day, 1 week, 1 month follow-up) | 4 | personal |
| 9 | Deliver the unexpected | 1 | gift |
| 10 | Events and community involvement | 1–5 | event |

**Total range: 27–36 touches per year when combined.**

---

## The 3R Framework

Braden's workshop framework. Every touch, every feature, every interaction should serve at least one R.

### Relevant
- Communications are timely and personalized
- Based on deep knowledge (Mackay 66)
- Connected to what's happening in their life
- Example: "Saw your daughter graduated — congratulations!" (from Mackay family data)

### Remarkable
- Delivering unexpected value
- Going beyond what's expected
- Making people talk about you to others
- Example: Sending a BBQ box for Father's Day (from Keith Roy's calendar)

### Remembered
- Consistent visibility throughout the year
- Not just during transactions
- Top of mind when someone mentions real estate
- Example: Monthly market update emails + quarterly personal calls = always present

---

## Relationship Levels

Three tiers based on RRI methodology:

### Level 1: Satisfied (★)
- Had an okay experience
- May or may not use you again
- NOT likely to refer you
- **Agent goal:** Move to Loyal through consistent touches

### Level 2: Loyal (★★★)
- Will likely use you again
- But referrals CANNOT be counted on
- **Agent goal:** Move to Advocacy through remarkable experiences

### Level 3: Advocacy (★★★★★)
- Raving fans
- Actively spread the word without being asked
- Passionate about who you are
- You become a topic of their conversations
- **This is the goal.** This is where predictable referral income comes from.

---

## The Referral Worthiness Assessment

From RRI. Used in HRM onboarding.

### Step 1: Current Rating (1–10)
"How well are you doing today with repeat and referral business?"

### Step 2: Runway Question
"If you stopped actively marketing and prospecting today, how long until you run out of business?"
Options: 1 day | 1 week | 1 month | 1 year | 5 years

### Step 3: Goal Rating (1–10)
"Where do you want to be in 3, 6, or 12 months?"

**HRM tracks this over time** — the gap between current and goal drives motivation and measures the impact of using the system.

---

## Memory Jogger Categories

Used during onboarding to help agents recall everyone they know. From RRI.

Accountant, Advertising, Airlines, Antiques, Apartments, Architect, Associates, Athletics, Attorney, Auctioneer, Auditor, Automobile, Babysitters, Band/Orchestra, Banking, Banquets, Barber, Baseball, Basketball, Bicycles, Billiards, Boats, Bookkeeping, Bowling, Broadcasting, Builders, Buses, Butchers, Camping, Caretakers, Chiropractors, Church, Cleaners, Close Friends, Clubs, Coworkers, Colleges, Computers, Consulting, Contractors, Copying, Cosmetics, Couriers, Crafts, Credit Union, Cruises, Day Care, Deliveries, Dentists, Dermatologists, Designers, Detectives, Diaper Service, DJs, Doctors, Driving Ranges, Dry Cleaners, Dry Wall, Electrician, Engineering, Entertainment, Eye Care, Family Members, Farming, Fire Fighter, Florists, Food Service, Fund Raising, Furniture, Gardens, Gift Shops, Golfing, Government, Graphic Arts, Grocery Store, Gymnastics, Hair/Nail Salon, Handy Person, Hardware, Health Clubs, Health Insurance, Hiking, Horses, Hospitals, Hotels, Hunting, Ice Skating, Jewelry, Judo/Karate, Leasing, Libraries, Management, Manufacturing, Military Personnel, Mobile Homes, Mobile Services, Mortgage Advisors, Movers, Past Associates, Police, Publishers, Real Estate, Security Systems, Social Services, Stocks and Bonds, Surveyors, Teachers, Title Companies, Training, Unions, Vendors, Volunteers

---

## Database ROI Calculation

The core financial metric. Every agent should know these numbers.

| Variable | Description | Example |
|---|---|---|
| A | Total advocates in database | 100 |
| B | Transactions/year from database | 15 |
| C | % Return (B/A × 100) | 15% |
| D | Average commission | $15,000 |
| E | Yearly revenue from database (B × D) | $225,000 |
| F | Value of one advocate (E / A) | $2,250 |

**Target:** 10–20% return rate. Below 10% = system needs work. Above 15% = strong performance.

**MVP tracks:** A (contacts), B (transactions), C (% return). Commission tracking (D, E, F) is post-MVP.

---

## Real-World Calendar Example (Keith Roy)

A top-producing RE/MAX agent's actual annual calendar. Five channels running every month.

### Monthly Recurring (every month):
- Market Update Video Email
- CMAs × 20 (targeted subset)
- Personal Lunches
- Home Visits with surprises × 20
- Birthday Boxes
- Monthly Online Info Session Ad
- FB Community Marketing

### Seasonal One-Offs:
- January: Tax Assessment email
- February: Formal invitation to client event (wax-sealed envelope), Real Estate Health Survey
- March: Online client entertainment event
- April: Easter Drop Off × 150, Hot New Listing email
- May: Mother's Day Card with Donation
- June: Father's Day Card with Donation, Q2 Micro-event
- July: Listing Postcard
- September: Homeshow, Fall event invitation
- October: 40 for 40 Fundraiser, Q4 Micro-event
- December: Christmas Letter, Christmas Video, Poinsettia × 150

### Key Insight
This calendar runs multiple touch types SIMULTANEOUSLY every month. It's not "one touch per month" — it's layered touches across channels creating consistent multi-dimensional visibility. HRM's calendar should model this layered approach.

---

## Cadence Variation Rules

Touches vary by BOTH segment AND relationship tier.

### Principle
- Advocates deserve more attention — they're your most valuable contacts
- Strategic alliances have different needs than buyers/sellers
- Not everyone gets the same treatment — that's the point of segmentation

### Example Cadence Matrix

| Segment | Advocacy | Loyal | Satisfied |
|---|---|---|---|
| Buyer/Seller | 36/year | 30/year | 24/year |
| Referring Agent | 24/year | 18/year | 12/year |
| Strategic Alliance | 24/year | 18/year | 12/year |

These defaults are generated by the Calendar Wizard and stored in touch_plans.cadence_rules. Agents can customize.
