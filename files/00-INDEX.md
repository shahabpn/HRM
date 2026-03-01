# HRM Knowledge Base — Index

## How to Use This Knowledge Base

These documents are the source of truth for the HRM project. Upload all files to Claude Code (or any AI development context) before starting development. Each document covers a specific domain:

| # | File | Purpose | When to Reference |
|---|---|---|---|
| 01 | PROJECT-OVERVIEW.md | What HRM is, business model, team, target user, timeline | Project kickoff, decision-making, scope questions |
| 02 | ARCHITECTURE.md | Tech stack, route map, API routes, dev phases, env vars | Setting up the project, deployment, infrastructure decisions |
| 03 | DATABASE-SCHEMA.md | Complete Supabase/PostgreSQL schema, all tables, RLS, indexes | Writing any database queries, migrations, or data operations |
| 04 | FEATURES.md | Every feature with acceptance criteria and interaction specs | Building any feature, writing tests, reviewing PRs |
| 05 | UI-UX-DESIGN.md | Design system, component inventory, responsive behavior, empty states | Building any UI component, responsive design, styling decisions |
| 06 | PRODUCT-METHODOLOGY.md | Mackay 66, LRS 5 steps, 3R framework, touch types, referral math | Understanding WHY a feature exists, domain terminology, content decisions |
| 07 | DEVELOPMENT-CONVENTIONS.md | Project structure, naming, TypeScript types, Supabase patterns, libraries | Writing any code, naming things, structuring files |

## Quick Reference

**Stack:** Next.js 14+ (App Router) + Chakra UI + Supabase (cloud) + Vercel + GitHub

**Auth:** Email/password via Supabase Auth

**Free tier limit:** 50 contacts

**Kanban columns:** Overdue | Due Today | Due This Week | Up to Date | No Plan

**Contact segments:** buyer_seller | referring_agent | strategic_alliance

**Relationship tiers:** satisfied | loyal | advocacy

**Health score:** (profile_depth × 0.35) + (touch_consistency × 0.65)

**Touch target:** 27–36 per contact per year, varies by segment × tier

**MVP timeline:** 6–8 weeks across 4 phases

## Document Versions

All documents: v1.0 — February 19, 2026
