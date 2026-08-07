# Gating docs.nova-guide.com with Cloudflare Access (Zero Trust) — $0 setup

Goal: require approved team members to sign in before **any** page of
`docs.nova-guide.com` is served, without changing how the docs are built or hosted.
We put a login gate **in front of** the existing GitHub Pages deployment using the
**free** Cloudflare + Cloudflare Access (Zero Trust) plans.

> **Why not just make the repo private?** Making a GitHub repo private does **not**
> gate its GitHub Pages site — Pages serves publicly even from a private repo. Repo
> permissions protect the *source*, not the *published site*. That is exactly why we
> use Cloudflare Access instead.

---

## Current setup (as deployed today)

| Thing | Current state |
|---|---|
| **Docs repo** | `srhughes0730-cmyk/Nova-Guide` (**private**) |
| **Docs host** | **GitHub Pages** on that repo |
| **Docs generator** | **Docsify** — client-side JS; markdown rendered in the browser (no server build step; hash-router SPA served from an `index.html`) |
| **Docs URL** | `https://docs.nova-guide.com/` — HTTPS live, GitHub-issued cert, "Enforce HTTPS" on |
| **Docs DNS today** | `CNAME  docs → srhughes0730-cmyk.github.io` (at GoDaddy) |
| **Apex site** | `nova-guide.com` = the marketing site, separate **public** repo `nova-guide-site`, GitHub Pages, apex `A` records → GitHub Pages IPs (`185.199.108–111.153`), `CNAME www → srhughes0730-cmyk.github.io` |
| **DNS managed at** | **GoDaddy** (registrar + current DNS host) |

**What changes:** DNS for `nova-guide.com` moves to Cloudflare (nameserver switch),
`docs.` gets proxied through Cloudflare, and an Access application requires login.
**What does NOT change:** the docs repo, the Docsify build, the GitHub Pages hosting,
and the apex marketing site all stay exactly as they are.

---

## Cost

- Cloudflare **Free** plan (DNS + proxy + Universal SSL): **$0**
- Cloudflare **Zero Trust Free** plan: **$0**, up to **50 users**
- **One-time PIN** login method: **$0**, no identity provider setup required
- **Total: $0.** (Heads-up: Zero Trust onboarding may ask you to add a card on file
  even for the Free plan — you are not charged as long as you stay on Free / ≤ 50 seats.)

---

## Manual dashboard steps vs. repo steps

- **Everything below is done in the Cloudflare and GoDaddy dashboards.**
- **No repo changes are required.** The docs repo, its `CNAME` file
  (`docs.nova-guide.com`), and the Docsify files stay untouched.

---

## Step 0 — Before you start

- [ ] Confirm `https://docs.nova-guide.com/` currently loads over HTTPS (it does today).
      Cloudflare "Full" mode relies on GitHub's cert already being valid.
- [ ] Have your GoDaddy login ready (you'll change nameservers there).
- [ ] Decide the allow-list: either specific team emails, or an email **domain**
      (e.g. everyone `@your-team-domain.com`).

---

## Step 1 — Add nova-guide.com to a free Cloudflare account

- [ ] Create/sign in to a free account at <https://dash.cloudflare.com/sign-up>.
- [ ] **Add a site** → enter `nova-guide.com` → choose the **Free** plan.
- [ ] Cloudflare scans and **imports your existing DNS records**. Let it.

---

## Step 2 — VERIFY every imported record BEFORE switching nameservers

⚠️ **This is the highest-risk step.** Moving nameservers moves **all** DNS —
including email — to Cloudflare. If a record is missing, that service goes dark.
Do not proceed to Step 3 until this table matches.

Confirm Cloudflare imported all of these (add any it missed, **DNS** tab):

- [ ] `A  @  185.199.108.153`
- [ ] `A  @  185.199.109.153`
- [ ] `A  @  185.199.110.153`
- [ ] `A  @  185.199.111.153`
- [ ] `CNAME  www  → srhughes0730-cmyk.github.io`
- [ ] `CNAME  docs → srhughes0730-cmyk.github.io`
- [ ] **All email records** — any `MX`, and any `TXT` for `SPF`, `DKIM`, `DMARC`,
      plus domain-verification `TXT` records. **Email breaks if these are missing.**
- [ ] Any `AAAA` records you previously added (optional IPv6).

> Tip: open the current GoDaddy DNS zone in another tab and compare line-by-line.

---

## Step 3 — Switch nameservers at GoDaddy (one-time domain move)

- [ ] Cloudflare shows you **two nameservers** (e.g. `xxx.ns.cloudflare.com`).
- [ ] GoDaddy → your domain → **Nameservers** → **Change** → **Enter my own
      nameservers (custom)** → paste Cloudflare's two → **Save**.
- [ ] Back in Cloudflare, wait for the domain to show **Active** (minutes to a few
      hours; up to 24h). Cloudflare emails you when it's done.

Until it's Active, everything keeps resolving from GoDaddy, so there's no outage
window if Step 2 was done correctly.

---

## Step 4 — Set proxy status (orange vs. grey cloud)

In Cloudflare **DNS**, the cloud icon per record controls proxying. Access only
works on **proxied** hostnames.

- [ ] `docs` → **Proxied (orange cloud)** ← required for the gate
- [ ] `@` (apex) → **DNS only (grey cloud)** ← keep marketing site on GitHub's own TLS
- [ ] `www` → **DNS only (grey cloud)**
- [ ] Email `MX`/`TXT` → always **DNS only (grey)** (proxy doesn't apply to mail)

> Keeping the apex/www grey means the marketing site's GitHub Pages TLS and behavior
> are completely unaffected by this change. Only `docs.` goes through Cloudflare.

---

## Step 5 — TLS sequence so the docs don't break (GitHub Pages behind Cloudflare)

The one thing that breaks GitHub-Pages-behind-Cloudflare is an SSL **mode mismatch**.

- [ ] Confirm the docs cert is already provisioned (Step 0 — it is).
- [ ] Cloudflare → **SSL/TLS → Overview** → set encryption mode to **Full**.
      - **Full** = Cloudflare → GitHub origin over HTTPS. ✅ Correct here.
      - **Flexible** = Cloudflare → origin over HTTP. ❌ Causes a **redirect loop**
        with GitHub's "Enforce HTTPS". Do **not** use Flexible.
      - (**Full (strict)** also works, since GitHub serves a valid public cert — but
        **Full** is the safe default.)
- [ ] Leave GitHub Pages **"Enforce HTTPS" ON**. With SSL mode = Full this is
      consistent (both hops are HTTPS) and there is no loop.
- [ ] Optional: Cloudflare → **SSL/TLS → Edge Certificates** → enable
      **Always Use HTTPS** (edge-level http→https redirect).

> Expected quirk: after `docs.` is proxied, GitHub's **Pages → "DNS check"** may show
> a warning, because the hostname now resolves to Cloudflare IPs instead of GitHub's.
> This is normal and the site keeps serving — do **not** remove the custom domain in
> GitHub in response to that warning.

---

## Step 6 — Create the Access application (the actual gate)

- [ ] Cloudflare dash → **Zero Trust** → complete the free onboarding (pick a team
      name like `nova-guide`; choose the **Free** plan). Note your **team domain**
      `https://<team-name>.cloudflareaccess.com` — you'll need it for Google below.
- [ ] **Access → Applications → Add an application → Self-hosted and private → Public DNS.**
      > ⚠️ **Trap (learned the hard way):** pick the **Public DNS** tab, **not
      > "Private destinations."** The private path opens the form with
      > `?destination=private-ip` and silently enables a clientless browser-isolation
      > flag, so **Create** fails with:
      > `use_clientless_isolation_app_launcher_url can only be enabled for apps with private destinations`.
      > `docs.nova-guide.com` is a **public** hostname, so it must be Public DNS. If you
      > see that error, either turn OFF the **"Allow access through browser-based RDP,
      > SSH, or VNC sessions"** toggle, or Cancel and restart via the Public DNS tile.
- [ ] **Application configuration:**
      - Application name: `Nova Guide Docs`
      - Session duration: e.g. `24 hours`
      - **Destination (public hostname):** subdomain `docs`, domain `nova-guide.com`
        (path left blank = gate the entire site).
      - Leave the **browser-based RDP/SSH/VNC** toggle **OFF** — not needed for a docs gate.
- [ ] **Add a policy:**
      - Policy name: `Team`
      - Action: **Allow**
      - Configure rules:
        - **Emails** → add each approved address **individually** (exact match). ← use
          this for named people.
        - **Emails ending in** → `@your-team-domain.com` — only for gating a *whole*
          domain. **Do NOT use this for individuals** (e.g. `@gmail.com` would let in
          *every* Gmail user; a personal address typed here matches nobody).
        - **Email lists** → reusable group you maintain.
      > ⚠️ Two things that lock people out: (1) include the **Cloudflare-owner email**
      > so you don't lock *yourself* out; (2) every allowed email must be a **real
      > inbox** — `nova-guide.com` has no MX/email, so `@nova-guide.com` addresses can
      > never receive the one-time PIN.
- [ ] **Login methods / identity:**
      - **One-time PIN** is enabled by default and needs zero setup — approved users
        get a 6-digit code emailed to them each login (shown as a **"Cloudflare"**
        button on the login page — it is *not* a Cloudflare account; it just asks for
        their email and mails a code). Good enough to start.
      - *(Optional)* Add **Google** for a "Sign in with Google" button:
        - Find **Login methods** via the **⌘K Quick search** ("login methods" /
          "identity providers"). The dashboard moved it — in the current layout it's
          under **Integrations → Identity providers**, *not* "Settings → Authentication."
        - Add → **Google** (generic; not "Google Workspace" unless you use Workspace).
          It needs a **Google Cloud OAuth client** (Web application) with **Authorized
          redirect URI** = `https://<team-name>.cloudflareaccess.com/cdn-cgi/access/callback`.
          Paste the resulting **Client ID + Secret** into Cloudflare; leave PKCE off.
        - ⚠️ In Google Cloud, the **OAuth consent screen** must be **Published** (or add
          each teammate as a **Test user**) or they'll hit "access blocked / app not
          verified." Basic `email`/`profile`/`openid` scopes need no Google review.
      - With **"Accept all available identity providers"** ON in the app, any method you
        add (PIN, Google) appears automatically — the **Team** policy still governs who
        actually gets in.
- [ ] Save. Access begins enforcing within ~a minute.

---

## Step 7 — Verify

- [ ] **Incognito** window → visit `https://docs.nova-guide.com/`.
      You should hit the **Cloudflare Access login screen before any docs render.**
- [ ] Enter an **approved** email → receive the one-time PIN (or Google) → you're in,
      docs load normally.
- [ ] Try a **non-listed** email → you should be **denied**.
- [ ] Visit `https://nova-guide.com/` and `/privacy` and `/support` → the marketing
      site still loads publicly with **no** login prompt (apex stayed grey-cloud).
- [ ] `http://docs.nova-guide.com/` redirects to `https://` (no cert warning, no loop).

---

## Managing team members later

- **Add someone:** Zero Trust → **Access → Applications → Nova Guide Docs → policy
  `Team`** → add their email → Save. (Or add them to the Email list / rely on the
  domain rule.) They can log in immediately.
- **Remove someone:** remove their email from the policy/list → Save. Optionally
  **Zero Trust → Logs → Access** to review who's authenticated, and revoke sessions.
- **Free-plan limit:** up to **50 users**.

---

## Complications specific to THIS deployment — and how to handle them

1. **Nameserver move is the real risk, not the docs.** In practice this domain had
   **no MX and no apex SPF** (only a default `_dmarc` TXT), so email wasn't affected —
   but the Cloudflare import *did* also carry over records `dig` didn't show (a `pay`
   subdomain and `_domainconnect`). Mitigation: Step 2 — compare the import against
   GoDaddy's own DNS page, not just a `dig`, before switching in Step 3.
2. **Two GitHub Pages sites share one domain** (`nova-guide.com` apex = marketing,
   `docs.` = docs). Only proxy `docs.`; keep the apex/www grey so the marketing site's
   GitHub TLS is untouched (Step 4).
3. **Docsify is client-side.** Because the gate is at the network edge (Cloudflare
   intercepts the request before GitHub responds), it protects the HTML, the markdown
   files, and the JS equally — there's no "API" that bypasses the gate. No Docsify
   config changes needed.
4. **GitHub's DNS-check warning after proxying** is cosmetic (Step 5 note) — leave the
   custom domain in place.
5. **Don't use SSL "Flexible."** It's the one setting that will break the proxied docs
   (redirect loop with Enforce HTTPS). Use **Full** (Step 5).
6. **Add the app via "Public DNS," never "Private destinations."** The private path
   trips `use_clientless_isolation_app_launcher_url can only be enabled for apps with
   private destinations` on Create (Step 6).
7. **Policy selector matters.** Use **"Emails"** (exact) for individuals — **"Emails
   ending in"** is domain-wide and will either match nobody or let in an entire mail
   provider. Always include the owner email (Step 6).
8. **Team domain must match Google's redirect URI exactly.** The dashboard may show a
   legacy team subdomain on the login card while the live app uses another — trust the
   one the app actually redirects to, and register that in the Google OAuth client
   (Step 6). A mismatch surfaces as `redirect_uri_mismatch`.

---

## Rollback (if you ever want the docs public again)

- Zero Trust → **Access → Applications** → delete the `Nova Guide Docs` application
  (removes the gate; site becomes public again), **or**
- Cloudflare **DNS** → set `docs` back to **DNS only (grey)** to stop proxying.
- Nameservers can be pointed back to GoDaddy later if desired, but that's a separate,
  larger change and isn't needed just to remove the gate.


## Docs sidebar / content staleness (added 2026-08-07)

Symptom: docs.nova-guide.com shows an outdated sidebar or page content even
though the GitHub Pages deploy succeeded (observed 2026-08-07: sidebar ended at
ADR 0019 while the repo had 0021).

Cause: Docsify fetches `_sidebar.md` and page `.md` files at runtime as plain
assets, and Cloudflare's edge cache serves stale copies of them.

Fix (one-time):
1. Cloudflare dashboard → nova-guide.com zone → Caching → Configuration →
   **Purge Everything** (or purge by hostname `docs.nova-guide.com`).
2. Rules → Cache Rules → create rule "Docs bypass":
   - When: Hostname equals `docs.nova-guide.com`
   - Then: **Bypass cache**
   (The docs site is small and behind Access anyway — edge caching buys nothing
   and costs freshness.)

After that, every push to main appears on the site as soon as the
"Deploy docs to GitHub Pages" action finishes.
