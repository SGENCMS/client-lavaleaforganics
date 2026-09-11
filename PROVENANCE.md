# PROVENANCE

A capture of **every page** of `https://lavaleaforganics.com/`, then deliberately modified for
public hosting. Because it was modified, **this bundle is not a faithful record of what the
server sent** and must not be cited as one. Every deviation is listed below.

## Capture

| | |
| --- | --- |
| Source | `https://lavaleaforganics.com/` — 11 pages (the site's sitemap: 10 live pages + its 404 page) |
| Captured | 2026-09-10, one page at a time |
| Method | clone-site pipeline, Clone Stages 1–5 (Playwright + CDP), multi-page shape |
| Age gate | Captured past the 21+ gate by seeding `age_verification=verified` — the cookie the site's own `age-verification.min.js` sets on "YES". No other cookie was seeded. |
| HTML source | For 10 of 11 pages, the raw body of the same response whose inline styles were captured, so builder element ids and their styles stay consistent. The 404 page is served with HTTP 404, which the capture step does not keep as a body, so it used a second request; that page has no id-keyed styles, so nothing was lost. |

## Deviations from the captured bytes

"Rendered" = whether the change can affect what is painted.

### Leak and identity hardening (every page)

| # | Change | Rendered? |
| --- | --- | --- |
| 1 | Global transport guard (`fetch` / `XMLHttpRequest` / `sendBeacon`) + `<meta name="referrer" content="no-referrer">` inserted as the first children of `<head>` | No |
| 2 | `<title>` prefixed `UNOFFICIAL PREVIEW — `; `meta[name=description]` replaced with a preview disclosure | No |
| 3 | `og:title` / `og:url` / `og:site_name` / `og:description` rewritten to preview identity; `og:url` points at each page's own preview URL | No |
| 4 | `og:image`, `og:image:secure_url`, `twitter:image` deleted (they hotlinked the client's logo from the client's server); `twitter:*` rewritten, card downgraded to `summary` | No |
| 5 | `robots` `index, follow, …` → `noindex, nofollow, noarchive, nosnippet` | No |
| 6 | `google-site-verification` meta deleted | No |
| 7 | `schema.org` JSON-LD blocks deleted (business identity; one also carried an internal hostname) | No |
| 8 | `Defaults.base_url` / `admin_url` / `current_url` blanked (`admin_url` disclosed the client's back-office path) | No |
| 9 | `window.__SG_TRACK__` analytics config deleted | No |
| 10 | Search form `action` removed (it posted visitors' queries to the client's server); original kept as `data-action-removed-at-publish` | No |
| 11 | Live Google Tag Manager `<noscript><iframe>` deleted | No |
| 12 | 75 hotlinked client-origin asset references rewritten to byte-identical local copies, resolved through the capture manifests **by content hash**, never by filename | No (same bytes) |

### Removed files (captured, never part of the site's own content)

| # | Removed | Why |
| --- | --- | --- |
| 13 | `assets/front/js/sg-analytics.js`, `sg-collect.html` | First-party analytics tracker and its collector stub |
| 14 | Captured Google Tag Manager, Microsoft Clarity (11 hosts), and an identity/ad-tech platform's scripts | Tracking payloads; unreferenced |
| 15 | Captured Google Maps / Places RPC responses | One held a signed Google service-account bearer token minted for the capture session (see below) |
| 16 | A duplicate copy of the e-commerce menu loader and a captured image directory from another business's website | Unreferenced |
| 17 | A capture directory from the client's staging server | Unreferenced — every image in it already resolves via a local copy |
| 18 | Mirrored duplicate copies of three page documents and of one placeholder image | Unreferenced once the page links were repaired (#23) |

### Live embeds made inert

| # | Change | Rendered? |
| --- | --- | --- |
| 19 | **Ordering menu (`shop.html`)**: the e-commerce provider's loader injected an iframe serving the client's live, orderable menu. Loader removed; replaced by an inert placeholder of the identical structure and size (`#dutchie--embed__container` > iframe, `width:100%`, `height:100vh`, **no `src`**). | Same layout; the live capture shows this region blank |
| 20 | **Maps (9 iframes on 8 pages)**: Google Maps embeds use the client's API key, which is referrer-locked to their domain — on this preview each rendered Google's "Oops! Something went wrong" (`RefererNotAllowedMapError`) while still sending requests with the key. Each iframe element is kept (so every iframe-targeted style still applies) but has no `src`; its `srcdoc` shows a static image of that same map as the live site renders it, captured with page overlays hidden. | Map region: a picture instead of an interactive map |
| 21 | **Unloadable images (`farmington-nm-dispensary.html`)**: 9 images hotlinked from a third-party host whose TLS certificate is not valid for that hostname (`ERR_CERT_COMMON_NAME_INVALID` in Chromium) — broken for every visitor on the live site too. 10 URL references replaced with a 1×1 transparent image; 2 preloads for them removed. | Same as a failed load |

### Internal identifiers

| # | Change | Rendered? |
| --- | --- | --- |
| 22 | An internal staging hostname (13 references: an accessibility script's selector on every page, and two body links pointing *at* the staging host) → the client's public host; a tenant path segment that is not public on the client's site (43 references) → the neutral `sites/media/` | No (same bytes, new path) |

### Multi-page structure repairs

The capture pipeline's multi-page emitter introduced defects that would have broken the site's
behaviour. Each was repaired here and each is a known pipeline defect rather than site content.

| # | Defect | Repair |
| --- | --- | --- |
| 23 | Root-relative links between pages (`/about-us`, `/service-areas/…`) were not rewritten — on this host they would 404 | Rewritten to the sibling files |
| 24 | 9 of 11 `<link rel="canonical">` tags had been rewritten to local files (one to a raw duplicate page) | Restored to each page's own client URL |
| 25 | The site URL had been string-substituted *inside* longer URLs: 30 × `href="index.htmlsites/…"`, 6 × `href="shop.html/…"` | Repaired to the real local file (`sites/…`), or to `404.html` for links that are dead on the live site |
| 26 | 1 link is an unrendered `{{site_url}}` template string on the client's page (dead on the live site) | → `404.html` |
| 27 | Links to `/learn` (dead on the live site) | → `404.html`, where the real site sends visitors |
| 28 | Every page's inline `<style>` elements had been merged into one block. The source ships them separately, so an unbalanced rule in one (a stray `}`) was contained; merged, it swallowed the next rule — silently dropping a section's 152 px/100 px padding on `contact-us` | Split back into separate `<style>` elements at the capture's own chunk boundaries: 11 blocks → 298 elements, same order, same bytes |
| 29 | Two Google Fonts stylesheets from **inside** the map iframes' documents were merged into the pages' shared `chrome.css` — the live pages never load them | Both chunks removed from `chrome.css` (`@font-face` rules only, 165,965 bytes; the file re-parses with 0 errors), then the 115 font files only they referenced (Google Sans, Google Sans Text, Roboto) deleted. No page requested any of them at render (checked on all 11 pages at 2 widths). Google Sans is not an open-licensed family, so this also ends re-serving it. |
| 30 | Internal build-pipeline markers on `<style>` tags | Removed |

### Publishing

| # | Change |
| --- | --- |
| 31 | `.nojekyll`, `.gitattributes`, `.gitignore`, `README.md`, this file added |
| 32 | Pipeline `README.md` replaced |

## Deliberately NOT changed

| | Why |
| --- | --- |
| `<link rel="canonical">` → the client's URL, on every page | Correct for a duplicate; deliberately different from `og:url` |
| Links to pages outside this set (cart, checkout, online menu) | Intended for a preview; mitigated by `no-referrer`, not blocked |
| Favicon (client logo, local copy) | Matches house practice on sibling previews |
| `<meta name="sgen_version">` | Present on the live public site; discloses nothing new |
| `/* === source: … === */` comments in `chrome.css` | Inert provenance markers naming each stylesheet's public source URL |
| Cart endpoint URLs in inline configuration | Neutralised at the transport layer by the guard |

## A captured credential, and repository history

An earlier publish of this repository (single-page shape) contained a captured Google
service-account bearer token (`scope: maps-platform.places.details`, one-hour validity), found by a
pre-publish audit after the first push. It had already expired when found and belonged to Google's
own service account, not the client's. It was removed and the commit force-pushed. A force-push
makes the old commit unreachable but does not delete it: GitHub can still serve it by commit hash.
That earlier version's own `PROVENANCE.md` also named internal hostnames. **Only deleting and
recreating the repository removes that history.**

## How the pixel comparison was taken

The capture step's standard screenshot harness clicks every toggle on a page *before* it takes its
screenshots. On these pages that left the accessibility drawer and the mobile menu open in the build
screenshots and scored them 21–89% against the live references. It is a measurement artefact —
a fresh load of every page has both closed. The figures in README.md come from build screenshots
taken with the capture step's own pre-screenshot sequence and viewports, without the click walk,
compared by the unmodified Stage 4 script against the same live references.

## Not published

`audit.json` — the pipeline's audit record. It carries absolute build paths from the capture machine.

## Verification after modification

- **Stage 4 pixel diff** (clean capture, 11 pages × 6 viewports, gate 0.95): 8/11 pass as measured; index, service-areas and 404 fall below solely at the static map frame, which the live capture shows blank. Excluding the map frames, every page scores 96.99–100%. Full table in README.md.
- **Stage 5 bundle audit** on this published tree: 12/12 gates; 0 hard, 2 soft (source-derived CSS).
- **Gate 13 (runtime off-origin)**, all 11 pages: 0 escapes, 0 off-origin navigations. Its formal verdict is FAIL because its exerciser is interrupted by in-site navigation, and it fails closed on an interrupted exercise; its count is therefore a floor.
- **Publish gate**: every page and every file (docs included) — no client identity in unfurl tags, no tracker, no live embed, no hotlinked or off-allowlist subresource, no credential, no internal hostname or tenant identifier, no unfilled placeholder.
- **Rendering and links**, served at this preview's subpath: 22/22 page-viewports render with 0 missing images and 0 local 404s; 0 relative links fail to resolve. All 16 visible static-map frames paint their image, with 0 off-origin requests.
