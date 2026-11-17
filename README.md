# mangue.dev

Static website for **Mangue** and its iOS app **Stock Hush**. Plain HTML and CSS: no framework, no build step, no npm, no analytics, no third-party requests. Hosted on GitHub Pages at `https://mangue.dev`.

```
/index.html                     Mangue landing page (yellow)
/stockhush/index.html           Stock Hush product page
/stockhush/privacy/index.html   Privacy policy (DRAFT, has [CONFIRM] markers)
/stockhush/terms/index.html     Terms (DRAFT, has [CONFIRM] markers)
/stockhush/support/index.html   Support / FAQ
/404.html                       On-brand 404 (GitHub Pages serves it automatically)
/CNAME                          mangue.dev
/.nojekyll                      tells GitHub Pages to serve files as-is
/robots.txt, /sitemap.xml, /site.webmanifest, /favicon.ico
/assets/css/                    base.css (shared), phone.css (phone mockups), mangue.css, stockhush.css
/assets/fonts/                  Inter variable font, Latin subset (SIL OFL, see OFL.txt)
/assets/img/                    logos, Stock Hush icon, og-*.png social images, avatar.png
/assets/                        favicon.svg, apple-touch-icon.png, icon-192.png, icon-512.png
```

## Preview locally

```bash
python3 -m http.server 8000
```

Open http://localhost:8000. (The local server doesn't serve `404.html` for missing paths, but GitHub Pages does. Open `/404.html` directly to preview it.)

## Deploy to GitHub Pages

1. Create an **empty public repository** on GitHub, for example `mangue-site`. Don't add a README, license or .gitignore.
2. Push this repo (see "Push" below).
3. On GitHub: **Settings → Pages**
   - **Source:** Deploy from a branch
   - **Branch:** `main`, folder `/ (root)`, then **Save**
   - **Custom domain:** `mangue.dev`, then **Save**. The `CNAME` file in this repo already contains it.
4. Set up DNS at Porkbun (below). Wait until GitHub's DNS check passes; this can take from a few minutes to a few hours.
5. Tick **Enforce HTTPS** once it becomes available. GitHub issues the certificate automatically.
6. Optional but recommended: **Settings → Pages → Verify a custom domain** (account-level Settings → Pages) to protect the domain from takeover.

### Push

```bash
git remote add origin git@github.com:raghavverma190/mangue-site.git
git push -u origin main
```

(HTTPS alternative: `git remote add origin https://github.com/raghavverma190/mangue-site.git`)

## Porkbun DNS records

In Porkbun: **Domain Management → mangue.dev → DNS**.

**First delete** Porkbun's default records:
- the `ALIAS` record on the root (`mangue.dev` → `pixie.porkbun.com`)
- the wildcard `CNAME` record (`*.mangue.dev` → `pixie.porkbun.com`)

**Then add:**

| Type  | Host (leave blank = root) | Answer                         | TTL |
|-------|---------------------------|--------------------------------|-----|
| A     | *(blank)*                 | `185.199.108.153`              | 600 |
| A     | *(blank)*                 | `185.199.109.153`              | 600 |
| A     | *(blank)*                 | `185.199.110.153`              | 600 |
| A     | *(blank)*                 | `185.199.111.153`              | 600 |
| CNAME | `www`                     | `raghavverma190.github.io`  | 600 |

**Optional, for IPv6:**

| Type | Host      | Answer                 |
|------|-----------|------------------------|
| AAAA | *(blank)* | `2606:50c0:8000::153`  |
| AAAA | *(blank)* | `2606:50c0:8001::153`  |
| AAAA | *(blank)* | `2606:50c0:8002::153`  |
| AAAA | *(blank)* | `2606:50c0:8003::153`  |

Leave any MX/TXT records for email (for example `hello@` and `support@`) untouched. With `www` pointed at GitHub and `mangue.dev` as the custom domain, GitHub redirects `www.mangue.dev` to `mangue.dev`.

Check it from a terminal:

```bash
dig +short mangue.dev A
```

```bash
dig +short www.mangue.dev CNAME
```

## Swapping TestFlight for the App Store

Every beta CTA links to the same URL, written out in full so the buttons work without JavaScript. Each one is tagged `data-cta="testflight"`.

Current link: `https://testflight.apple.com/join/eSAuBwD7`

**1. Replace the URL everywhere** (macOS `sed`; put in your real App Store URL):

```bash
grep -rl 'https://testflight.apple.com/join/eSAuBwD7' --include='*.html' . | xargs sed -i '' 's#https://testflight.apple.com/join/eSAuBwD7#https://apps.apple.com/app/id0000000000#g'
```

**2. Update the button labels.** The CTA wording is "Join the beta" / "Join the TestFlight beta". Find them with:

```bash
grep -rn 'data-cta="testflight"' --include='*.html' .
```

Change the text (for example "Download on the App Store"). Optionally rename `data-cta="testflight"` to `data-cta="appstore"`.

**3. Optional: use Apple's badge.** Download the official "Download on the App Store" badge SVGs from Apple's Marketing Resources (https://developer.apple.com/app-store/marketing/guidelines/). Save them in `/assets/img/`, then swap the CTA's text for:

```html
<a data-cta="appstore" href="https://apps.apple.com/app/id0000000000" rel="noopener">
  <img src="/assets/img/app-store-badge.svg" alt="Download on the App Store" width="120" height="40">
</a>
```

Also update the "iOS · TestFlight beta" chip on `/index.html` and the "Go quiet." CTA sentence on `/stockhush/index.html` ("Join the TestFlight beta and tell us how it feels.").

## Updating content

- **Copy** lives directly in each `index.html`. Edit the text and commit.
- **Colors** are CSS variables at the top of `assets/css/mangue.css` (yellow theme) and `assets/css/stockhush.css` (app palette; the dark palette is under `@media (prefers-color-scheme: dark)`).
- **Pricing** is in the "Pricing" section of `stockhush/index.html`, and repeated in `stockhush/terms/index.html` and the JSON-LD block in the `<head>` of `stockhush/index.html`. Change all three together.
- **FAQ** appears in two places: `stockhush/index.html` (the `<dl class="faq">` list) and `stockhush/support/index.html` (the `<details>` blocks).
- **Phone mockups** are HTML in the pages, styled by `assets/css/phone.css`. Each one has a `role="img"` and an `aria-label` describing it. If you change the visible text, update the label too.
- **Legal pages:** search for `[CONFIRM` and replace each one. Each page ends with a yellow-bordered "items to confirm" list; delete that `<section class="confirm-list">` once everything is filled in, and set the effective date.
- **New pages:** copy an existing page, then update its `<title>`, `<meta name="description">`, `<link rel="canonical">` and OG/Twitter tags, and add it to `sitemap.xml`.
- **Social images** (`assets/img/og-*.png`, 1200×630) were rendered from HTML with headless Chrome. To change them, edit and re-export in any tool at 1200×630.
- **Year:** the footers say "© 2026". Update it by hand each year:

```bash
grep -rl '© 2026' --include='*.html' . | xargs sed -i '' 's/© 2026/© 2027/g'
```

## Notes

- Fonts: Inter is self-hosted. `InterVariable-latin.woff2` is a Latin subset of the official Inter 4.1 release by Rasmus Andersson, licensed under the SIL Open Font License (`assets/fonts/OFL.txt`).
- The logo SVGs (`assets/img/logo-ink.svg`, `logo-yellow.svg`, `logo-white.svg`) are outlined Inter Bold paths, so they don't depend on the font being installed.
- The Stock Hush icon (`assets/img/stockhush-icon.svg`) is a vector redraw traced from the app icon in the app design PDF. Replace it with an export of the real icon file if you have one.
