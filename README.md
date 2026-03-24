# micahbank2-24hourpharmacy-plan[README (1).md](https://github.com/user-attachments/files/26215126/README.1.md)
# 24HourPharmacy.com

A content and tools website that helps people find 24-hour pharmacies across the United States and save money on prescriptions. Built on WordPress with custom React widgets, a programmatic city page pipeline, and a multi-stream affiliate monetization model.

This repo contains all custom code. WordPress core, GeneratePress Pro, and third-party plugins are managed on the server and are not tracked here.

---

## What this is

An exact-match domain site targeting "24 hour pharmacy near me" and related high-intent local health queries. Users are often in genuine emergencies - out of medication at midnight, traveling, looking for urgent care alternatives. The site serves that need directly and monetizes through prescription savings tools, insurance lead gen, and telehealth referrals.

This is **not** an online pharmacy. It does not dispense medications, handle prescriptions, or store any health data.

---

## Repo structure

```
├── wordpress/
│   ├── theme/                  GeneratePress child theme
│   │   ├── style.css
│   │   ├── functions.php
│   │   ├── front-page.php
│   │   ├── single-city.php
│   │   ├── archive-state.php
│   │   ├── header.php
│   │   └── footer.php
│   └── plugin/                 Custom plugin: 24hr-pharmacy-tools
│       ├── 24hr-pharmacy-tools.php
│       ├── post-types.php
│       ├── taxonomies.php
│       ├── meta-boxes.php
│       ├── schema.php
│       ├── shortcodes.php
│       ├── settings.php
│       ├── rewrite-rules.php
│       └── rest-api.php
├── widgets/                    React widgets (Vite, IIFE output)
│   ├── pharmacy-finder/        [pharmacy_finder] shortcode
│   ├── discount-card/          [discount_card] shortcode
│   ├── price-checker/          [price_checker] shortcode
│   └── open-now-checker/       [open_now_checker] shortcode
├── data/
│   ├── cities.json             Target cities with coordinates and population
│   ├── scripts/                Python data pipeline
│   └── pharmacies/             Raw Google Places API response cache
├── CLAUDE.md                   Instructions for Claude Code
├── .env.example                Environment variable reference (no real values)
└── README.md
```

---

## Tech stack

| Layer | Choice |
|-------|--------|
| CMS | WordPress 6.x |
| Theme | GeneratePress Pro + child theme |
| Hosting | Hostinger Business |
| CDN / Security | Cloudflare (free tier) |
| Caching | LiteSpeed Cache |
| React widgets | React 18, Vite (library mode, IIFE output) |
| Widget styling | CSS custom properties from GeneratePress (no Tailwind) |
| Data pipeline | Python 3.11 |
| Maps / Places | Google Maps Platform |
| Analytics | Google Analytics 4 + Search Console |
| Email | MailerLite |

---

## React widgets

Each widget is a standalone React app compiled by Vite in library mode to a single IIFE bundle. Bundles are placed inside the custom plugin and loaded via WordPress shortcodes.

**Key rules:**
- Styled with CSS custom properties from the GeneratePress theme, not Tailwind
- All API keys and affiliate codes passed via `wp_localize_script()` from `wp_options`
- Nothing sensitive is hardcoded in this repo
- Each widget loads asynchronously to avoid blocking page render
- Bundles are individual per widget for now; consolidate into a shared bundle once 4+ widgets exist on the same pages

Build a widget:

```bash
cd widgets/discount-card
npm install
npm run build
# Output: dist/discount-card.iife.js
# Copy to: wordpress/plugin/assets/js/
```

---

## Data pipeline

The Python pipeline in `data/scripts/` generates city pages programmatically:

1. Reads `data/cities.json` (city name, state, lat/lng, population)
2. Queries Google Places API for 24-hour pharmacies within 25km
3. Filters results to verified 24-hour or late-night locations
4. Generates WordPress post payloads (title, slug, HTML, meta, JSON-LD schema)
5. Pushes drafts to WordPress via REST API using application passwords
6. Human review and 500+ word unique editorial content added before publishing

Raw API responses are cached to `data/pharmacies/raw/` to avoid redundant API calls.

Rate limit: max 10 Google Places API requests per second.

---

## Environment variables

**Never commit real values.** Use `.env.example` as a reference.

All sensitive values are stored in one of two places:
- **Local `.env`** for running data pipeline scripts
- **WordPress options table** (via plugin Settings page in WP admin) for values needed by the frontend

The plugin Settings page stores: Google Maps API key, all affiliate BIN/PCN/Group codes, network publisher IDs. These are passed to React via `wp_localize_script()`.

See `.env.example` for the full list of required variables.

---

## Deployment

There is no CI/CD pipeline yet. Deploy manually:

**Child theme:**
```
Zip wordpress/theme/
Upload via WP Admin > Appearance > Themes > Add New > Upload
```

**Custom plugin:**
```
Zip wordpress/plugin/
Upload via WP Admin > Plugins > Add New > Upload Plugin
```

**Widget bundles:**
```
Build via Vite (see above)
Copy .iife.js file into plugin/assets/js/
Redeploy plugin zip
```

**Data scripts:**
```
Run locally
Scripts push content to WordPress via REST API
Requires WP_APP_USER and WP_APP_PASSWORD in .env
```

To automate: add a GitHub Actions workflow that SFTPs changed theme/plugin files to Hostinger on push to `main`.

---

## WordPress security

Hardening is documented in `security-and-ops.md`. Key items:

- `DISALLOW_FILE_EDIT` and `FORCE_SSL_ADMIN` set in `wp-config.php`
- Wordfence active with brute force protection, 2FA on admin, XML-RPC blocked
- Google Maps API key restricted to `24hourpharmacy.com/*` in Google Cloud Console
- Cloudflare WAF rate-limits `/wp-login.php` and `/xmlrpc.php`
- UpdraftPlus runs daily backups to Google Drive

---

## Content rules

This is a YMYL (Your Money or Your Life) site. Google applies elevated quality standards to health content.

- Every city page requires 500+ words of genuinely unique editorial content. No thin programmatic output.
- All pages with affiliate links require a visible FTC disclosure above the fold.
- Every page requires the medical disclaimer in the footer.
- H2s should be written as questions answered directly in the opening sentence (GEO / AI Overviews optimization).
- Schema required on all city pages: `LocalBusiness`, `FAQPage`, `BreadcrumbList`.
- Schema required on guide content: `HowTo`, `MedicalWebPage`.

See `CLAUDE.md` for the full content generation instructions used by Claude Code.

---

## URL structure

```
/                                    Homepage
/city/{state}/{city-slug}/           City page
/state/{state-slug}/                 State directory
/pharmacy/{slug}/                    Individual pharmacy
/savings/discount-card/              Discount card hub
/savings/home-testing/               At-home health testing
/insurance/                          Insurance comparison
/telehealth/                         Telehealth hub
/telehealth/glp1/                    GLP-1 telehealth comparison
/seniors/medical-alerts/             Medical alert systems
/mental-health-resources/            Mental health platforms
/blog/{post-slug}/                   Blog
/tools/pharmacy-finder/              Standalone finder tool
/advertise/                          Direct advertising for independent pharmacies
/about/                              About + author bio
/affiliate-disclosure/               FTC disclosure (required)
/privacy-policy/                     Privacy policy (required)
/disclaimer/                         Medical disclaimer (required)
```

---

## Performance targets

| Metric | Target |
|--------|--------|
| LCP | < 2.5s |
| CLS | < 0.1 |
| INP | < 200ms |
| TTFB | < 600ms |
| Mobile Lighthouse | > 90 |

Ad network approval (Raptive/Mediavine) requires strong Core Web Vitals. Test with PageSpeed Insights before applying.
