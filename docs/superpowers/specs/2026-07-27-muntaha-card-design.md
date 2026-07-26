# Muntaha-Card — Personal Portfolio Site

**Date:** 2026-07-27
**Owner:** Sadaqatul Muntaha
**File:** `index.html` (single, self-contained, no build step) — forked from `safwandotcom/safwan-card`
**Deploy:** GitHub repo `safwandotcom/Muntaha-Card`, imported by the user into their existing Vercel project at `sadaqatulmuntaha.xyz`

## Goal

Produce a portfolio site for Sadaqatul Muntaha (Finance & Operations Professional) using
the exact visual system, section rhythm, and mechanics of `safwan-card` — same typography,
palette, vCard button, and blog subsystem — populated with her own content from her
LinkedIn-exported resume PDF.

## Source of truth

- Content: `Profile (1).pdf` (LinkedIn PDF export), supplemented by user-supplied answers
  (photo, phone, accent color, section decisions) in this conversation.
- Template/visual system: `safwandotcom/safwan-card` `index.html` (Archivo type, white/black
  + one electric-blue accent `#1F35FF`, `.hero`/`.xp-item`/`.edu-grid`/`.cert` component
  patterns, client-side vCard-as-Blob download, blog section under `blog/`).

## Decisions (approved)

- **Accent color:** same electric blue `#2e0bfc` as safwan-card (no new color).
- **Palette/type:** identical to safwan-card — true white background, near-black ink,
  Archivo superfamily, no serif/mono.
- **Structure:** Hero (+ sticky vCard button) → About → Experience timeline → Skills →
  Education & Credentials → Blog → Contact. **No "Beyond the desk" section** (dropped
  per user).
- **Photo:** downloaded from user-supplied LinkedIn CDN URL, embedded as a local asset
  (not hot-linked, since the LinkedIn URL is a signed/expiring link).
- **Banner:** no raster banner asset exists in safwan-card — its "banner" is a CSS
  radial-gradient glow in the accent color behind the hero text. Reused as-is, tinted
  with the same blue.

## Content

### Hero
- Name: Sadaqatul Muntaha
- Role line: "Finance & Operations Professional @ Chevron Bangladesh"
- Sub-line: "I turn client relationships, compliance work, and messy operations into
  systems that hold up under scale — from legal records at Chevron to a 40,000-person
  world stage."
- Chips: Dhaka, Bangladesh · BRAC University · IELTS 7.0
- vCard fields: FN Sadaqatul Muntaha · TITLE Finance & Operations Professional ·
  ORG Chevron Bangladesh · TEL +8801626253263 · EMAIL sadaqatulmuntaha80@gmail.com ·
  URL https://www.linkedin.com/in/sadaqatul-muntaha-937207330/

### About
Heading: "Process-driven. People-first."

Body (personal voice, drawn from the resume Summary — the live LinkedIn page could not
be fetched, HTTP 999/anti-bot block, so the PDF export is the source of truth):

> I'm a finance and operations professional who's spent the last four years turning
> client relationships, messy processes, and compliance work into systems that actually
> hold up — at Chevron, in international student admissions, and everywhere in between.
> I'm currently digitizing legal records at Chevron Bangladesh while finishing my BBA in
> Management Information Systems at BRAC University. Off the clock, I coordinated
> operations for 40,000+ participants across 158 countries at the 25th World Scout
> Jamboree — recognized with Bangladesh's President Scouts Award, its highest national
> honor. I'm at my best turning ambiguity into a process, and a process into a result
> someone can measure.

### Experience timeline (reverse-chronological, `.xp-item` pattern)
Chevron is the headline/current entry; all other resume roles follow in full:

1. **Legal Intern**, Chevron, Gulshan — May 2026 – July 2026 (current)
2. **Admission Officer**, TSG Education, Dhaka — Jan 2026 – Mar 2026
3. **Branding & Marketing Executive**, TSG Education, Dhaka — Oct 2024 – Dec 2025
4. **Associate Manager**, marketUP, Dhaka — Feb 2023 – Sep 2023
5. **Junior Engagement Officer**, X - Integrated Marketing Agency, Dhaka — Aug 2022 – Aug 2023
6. **Customer Relationship Officer**, EnglishBoli, Dhaka — Jul 2021 – Dec 2022
7. **Sales Executive**, Tele Musketeers, Dhaka — Feb 2021 – Aug 2021

Bullets carry over from the resume verbatim per role; bold the standout metrics (60%+
conversion, 95%+ on-time, 1000+ records, etc.) the way safwan-card bolds its own metrics.
User has confirmed it's fine to publish these employer names and metrics publicly.

### Skills (4 rows, banking/finance-oriented — mirrors safwan-card's `.skrow` pattern)
- **Finance & Analytics** — SQL, Power BI, Python, Google Analytics, Data Analysis, Advanced Excel
- **Client & Relationship Management** — Account Management, CRM Systems, Consultative Selling, Client Counseling
- **Operations & Compliance** — Process Optimization, Risk & Compliance Management, SOP Development, Project & Budget Management
- **Leadership & Communication** — Team Coordination, Cross-cultural Communication, Public Speaking, Multi-stakeholder Management

### Education & Credentials
- Main degree block: BRAC University — Bachelors in Business Administration,
  Management Information Systems (Dec 2022 – Jul 2025)
- Secondary line: Higher & Secondary School Certificate, Viqarunnisa Noon School and
  College (2016–2020)
- Certs row (`.cert` pattern): IELTS 7.00 · Duke of Edinburgh's International Award ·
  President Scouts Award (Bangladesh's highest national scouting honor)

### Blog
One starter post: **"What 40,000 Scouts Taught Me About Operations"**, in the same
`blog/` subsystem safwan-card uses (`blog/blog.css`, `blog/index.html`,
`blog/posts/*.html`).

### Contact
Same "Let's talk" section/mechanics as safwan-card, pointed at Sadaqatul's email/LinkedIn.

## Non-goals

- No "Beyond the desk" / scouting-lifestyle section (the Jamboree story lives in About
  instead).
- No new accent color, no new type system, no structural redesign beyond swapping
  "repo cards" (safwan's dev-specific pattern folded into his own top timeline entry —
  there was never a separate repo-card section to replace) for Muntaha's own top entry.
- No DNS/Vercel configuration — user already has `sadaqatulmuntaha.xyz` wired to a
  Vercel project and will import the new repo themselves.

## Verification

Browser screenshots at desktop (~1440) and mobile (~390) widths; vCard downloads and
opens correctly with all fields; contrast ≥4.5:1 body; no horizontal scroll; blog post
renders and links back to the index.
