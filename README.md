# client-lavaleaforganics

Static preview bundle of **lavaleaforganics.com** (Lava Leaf Organics — Farmington, NM
cannabis dispensary) — cloned landing page, captured by the clone-site pipeline
(Clone Stages 1–5).

**Preview: https://sgencms.github.io/client-lavaleaforganics/**

Pure static. No build step, no dependencies, no backend. Open `index.html` or use the
preview link above.

> This is an **unofficial development copy** published for build review. It is not
> operated by, affiliated with, or endorsed by Lava Leaf Organics. The real site is
> https://lavaleaforganics.com/.

## Pages

| File | Source |
| --- | --- |
| `index.html` | `https://lavaleaforganics.com/` |

All CSS is inlined into a single `<style>` block in `<head>` (output-spec §C.2, SP shape),
so there is no sibling stylesheet to serve.

### What this page actually is

The source is a full single-page homepage behind a 21+ age gate: hero, product-category
grid, an Instagram strip, an FAQ accordion, a brands rail, a testimonials carousel, store
info with an embedded Google Map, and a long "first visit" ordering guide.

The capture was taken **past the age gate**. The gate's own markup and CSS are still in the
bundle, but the cookie it sets (`age_verification=verified`) was seeded at capture time, so
the page renders as a returning visitor sees it.

## Verification

Captured and checked by the pipeline, not by eye.

| Gate | Result |
| --- | --- |
| Stage 4 — pixel diff vs live source, 6 viewports | **PASS 6/6** |
| Stage 5 — bundle audit, re-run on **this published tree** | **11/12** |
| Stage 5 — Layer 2 assertions | **0 hard, 0 soft** |
| Stage 5 — Gate 13, runtime off-origin requests | **FAIL — 2 escapes (the Google Map)** |

Per-viewport pixel match against the live site, measured on the **hardened** bundle
(gate is 0.95):

| Viewport | Match |
| --- | --- |
| 1440×900 large desktop | 96.984% |
| 1280×800 standard desktop | 99.663% |
| 1024×768 large tablet | 99.641% |
| 768×1024 small tablet | 99.329% |
| 430×932 large phone | 99.796% |
| 390×844 standard phone | 99.656% |

These are re-derived from the **final published tree**, not carried over from an earlier
revision — a sibling preview's README once shipped gate numbers copied from a pre-flatten
bundle, so they are re-run here rather than reused.

The residual is **not** drift: it is the two genuinely dynamic regions of the page — the live
Instagram photo strip and the rotating testimonials carousel — which serve different content
on every load. Verified by reading the diff images, where the mismatch is confined to exactly
those two blocks. It is also why the per-viewport figures move between runs: an immediately
preceding run of the same bundle scored 99.808% at 1440 and 100.000% at 430. **Any single
run's number is a sample of that dynamic content, not a fidelity measurement to three
decimal places.** Every run has passed 6/6.

### Why Gate 1 fails

Gate 1 wants the pipeline's `project/` subdirectory. It is flattened to the repo root here
so GitHub Pages can serve `/client-lavaleaforganics/` directly. The capture is not wrong;
the published layout differs from the pipeline's by design.

### Why Gate 13 fails, and what it means

Gate 13 renders the bundle over HTTP and fails on any programmatic off-origin request. It
reports **2 escapes**, both from the embedded Google Map:

- `maps.googleapis.com/maps/api/js?key=…`
- `maps.gstatic.com/maps-api-v3/embed/js/…`

They are **not** tracking. They are the map the page renders. Removing them would blank a
visible part of the design this preview exists to review, so the map is left intact and
surfaced here rather than hidden — the same call made for the search form on
`client-treeoflifenv`.

**The honest cost:** that request carries the *client's own* Google Maps API key, so map
loads from this preview draw on the client's Maps quota. Nothing else in the bundle reaches
any third party.

Before hardening this gate reported **11** escapes, including
`https://lavaleaforganics.com/do_shopping/cart_state` — a call to the client's **production**
server that returns a session cookie, writing anonymous preview visitors into the client's
logs. That class is now blocked (see change 4).

## Changes made to the capture

This is a public copy of a client's page, so the following were changed from what was
captured. They are deliberate, and they are the only edits to the source markup. Every one
is recorded in `PROVENANCE.md`.

1. **`noindex, nofollow, noarchive, nosnippet`.** The source served
   `index, follow, max-image-preview:large`. A public duplicate of a client's page must not
   compete with the client's own site in search results.
2. **Social/unfurl metadata rewritten.** The source `og:`/`twitter:` tags carried the
   client's real title, `og:site_name`, description and a **hotlinked** logo, with `og:url`
   pointing at `lavaleaforganics.com`. `noindex` governs search indexers only — it does
   **not** stop link-preview crawlers (Slack, Teams, Facebook, X, LinkedIn, Discord,
   iMessage), so pasting this URL rendered a card indistinguishable from a share of the
   official dispensary, and drew image bandwidth off the client's own server. The tags now
   identify the page as an unofficial preview and point at this preview's own URL;
   `og:image`, `og:image:secure_url` and `twitter:image` were removed outright. Three
   `schema.org` JSON-LD blocks (`Organization`, `WebSite`, and a `Store` block carrying the
   full name/address/phone **and the internal staging hostname**) were removed for the same
   reason. None of this is rendered, so the pixel match is unaffected.
3. **Google Search Console token removed.** The source shipped a
   `google-site-verification` meta tag; a site-ownership proof must not be republished on a
   host the client does not control.
4. **All programmatic calls to the client's production API are blocked**, by a global
   transport guard in the first `<script>` of `index.html` that wraps `fetch`,
   `XMLHttpRequest` and `navigator.sendBeacon` and rejects anything off-origin. It guards
   the **transport, not each caller**, because the production URLs are injected as inline
   config (`Defaults.base_url`, `window.__SG_TRACK__`) and consumed by other files — a
   per-file guard could never cover them.
5. **First-party SGEN analytics removed.** The source carried
   `window.__SG_TRACK__ = {"url":"https://lavaleaforganics.com/sg-collect",…}` and shipped
   `assets/front/js/sg-analytics.js`, which POSTs pageviews and interaction events to the
   client's collector. Both are gone, along with the mirrored `sg-collect.html` stub.
6. **Google Tag Manager removed.** A live `<noscript><iframe>` pointing at the client's real
   container (`GTM-NPSTL9W8`) would have fired the client's analytics on every preview
   visit. The tag is gone, and the orphaned GTM (849 KB), Microsoft Clarity (4 hosts) and
   `app.tryumbrella.com` (365 KB) payloads were pruned from `_xorigin/`.
7. **Client back-office path blanked.** `Defaults.admin_url` disclosed
   `https://lavaleaforganics.com/sg-admin/`. `base_url` and `current_url` were blanked with it.
8. **Hotlinked images localised.** Nine images — the logo and the eight Instagram tiles —
   were still being fetched from the client's server at render time. They now resolve to
   byte-identical local copies, matched by **content hash** (filename matching would have
   picked the wrong files: collision-renaming meant `5.webp` and `5-2.webp` are different images).
9. **Search form action removed.** It posted the visitor and their query to
   `https://lavaleaforganics.com/search`, which answers and mints a session.
10. **`no-referrer` referrer policy.** Outbound navigation still goes to the client's real
    site by design — but without this the preview's URL travels as the `Referer` and lands
    in the client's own analytics as an unexplained traffic source.

`<link rel="canonical">` still points at `https://lavaleaforganics.com/` — correct, since
the client's page is the canonical original. This is deliberately **not** treated the same
way as `og:url`: canonical consolidates search signal to the real site, whereas `og:` drives
unfurl cards that must not impersonate the business.

`audit.json` is **not** published. It carries absolute build paths from the machine that
made the capture.

## Known limits

- **The cart drawer stays empty.** It hydrates from `/dispenza/ajax/cart_html` on a backend
  that does not exist here, and the same-origin guard now refuses the call. It renders as an
  inert empty shell and nothing leaves the browser. No static bundle can satisfy it.
- **Navigation still leaves the preview.** Clicking a nav link (HOME, SHOP, ABOUT, CONTACT,
  ORDER NOW) takes you to the real site. That is intended for a preview, and is *not* blocked
  by the guard, which covers programmatic requests only. The `no-referrer` policy keeps this
  preview's URL out of the client's analytics.
- **The favicon is the client's logo**, served from a local copy. The browser tab therefore
  shows the client's mark.

## Residual risks not addressed here

Recorded so they are not mistaken for oversights.

- **No on-page disclosure.** Nothing *rendered* tells a visitor this is not the official
  site — the disclosure lives in `<head>` metadata and in this file. A visible banner would
  fix it but would break the pixel fidelity that is the point of the artifact.
- **The public org exposes the client roster.** `SGENCMS` hosts several public `client-*`
  Pages sites naming other businesses. That is an org-level hosting decision.
- **No framing or CSP headers.** GitHub Pages cannot set response headers, so this
  pixel-accurate replica can be embedded in an iframe by anyone.
- **The commit metadata is public**, including the committer email.
- **The Google Map draws on the client's Maps API key** (see Gate 13 above).
- **Proprietary webfonts are re-served.** `_xorigin/fonts.gstatic.com/s/googlesans*/`
  contains ~100 Google Sans / Google Sans Text `.woff2` files, captured because the mirrored
  Maps embed references them. Google Sans is **not** an open-licensed family like the rest of
  Google Fonts, and re-serving it from a third-party host is not clearly permitted. They were
  left in place rather than deleted because they *are* referenced — removing them would change
  what the map renders — so this is flagged as a decision, not treated as settled.

## License / ownership

All site content, imagery, trademarks and branding belong to Lava Leaf Organics. This
repository is an unaffiliated development artifact and asserts no rights over them.
