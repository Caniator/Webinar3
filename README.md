# Aerones Webinar Landing Page — Vercel Handoff

Static site for **webinar.aerones.com** — the registration page for *From Inspection to Insight: The Aerones Wind Operations Stack*, live on June 11, 2026.

This folder is a self-contained static site. No build step. Vercel serves it as-is.

---

## What's in this folder

```
.
├── index.html           # The page — React + Tailwind, all in one file
├── favicon.png          # Aerones logomark (32×32 + apple-touch fallback)
├── og-image.png         # 1200×630 social card
├── robots.txt
├── sitemap.xml
├── vercel.json          # Cache + security headers
├── .gitignore
├── README.md            # ← you are here
└── assets/
    ├── logo/            # Aerones wordmark, color + white
    ├── patterns/        # Blueprint texture used behind hero + form section
    ├── photos/          # Hero laptop product shot
    └── speakers/        # Andis + Nauris headshots
```

---

## How the page works

It's a single React component rendered at runtime via Babel-standalone. There is **no build step** — `index.html` ships everything inline. Pros: anyone can edit it in a text editor. Cons: ~150 KB of React/Babel served per page load (mitigated by the cache headers in `vercel.json`).

The registration form is a **Pipedrive hosted webform** embedded via Pipedrive's auto-resizing loader. On submit, Pipedrive redirects the user to the Zoom Webinar signup page; Zoom emails the join link. **No middleware, no API tokens on our side.**

To change the form (fields, redirect URL, styling) edit it in the Pipedrive web UI — the embed picks up changes automatically.

To change the form **URL** (e.g. a new webinar), edit the `PIPEDRIVE_FORM_URL` constant near the top of the `<script type="text/babel">` block in `index.html`.

---

## Deploy to Vercel — first time

1. **Create a Git repo** with this folder's contents at the root:
   ```bash
   cd vercel-build
   git init -b main
   git add .
   git commit -m "Initial commit — webinar landing"
   git remote add origin git@github.com:<aerones-org>/webinar-landing.git
   git push -u origin main
   ```

2. **Import to Vercel:** dashboard → *Add New… → Project* → pick the repo.
   - **Framework Preset:** `Other`
   - **Root Directory:** `./` (leave default)
   - **Build Command:** *(leave blank)*
   - **Output Directory:** *(leave blank — Vercel serves the root)*
   - **Install Command:** *(leave blank)*
   - Click **Deploy**.

3. **Wait ~20 seconds.** Vercel gives you a `*.vercel.app` preview URL.

4. **Sanity check** the preview:
   - Page loads, no console errors
   - "Save my seat" / "Register for June 11" both scroll to the Pipedrive form
   - The form renders and submits to Pipedrive (which redirects to Zoom)
   - On mobile (DevTools or a real device): layout looks right, no horizontal scroll

5. **Point `webinar.aerones.com` at Vercel:**
   - In Vercel project → *Settings → Domains* → **Add Domain** → enter `webinar.aerones.com`
   - Vercel will show one of two records to add at your DNS host (CNAME to `cname.vercel-dns.com` is most common)
   - Add it, wait for DNS to propagate (usually <5 min, sometimes longer)
   - Vercel auto-provisions an SSL cert

---

## Deploy updates

Push to `main`. Vercel deploys automatically. The HTML is served with `Cache-Control: max-age=0, must-revalidate` so changes are visible immediately on refresh.

Static assets (`/assets/*`) are served with a 1-year immutable cache. If you replace an asset (e.g. swap a speaker photo with the same filename), **change the filename** to bust the cache.

---

## Editing the page

### Update text / copy

Open `index.html` and search for the string. Most copy lives near the top of each component (`Hero`, `Agenda`, `Speakers`, `FAQ_ITEMS`, `Footer`).

### Change brand colors

The Tailwind palette is defined in the inline `tailwind.config = { ... }` block near the top of `index.html`. Change the hex values under `colors.ae`.

### Swap speaker photos

Drop new files into `assets/speakers/` (PNG or JPG, square, ~1600×1600). Update the `src:` values in the `Speakers` component.

If a face doesn't sit right inside the square crop, edit the `objectPosition` in the `Speakers` component — values are `"<horizontal>% <vertical>%"` (e.g. `"50% 35%"` means horizontally centered, faces shown 35% from the top).

### Change the Pipedrive form

Edit the form in Pipedrive (Leads → Webforms). Layout / fields / redirect URL changes are picked up automatically.

To swap the form **entirely** (different form, new webinar), update `PIPEDRIVE_FORM_URL` near the top of the `<script>` block in `index.html`.

---

## Analytics (optional, not yet wired)

To add **Vercel Analytics** (recommended — privacy-friendly, no cookie banner needed in EU):
- In Vercel dashboard → project → *Analytics* tab → Enable
- Add this line just before `</head>`:
  ```html
  <script defer src="/_vercel/insights/script.js"></script>
  ```

To add **Plausible** instead:
```html
<script defer data-domain="webinar.aerones.com" src="https://plausible.io/js/script.js"></script>
```

Note: because the Pipedrive form is in an iframe, you won't get true "form submit" conversion events from the client side. Track conversions inside Pipedrive (lead created = submit).

---

## Troubleshooting

**"The Pipedrive form doesn't show up."**
The form loader script (`https://webforms.pipedrive.com/f/loader`) may be blocked by ad blockers in development. Test in an incognito window without extensions. In production this is rarely an issue for end users — Pipedrive forms are common and uBlock/Adblock don't block them.

**"OG card preview looks wrong on LinkedIn / Slack."**
LinkedIn aggressively caches OG images. Use the [LinkedIn Post Inspector](https://www.linkedin.com/post-inspector/) to force a refresh after each deploy.

**"Page loads slowly on first visit."**
First visit pulls ~150 KB of React + Babel + Tailwind from unpkg/jsdelivr. Subsequent visits are cached. If page-speed becomes a priority for a future campaign, the migration target is a Next.js or Vite static export (pre-compiled JSX, no runtime Babel). I'd estimate a half-day rebuild for someone who knows React.

**"I changed brand colors and the CTAs still look blue."**
There's a small runtime style override at the bottom of the App component that hard-sets the CTA color from `TWEAK_DEFAULTS.primaryCta`. Update that constant *and* the Tailwind palette together.

---

## Contacts

- **Webinar inquiries:** info@aerones.com
- **Page maintenance:** whoever owns marketing-eng at Aerones
