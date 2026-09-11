# client-lavaleaforganics

Static preview bundle of **lavaleaforganics.com** (Lava Leaf Organics — Farmington, NM
cannabis dispensary) — **all pages of the site**, captured by the clone-site pipeline
(Clone Stages 1–5) and hardened for public hosting.

**Preview: https://sgencms.github.io/client-lavaleaforganics/**

Pure static. No build step, no dependencies, no backend. Open `index.html` or use the
preview link above. Navigation between the pages stays inside the preview.

> This is an **unofficial development copy** published for build review. It is not
> operated by, affiliated with, or endorsed by Lava Leaf Organics. The real site is
> https://lavaleaforganics.com/.

## Pages

| File | Source |
| --- | --- |
| `index.html` | `https://lavaleaforganics.com/` |
| `about-us.html` | `https://lavaleaforganics.com/about-us` |
| `contact-us.html` | `https://lavaleaforganics.com/contact-us` |
| `farmington-nm-dispensary.html` | `https://lavaleaforganics.com/farmington-nm-dispensary` |
| `privacy-policy.html` | `https://lavaleaforganics.com/privacy-policy` |
| `service-areas.html` | `https://lavaleaforganics.com/service-areas` |
| `downtown-farmington.html` | `https://lavaleaforganics.com/service-areas/downtown-farmington` |
| `north-farmington.html` | `https://lavaleaforganics.com/service-areas/north-farmington` |
| `west-farmington.html` | `https://lavaleaforganics.com/service-areas/west-farmington` |
| `shop.html` | `https://lavaleaforganics.com/shop` |
| `404.html` | `https://lavaleaforganics.com/404` (the site's own "page not found" page) |

Shared styles live in `chrome.css`; each page carries its own inline styles.

### How "all pages" was determined

The page set is the site's own sitemap (`/sitemaps/sitemap-pages.xml`, 11 URLs), cross-checked
by crawling every internal link on every one of those pages. The crawl surfaced 22 further
URLs, and none of them is a separate page:

- ~20 are `/shop?dtche[...]=…` — filter and sort states of the same `/shop` menu, not pages.
- `/learn`, `/shop/accessories/accessories` and five `/shop/albuquerque/…` links **are broken on
  the live site itself**: each redirects to, or returns, the client's 404 page. In this preview
  they point at `404.html`, which is where a visitor to the real site ends up.
- One link is an unrendered template string (`{{site_url}shop…`) on the client's page, which also
  lands on their 404 page. Same treatment.

`robots.txt` disallows `/register`, `/login` and `/sg-admin/`; none of them were captured.

## Verification

Captured and checked by the pipeline, not by eye.

| Check | Result |
| --- | --- |
| Stage 4 — pixel diff vs live source, 11 pages × 6 viewports | **8 of 11 pass as measured; 3 are below 0.95 (index, service-areas, 404) — each solely because of the static map frame (see below). Outside the map frames every page is 96.99–100%.** |
| Stage 5 — bundle audit, re-run on **this published tree** | **12 / 12 gates; Layer 2: 0 hard, 2 soft (source-derived `transition: all` and a hardcoded colour in `chrome.css`)** |
| Gate 13 — runtime off-origin requests, all 11 pages | **0 off-origin requests, 0 off-origin navigations. Formal verdict FAIL: its exerciser clicks in-site links, which navigate between pages (22 times), and it fails closed when an exercise is interrupted** |
| Publish gate — per-page and tree-wide leak assertions | **PASS — 11 pages, every file (docs included)** |
| Rendering — every page at 2 viewports, served at this preview's subpath | **22 / 22 page-viewports render; 0 missing images; 0 local 404s** |
| Links — every relative link on every page resolves to a file in the bundle | **0 unresolved** |

Per-page pixel match against the live site (gate is 0.95):

| Page | Match, worst → best viewport | Outside the map frames |
| --- | --- | --- |
| `index.html` | 94.888% → 97.716% | 96.99% → 99.99% |
| `about-us.html` | 95.555% → 97.965% | 98.02% → 100.00% |
| `contact-us.html` | 98.736% → 99.372% | 100.00% |
| `farmington-nm-dispensary.html` | 100.000% | — (no map) |
| `privacy-policy.html` | 100.000% | — (no map) |
| `service-areas.html` | 92.981% → 95.559% | 100.00% |
| `downtown-farmington.html` | 99.453% → 99.774% | 100.00% |
| `north-farmington.html` | 99.525% → 99.823% | 100.00% |
| `west-farmington.html` | 99.484% → 99.808% | 100.00% |
| `shop.html` | 99.990% → 100.000% | — (no map) |
| `404.html` | 91.982% → 94.309% | 99.95% → 99.98% |

**Why three pages are below 0.95.** On those pages the live reference capture shows the map area
*blank*: the site loads its map lazily and it had not painted when the page was photographed. This
preview shows a static image of the map there (change 8) — which is what a real visitor to the live
site sees once the map loads. The third column measures each page with the map frames excluded:
everything else matches. The map was not blanked to pass the gate.

The pixel figures come from build screenshots taken with the capture step's own pre-screenshot
sequence (load, scroll the whole page, return to top, settle, then screenshot each viewport). The
pipeline's standard harness takes its screenshots *after* clicking every toggle on the page, which
left the site's accessibility drawer and mobile menu open in the captures and scored the same
pages 21–89% — a measurement artefact, not a difference in the pages. See PROVENANCE.md.

Where a page is below 100%, the residual is the site's genuinely dynamic content — the live
Instagram strip and the rotating testimonials — which serves different items on every load.

## Changes made to the capture

This is a public copy of a client's site, so the following were changed from what was captured.
They are deliberate. Every one is listed, with counts, in `PROVENANCE.md`.

1. **`noindex, nofollow, noarchive, nosnippet`** on every page. The source served
   `index, follow`. A public duplicate must not compete with the client's site in search.
2. **Social/unfurl metadata rewritten** on every page. `og:`/`twitter:` tags named the client and
   hotlinked their logo from their own server; `noindex` does not stop link-preview crawlers
   (Slack, Teams, Facebook, X, LinkedIn, Discord, iMessage), so a pasted link rendered a card
   indistinguishable from the official business. They now identify each page as an unofficial
   preview and point at the preview's own URL; social images were removed. `schema.org` JSON-LD
   blocks asserting the business identity were removed. None of this is rendered.
3. **Google Search Console ownership token removed.**
4. **All programmatic calls to the client's production systems are blocked** by a global
   transport guard, the first `<script>` on every page, wrapping `fetch`, `XMLHttpRequest` and
   `navigator.sendBeacon`. It guards the transport rather than each caller, because the
   production URLs arrive as inline configuration consumed by other files.
5. **First-party analytics, Google Tag Manager, Microsoft Clarity and an identity/ad-tech
   script removed**, along with their captured payloads.
6. **The client's back-office path blanked** from inline configuration.
7. **The live ordering menu is not embedded.** `/shop` loads the client's real, orderable menu from
   their e-commerce provider in an iframe. A public, unofficial copy must not host a working order
   flow, so it is replaced by an inert placeholder of exactly the same size (an empty frame that
   loads nothing). The live capture of this page shows that region blank anyway.
8. **Maps are static images.** The site's Google Maps embeds use the client's API key, which is
   locked to their own domain: on this preview every map showed Google's "Oops! Something went
   wrong" box while still sending requests with their key. Each map is now a static image of the
   same map as the live site renders it, in the same frame, so the layout is unchanged.
9. **Hotlinked images localised** to byte-identical local copies (matched by content hash, never
   by filename).
10. **Images that are broken for everyone were neutralised.** One page loads nine images from a
    third-party host whose TLS certificate is not valid for that hostname, so no browser can load
    them — they are broken on the live site too. Their URLs were replaced with a transparent
    placeholder so the preview does not send visitors to that host.
11. **Internal infrastructure identifiers removed**: a staging hostname and a tenant path segment
    that are not public on the client's site were replaced with neutral equivalents.
12. **Page structure repairs, so the multi-page bundle behaves like the site** (details in
    PROVENANCE.md): links between pages rewritten to the sibling files; canonical links restored to
    the client's URLs; links that the capture had corrupted repaired; each page's inline styles
    split back into the separate `<style>` elements the source uses, so a malformed rule in one
    cannot swallow the next (it had silently dropped a section's spacing on `contact-us`).
13. **Search form action removed**, and a **`no-referrer`** policy added so outbound navigation does
    not reveal this preview's URL to the client's analytics.

`<link rel="canonical">` points at the client's page on every page — correct for a duplicate,
and deliberately different from `og:url`, which drives unfurl cards.

`audit.json` is **not** published: it carries absolute build paths from the capture machine.

## Known limits

- **The cart and the shop menu are inert.** The cart hydrates from a backend that does not exist
  here; the menu is deliberately not embedded (change 7).
- **Maps are pictures**, not interactive maps (change 8).
- **Links to pages outside this set** (the cart, checkout, the online menu) still go to the real
  site. That is intended for a preview; the guard blocks programmatic requests, not navigation.
- **The favicon is the client's logo**, served from a local copy.

## Residual risks not addressed here

- **An earlier version of this repository remains fetchable by commit hash.** It was replaced by a
  force-push, which makes the old commit unreachable but does not delete it from GitHub. That
  version included an expired third-party credential and internal hostnames. Only deleting and
  recreating the repository removes it.
- **No on-page disclosure.** Nothing *rendered* says this is not the official site; the disclosure
  lives in `<head>` metadata and in this file. A banner would break pixel fidelity.
- **The public organisation lists other clients' previews.** An org-level hosting decision.
- **No framing or CSP headers** — GitHub Pages cannot set response headers.
- **Commit metadata is public**, including the committer email.

## License / ownership

All site content, imagery, trademarks and branding belong to Lava Leaf Organics. This repository
is an unaffiliated development artifact and asserts no rights over them.
