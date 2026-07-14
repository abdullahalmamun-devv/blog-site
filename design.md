# Design Guidelines — iamabdullah.dev

## Brand Direction

The domain (`iamabdullah.dev`) reads as a developer portfolio at first glance — the design must actively counter that and signal "marketer/creator reviewing tools," not "software engineer's personal site." Favor warm, editorial, human tones over sterile dev-blog aesthetics (avoid pure terminal-black/neon-green dev themes).

## Color Palette

| Role | Color | Hex | Usage |
|---|---|---|---|
| Primary | Deep Indigo | `#3730A3` | Headers, primary buttons, links |
| Secondary | Warm Amber | `#F59E0B` | CTAs, "Verdict" badges, highlights |
| Accent | Coral | `#F97066` | Rating stars, alert/warning badges (e.g., "con" markers) |
| Success | Emerald | `#10B981` | "Pro" markers, positive verdict badges |
| Neutral background | Off-white | `#FAFAF9` | Page background (not pure white — softer, editorial feel) |
| Neutral text | Slate | `#1E293B` | Body text |
| Muted text | Slate-500 | `#64748B` | Captions, metadata (dates, read time) |

Tailwind config: extend the default palette with these as `primary`, `secondary`, `accent`, `success` — do not use raw hex values inline in components.

## Typography

- **Headings:** A humanist sans-serif with character — e.g., "Sora" or "Lexend" (via `next/font/google`) — avoid generic system-UI/Inter-only look to prevent the site feeling like a SaaS dashboard.
- **Body:** "Inter" or "Source Sans 3" for readability at long-form review length (1,800–2,500 words per post).
- **Scale:** Use Tailwind's default type scale (`text-sm` through `text-4xl`); H1 `text-4xl font-bold`, H2 `text-2xl font-semibold`, body `text-base leading-relaxed`.
- Line length for body content capped at `max-w-prose` (~65-75 characters) for readability.

## Component Styling Conventions (Tailwind)

- **Utility-first only** — no custom CSS files beyond `globals.css` for font-face and base resets. No CSS modules, no styled-components (per `rules.md`).
- **Spacing scale:** stick to Tailwind's default spacing scale (4px increments) — no arbitrary pixel values unless truly necessary (`p-[13px]` should be rare/justified).
- **Rounded corners:** `rounded-lg` as the default for cards/buttons — consistent, soft, approachable (not sharp/technical).
- **Shadows:** subtle only — `shadow-sm` for cards, avoid heavy drop shadows (keeps the "editorial," not "SaaS dashboard," feel).
- **Buttons:** primary CTA = solid `bg-secondary` (amber) with dark text for contrast and warmth; secondary/ghost buttons = outlined `border-primary text-primary`.

## Key Component Patterns

- **QuickVerdictBox:** Card with colored left border (`border-l-4 border-secondary`), shows tool name, star rating (accent coral stars), one-line verdict, "Best for" tag.
- **ProsConsTable:** Two-column layout, pros in `success`-tinted background chips, cons in `accent`-tinted background chips — never a plain bullet list, needs visual weight to signal honesty/balance.
- **PricingTable:** Simple bordered table, highlight the "best value" plan column with a `bg-secondary/10` tint and a small badge.
- **Author bio block:** Photo (real, not illustration/avatar) + 2-3 line bio + credibility markers (years of experience, ad spend managed) — appears on every post footer, not just the About page, to reinforce E-E-A-T on every page Google crawls.

## Admin Panel Design (Separate from Public Branding)

The public site avoids a "SaaS dashboard" look on purpose (see above) — the admin panel is the opposite: it's a private tool for one person, so **utilitarian and dense is fine here**, prioritize speed of use over brand personality.

- Reuse the same Tailwind config/color tokens for consistency, but it's acceptable to lean on `primary`/neutral grays more heavily and skip the warm editorial styling.
- Data-dense tables (post list) are fine — no need for generous whitespace like the public blog.
- Status badges: `draft` = neutral gray, `scheduled` = amber (`secondary`), `published` = emerald (`success`) — consistent, scannable at a glance in the post list.
- The Tiptap editor toolbar should stay minimal — only the formatting options actually used in review posts (headings, bold, links, tables, images) — no need to expose every possible rich-text feature.
- Mobile responsiveness for `/admin` is a "nice to have," not a hard requirement for MVP — Abdullah will primarily manage content from a desktop.

## What to Avoid

- No dark-mode-only terminal aesthetics, monospace body fonts, or matrix/code-themed visuals **on the public site** — this actively works against the "I'm a marketer, trust my reviews" positioning. (The admin panel is exempt from this — a plain, functional dark or light dashboard theme is fine there.)
- No stock photography of generic "business people in suits" — use real screenshots and a real author photo instead.
- No dense, cluttered dashboard-style layouts on content pages — long-form reading needs generous whitespace.
