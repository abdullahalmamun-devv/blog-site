# PRD — iamabdullah.dev

## Core Problem

Meta Ads managers and solo marketers are flooded with AI/automation tools claiming to save time and money, but most reviews online are either AI-generated shallow content or thinly-disguised affiliate spam with no real usage data. There is no trusted, personal-brand resource that reviews these tools based on actual ad-spend experience, with real numbers, region-specific context (payments, support, currency for South Asian users), and honest pros/cons.

## Target Users

**Primary:** Freelance/solo Meta Ads managers and media buyers (budget-conscious, DIY, often managing $500–$10K/month ad spend) who are evaluating whether a new AI/automation tool is worth the subscription cost.

**Secondary:** Small agency owners running Meta Ads for clients, looking for tools to scale operations without hiring more staff.

**Tertiary (Phase 2+):** Broader SaaS/marketing-tool buyers once the site expands beyond the Meta Ads pillar.

## Primary Features (MVP)

1. **Blog / Review Engine** — DB-backed posts with three content types: single tool review, comparison, roundup. Each post supports a standardized layout (quick verdict box, pricing table, pros/cons, FAQ).
2. **Custom Admin Panel** — Password-protected `/admin` area for full content management: create/edit/delete posts, rich text editing, draft/scheduled/published states, publish-date scheduling, manual drag-and-drop reordering of featured posts, and a media library for image uploads.
3. **Author/Brand Page** — Establishes E-E-A-T: real experience, campaign history, credibility.
4. **Affiliate Link Management** — Cloaked `/go/[slug]` redirect links with click tracking (stored in DB) and proper `rel="sponsored nofollow"` attributes.
5. **Email Capture** — Newsletter signup embedded in posts and a dedicated page, stored via third-party ESP (e.g., ConvertKit) or DB-backed initially.
6. **SEO Infrastructure** — Dynamic metadata, sitemap, robots.txt, JSON-LD (Review + Article schema) per post, editable per-post via the admin panel.
7. **Comments (Phase 2)** — Giscus-based, no custom DB needed.

## Explicit Non-Goals (MVP)

- No public user accounts / authentication for readers (only the admin has a login)
- No payment processing (affiliate-driven, not a paid product — yet)
- No multi-author support initially (single admin user; schema is RBAC-ready for later)
- No content approval workflow (single admin publishes directly; add review/approval only if a second author joins later)
