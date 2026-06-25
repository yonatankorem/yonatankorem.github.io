# Yonatan Korem — logo pack

A three-layer stacked mark in cool blue, with the darkest sheet anchored on the
bottom and translucent overlaps. Built from plain polygons, so the SVGs render
identically everywhere and the look survives down to a 16px favicon.

## Files

**Mark (transparent)**
- `mark.svg` — blue mark, for light backgrounds
- `mark-dark.svg` — light-blue mark, for dark backgrounds
- `mark-blue-512.png`, `mark-light-512.png` — raster versions

**Favicon / app icon (navy tile)**
- `favicon.svg` — scalable, primary
- `favicon-16.png`, `favicon-32.png`, `favicon-48.png`
- `apple-touch-icon.png` (180px)
- `icon-512.png` — PWA / social

**Email**
- `signature-logo.png` — 48px @2x tile for signatures
- `email-signature.html` — Brevo-ready signature block

**Code**
- `site-snippet.html` — favicon `<link>` tags + header lockup

## Add to your site
1. Drop this `assets/` folder into your site repo.
2. Paste the favicon `<link>` tags from `site-snippet.html` into your `<head>`.
3. Use the inline-SVG header lockup (also in `site-snippet.html`) for the nav.

## Add to Brevo
1. Push `assets/` to your GitHub Pages repo so the logo is public at
   `https://yonatankorem.github.io/assets/signature-logo.png`
   (or upload `signature-logo.png` to Brevo's image library and use that URL).
2. In Brevo, insert an HTML block / signature and paste the table from
   `email-signature.html`. Double-check the image `src` URL.

## Colors
- Blue: `#2563EB`  (alpha .30 / .58 / .92, top→bottom)
- Light blue (on dark): `#60A5FA`  (alpha .55 / .74 / .95)
- Navy tile: `#0B1220`
