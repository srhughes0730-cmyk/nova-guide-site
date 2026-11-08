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
| `schools/approve.html` | Step-by-step Google Admin console approval guide | `https://nova-guide.com/schools/approve` |
| `schools/schools.css` | Shared styles (incl. print rules) for both For Schools pages | — |
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

- **Beta (now):** set `#get-link`'s href in `get.html` to the public TestFlight
  invite URL (App Store Connect → TestFlight → your app → Public Link). Keep the
  "Join the TestFlight beta" button.
- **Go-live:** in `get.html`, point `#get-link` at the App Store product URL
  (`https://apps.apple.com/app/idXXXXXXXXXX`) and swap the beta button for the
  official "Download on the App Store" badge — the markup is stubbed in an HTML
  comment right below the beta button. The QR code does **not** change.

> Note: Apple's marketing guidelines reserve the official "Download on the App
> Store" badge for links to an actual App Store product page, so it's used only
> at go-live — not for the TestFlight link during beta.

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
