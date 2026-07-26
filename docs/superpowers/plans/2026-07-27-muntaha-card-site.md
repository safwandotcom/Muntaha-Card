# Muntaha-Card Site Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `Muntaha-Card`, a single-file static portfolio site for Sadaqatul Muntaha, cloned in visual system from `safwandotcom/safwan-card` and populated with her resume content, then push it to a new public GitHub repo `safwandotcom/Muntaha-Card`.

**Architecture:** One self-contained `index.html` (no build step, no framework) plus a `blog/` subsystem (`blog/index.html`, `blog/blog.css`, `blog/posts/*.html`), copied structurally from `safwan-card` and edited section-by-section. Images (portrait photo, QR code, icons) are embedded as base64 data URIs directly in the HTML, matching safwan-card's existing pattern exactly.

**Tech Stack:** Plain HTML/CSS/vanilla JS. Python 3.12 + Pillow (already installed at `C:/Program Files/Python312/python`) for generating icon/OG-image assets. Python `qrcode` package (already installed) for the QR code. `gh` CLI (already authenticated as `safwandotcom`) for repo creation.

## Global Constraints

- Reuse safwan-card's exact palette verbatim: `--accent:#2e0bfc`, `--accent-ink:#2408c4`, `--accent-soft:#eeeafe`, `--ink:#16161f`, `--ink-2:#454553`, `--muted:#6c6c7b`, background `#fff` (user chose "keep the same electric blue" — do not introduce a new color).
- Reuse safwan-card's exact typography/CSS (the `<style>` block) verbatim — only the content inside `<body>` and the `<head>` meta/JSON-LD text change.
- No "Beyond the desk"/Scouting section — it is dropped per the approved spec at `docs/superpowers/specs/2026-07-27-muntaha-card-design.md`.
- No DNS/Vercel work of any kind — the user imports the repo into their existing `sadaqatulmuntaha.xyz` Vercel project themselves.
- No fabricated content: every fact/number/bullet must trace back to `Profile (1).pdf` or an explicit answer the user gave in conversation. Do not invent employer logos, course lists, or availability claims not supported by the resume.
- Source template lives at `C:/Users/HP/AppData/Local/Temp/claude/E--My-Website/e4374400-85be-464b-9e99-6592bcf9bd8b/scratchpad/safwan-card` (already cloned locally, read-only reference — never edit this directory).
- Target repo lives at `C:/Users/HP/AppData/Local/Temp/claude/E--My-Website/e4374400-85be-464b-9e99-6592bcf9bd8b/scratchpad/Muntaha-Card` (git already initialized, spec doc already committed as `87ddd0a`).
- Muntaha's downloaded photo is at `C:/Users/HP/AppData/Local/Temp/claude/E--My-Website/e4374400-85be-464b-9e99-6592bcf9bd8b/scratchpad/muntaha-photo.jpg` (400×400 JPEG, already verified).

---

### Task 1: Scaffold the repo from safwan-card + generate personalized image assets

**Files:**
- Create: `index.html` (temporary full copy, edited in later tasks)
- Create: `blog/blog.css`, `blog/index.html`, `blog/posts/` (copied, edited in Task 9)
- Create: `favicon.svg`
- Create: `favicon.ico`, `favicon-32.png`, `apple-touch-icon.png`, `icon-192.png`, `icon-512.png`
- Create: `og.png`
- Create: `robots.txt`, `site.webmanifest`, `sitemap.xml`, `README.md` (copied, edited in Task 10)
- Create: `assets/photo_base64.txt`, `assets/qr_base64.txt` (scratch text files, not shipped — used to splice base64 into HTML in later tasks; delete before final push in Task 12)

**Interfaces:**
- Produces: `assets/photo_base64.txt` — single line, the full `data:image/jpeg;base64,...` URI for Muntaha's photo, consumed by Tasks 3, 5, 8.
- Produces: `assets/qr_base64.txt` — single line, the full `data:image/png;base64,...` URI for the QR code, consumed by Task 8.
- Produces: `og.png`, `favicon.ico`, `favicon-32.png`, `apple-touch-icon.png`, `icon-192.png`, `icon-512.png`, `favicon.svg` — referenced by `<link>`/`<meta>` tags added in Task 4.

- [ ] **Step 1: Copy the template files as a starting point**

```bash
SRC="/c/Users/HP/AppData/Local/Temp/claude/E--My-Website/e4374400-85be-464b-9e99-6592bcf9bd8b/scratchpad/safwan-card"
DST="/c/Users/HP/AppData/Local/Temp/claude/E--My-Website/e4374400-85be-464b-9e99-6592bcf9bd8b/scratchpad/Muntaha-Card"
cp "$SRC/index.html" "$DST/index.html"
mkdir -p "$DST/blog/posts" "$DST/assets"
cp "$SRC/blog/blog.css" "$DST/blog/blog.css"
cp "$SRC/robots.txt" "$DST/robots.txt"
cp "$SRC/site.webmanifest" "$DST/site.webmanifest"
cp "$SRC/sitemap.xml" "$DST/sitemap.xml"
cp "$SRC/README.md" "$DST/README.md"
```

- [ ] **Step 2: Generate the "M" monogram favicon.svg** (same style as safwan-card's "S" version, same colors)

Write to `favicon.svg`:

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64">
  <rect width="64" height="64" rx="14" fill="#2e0bfc"/>
  <text x="50%" y="53%" dy=".05em" text-anchor="middle" dominant-baseline="middle"
        font-family="'Segoe UI',Arial,sans-serif" font-weight="800" font-size="40" fill="#fff">M</text>
  <circle cx="45" cy="45" r="4.5" fill="#fff"/>
</svg>
```

- [ ] **Step 3: Generate the raster icon set with Pillow**

Save as `Muntaha-Card/generate_icons.py` and run it, then delete the script (it's a one-off asset generator, not part of the shipped site):

```python
from PIL import Image, ImageDraw, ImageFont

ACCENT = (46, 11, 252, 255)
WHITE = (255, 255, 255, 255)

def make_icon(size, corner_ratio=0.22):
    img = Image.new("RGBA", (size, size), (0, 0, 0, 0))
    d = ImageDraw.Draw(img)
    r = int(size * corner_ratio)
    d.rounded_rectangle([0, 0, size - 1, size - 1], radius=r, fill=ACCENT)
    font_size = int(size * 0.62)
    font = ImageFont.truetype("C:/Windows/Fonts/arialbd.ttf", font_size)
    bbox = d.textbbox((0, 0), "M", font=font)
    tw, th = bbox[2] - bbox[0], bbox[3] - bbox[1]
    d.text(((size - tw) / 2 - bbox[0], (size - th) / 2 - bbox[1]), "M", font=font, fill=WHITE)
    dot_r = max(2, int(size * 0.07))
    cx, cy = int(size * 0.70), int(size * 0.70)
    d.ellipse([cx - dot_r, cy - dot_r, cx + dot_r, cy + dot_r], fill=WHITE)
    return img

sizes = {"favicon-32.png": 32, "apple-touch-icon.png": 180, "icon-192.png": 192, "icon-512.png": 512}
for name, sz in sizes.items():
    make_icon(sz).save(name)

# favicon.ico with multiple embedded sizes, matching safwan-card's 16/32px pattern
make_icon(32).save("favicon.ico", sizes=[(16, 16), (32, 32)])
print("icons written")
```

```bash
"/c/Program Files/Python312/python" generate_icons.py
rm generate_icons.py
```

- [ ] **Step 4: Verify the icon set**

```bash
"/c/Program Files/Python312/python" -c "
from PIL import Image
import os
for f in ['favicon-32.png','apple-touch-icon.png','icon-192.png','icon-512.png']:
    im = Image.open(f); print(f, im.size, im.mode)
print('favicon.ico', os.path.getsize('favicon.ico'), 'bytes')
"
```
Expected: sizes print as `(32, 32) RGBA`, `(180, 180) RGBA`, `(192, 192) RGBA`, `(512, 512) RGBA`, and `favicon.ico` size > 0.

- [ ] **Step 5: Generate `og.png`** (1200×630, same layout as safwan-card's: blue left bar, blue uppercase eyebrow role, bold black name, gray sub-line, bold domain + blue dot)

Save as `Muntaha-Card/generate_og.py`, run, then delete:

```python
from PIL import Image, ImageDraw, ImageFont

W, H = 1200, 630
ACCENT = (46, 11, 252)
INK = (22, 22, 31)
MUTED = (76, 76, 90)

img = Image.new("RGB", (W, H), "white")
d = ImageDraw.Draw(img)
d.rectangle([0, 0, 14, H], fill=ACCENT)

eyebrow_font = ImageFont.truetype("C:/Windows/Fonts/arialbd.ttf", 28)
name_font = ImageFont.truetype("C:/Windows/Fonts/arialbd.ttf", 78)
sub_font = ImageFont.truetype("C:/Windows/Fonts/arial.ttf", 32)
domain_font = ImageFont.truetype("C:/Windows/Fonts/arialbd.ttf", 34)

d.text((96, 118), "FINANCE & OPERATIONS PROFESSIONAL", font=eyebrow_font, fill=ACCENT)
d.text((94, 170), "Sadaqatul", font=name_font, fill=INK)
d.text((94, 260), "Muntaha", font=name_font, fill=INK)
d.text((96, 410), "I turn client relationships and messy operations into", font=sub_font, fill=MUTED)
d.text((96, 450), "systems that hold up under scale.", font=sub_font, fill=MUTED)
d.text((96, 510), "sadaqatulmuntaha.xyz", font=domain_font, fill=INK)
bbox = d.textbbox((96, 510), "sadaqatulmuntaha.xyz", font=domain_font)
d.ellipse([bbox[2] + 12, bbox[3] - 14, bbox[2] + 26, bbox[3]], fill=ACCENT)

img.save("og.png")
print("og.png written", img.size)
```

```bash
"/c/Program Files/Python312/python" generate_og.py
rm generate_og.py
```

- [ ] **Step 6: Prepare the photo base64 asset file**

```bash
"/c/Program Files/Python312/python" -c "
import base64
data = open('../muntaha-photo.jpg','rb').read()
b64 = base64.b64encode(data).decode()
open('assets/photo_base64.txt','w').write('data:image/jpeg;base64,' + b64)
print('photo_base64.txt written, length', len(b64))
"
```
Expected: prints a length > 0 (the file should be ~55,000 characters for a 41,719-byte JPEG).

- [ ] **Step 7: Generate the QR code (encodes the live site URL) and its base64 asset file**

```bash
"/c/Program Files/Python312/python" -c "
import qrcode, base64, io
img = qrcode.make('https://sadaqatulmuntaha.xyz/', box_size=8, border=2)
buf = io.BytesIO()
img.save(buf, format='PNG')
b64 = base64.b64encode(buf.getvalue()).decode()
open('assets/qr_base64.txt','w').write('data:image/png;base64,' + b64)
print('qr_base64.txt written, length', len(b64))
"
```
Expected: prints a length > 0.

- [ ] **Step 8: Commit the scaffold**

```bash
git add index.html blog favicon.svg favicon.ico favicon-32.png apple-touch-icon.png icon-192.png icon-512.png og.png robots.txt site.webmanifest sitemap.xml README.md
git commit -m "Scaffold Muntaha-Card from safwan-card template + generate M-monogram icons, OG image, QR code"
```
(`assets/*.txt` are intentionally left untracked — they're scratch files for later tasks, deleted in Task 12.)

---

### Task 2: Rewrite the `<head>` meta/SEO block

**Files:**
- Modify: `index.html` (the `<head>...</head>` region, lines 1–~75 of the copied file — everything before the `<style>` block)

**Interfaces:**
- Consumes: nothing from other tasks.
- Produces: page `<title>`, meta description, canonical URL, OG/Twitter tags, JSON-LD — read by nothing programmatically, but must stay consistent with content added in Tasks 4–8 (same name, same role string, same domain).

- [ ] **Step 1: Replace the identity meta tags**

Using the Edit tool, replace every occurrence of safwan's identity strings in the `<head>` with Muntaha's:

| Find | Replace with |
|---|---|
| `Mohammed Safwanul Islam — AI & Product Engineer` (title/og:title/twitter:title) | `Sadaqatul Muntaha — Finance & Operations Professional` |
| `Mohammed Safwanul Islam — AI & Product Engineer at PreCognise. I build software people actually rely on.` (description/og:description/twitter meta) | `Sadaqatul Muntaha — Finance & Operations Professional at Chevron Bangladesh. I turn client relationships and messy operations into systems that hold up under scale.` |
| `Mohammed Safwanul Islam` (author, og:site_name) | `Sadaqatul Muntaha` |
| `Mohammed Safwanul Islam, Safwan, Safwanul, AI Engineer, Product Engineer, PreCognise, full-stack developer, Next.js, machine learning` (keywords) | `Sadaqatul Muntaha, Muntaha, Finance Professional, Operations Manager, Client Relationship Management, BRAC University, Chevron Bangladesh` |
| `https://safwandotcom.xyz/` (canonical, og:url, all absolute URLs) | `https://sadaqatulmuntaha.xyz/` |
| `AI & Product Engineer at PreCognise. I build software people actually rely on.` (og:image:alt) | `Sadaqatul Muntaha — Finance & Operations Professional` |

Leave `<meta name="theme-color" content="#2e0bfc">` and the `og:image:width`/`og:image:height` (1200/630) unchanged.

- [ ] **Step 2: Replace the JSON-LD Person schema**

Find the `<script type="application/ld+json">` block containing `"@type": "Person"` and replace its `name`, `url`, `jobTitle`, `worksFor`, `email`, `telephone`, `sameAs` fields:

```json
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Sadaqatul Muntaha",
  "url": "https://sadaqatulmuntaha.xyz/",
  "jobTitle": "Finance & Operations Professional",
  "worksFor": { "@type": "Organization", "name": "Chevron Bangladesh" },
  "email": "sadaqatulmuntaha80@gmail.com",
  "telephone": "+8801626253263",
  "sameAs": ["https://www.linkedin.com/in/sadaqatul-muntaha-937207330/"]
}
```
(Keep whatever second JSON-LD block exists, e.g. a `WebSite` schema, updating its `url`/`name` fields the same way.)

- [ ] **Step 3: Verify no leftover safwan strings remain in the head**

```bash
grep -n "Safwanul\|PreCognise\|safwandotcom" index.html | grep -v "^[0-9]*:</style>" | head -20
```
Expected at this point: only matches inside the not-yet-edited `<body>` (Tasks 3–8 will remove those); zero matches within the `<head>` block (lines 1–~75). Manually confirm the reported line numbers for any match are ≥ the line where `<style>` begins.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Rewrite index.html head/meta/JSON-LD for Sadaqatul Muntaha"
```

---

### Task 3: Rewrite the masthead + hero section

**Files:**
- Modify: `index.html` — the `<header class="masthead">...</header>` block and the `<section class="hero">...</section>` block (immediately following `<main id="top">`)

**Interfaces:**
- Consumes: `assets/photo_base64.txt` (from Task 1) for the hero portrait `<img>` and the `save-btn` avatar `<img>`.
- Produces: nothing consumed elsewhere, but the vCard JS in Task 8 duplicates this same photo base64 independently via its own `PORTRAIT_DATA_URI` constant.

- [ ] **Step 1: Replace the masthead**

Replace:
```html
<a href="#top" class="brand">Safwanul<b>.</b></a>
```
with:
```html
<a href="#top" class="brand">Muntaha<b>.</b></a>
```

Replace the `save-btn`'s `aria-label` and visible label text:
```html
<button class="save-btn" type="button" data-vcard aria-label="Save Sadaqatul Muntaha to your contacts">
  <img class="save-av" src="PASTE_PHOTO_BASE64_HERE" alt="">
  <span class="save-tx"><span class="full">Save my card</span><span class="mini">Save</span></span>
</button>
```
Replace `PASTE_PHOTO_BASE64_HERE` with the exact single-line content of `assets/photo_base64.txt` (use the Read tool to get the file's content, then Edit to splice it in — do not retype it by hand).

Nav links (`#about`, `#experience`, `#skills`, `#education`, `blog/index.html`, `#contact`) stay unchanged — the target section IDs are preserved throughout this plan.

- [ ] **Step 2: Replace the hero content**

```html
<div class="hero-main reveal">
  <span class="avail"><span class="dot"></span>Open to Finance & Operations roles · Dhaka-based</span>
  <h1>Sadaqatul Muntaha</h1>
  <div class="hero-role">
    Finance &amp; Operations Professional
    <span class="at">at</span>
    Chevron Bangladesh
  </div>
  <p class="hero-sub">I turn client relationships, compliance work, and messy operations into systems that hold up under scale — from legal records at Chevron to a 40,000-person world stage.</p>
  <div class="hero-chips">
    <span class="chip"><b>BBA</b> Management Information Systems · BRAC University</span>
    <span class="chip">Dhaka, Bangladesh</span>
    <span class="chip"><b>IELTS</b> 7.0</span>
  </div>
  <div class="hero-actions">
    <button class="btn btn-solid" type="button" data-vcard>Save my card to your phone</button>
    <a class="btn btn-ghost" href="#experience">See what I've done</a>
  </div>
</div>
<div class="hero-portrait reveal">
  <figure>
    <img src="PASTE_PHOTO_BASE64_HERE" alt="Sadaqatul Muntaha">
    <span class="badge"><span class="d"></span>Open to opportunities</span>
  </figure>
</div>
```
Replace both `PASTE_PHOTO_BASE64_HERE` placeholders with the exact content of `assets/photo_base64.txt` (same value as Step 1 — one photo, embedded twice, matching safwan-card's own duplication pattern).

Note: safwan-card's hero-role includes an inline employer-logo `<img class="pc-logo-img">` — omit that entirely for Muntaha (no Chevron logo asset, and using a third-party corporate logo without one isn't appropriate); the plain text "at Chevron Bangladesh" replaces it.

- [ ] **Step 3: Verify in a browser**

Open `index.html` directly in a browser (`start index.html` on Windows, or via the `run` skill) and confirm:
- The nav bar shows "Muntaha." and a small round avatar next to "Save my card".
- The hero shows Sadaqatul Muntaha's name, role line, photo, and three chips.
- No broken image icon (confirms the base64 splice was well-formed).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add Sadaqatul Muntaha's masthead and hero content"
```

---

### Task 4: Rewrite the About section

**Files:**
- Modify: `index.html` — `<section class="sec" id="about">...</section>`

**Interfaces:**
- Consumes: nothing.
- Produces: nothing consumed elsewhere.

- [ ] **Step 1: Replace the section content**

```html
<section class="sec" id="about">
  <div class="wrap">
    <div class="sec-head reveal"><h2>Process-driven.<br>People-first.</h2></div>
    <div class="about-body reveal">
      <div>
        <p>I'm a <b>finance and operations professional</b> who turns messy processes into ones that hold up.</p>
        <p>I'm a finance and operations professional who's spent the last four years turning client relationships, messy processes, and compliance work into systems that actually hold up — at Chevron, in international student admissions, and everywhere in between. I'm currently digitizing legal records at Chevron Bangladesh while finishing my BBA in Management Information Systems at BRAC University. Off the clock, I coordinated operations for 40,000+ participants across 158 countries at the 25th World Scout Jamboree — recognized with Bangladesh's President Scouts Award, its highest national honor. I'm at my best turning ambiguity into a process, and a process into a result someone can measure.</p>
      </div>
      <dl class="about-facts">
        <div class="fact"><dt>Now</dt><dd>Legal Intern, Chevron Bangladesh</dd></div>
        <div class="fact"><dt>Based</dt><dd>Dhaka, Bangladesh</dd></div>
        <div class="fact"><dt>Studied</dt><dd>BBA, Management Information Systems · BRAC University</dd></div>
        <div class="fact"><dt>Languages</dt><dd>English · Bengali</dd></div>
        <div class="fact"><dt>Focus</dt><dd>Banking &amp; Finance</dd></div>
      </dl>
    </div>
  </div>
</section>
```

- [ ] **Step 2: Verify**

```bash
grep -n "Process-driven\|President Scouts Award\|BBA, Management Information Systems" index.html
```
Expected: 3+ matches, all within the `id="about"` section.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add About section content"
```

---

### Task 5: Rewrite the Experience timeline

**Files:**
- Modify: `index.html` — `<section class="sec" id="experience">...</section>` (the `.xp` list of `.xp-item` blocks)

**Interfaces:**
- Consumes: nothing.
- Produces: nothing consumed elsewhere.

- [ ] **Step 1: Replace the section lede and all `.xp-item` entries**

```html
<div class="sec-head reveal"><h2>What I've done</h2></div>
<p class="sec-lede">From legal ops to marketing pipelines — growing responsibility, and results you can measure.</p>
</div>
<div class="xp reveal">

  <div class="xp-item">
    <div class="xp-meta">
      <div class="role">Legal Intern</div>
      <div class="co">Chevron</div>
      <div class="when">May 2026 — Present</div>
      <div class="where">Gulshan, Dhaka</div>
    </div>
    <div class="xp-body">
      <ul>
        <li>Digitized <b>1000+ corporate legal records</b> through high-volume scanning &amp; metadata tagging, optimizing document retrieval and ensuring regulatory compliance.</li>
        <li>Enforced a strict <b>6-field naming convention</b>, reducing data inconsistencies and ensuring accuracy in legal documentation and audit readiness.</li>
        <li>Optimized the document archiving pipeline bridging hardcopy and digital systems, reducing manual data-entry errors and improving operational efficiency.</li>
        <li>Managed data synchronization for legal registers, tracking contract statuses to support seamless information retrieval and compliance requirements.</li>
      </ul>
      <div class="xp-tags"><span>Compliance</span><span>Legal Ops</span><span>Document Digitization</span><span>Data Governance</span></div>
    </div>
  </div>

  <div class="xp-item">
    <div class="xp-meta">
      <div class="role">Admission Officer</div>
      <div class="co">TSG Education</div>
      <div class="when">Jan 2026 — Mar 2026</div>
      <div class="where">Dhaka, Bangladesh</div>
    </div>
    <div class="xp-body"><ul>
      <li>Managed the admissions pipeline for <b>50+ international clients</b> across 5 destinations, converting <b>60%+</b> of qualified leads into enrollments through strategic relationship building.</li>
      <li>Coordinated application processing and verification for 40+ clients quarterly, ensuring <b>95%+ on-time</b> submissions and regulatory compliance.</li>
      <li>Established strategic relationships with 15+ partner institutions, securing 10+ placements monthly and managing multi-stakeholder communications.</li>
      <li>Executed targeted campaigns via social media and events, generating <b>200+ qualified leads</b> monthly through data-driven marketing strategies.</li>
    </ul></div>
  </div>

  <div class="xp-item">
    <div class="xp-meta">
      <div class="role">Branding &amp; Marketing Executive</div>
      <div class="co">TSG Education</div>
      <div class="when">Oct 2024 — Dec 2025</div>
      <div class="where">Dhaka, Bangladesh</div>
    </div>
    <div class="xp-body"><ul>
      <li>Led digital marketing strategy across 5+ platforms (Social Media, Google Ads, CRM), analyzing data to optimize lead generation and conversion metrics.</li>
      <li>Managed monthly marketing budgets with ROI tracking, ensuring efficient capital allocation and measurable campaign performance.</li>
      <li>Executed <b>15+ campaigns monthly</b>, monitoring performance through analytics and KPI dashboards, achieving consistent engagement growth.</li>
      <li>Utilized JIRA task management and CRM systems for workflow optimization, improving team efficiency and project delivery timelines.</li>
      <li>Provided client counseling for international student enrollments, demonstrating strong relationship management and advisory capabilities.</li>
    </ul></div>
  </div>

  <div class="xp-item">
    <div class="xp-meta">
      <div class="role">Associate Manager</div>
      <div class="co">marketUP.</div>
      <div class="when">Feb 2023 — Sep 2023</div>
      <div class="where">Dhaka, Bangladesh</div>
    </div>
    <div class="xp-body"><ul>
      <li>Managed digital marketing campaigns for multiple clients, coordinating team workflows and ensuring delivery of quality outputs within timelines.</li>
      <li>Oversaw campaign execution and team performance, implementing quality-control measures and optimizing operational processes.</li>
      <li>Gathered client requirements, provided strategic recommendations, and implemented customized solutions to drive business growth.</li>
    </ul></div>
  </div>

  <div class="xp-item">
    <div class="xp-meta">
      <div class="role">Junior Engagement Officer</div>
      <div class="co">X - Integrated Marketing Agency</div>
      <div class="when">Aug 2022 — Aug 2023</div>
      <div class="where">Dhaka, Bangladesh</div>
    </div>
    <div class="xp-body"><ul>
      <li>Managed client accounts end-to-end, supporting campaign execution and ensuring <b>95%+ client satisfaction</b> through responsive communication and problem resolution.</li>
      <li>Monitored campaign performance metrics using Google Analytics and reporting dashboards, providing data-driven insights for decision-making.</li>
      <li>Developed Standard Operating Procedures (SOPs) for client engagement workflows, improving team efficiency and process standardization.</li>
      <li>Coordinated cross-functional teams to deliver projects on schedule, demonstrating project management and operational coordination skills.</li>
    </ul></div>
  </div>

  <div class="xp-item">
    <div class="xp-meta">
      <div class="role">Customer Relationship Officer</div>
      <div class="co">EnglishBoli</div>
      <div class="when">Jul 2021 — Dec 2022</div>
      <div class="where">Dhaka, Bangladesh</div>
    </div>
    <div class="xp-body"><ul>
      <li>Managed customer inquiries and resolved issues through email and chat support, maintaining high satisfaction levels and retention rates.</li>
      <li>Converted prospective customers into active enrollments through consultative selling and relationship building.</li>
      <li>Coordinated with stakeholders to ensure a seamless customer experience and achieved enrollment targets.</li>
    </ul></div>
  </div>

  <div class="xp-item">
    <div class="xp-meta">
      <div class="role">Sales Executive</div>
      <div class="co">Tele Musketeers</div>
      <div class="when">Feb 2021 — Aug 2021</div>
      <div class="where">Dhaka, Bangladesh</div>
    </div>
    <div class="xp-body"><ul>
      <li>Achieved monthly sales targets through strategic lead management and client acquisition initiatives.</li>
      <li>Managed the sales pipeline, maintaining client relationships and closing deals to meet performance metrics.</li>
      <li>Demonstrated the ability to identify market opportunities and convert prospects into revenue-generating customers.</li>
    </ul></div>
  </div>

</div>
```

Note: this replaces the entire contents between `<div class="sec-head reveal"><h2>What I've done</h2></div>` and the `</div>` that closes `.xp` — use the Edit tool with the full old block (from `<h2>What I've done</h2>` through the last `.xp-item` in safwan-card's version) as `old_string` and the block above as `new_string`.

- [ ] **Step 2: Verify all 7 roles are present and safwan's are gone**

```bash
grep -c "xp-item" index.html
grep -n "PreCognise\|Grand Homes Venture\|KidoCode\|UCSI Group\|JobStreet\|Showaround" index.html
```
Expected: `xp-item` count = 7; second command returns no matches inside the experience section (it's fine if `PreCognise`/etc. still appear elsewhere in the file — later tasks or the head clean this up; but the *experience* section itself should have zero).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add Experience timeline (Chevron, TSG Education, marketUP, and 4 more roles)"
```

---

### Task 6: Rewrite the Skills section

**Files:**
- Modify: `index.html` — `<section class="sec" id="skills">...</section>` (the `.skrow` rows)

**Interfaces:**
- Consumes: nothing.
- Produces: nothing consumed elsewhere.

- [ ] **Step 1: Replace the skills rows**

```html
<div class="sec-head reveal"><h2>What I work with</h2></div>
<div class="skills reveal">
  <div class="skrow"><h3>Finance &amp; Analytics</h3><div class="tags"><span>SQL</span><span>Power BI</span><span>Python</span><span>Google Analytics</span><span>Data Analysis</span><span>Advanced Excel</span></div></div>
  <div class="skrow"><h3>Client &amp; Relationship Management</h3><div class="tags"><span>Account Management</span><span>CRM Systems</span><span>Consultative Selling</span><span>Client Counseling</span></div></div>
  <div class="skrow"><h3>Operations &amp; Compliance</h3><div class="tags"><span>Process Optimization</span><span>Risk &amp; Compliance Management</span><span>SOP Development</span><span>Project &amp; Budget Management</span></div></div>
  <div class="skrow"><h3>Leadership &amp; Communication</h3><div class="tags"><span>Team Coordination</span><span>Cross-cultural Communication</span><span>Public Speaking</span><span>Multi-stakeholder Management</span></div></div>
</div>
```

- [ ] **Step 2: Verify**

```bash
grep -c "skrow" index.html
```
Expected: 4.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add Skills section (finance/banking-oriented)"
```

---

### Task 7: Rewrite Education & Credentials, and remove the Scouting section

**Files:**
- Modify: `index.html` — `<section class="sec" id="education">...</section>` (the `.edu-grid`), and delete the entire `<section class="sec" id="scouting">...</section>` block that follows it

**Interfaces:**
- Consumes: nothing.
- Produces: nothing consumed elsewhere. Deleting the Scouting section means the masthead nav (Task 3) correctly has no `#scouting` link — confirm it never had one (safwan-card's nav doesn't link to `#scouting` either, so no nav change is needed here).

- [ ] **Step 1: Replace the edu-grid**

```html
<div class="sec-head reveal"><h2>Education &amp; credentials</h2></div>
<div class="edu-grid reveal">
  <div class="degree">
    <h3>Bachelor of Business Administration — Management Information Systems</h3>
    <div class="school">BRAC University · Dhaka, Bangladesh</div>
    <div class="dmeta">Dec 2022 — Jul 2025</div>
  </div>
  <div class="degree">
    <h3>Higher &amp; Secondary School Certificate</h3>
    <div class="school">Viqarunnisa Noon School and College</div>
    <div class="dmeta">2016 — 2020</div>
  </div>
  <div class="certs">
    <div class="cert"><span class="ci">7.0</span><span><span class="ct">IELTS — English Proficiency</span><br><span class="cs">British Council / IDP</span></span></div>
    <div class="cert"><span class="ci">DofE</span><span><span class="ct">International Award</span><br><span class="cs">Duke of Edinburgh's Award</span></span></div>
    <div class="cert"><span class="ci">⚜</span><span><span class="ct">President Scouts Award</span><br><span class="cs">Bangladesh Scouts — highest national honor</span></span></div>
  </div>
</div>
```

- [ ] **Step 2: Delete the Scouting section entirely**

Using the Edit tool, remove the whole block, from the `<!-- ============ SCOUTING ============ -->` comment through its closing `</section>`, leaving the `<!-- ============ WRITING ============ -->` comment (or equivalent "Latest writing" section) as the very next thing after Education's closing `</section>`.

- [ ] **Step 3: Verify**

```bash
grep -n "id=\"scouting\"\|Beyond the desk\|President's Scout Award\|Institute of Engineers Malaysia" index.html
grep -n "President Scouts Award\|IELTS" index.html
```
Expected: first command returns no matches (old scouting section and safwan's own cert list gone); second command returns matches inside `id="education"`.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add Education & Credentials, remove Scouting section"
```

---

### Task 8: Rewrite the Writing preview, Contact/vCard section, and footer

**Files:**
- Modify: `index.html` — `<section class="sec" id="writing">...</section>`, `<section class="contact" id="contact">...</section>`, `<footer>...</footer>`, and the `<script>` block containing `buildVCard()`

**Interfaces:**
- Consumes: `assets/photo_base64.txt` (Task 1) for the `dcard-av` image and the `PORTRAIT_DATA_URI` JS constant; `assets/qr_base64.txt` (Task 1) for the `.qr` image.
- Produces: nothing consumed elsewhere.

- [ ] **Step 1: Replace the Writing preview block**

```html
<div class="sec-head reveal" style="display:flex;justify-content:space-between;align-items:flex-end;gap:1rem;max-width:none">
  <h2>Latest writing</h2>
  <a href="blog/index.html" style="font-size:.85rem;font-weight:600;color:var(--accent);white-space:nowrap">All writing →</a>
</div>
<div class="reveal" style="border-top:1px solid var(--line)">
  <a href="blog/posts/what-40000-scouts-taught-me-about-operations.html" style="display:block;padding-block:clamp(1.4rem,3vw,1.9rem);border-bottom:1px solid var(--line);transition:background .18s var(--e)" onmouseover="this.style.background='var(--accent-soft)'" onmouseout="this.style.background='transparent'">
    <div style="display:flex;align-items:center;gap:.7rem;font-size:.8rem;color:var(--muted);margin-bottom:.5rem">
      <time datetime="2026-07-27">Jul 27, 2026</time>
      <span style="width:3px;height:3px;border-radius:50%;background:var(--line-2)"></span>
      <span>2 min read</span>
    </div>
    <h3 style="font-size:clamp(1.2rem,1rem + 1vw,1.55rem)">What 40,000 Scouts Taught Me About Operations</h3>
    <p style="color:var(--ink-2);margin-top:.4rem;max-width:60ch">Coordinating logistics for 40,000+ people across 158 countries taught me more about operations than any job did.</p>
  </a>
</div>
```

- [ ] **Step 2: Replace the Contact/dcard section**

```html
<div class="contact-head reveal">
  <h2>Let's talk.</h2>
  <p>Hiring for a finance or operations role, or just curious — save my card in one tap, or point your phone camera at the code.</p>
</div>
<div class="contact-grid">
  <div class="dcard reveal" id="dcard">
    <div class="sheen"></div>
    <div class="dcard-top">
      <img class="dcard-av" src="PASTE_PHOTO_BASE64_HERE" alt="">
      <div>
        <div class="dcard-name">Sadaqatul Muntaha</div>
        <div class="dcard-role">Finance &amp; Operations Professional · Chevron Bangladesh</div>
      </div>
    </div>
    <div class="dcard-links">
      <a href="mailto:sadaqatulmuntaha80@gmail.com"><span class="lbl">Email</span><span class="val">sadaqatulmuntaha80@gmail.com</span></a>
      <a href="tel:+8801626253263"><span class="lbl">Phone</span><span class="val">+880 1626 253263</span></a>
      <a href="https://www.linkedin.com/in/sadaqatul-muntaha-937207330/" target="_blank" rel="noopener"><span class="lbl">LinkedIn</span><span class="val">in/sadaqatul-muntaha</span></a>
    </div>
    <button class="btn btn-solid" type="button" data-vcard>＋ Save my card to your phone</button>
  </div>
  <div class="scan reveal">
    <span class="scan-badge">Point &amp; save</span>
    <img class="qr" src="PASTE_QR_BASE64_HERE" alt="QR code linking to sadaqatulmuntaha.xyz">
    <div class="scan-t">Scan to visit</div>
    <div class="scan-s">Point your phone camera here to open my site.</div>
  </div>
</div>
```
Note: safwan-card's dcard has an inline employer-logo `<img class="pc-logo-img">` next to the role text — omit it here (same reasoning as Task 3: no Chevron logo asset).

Replace `PASTE_PHOTO_BASE64_HERE` with the content of `assets/photo_base64.txt`, and `PASTE_QR_BASE64_HERE` with the content of `assets/qr_base64.txt`.

- [ ] **Step 3: Replace the footer**

```html
<footer>
  <div class="wrap">
    <span>© 2026 Sadaqatul Muntaha · Dhaka, Bangladesh</span>
    <span>Made with care · <a href="https://www.linkedin.com/in/sadaqatul-muntaha-937207330/" target="_blank" rel="noopener">linkedin.com/in/sadaqatul-muntaha</a></span>
  </div>
</footer>
```

- [ ] **Step 4: Replace the vCard-building script**

Find the `<script>` block containing `PORTRAIT_DATA_URI` and `buildVCard()`. Replace the constant and the VCARD field array:

```js
const PORTRAIT_DATA_URI = "PASTE_PHOTO_BASE64_HERE";

function buildVCard(){
  const b64=(PORTRAIT_DATA_URI.split(",")[1]||"");
  const head="PHOTO;ENCODING=b;TYPE=JPEG:";
  const photo=[];
  if(b64){photo.push(head+b64.slice(0,75-head.length));let r=b64.slice(75-head.length);while(r.length){photo.push(" "+r.slice(0,74));r=r.slice(74);}}
  return ["BEGIN:VCARD","VERSION:3.0","N:Muntaha;Sadaqatul;;;","FN:Sadaqatul Muntaha",
    "TITLE:Finance & Operations Professional","ORG:Chevron Bangladesh","EMAIL;TYPE=INTERNET,HOME:sadaqatulmuntaha80@gmail.com",
    "TEL;TYPE=CELL:+8801626253263","URL:https://sadaqatulmuntaha.xyz",
    "X-SOCIALPROFILE;TYPE=linkedin:https://www.linkedin.com/in/sadaqatul-muntaha-937207330/",
    "URL;TYPE=linkedin:https://www.linkedin.com/in/sadaqatul-muntaha-937207330/","ADR;TYPE=HOME:;;;Dhaka;;;Bangladesh",
    ...photo,"END:VCARD"].join("\r\n");
}
```

Replace `PASTE_PHOTO_BASE64_HERE` with the content of `assets/photo_base64.txt` (third embedding of the same photo — matches safwan-card's own duplication of hero + save-btn + JS constant).

Further down in the same script, find:
```js
const a=document.createElement("a");a.href=url;a.download="Mohammed-Safwanul-Islam.vcf";
```
and
```js
}catch(e){window.location.href="mailto:safwan24bd@gmail.com";}
```
Replace with:
```js
const a=document.createElement("a");a.href=url;a.download="Sadaqatul-Muntaha.vcf";
```
```js
}catch(e){window.location.href="mailto:sadaqatulmuntaha80@gmail.com";}
```

- [ ] **Step 5: Verify no safwan strings remain anywhere in the file**

```bash
grep -in "safwan\|precognise\|Mohammed" index.html
```
Expected: no output at all.

```bash
grep -c "data:image" index.html
```
Expected: at least 3 (photo × 3 embeddings + QR × 1, plus any small inline logo/icon references already in the CSS, so ≥4 is also fine — just confirm it's not 0, which would mean a splice failed).

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Add Writing preview, Contact/vCard section, and footer for Sadaqatul Muntaha"
```

---

### Task 9: Build the blog subsystem

**Files:**
- Modify: `blog/index.html` (already copied in Task 1)
- Create: `blog/posts/what-40000-scouts-taught-me-about-operations.html`
- `blog/blog.css` stays as copied — no personalized content in it (verify in Step 1).

**Interfaces:**
- Consumes: nothing.
- Produces: `blog/posts/what-40000-scouts-taught-me-about-operations.html`, linked from `index.html` (Task 8) and `blog/index.html` (this task).

- [ ] **Step 1: Confirm blog.css has no personal content to change**

```bash
grep -in "safwan\|precognise\|mohammed" blog/blog.css
```
Expected: no output (it's pure CSS with no names in it — if this returns matches, edit them out the same way as other tasks before continuing).

- [ ] **Step 2: Rewrite `blog/index.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Writing — Sadaqatul Muntaha</title>
<meta name="description" content="Notes and essays by Sadaqatul Muntaha on operations, client relationships, and leadership.">
<meta name="author" content="Sadaqatul Muntaha">
<meta name="robots" content="index,follow,max-image-preview:large">
<link rel="canonical" href="https://sadaqatulmuntaha.xyz/blog/">
<meta name="theme-color" content="#2e0bfc">
<link rel="icon" href="/favicon.ico" sizes="any">
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">
<link rel="manifest" href="/site.webmanifest">
<meta property="og:type" content="website">
<meta property="og:site_name" content="Sadaqatul Muntaha">
<meta property="og:title" content="Writing — Sadaqatul Muntaha">
<meta property="og:description" content="Notes and essays on operations, client relationships, and leadership.">
<meta property="og:url" content="https://sadaqatulmuntaha.xyz/blog/">
<meta property="og:image" content="https://sadaqatulmuntaha.xyz/og.png">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:image" content="https://sadaqatulmuntaha.xyz/og.png">
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Blog",
  "name": "Writing — Sadaqatul Muntaha",
  "url": "https://sadaqatulmuntaha.xyz/blog/",
  "author": { "@type": "Person", "name": "Sadaqatul Muntaha", "url": "https://sadaqatulmuntaha.xyz/" }
}
</script>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Bricolage+Grotesque:opsz,wght@12..48,400..800&family=Geist:wght@300..700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="blog.css">
</head>
<body>

<header class="masthead">
  <div class="masthead-in">
    <a href="../index.html" class="brand">Muntaha<b>.</b></a>
    <nav class="nav" aria-label="Primary">
      <a href="../index.html#about">About</a>
      <a href="../index.html#experience">Experience</a>
      <a href="index.html" class="here">Writing</a>
      <a href="../index.html#contact">Contact</a>
    </nav>
  </div>
</header>

<main>
  <section class="blog-hero wrap">
    <p class="eyebrow">Writing</p>
    <h1>Thoughts, in progress.</h1>
    <p>Notes on operations, client relationships, and the kind of leadership that shows up in the details. No schedule — just when something's worth writing down.</p>
  </section>

  <section class="wrap">
    <div class="post-list">

      <a class="post-row" href="posts/what-40000-scouts-taught-me-about-operations.html">
        <div class="post-meta">
          <time datetime="2026-07-27">Jul 27, 2026</time>
          <span class="dot"></span>
          <span>2 min read</span>
        </div>
        <h2 class="post-title">What 40,000 Scouts Taught Me About Operations</h2>
        <p class="post-excerpt">Coordinating logistics for 40,000+ people across 158 countries taught me more about operations than any job did.</p>
        <span class="more">Read →</span>
      </a>

    </div>
  </section>
</main>

<footer class="blog-footer">
  <div class="wrap">
    <span>© 2026 Sadaqatul Muntaha</span>
    <a href="../index.html">← Back to portfolio</a>
  </div>
</footer>

</body>
</html>
```

- [ ] **Step 3: Write `blog/posts/what-40000-scouts-taught-me-about-operations.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>What 40,000 Scouts Taught Me About Operations — Muntaha</title>
<meta name="description" content="Coordinating logistics for 40,000+ people across 158 countries taught me more about operations than any job did.">
<meta name="author" content="Sadaqatul Muntaha">
<meta name="robots" content="index,follow,max-image-preview:large">
<link rel="canonical" href="https://sadaqatulmuntaha.xyz/blog/posts/what-40000-scouts-taught-me-about-operations.html">
<meta name="theme-color" content="#2e0bfc">
<link rel="icon" href="/favicon.ico" sizes="any">
<link rel="icon" href="/favicon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="/apple-touch-icon.png">
<link rel="manifest" href="/site.webmanifest">
<meta property="og:type" content="article">
<meta property="og:site_name" content="Sadaqatul Muntaha">
<meta property="og:title" content="What 40,000 Scouts Taught Me About Operations">
<meta property="og:description" content="Coordinating logistics for 40,000+ people across 158 countries taught me more about operations than any job did.">
<meta property="og:url" content="https://sadaqatulmuntaha.xyz/blog/posts/what-40000-scouts-taught-me-about-operations.html">
<meta property="og:image" content="https://sadaqatulmuntaha.xyz/og.png">
<meta property="article:published_time" content="2026-07-27">
<meta property="article:author" content="Sadaqatul Muntaha">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:image" content="https://sadaqatulmuntaha.xyz/og.png">
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "headline": "What 40,000 Scouts Taught Me About Operations",
  "description": "Coordinating logistics for 40,000+ people across 158 countries taught me more about operations than any job did.",
  "image": "https://sadaqatulmuntaha.xyz/og.png",
  "datePublished": "2026-07-27",
  "dateModified": "2026-07-27",
  "author": { "@type": "Person", "name": "Sadaqatul Muntaha", "url": "https://sadaqatulmuntaha.xyz/" },
  "publisher": { "@type": "Person", "name": "Sadaqatul Muntaha" },
  "mainEntityOfPage": { "@type": "WebPage", "@id": "https://sadaqatulmuntaha.xyz/blog/posts/what-40000-scouts-taught-me-about-operations.html" }
}
</script>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Bricolage+Grotesque:opsz,wght@12..48,400..800&family=Geist:wght@300..700&display=swap" rel="stylesheet">
<link rel="stylesheet" href="../blog.css">
</head>
<body>

<header class="masthead">
  <div class="masthead-in">
    <a href="../../index.html" class="brand">Muntaha<b>.</b></a>
    <nav class="nav" aria-label="Primary">
      <a href="../../index.html#about">About</a>
      <a href="../../index.html#experience">Experience</a>
      <a href="../index.html" class="here">Writing</a>
      <a href="../../index.html#contact">Contact</a>
    </nav>
  </div>
</header>

<main>
  <article>
    <header class="article-head reading">
      <a class="back" href="../index.html">← All writing</a>
      <p class="eyebrow" style="margin-top:1.5rem">Essay</p>
      <h1>What 40,000 Scouts Taught Me About Operations</h1>
      <div class="meta">
        <time datetime="2026-07-27">Jul 27, 2026</time>
        <span class="dot"></span>
        <span>2 min read</span>
      </div>
    </header>

    <div class="prose reading">
      <p>In 2023 I helped coordinate logistics for the 25th World Scout Jamboree — 40,000+ participants from 158 countries, all moving through the same campsite, the same meal lines, the same shuttle schedules, at the same time. Nothing about that scale is intuitive. You learn operations by watching a queue that isn't supposed to exist form in front of you, and then figuring out, in real time, why.</p>

      <h2>A process is just a promise you can keep at scale</h2>
      <p>The plans on paper were beautiful. The moment 40,000 people touched them, the plans became suggestions. What actually held the event together wasn't the schedule — it was a handful of simple, repeatable rules everyone could follow without needing to ask a supervisor: which line, which wristband color, which exit. The best process is the one nobody has to think about.</p>

      <blockquote>A system that only works when everyone follows instructions perfectly isn't a system — it's a hope.</blockquote>

      <h2>What actually changed</h2>
      <p>Three things I've carried into every role since:</p>
      <ul>
        <li><strong>Design for the tired version of the person.</strong> Plan for someone on hour fourteen, not the version reading the manual fresh.</li>
        <li><strong>Escalation paths matter more than the plan.</strong> Things will go wrong; what matters is how fast the right person finds out.</li>
        <li><strong>Cross-cultural clarity beats clever wording.</strong> With 158 nationalities in the same camp, the instructions that worked needed no translation to understand.</li>
      </ul>

      <h3>A small example</h3>
      <p>One evening, a shuttle route quietly stopped making sense — buses were arriving faster than people could board, and a bottleneck started building at the gate. No one had scheduled a fix for that. We rerouted boarding on the spot, reusing the same wristband-color system built for the meal lines, and the backup cleared in under ten minutes. It worked because it was simple enough to repurpose under pressure.</p>

      <hr>
      <p>I've applied that same instinct everywhere since — in a legal team's document backlog, in an admissions pipeline, in a marketing calendar. The scale is different. The lesson is the same: good operations isn't about controlling every variable, it's about building something simple enough to survive contact with 40,000 real people.</p>
    </div>

    <footer class="article-foot reading">
      <a class="back" href="../index.html">← All writing</a>
      <a class="back" href="../../index.html#contact">Say hello →</a>
    </footer>
  </article>
</main>

<footer class="blog-footer">
  <div class="wrap">
    <span>© 2026 Sadaqatul Muntaha</span>
    <a href="../../index.html">← Back to portfolio</a>
  </div>
</footer>

</body>
</html>
```

- [ ] **Step 4: Verify links resolve**

```bash
grep -o 'href="[^"]*"' blog/index.html blog/posts/what-40000-scouts-taught-me-about-operations.html
```
Manually confirm each relative path (`../index.html`, `index.html`, `posts/...html`, `../blog.css`, etc.) points to a file that exists in the repo.

- [ ] **Step 5: Commit**

```bash
git add blog
git commit -m "Add blog subsystem with 'What 40,000 Scouts Taught Me About Operations' post"
```

---

### Task 10: Update robots.txt, site.webmanifest, sitemap.xml, README.md

**Files:**
- Modify: `robots.txt`, `site.webmanifest`, `sitemap.xml`, `README.md`

**Interfaces:**
- Consumes: nothing.
- Produces: nothing consumed elsewhere.

- [ ] **Step 1: `robots.txt`**

```
User-agent: *
Allow: /

Sitemap: https://sadaqatulmuntaha.xyz/sitemap.xml
```

- [ ] **Step 2: `site.webmanifest`**

```json
{
  "name": "Sadaqatul Muntaha",
  "short_name": "Muntaha",
  "description": "Sadaqatul Muntaha — Finance & Operations Professional. I turn client relationships and messy operations into systems that hold up under scale.",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#2e0bfc",
  "icons": [
    { "src": "/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icon-512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/favicon.svg", "sizes": "any", "type": "image/svg+xml" }
  ]
}
```

- [ ] **Step 3: `sitemap.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://sadaqatulmuntaha.xyz/</loc>
    <lastmod>2026-07-27</lastmod>
    <changefreq>monthly</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://sadaqatulmuntaha.xyz/blog/</loc>
    <lastmod>2026-07-27</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.8</priority>
  </url>
  <url>
    <loc>https://sadaqatulmuntaha.xyz/blog/posts/what-40000-scouts-taught-me-about-operations.html</loc>
    <lastmod>2026-07-27</lastmod>
    <changefreq>yearly</changefreq>
    <priority>0.6</priority>
  </url>
</urlset>
```

- [ ] **Step 4: `README.md`**

```markdown
# Muntaha-Card
Personal portfolio site for Sadaqatul Muntaha, based on her CV and LinkedIn profile.
```

- [ ] **Step 5: Commit**

```bash
git add robots.txt site.webmanifest sitemap.xml README.md
git commit -m "Update robots.txt, manifest, sitemap, and README for Muntaha-Card"
```

---

### Task 11: Full-page verification pass

**Files:** none modified — read-only verification of everything built in Tasks 1–10.

**Interfaces:**
- Consumes: the finished `index.html`, `blog/index.html`, `blog/posts/what-40000-scouts-taught-me-about-operations.html`.

- [ ] **Step 1: Global safwan-content sweep**

```bash
grep -rin "safwan\|precognise\|mohammed" --include="*.html" --include="*.json" --include="*.xml" --include="*.txt" .
```
Expected: no output anywhere in the repo.

- [ ] **Step 2: Open the site in a browser and visually verify**

Use the `run` skill (or `start index.html` directly) to open the page. Confirm:
- Desktop width (~1440px): hero, about, experience (7 roles), skills (4 rows), education (2 degree blocks + 3 certs), writing preview, contact card with QR, footer — all render without a broken-image icon and without horizontal scroll.
- Mobile width (~390px, via browser devtools device toolbar): same sections stack correctly, no horizontal scroll, nav collapses/works as safwan-card's does.
- Click "Save my card to your phone" — confirm a `Sadaqatul-Muntaha.vcf` file downloads and the on-screen toast ("Saved — check your downloads") appears.
- Open the downloaded `.vcf` in a text editor and confirm it contains `FN:Sadaqatul Muntaha`, `TEL;TYPE=CELL:+8801626253263`, `EMAIL;TYPE=INTERNET,HOME:sadaqatulmuntaha80@gmail.com`, and a `PHOTO;ENCODING=b;TYPE=JPEG:` field with base64 content.
- Click "All writing →", then the blog post link, then "← All writing" and "← Back to portfolio" — confirm all four navigate correctly.

- [ ] **Step 3: Clean up scratch asset files**

```bash
rm -rf assets
```
(`assets/photo_base64.txt` and `assets/qr_base64.txt` were only scratch intermediates for splicing base64 into the HTML — the actual data now lives inline in `index.html`.)

- [ ] **Step 4: Final diff review and commit**

```bash
git status
git add -A
git commit -m "Remove scratch asset files after verification" --allow-empty
```

---

### Task 12: Create the GitHub repo and push

**Files:** none — this is a `gh`/`git` operation only.

**Interfaces:** none.

- [ ] **Step 1: Create the public GitHub repo**

```bash
gh repo create safwandotcom/Muntaha-Card --public --description "Personal portfolio site for Sadaqatul Muntaha, based on her CV and LinkedIn profile." --source=. --remote=origin
```

- [ ] **Step 2: Push**

```bash
git push -u origin main
```
(If the local default branch isn't `main`, run `git branch -M main` first, matching `safwan-card`'s branch name.)

- [ ] **Step 3: Verify**

```bash
gh repo view safwandotcom/Muntaha-Card --json name,url,pushedAt
```
Expected: `name` is `Muntaha-Card`, `pushedAt` is recent.

- [ ] **Step 4: Report the repo URL back to the user**

Tell the user the repo is live at `https://github.com/safwandotcom/Muntaha-Card` and ready for them to import into their `sadaqatulmuntaha.xyz` Vercel project.

---

## Self-Review Notes

- **Spec coverage:** Hero/vCard ✓ (Task 3, 8), About ✓ (Task 4), Experience timeline with all 7 roles headlining Chevron ✓ (Task 5), Skills ✓ (Task 6), Education & Credentials ✓ (Task 7), Scouting section dropped ✓ (Task 7), Blog with starter post ✓ (Task 9), Contact ✓ (Task 8), accent color unchanged ✓ (Global Constraints), photo/phone from user ✓ (Task 1/3/8), repo creation ✓ (Task 12), no Vercel/DNS work ✓ (excluded throughout).
- **Placeholder scan:** the only literal placeholder tokens are `PASTE_PHOTO_BASE64_HERE` / `PASTE_QR_BASE64_HERE` / `PASTE_PHOTO_BASE64_HERE`, and each is immediately followed by an explicit instruction naming the exact source file to splice in — these are splice markers for a mechanical copy step, not unresolved design decisions.
- **Type/name consistency:** `data-vcard` attribute, `buildVCard()`, `addToContacts()`, `showToast()`, section IDs (`#about`, `#experience`, `#skills`, `#education`, `#contact`), and CSS classes (`.xp-item`, `.skrow`, `.edu-grid`, `.cert`, `.dcard`) are used identically to safwan-card's existing implementation across all tasks — no renamed identifiers.
