---
name: iamabdullah-dev-project
description: Use this skill for ANY work inside the iamabdullah.dev project — writing code, creating files, planning a feature, fixing a bug, or answering "what should I do next." This is the authoritative source for how this specific project must be built: phase discipline, architecture boundaries, coding rules, design conventions, and the memory-tracking workflow. Always consult this before touching code, even for small changes.
---

# iamabdullah.dev — Project Skill

## What this project is

A personal-brand, SEO-focused review blog (Next.js, DB-backed, custom admin panel) reviewing AI/automation tools — starting with a Meta Ads tools pillar niche, structured to expand into adjacent SaaS categories later without a rebuild. Full context lives in `prd.md`, `architecture.md`, `rules.md`, `phases.md`, and `design.md` at the project root — this skill file tells you how to use those documents, not what's in them. Read the actual docs for details; this file is the operating procedure.

## Before doing anything

1. Read `memory.md` first, every session. It has "Current Phase," "Completed Features," "Active Bugs," and "Open Questions" — this tells you exactly where the project stands. Never assume; check.
2. If `memory.md` says a phase is in progress, resume it. Do not jump to a later phase in `phases.md` because it seems more interesting or urgent — sequential order is a hard constraint, not a suggestion.
3. If the user's request doesn't map to the current phase's checklist item, ask before proceeding — either the phase order needs revisiting (rare, ask explicitly) or the request should wait.

## Non-negotiable boundaries (full detail in rules.md — this is the summary you must never violate)

- **Server Actions are the default** for mutations. Route Handlers only for: external webhooks, non-HTML responses (RSS/sitemap), or the redirect/cron endpoints already defined in `architecture.md`. Don't invent a new API route when a Server Action would do.
- **Every `/admin/**` route is session-protected** via `middleware.ts`, AND every admin Server Action re-verifies the session server-side. Both layers, always — never ship one without the other.
- **No new libraries outside the approved list in `rules.md` §3** without explicitly asking the user first. This includes tempting shortcuts like a state management library or a CSS-in-JS package — the constraint is intentional, not an oversight to route around.
- **Content lives in Postgres, not git-committed MDX files.** If you find yourself about to create a `.mdx` file under `/content`, stop — that was the old architecture before the admin panel was added. Posts go through `lib/posts.ts` and the `posts` table.
- **No plaintext secrets in code, ever** — not even "temporarily for testing." `.env.local` only.
- Performance targets (LCP < 2.5s, CLS < 0.1, INP < 200ms) apply to **public pages**, not the admin panel — don't over-engineer admin UI performance at the cost of shipping speed.

## Design conventions (full detail in design.md)

- Public site: warm/editorial (Sora or Lexend headings, Inter body, indigo/amber/coral palette, `rounded-lg`, `shadow-sm`) — actively avoid a "SaaS dashboard" or dev-portfolio look, since the `.dev` domain already pulls that direction.
- Admin panel: plain, dense, utilitarian is fine — don't spend effort making `/admin` beautiful. Speed of use over brand personality there.
- Never introduce a new color, font, or component style without checking `design.md` first — consistency matters more than any single component looking slightly better.

## Content/SEO discipline

- Every post needs a `category` value (free-form, not enum) — this is what keeps the site expandable beyond Meta Ads later. Never hardcode a niche assumption into a route, filename, or schema field.
- Affiliate links are never inline — always through `lib/affiliate-links.ts`, referenced by tool slug.
- Any change to a published post's content must trigger `revalidatePath`/`revalidateTag` — stale public pages after an edit is a bug, not a minor issue.

## End-of-session workflow (do this every time, not just when asked)

1. Check off completed items in `phases.md`'s current phase inside `memory.md`'s "Completed Features" section.
2. If you deviated from `architecture.md` or `rules.md` for a good reason, log it under "Recent Structural Decisions" in `memory.md` with the date and the reasoning — don't let undocumented drift accumulate.
3. If you hit something unresolved (missing decision, ambiguous requirement), add it to "Open Questions / Blockers" instead of silently guessing and moving on.
4. Update "Current Phase" if the phase changed during this session.

## When something isn't covered by the docs

Ask the user. Don't infer a decision on architecture, library choice, or design direction that isn't already written down — these 6 files are the source of truth specifically so decisions don't get re-litigated or silently drift session to session. A quick question now is cheaper than an undocumented decision that contradicts the docs later.
