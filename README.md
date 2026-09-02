# Atlas Marketing Site

The public marketing and legal pages for Atlas, split out of the main
[`z1-atlas`](https://github.com/snwokoagbara/z1-atlas) app repo so they can be
built and deployed on their own, independent of the FastAPI app's release
cycle. Six static pages, no build step, no server-side templating.

Split from `snwokoagbara/z1-atlas` @ `83c9dcf77b6081a48a91a485d359b4df0b722afd`
(2026-09-01), where these files lived at
`server_files/docker/forecast/static/`. That app repo still serves the same
six routes today (`app.py`'s `landing_page` / `privacy_page` /
`delete_my_data_page` handlers and friends) — nothing there has been removed
or repointed. Until the app repo is updated to defer to this one, content
edits made here need to be hand-applied to both places.

## Pages

| File | Route |
|---|---|
| `index.html` | `/` |
| `product.html` | `/product` |
| `security.html` | `/security` |
| `about.html` | `/about` |
| `privacy.html` | `/privacy-policy` |
| `delete-my-data.html` | `/delete-my-data` |

Every page shares five images at the repo root (`atlas-mark.png`,
`dodan-colour.png`, `shot-overview.jpg`, `shot-calllist.jpg`,
`shot-accountant.jpg`), referenced by root-absolute path (e.g. `/atlas-mark.png`)
so they resolve correctly regardless of which page is current.

There is no shared stylesheet or shared nav/footer partial — each page carries
its own inline `<style>` block, its own `<link>` to Google Fonts (Figtree),
and its own copy of the nav and footer markup. That's inherited from how these
pages already lived in the app repo (see that repo's `CLAUDE.md`, rule 5): if
you change the brand (font, colors, nav links), change all six by hand.

## Hosting

Flat static files — any static host works. The five pages other than
`index.html` need clean-URL routing (`/product` → `product.html`, etc.);
Netlify, Vercel, Cloudflare Pages and GitHub Pages all do this by default for
a file of the same name, and an nginx/Caddy `try_files` rule works too.

**Deployed on GitHub Pages** (repo Settings → Pages → Deploy from a branch →
`master` → `/`), `.nojekyll` present so GitHub serves the raw files instead of
running its default Jekyll build. Default URL:
`https://snwokoagbara.github.io/atlas-marketing/`.

**That default URL serves the site under a `/atlas-marketing/` path prefix,
not domain root — every asset and nav link here is root-absolute
(`/atlas-mark.png`, `/product`, ...) on purpose, to match the live
`atlas.ricroot.com` paths exactly, so on the bare `github.io` URL every one of
those 404s.** This only resolves correctly once a custom domain is attached
(Settings → Pages → Custom domain, plus a `CNAME` file here and a DNS record
at the registrar — none of which is set up yet). Until then, treat the
`github.io` URL as a build-succeeded check, not a working preview.

## Off-origin by design

This site does not run its own backend. Two things on `landing.html` and
`about.html` deliberately point at the live Atlas app instead of resolving
locally:

- **Sign in** (`/auth/login`) — the actual login page only exists in the app.
- **The demo-request / contact form** — posts to
  `https://atlas.ricroot.com/api/demo-request`, set via the `API_ORIGIN`
  constant near the bottom of each page's inline `<script>`. The form also
  omits a `Content-Type` header on purpose, so the cross-origin POST stays a
  CORS "simple request" — the app's `/api/demo-request` handler doesn't route
  `OPTIONS`, so a preflighted request would fail outright.

If the live app ever moves origins, update `API_ORIGIN` **and** the matching
`<form action="...">` in both files together — the constant can't reach the
no-JS fallback attribute.

## Local preview

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000/index.html` (or any other page directly by
filename — clean-URL routing isn't available on a bare file server).
