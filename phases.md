# Development Phases — iamabdullah.dev

Work through these sequentially. Do not start a phase until the previous one's checklist is complete.

## Phase 1 — Project Setup & Infrastructure
- [ ] Initialize Next.js 15 App Router project with TypeScript + Tailwind
- [ ] Set up Vercel project, connect GitHub repo, confirm auto-deploy on push
- [ ] Set up Neon Postgres DB (via Vercel Marketplace integration), connect via Drizzle, run first migration (empty schema)
- [ ] Define `posts`, `clicks`, `subscribers`, `admin_users` tables in `lib/schema.ts`
- [ ] Set up `.env.local` structure + Vercel environment variables
- [ ] Base layout, header, footer, global Tailwind theme (colors/fonts from `design.md`)

## Phase 2 — Auth & Admin Access
- [ ] Set up Auth.js (Credentials provider) with `lib/auth.ts`
- [ ] Seed script to create the single admin user (hashed password via `bcrypt`), run once — no public sign-up route
- [ ] `middleware.ts` protecting all `/admin/**` routes
- [ ] `/admin/login` page + session handling
- [ ] Confirm an unauthenticated visit to any `/admin/*` route redirects to login (test this explicitly before moving on)

## Phase 3 — Post Data Layer & Public Rendering
- [ ] Finalize `posts` table schema (title, slug, content, status, scheduled_at, published_at, category, tool metadata, SEO fields, order)
- [ ] `lib/posts.ts` — CRUD query functions used by both admin and public pages
- [ ] Blog index page (`/blog`) — queries published posts from DB, sorted by `order`/date
- [ ] Single post page (`/blog/[slug]`) — DB-backed, renders Markdown content via `next-mdx-remote/rsc`
- [ ] Reusable content components: `QuickVerdictBox`, `PricingTable`, `ProsConsTable`, `FAQAccordion`
- [ ] Manually insert 1–2 seed posts directly via DB/migration to validate the full render pipeline before the admin editor exists

## Phase 4 — Admin Panel: Post Management
- [ ] Admin dashboard (`/admin`) — recent posts, subscriber count, click stats overview
- [ ] Post list (`/admin/posts`) — filterable by status (draft/scheduled/published)
- [ ] Tiptap-based post editor (`/admin/posts/new`, `/admin/posts/[id]/edit`) with Markdown serialization
- [ ] Server Actions for create/update/delete, all re-verifying the session server-side
- [ ] Draft → Scheduled → Published status flow, with `scheduled_at` date/time picker
- [ ] `revalidatePath` called on every publish/edit so public pages update immediately
- [ ] Drag-and-drop reordering (`dnd-kit`) for featured/homepage post order

## Phase 5 — Scheduling & Media
- [ ] `/api/cron/publish-scheduled` Route Handler — checks due scheduled posts, flips status, revalidates
- [ ] `vercel.json` Cron config (every 5–15 min) targeting the above route, protected by `CRON_SECRET`
- [ ] Media library (`/admin/media`) — Vercel Blob upload integration, image list/delete
- [ ] Wire the editor's image insert to the media library instead of manual URL entry

## Phase 6 — SEO Infrastructure
- [ ] Shared metadata generator in `lib/seo.ts`, pulling per-post SEO fields from the DB, used by every page
- [ ] `app/sitemap.ts`, `app/robots.ts` — DB-backed, includes only published posts
- [ ] JSON-LD structured data (Article + Review schema) injected per post
- [ ] `app/feed.xml/route.ts` for RSS
- [ ] Open Graph image generation (static or `next/og` dynamic), editable per-post via admin

## Phase 7 — Affiliate Link System
- [ ] `lib/affiliate-links.ts` central registry
- [ ] `/app/go/[slug]/route.ts` — redirect handler with click logging to `clicks` table
- [ ] Confirm `rel="sponsored nofollow"` on all outbound affiliate anchors
- [ ] Add affiliate disclosure component, injected at top of every review post

## Phase 8 — Email Capture
- [ ] `NewsletterForm` client component + Server Action
- [ ] ConvertKit API integration in `lib/convertkit.ts`
- [ ] Fallback DB logging to `subscribers` table
- [ ] Embed form in post footer + dedicated `/newsletter` page

## Phase 9 — Author/Brand & Trust Pages
- [ ] `/about` page — author bio, real experience, E-E-A-T signals
- [ ] Homepage hero copy clarifying brand positioning (per `design.md` / branding fix)
- [ ] Contact/disclosure/privacy policy pages (legal minimum)

## Phase 10 — Comments & Analytics
- [ ] Giscus integration on post pages
- [ ] Vercel Analytics + GA4 tag
- [ ] Expand admin dashboard with click/subscriber trend charts (data already in DB from earlier phases)

## Phase 11 — Polish & Launch
- [ ] Lighthouse/Core Web Vitals audit against `rules.md` targets (public pages — admin panel is not held to the same SEO/perf bar)
- [ ] Accessibility pass (semantic HTML, alt text, contrast) on public pages
- [ ] 404/error page polish
- [ ] Final content QA on first batch of launch posts (per the 90-day content plan)
- [ ] Submit sitemap to Google Search Console + Bing Webmaster Tools
