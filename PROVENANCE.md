# PROVENANCE

A byte-faithful capture, then deliberately modified for public hosting. Because it was
modified, **this bundle is no longer a faithful record of what the server sent**, and must
not be cited as one. Every deviation is listed below.

## Capture

| | |
| --- | --- |
| Source | `https://lavaleaforganics.com/` |
| Captured | 2026-09-10 |
| Method | clone-site pipeline, Clone Stages 1–5 (Playwright + CDP) |
| Shape | SP (single page); all CSS inlined per output-spec §C.2 |
| Age gate | Capture taken past the 21+ gate by seeding `age_verification=verified`, the cookie the source's own `age-verification.min.js` sets on "YES". No other cookie was seeded. |

## Deviations from the captured bytes

Ordered as applied. "Rendered" = whether the change can affect the pixel output.

| # | Change | File | Rendered? |
| --- | --- | --- | --- |
| 1 | Global transport guard + `<meta name="referrer" content="no-referrer">` inserted as the first children of `<head>` | `index.html` | No |
| 2 | `<title>` prefixed `UNOFFICIAL PREVIEW — ` | `index.html` | No |
| 3 | `meta[name=description]` replaced with preview disclosure text | `index.html` | No |
| 4 | `og:title`, `og:url`, `og:site_name`, `og:description` rewritten to preview identity | `index.html` | No |
| 5 | `og:image`, `og:image:secure_url`, `twitter:image` **deleted** (hotlinked the client's logo from the client's server) | `index.html` | No |
| 6 | `twitter:card` → `summary`; `twitter:title`, `twitter:description` rewritten | `index.html` | No |
| 7 | `robots` `index, follow, max-image-preview:large, max-snippet:-1, max-video-preview:-1` → `noindex, nofollow, noarchive, nosnippet` | `index.html` | No |
| 8 | `meta[name=google-site-verification]` **deleted** | `index.html` | No |
| 9 | 3 × `schema.org` JSON-LD blocks **deleted** (`Organization`, `WebSite`, and a `Store` block with full NAP + the internal staging hostname) | `index.html` | No |
| 10 | `Defaults.base_url`, `Defaults.admin_url`, `Defaults.current_url` blanked (`admin_url` disclosed `/sg-admin/`). `age_verification_expiry` left intact. | `index.html` | No |
| 11 | `window.__SG_TRACK__` config **deleted** | `index.html` | No |
| 12 | Search form `action="https://lavaleaforganics.com/search"` removed; original value preserved as `data-action-removed-at-publish` | `index.html` | No |
| 13 | GTM `<noscript><iframe src="…googletagmanager.com/ns.html?id=GTM-NPSTL9W8">` **deleted** | `index.html` | No |
| 14 | 8 hotlinked Instagram images rewritten to local copies, matched by md5 | `index.html` | No (byte-identical files) |
| 15 | `assets/front/js/sg-analytics.js` **deleted** (11,239 B) | — | No (was unreferenced) |
| 16 | `sg-collect.html` **deleted** (0 B stub) | — | No |
| 17 | `grep.exe.stackdump` **deleted** (1,729 B) — a crash dump from the build machine's own tooling, not site content | — | No |
| 18 | `_xorigin/app.tryumbrella.com/` **deleted** (365 KB) | — | No |
| 19 | `_xorigin/www.googletagmanager.com/` **deleted** (849 KB) | — | No |
| 20 | `_xorigin/{c,www,z,scripts}.clarity.ms/` **deleted** (4 hosts) | — | No |
| 21 | `_xorigin/lavaleaforganics.staging.sgen.com/` **deleted** (8 files) — captured from SGEN's internal staging host. Unreferenced: every one of those images already resolves via a local `assets/in-pages/` copy, verified per filename before deleting. | — | No |
| 22 | Internal staging hostname removed from an inline accessibility script's CSS selector (`[href$="…staging.sgen.com/"]` → the public host) | `index.html` | No |
| 23 | `sites/sgen_lavaleaforganics_staging_sgen_com/` renamed to `sites/media/`, and its 23 image references rewritten | `index.html` + tree | No (same bytes, new path) |
| 24 | `data-clone-stage-3="sp-inline"` attribute stripped from the `<style>` tag — an internal build-pipeline marker. The sibling published preview ships no such attribute; this one does not either. | `index.html` | No |
| 24a | **`_xorigin/maps.googleapis.com/` deleted (18 files, 1.6 MB) — it contained a live signed Google service-account bearer JWT.** See below. | — | No (0 references) |
| 24b | `_xorigin/places.googleapis.com/` deleted (1 file) — a captured Places RPC response holding the client's business record. Unreferenced. | — | No |
| 24c | `_xorigin/maps.gstatic.com/` deleted (2 files, 253 KB) — unreferenced; the map loads these from Google directly. | — | No |
| 25 | `.nojekyll`, `.gitattributes`, `.gitignore`, `README.md`, this file added | — | No |
| 26 | `project/` flattened to repo root so Pages serves `/client-lavaleaforganics/` | — | No |

Changes 21–23 remove **SGEN's internal tenant slug and staging hostname**. The client's public
site already serves `sites/sgen_lavaleaforganics_com/…` so that path is mirrored verbatim per
output-spec §K.1 and kept; the `…_staging_sgen_com` variant is *not* public on the client's site
and is not reproduced here. This is a deliberate departure from strict path-mirroring, made
because the bundle is published to a public host.

## Unreferenced files that DO ship

An orphan sweep (nothing in the HTML/CSS/JS references them) leaves 34 files / ~2.2 MB:
Google Maps API JS chunks and unused Google font weights under `_xorigin/`, plus extra captured
images under `assets/full-library/`. They are kept because `assets/full-library/` is the
pipeline's by-design captured-image library, and the Google files are public CDN assets with no
disclosure value. They cannot affect rendering. Everything unreferenced that *did* carry
disclosure value — the tag-manager, Clarity and tryumbrella captures, the staging-host images,
the tooling crash dump — was deleted (changes 15–21).

Change 14 was resolved by **content hash via the Stage 1 manifest**, not by filename:
collision-renaming during emit means `5.webp` and `5-2.webp` are different images, so
filename matching would have silently substituted the wrong picture in four places.

## Deliberately NOT changed

| | Why |
| --- | --- |
| `<link rel="canonical">` → `https://lavaleaforganics.com/` | Correct for a duplicate: consolidates search signal to the real site. Deliberately treated differently from `og:url`, which drives unfurl cards and must not impersonate the business. |
| Outbound nav links to the client's site | Intended behaviour for a preview. Mitigated by `no-referrer`, not blocked. |
| Embedded Google Map | Rendered content. Removing it would blank a visible region of the design under review. It is the sole remaining off-origin request (Gate 13 reports 2 escapes) and it uses the **client's** Maps API key. |
| Favicon (client logo, local copy) | Matches house practice on sibling previews. |
| `/* === source: https://lavaleaforganics.com/assets/… === */` comments in the inline CSS | Inert provenance markers recording which stylesheet each block came from. They are comments, not loads — verified. |
| Cart endpoint URLs in inline config | Left in place as a record of what the source did; neutralised at the transport layer by the guard rather than by editing them out. |
| `<meta name="sgen_version" content="SGEN v1.0">` | Verified present on the **live public client site** and on the sibling published preview, so it discloses nothing that is not already public. Mirrored verbatim. |
| `_xorigin/fonts.gstatic.com/s/googlesans*/` (~100 `.woff2`) | Google Sans is proprietary, not open-licensed, and re-serving it from a third-party host is questionable. **Not** deleted, because the mirrored Maps embed references these files and removing them would change what the map renders. Flagged in README as a decision, not settled. |

## A captured credential, and why history was rewritten

`_xorigin/maps.googleapis.com/$rpc/…/GetPlaceWidgetMetadata.html` contained a **live signed
RS256 bearer token** minted by Google for the crawling session:

```
iss/sub : maps-js-embed-internal@place-ui-kit-js-internal.iam.gserviceaccount.com
scope   : https://www.googleapis.com/auth/maps-platform.places.details
iat/exp : 1789033382 / 1789036982  (one-hour validity)
```

This is the "captured third-party credential" case: Stage 1 mirrors **response bodies** from
third-party endpoints, and those responses are minted for *the machine doing the crawl*. It was
found by an adversarial pre-publish audit, not by the pipeline — the bundle passed every static
gate with the token inside it.

**The token was already expired** when it was found (~10 minutes past `exp`), and it is Google's
own internal service account, not the client's. It was nevertheless removed and the repository's
single commit was **amended and force-pushed**, because a forward commit does not remediate a
disclosure: the old blob stays fetchable at `raw.githubusercontent.com/<org>/<repo>/<old-sha>/…`.
Stated plainly: a force-push makes the old commit unreachable, but GitHub may retain unreachable
objects addressable by SHA until they are garbage-collected. Treat the token as having been
briefly public.

**Durable pipeline fix, still outstanding:** Clone Stage 1 should refuse to persist response
bodies from analytics / ad / identity / RPC hosts at capture time, so this class never reaches a
bundle.

## Not published

`audit.json` — the pipeline's own audit record. It carries absolute Windows build paths
from the capture machine. Gate numbers derived from it are quoted in `README.md` instead.

## Verification after modification

- Stage 4 pixel diff re-run on the FINAL published tree: **PASS 6/6** (96.984 / 99.663 /
  99.641 / 99.329 / 99.796 / 99.656). Hardening did not degrade fidelity. Figures move between
  runs because two page regions are genuinely dynamic; every run passed 6/6.
- Stage 5 on the published tree: **11/12** (Gate 1 fails on the flattened layout, by design).
- Gate 13 runtime off-origin: **11 escapes → 2**, the remainder being the Google Map.
