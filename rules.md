# Rules — Strict Coding Boundaries

These rules are non-negotiable for this project. Any generated code must comply.

## 1. Server Actions vs. API Routes

- **Default to Server Actions** for all form submissions and mutations originating from within the app (newsletter signup, any future contact form). Server Actions are the default; do not create an API Route unless one of the exceptions below applies.
- **Use a Route Handler (`/app/api/...` or `/app/go/...`) only when:**
  - The endpoint must be called by an external system (webhook receiver, e.g., ConvertKit webhook)
  - The response needs a non-HTML content type (RSS XML, sitemap XML, redirect responses)
  - The endpoint needs to be publicly fetchable via GET with cacheable semantics
- Never duplicate the same mutation logic in both a Server Action and an API Route — pick one, and if both are needed, extract shared logic into `/lib` and call it from both.

## 2. Error Handling Protocol

- Every Server Action must return a typed result object: `{ success: boolean; error?: string; data?: T }` — never throw raw errors to the client.
- Every Route Handler must wrap logic in try/catch and return proper HTTP status codes (400 for bad input, 500 for server errors) with a JSON error body — never leak stack traces to the response.
- Use `app/error.tsx` and `app/not-found.tsx` boundaries at the root and blog segment level for graceful UI-level failures.
- Log server-side errors with enough context (route, input params minus PII) — no `console.log` left in for debugging; use a single `lib/logger.ts` wrapper.
- Database write failures (e.g., click logging) must **never** block the affiliate redirect — redirect first (or in parallel), log asynchronously, fail silently with a server-side log if the DB write fails. A broken affiliate link because of a DB hiccup is unacceptable.

## 3. Library Restrictions

- **Allowed:** Next.js, React, Tailwind CSS, Drizzle ORM, `@neondatabase/serverless`, `next-mdx-remote` (for rendering DB-stored Markdown), `zod` (validation), `lucide-react` (icons), `next-auth`/Auth.js (admin authentication only), `bcrypt` (password hashing), `@tiptap/react` + extensions (admin rich text editor), `@vercel/blob` (media uploads), `dnd-kit` (drag-and-drop reordering in admin).
- **Not allowed without explicit approval:** any additional state management library (Redux, Zustand, etc.) — App Router + Server Components should make this unnecessary for this project's scope; any CSS-in-JS library (styled-components, emotion) — Tailwind only; any full third-party headless CMS SDK (Contentful, Sanity, Payload) — the admin panel is custom-built per `architecture.md`, not outsourced to a CMS product.
- All external API calls (ConvertKit, etc.) must be wrapped in a single `/lib/[service].ts` client file — never call `fetch` to a third-party API directly from a component or route handler.

## 4. Admin Panel & Auth Boundaries

- Every route under `/app/admin/**` must be protected by `middleware.ts` checking a valid Auth.js session — never rely on client-side checks alone (no "hide the button but leave the route open").
- The `/api/cron/publish-scheduled` route must validate a shared secret header (e.g. `CRON_SECRET` env var) before doing anything — it is a public URL by nature of being a Route Handler, so it must not trust the caller.
- No public sign-up/registration route exists anywhere in the app. The single admin account is created via a one-off seed script, never through a UI form.
- All admin mutations (create/edit/delete/reorder/schedule post) go through Server Actions that re-verify the session server-side, even though the page itself is already behind middleware — defense in depth, never trust that middleware alone is sufficient for a mutation.
- Passwords are never logged, and `bcrypt` comparison happens server-side only — never send a plaintext password comparison result structure that could leak timing information carelessly (use `bcrypt.compare`, not manual string equality).

## 5. Performance Best Practices

- Blog post pages must be statically generated (SSG) with ISR — no `force-dynamic` on post pages.
- All images through `next/image` with explicit `width`/`height` or `fill` + proper `sizes` — no unoptimized `<img>` tags.
- No client-side data fetching for content that is available at build time — Server Components fetch data directly, no unnecessary `useEffect` + `fetch` patterns.
- Keep Client Components minimal — mark `'use client'` only on components that truly need interactivity (forms, accordions, toggles). Default to Server Components everywhere else.
- Bundle size: no heavy client-side libraries for things achievable in CSS/Tailwind (e.g., no animation library for simple hover states).
- Lighthouse/Core Web Vitals targets: LCP < 2.5s, CLS < 0.1, INP < 200ms — checked before merging any PR that touches layout or images.

## 6. Content/Data Integrity Rules

- Affiliate URLs are never hardcoded in post content — always reference the central `lib/affiliate-links.ts` registry by tool slug.
- Every post is validated against a `zod` schema before being written to the DB (from both the admin Server Action and the cron publish job) — a post missing required fields (title, slug, rating) should be rejected with a clear error, not silently saved broken.
- Publishing (immediate or scheduled) must always go through `revalidatePath`/`revalidateTag` so the public site never serves stale content after an edit.
- No secrets (API keys, DB connection strings, `CRON_SECRET`, auth secrets) in code — `.env.local` only, never committed.
