# Nova Guide — Website

Static marketing + legal site for **nova-guide.com**, hosted free on GitHub Pages.

## Contents

| File | Purpose | Live URL (once deployed) |
|------|---------|--------------------------|
| `index.html` | Landing page | `https://nova-guide.com/` |
| `get.html` | Download / "Get the app" page — **QR code target** | `https://nova-guide.com/get` |
| `privacy.html` | Privacy policy (**fill in placeholders first**) | `https://nova-guide.com/privacy` |
| `support.html` | Support / contact page | `https://nova-guide.com/support` |
| `schools/index.html` | For Schools — overview for school IT administrators (what the app reads, data handling, compliance FAQ) | `https://nova-guide.com/schools/` |
| `press/index.html`, `press/kit/` | Press page: coverage ("In the news"), messaging-kit descriptions, fast facts, logo and screenshot downloads, video, media contact. Linked from the homepage NFTE line and every footer | `https://nova-guide.com/press/` |
| `schools/approve.html` | Step-by-step Google Admin console approval guide | `https://nova-guide.com/schools/approve` |
| `schools/schools.css` | Shared styles (incl. print rules) for both For Schools pages | — |
| `media/nova-guide-overview-720p.mp4`, `media/nova-guide-overview-poster.jpg` | One-minute overview video on the homepage (`#demo`) and its poster frame. Self-hosted, click to play. Generated from the booth loop; see below | — |
| `schools/*.pdf` | Print-ready handouts generated from the two pages | `https://nova-guide.com/schools/nova-guide-for-schools.pdf` |
| `CNAME` | Tells GitHub Pages the custom domain | — |
| `404.html` | Fallback for unknown paths | — |
| `.nojekyll` | Serves files as-is (skips Jekyll processing) | — |
| `DNS-SETUP.md` | Exact GoDaddy DNS records to add | — |

## Before you publish

Search-and-replace these placeholders across `privacy.html` and `support.html`:

- `[LEGAL ENTITY / DEVELOPER NAME]`
- `[CONTACT EMAIL]`  (e.g. `support@nova-guide.com`)
- `[MONTH DAY, YEAR]`  (privacy policy dates)
- `[OPTIONAL MAILING ADDRESS]`
- Resolve the `[CONFIRM ...]` / `[REVIEW ...]` notes in `privacy.html`

The privacy policy is a **draft** — because the app serves minors and touches Google
account/Calendar data, have it reviewed by someone qualified before App Store submission.

## Deploy (GitHub Pages)

1. Create a new **public** repo, e.g. `nova-guide-site`.
2. Add all these files to the repo root and push to `main`.
3. Repo → **Settings → Pages** → Source: *Deploy from a branch* → `main` / `/ (root)` → Save.
4. Still in **Settings → Pages**, set **Custom domain** to `nova-guide.com` → Save
   (this file already includes the matching `CNAME`).
5. Add the DNS records in `DNS-SETUP.md` at GoDaddy.
6. Once DNS resolves, tick **Enforce HTTPS** in Settings → Pages.

The two URLs Apple asks for at submission:
- Privacy Policy URL → `https://nova-guide.com/privacy`
- Support URL → `https://nova-guide.com/support`

## Download page / QR code (`get.html`)

Print material (conference, flyers, posters) encodes a QR code pointing at
**`https://nova-guide.com/get`** — a stable URL that never changes, so the QR is
printed once and reused forever. All "Get the app" links across the site funnel
into this page, which holds the **only** external download link (`#get-link`).

**Updating the download target (the one place you ever touch):**

- **Now (1.0 live):** `#get-link` in `get.html` is the official "Download on the
  App Store" badge pointing at the App Store product page,
  `https://apps.apple.com/app/id6817147370`. If the download target ever changes,
  update that one `href`; the QR code does **not** change.
- *History:* during the beta `#get-link` was a "Join the TestFlight beta" button
  pointing at the public TestFlight invite link.

> Note: Apple's marketing guidelines reserve the official "Download on the App
> Store" badge for links to an actual App Store product page — never use it for a
> TestFlight link.

## For Schools pages (`schools/`)

The two pages are the public versions of `docs/product/school-admin-one-pager.md` and
`docs/product/school-admin-approval-guide.md` in the app repo — **those Markdown files are
the source of truth**; update them first, then mirror the change here. The PDF handouts
are printed from the pages themselves (print styles live in `schools.css`), so regenerate
them after any content change:

```bash
CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
"$CHROME" --headless --no-pdf-header-footer --print-to-pdf=schools/nova-guide-for-schools.pdf "file://$PWD/schools/index.html?print"
"$CHROME" --headless --no-pdf-header-footer --print-to-pdf=schools/nova-guide-approval-guide.pdf "file://$PWD/schools/approve.html?print"
```

## Overview video (`media/`)

The homepage video is the **web cut** of the trade-show booth loop in the app repo
(`docs/brand/showcase/`): it skips the intro and the QR codes and ends on an end card
with the disclaimer. It is self-hosted on purpose: a YouTube or Vimeo embed would load
third-party scripts and cookies, against the site's no-tracking promise. To share it
on YouTube or LinkedIn, upload the same file there and link to it rather than embedding.
It only downloads when someone presses play (`preload="none"`), and GitHub Pages serves
the byte ranges iPhone Safari needs.

To regenerate it, run these from the app repo root after changing the loop:

```bash
python3 docs/brand/tools/build_booth_loop.py
NOVA_CUT=web NOVA_W=1280 NOVA_H=720 NOVA_BITRATE=1500000 swift docs/brand/tools/record_booth_loop.swift docs/brand/showcase/nova-guide-booth-loop.html nova-guide-site/media/nova-guide-overview-720p.mp4 30
NOVA_CUT=web NOVA_W=1280 NOVA_H=720 swift docs/brand/tools/record_booth_loop.swift docs/brand/showcase/nova-guide-booth-loop.html --stills /tmp/poster 6.8
sips -s format jpeg -s formatOptions 80 -z 720 1280 /tmp/poster/frame-006.80.png --out nova-guide-site/media/nova-guide-overview-poster.jpg
```

Keep the MP4 well under 50 MB, where GitHub starts warning about large files.

## Press page (`press/`)

Low-volume by design. For each new article or release, add one `<article>` at the top of
"In the news": date · outlet, the outlet's own headline, one or two sentences **in our own
words**, and a link out. Never copy coverage text. Check our wording against the brand
guidelines (`docs/brand/brand-guidelines.md` §7 in the app repo): no outcome promises, no
audience defined by a condition, no unverified statistics. The "About Nova Guide" text is the
fixed messaging kit (§8); change it there first. Keep personal details about the founder to
what has already been published (name and school), and route press contact to the company
inbox. Releases we write ourselves can be hosted in full on this page.

`press/kit/` holds the downloads: logos copied from `docs/brand/assets/logo/`, and the App
Store captures as full-resolution JPEGs (`sips -s format jpeg -s formatOptions 90`) with
600 px thumbnails in `kit/thumbs/`. Refresh them when the app's screens change.
