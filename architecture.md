# Architecture — iamabdullah.dev

## Tech Stack

| Layer | Choice |
|---|---|
| Framework | Next.js 15 (App Router, Server Components by default) |
| Content storage | Postgres (DB-backed posts, not git-based MDX — required for a custom admin panel with scheduling, edit/delete, and reordering) |
| Database | Postgres via Neon (added through the Vercel Marketplace integration — Vercel's own native "Vercel Postgres" product was discontinued in 2025; Neon is now the standard way to get a managed Postgres DB in a Vercel project), accessed with Drizzle ORM |
| Content editor | Tiptap (rich text editor, stores content as Markdown/MDX-compatible output) |
| Auth (admin only) | Auth.js (NextAuth) — Credentials provider, single admin user for MVP, RBAC-ready |
| Media storage | Vercel Blob (image uploads from the admin panel — screenshots, OG images) |
| Styling | Tailwind CSS |
| Hosting | Vercel |
| Email | ConvertKit API (or Resend for transactional) |
| Comments | Giscus (GitHub Discussions-based, no DB) |
| Analytics | Vercel Analytics + GA4 |

## Why DB-backed instead of git-based MDX

The original plan used git-committed MDX files, which is simple but means "publishing" = "writing code and pushing to GitHub" — no admin UI, no scheduling, no drag-and-drop reordering. Since a full custom admin panel is now required (new post, edit, delete, schedule, reorder), **posts move into Postgres** as the source of truth. This is a deliberate architecture change from the earlier version of this doc.

## High-Level Data Flow

1. **Content authoring:** Abdullah logs into `/admin` (Auth.js session) → writes/edits a post in the Tiptap editor → Server Action validates (`zod`) and writes to the `posts` table with `status: draft | scheduled | published`, plus `scheduled_at` if scheduled.
2. **Publishing (immediate):** On save with `status: published`, the Server Action calls `revalidatePath('/blog/[slug]')` and `revalidatePath('/blog')` — the public page updates within seconds, no redeploy needed.
3. **Publishing (scheduled):** A Vercel Cron job (`vercel.json`, runs every 5–15 min) hits `/api/cron/publish-scheduled` → queries posts where `status = 'scheduled' AND scheduled_at <= now()` → flips them to `published` → calls `revalidatePath` for each.
4. **Reader visits a post:** Server Component fetches the post row from Postgres via Drizzle at request time, rendered with ISR-style caching (`revalidate` tag-based, invalidated on publish/edit — not a fixed time interval anymore, since edits should reflect immediately).
5. **Reader clicks affiliate link:** Routed through `/go/[slug]` Route Handler → logs click (tool slug, timestamp, referring post, anonymized IP hash) to Postgres → 302 redirects to actual affiliate URL.
6. **Reader submits email:** Client component → Server Action → calls ConvertKit API (primary) and optionally logs to DB as backup → confirmation UI.
7. **Sitemap/RSS/JSON-LD:** Generated from the same `posts` table (query at request time or cached), so there is a single source of truth per post — no data duplication between DB and static files.
8. **Reordering (featured/homepage):** Admin drags posts in a list (`dnd-kit`) → Server Action updates an `order` integer column on affected rows → homepage query sorts by `order`.

## Folder Structure (App Router)

```
/app
  /(marketing)
    /page.tsx                  # Homepage
    /about/page.tsx            # Author bio / E-E-A-T page
  /blog
    /page.tsx                  # Post index / listing (DB-backed)
    /[slug]/page.tsx            # Single post (DB-backed, revalidated on publish/edit)
  /go
    /[slug]/route.ts            # Affiliate redirect + click logging (Route Handler)
  /feed.xml
    /route.ts                   # RSS generation
  /sitemap.ts                   # Sitemap generation
  /robots.ts                    # robots.txt generation
  /api
    /newsletter/route.ts         # Fallback API route for email capture (if not using Server Action)
    /cron
      /publish-scheduled/route.ts # Vercel Cron target — publishes due scheduled posts
    /auth/[...nextauth]/route.ts  # Auth.js handler

  /admin                          # ⚠ All routes below protected by middleware (admin-only session)
    /login/page.tsx               # Admin login
    /page.tsx                     # Dashboard (recent posts, subscriber count, click stats)
    /posts
      /page.tsx                    # Post list (filter by status: draft/scheduled/published)
      /new/page.tsx                 # New post editor
      /[id]/edit/page.tsx           # Edit existing post
    /media/page.tsx                # Media library (Vercel Blob uploads)
    /settings/page.tsx             # Affiliate link registry management (optional, later)

  middleware.ts                    # Protects /admin/* — redirects to /admin/login if unauthenticated
  layout.tsx
  globals.css

/components
  /ui                             # Reusable primitives (Button, Card, Badge)
  /blog                           # PostCard, QuickVerdictBox, PricingTable, ProsConsTable, FAQAccordion
  /forms                          # NewsletterForm
  /admin                          # PostEditor (Tiptap wrapper), PostList, StatusBadge, ReorderableList, MediaUploader
  layout                          # Header, Footer, AuthorBioBlock

/lib
  db.ts                           # Drizzle client init
  schema.ts                       # DB schema (posts, clicks, subscribers, admin_users)
  auth.ts                         # Auth.js config
  affiliate-links.ts              # Central registry of affiliate URLs per tool (single source of truth)
  seo.ts                          # Shared metadata/JSON-LD generator functions
  posts.ts                        # Post CRUD query functions (used by both public pages and admin)

/drizzle                          # Migration files
```

## Core DB Schema (posts table — key fields)

| Column | Type | Notes |
|---|---|---|
| `id` | uuid | Primary key |
| `slug` | text, unique | URL slug |
| `title` | text | |
| `content` | text | Markdown/MDX output from Tiptap |
| `status` | enum | `draft`, `scheduled`, `published` |
| `scheduled_at` | timestamp, nullable | Only set when `status = scheduled` |
| `published_at` | timestamp, nullable | Set when it actually goes live |
| `category` | text | Free-form (see taxonomy note below) |
| `tool_name`, `rating`, `affiliate_slug` | text/numeric | Review-specific metadata |
| `seo_title`, `seo_description`, `og_image_url` | text | Per-post SEO overrides |
| `order` | integer | Manual reorder/featured position |
| `created_at`, `updated_at` | timestamp | |

## Content Taxonomy (Built for Expansion, Not Locked to One Niche)

The strategy is to start narrow (Meta Ads tools) for topical authority, then expand into adjacent tool categories over time — the structure below must support that from day one, so no rebuild is needed later:

- The `category` column is free-form text, not a hardcoded enum — adding a new category is just publishing a post with a new value from the admin panel, no code/schema change.
- Post URLs stay category-agnostic (`/blog/[slug]`, not `/blog/meta-ads/[slug]`) — this avoids baking the current niche into every URL, which would break/require redirects when expanding later.
- The blog index page and any future homepage sections can filter/group by `category` once there's more than one — that's a query change against existing data, not a structural one.
- `lib/affiliate-links.ts` and the tool registry are keyed by tool slug, not by category — so a tool review isn't tied to a single niche bucket either.

## Tech Stack Integration Notes

- **Provisioning Neon:** Add it via Vercel Dashboard → Storage/Integrations → Neon Postgres. This auto-injects `DATABASE_URL` (and related env vars) into the project — no manual connection-string copying.
- **Drizzle + Neon:** Connection pooled via `@neondatabase/serverless` driver, compatible with Vercel's serverless/edge functions — avoid long-lived connections.
- **Tiptap editor output:** Store as Markdown (via a serializer, e.g. `tiptap-markdown`) so the public post renderer can reuse the same Markdown/MDX rendering pipeline (`next-mdx-remote/rsc`) regardless of whether content came from the old MDX files or the new editor.
- **Auth.js Credentials provider:** Single admin user row in `admin_users` table, password hashed with `bcrypt`. No public sign-up route exists — the admin account is seeded via a migration/script, not a form.
- **Vercel Cron:** Defined in `vercel.json` (`{"crons": [{"path": "/api/cron/publish-scheduled", "schedule": "*/10 * * * *"}]}`) — protect the route with a shared secret header so it can't be triggered by outside requests.
- **Affiliate link registry:** All affiliate URLs live in one `lib/affiliate-links.ts` file, never hardcoded inside post content — makes updating a changed affiliate URL a one-line change across the whole site.
- **Images:** All screenshots go through `next/image`, uploaded via the admin Media Library to Vercel Blob (not local `/public` — Vercel's filesystem is ephemeral per deploy), never external hotlinked images.
