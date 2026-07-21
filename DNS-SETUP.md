# GoDaddy DNS setup for nova-guide.com → GitHub Pages

Where to do this: GoDaddy → **My Products** → find `nova-guide.com` → **DNS** →
**Manage DNS / Manage Zones** → *DNS Records*.

Replace `srhughes0730-cmyk` below only if you deploy the site from a **different**
GitHub account/org. It should be the account that owns the site repo, followed by
`.github.io` (no repo name).

---

## Step 1 — Remove conflicting records first

GoDaddy usually ships a parked-page record. Before adding anything, **delete**:

- [ ] Any **A** record with **Name = `@`** that points to a GoDaddy IP (parked page)
- [ ] Any **CNAME** record with **Name = `www`** that points to a GoDaddy domain
      (e.g. `@`, `_domainconnect`, or a `godaddy` forwarding target)
- [ ] Any existing **Forwarding** on the domain (Domain settings → Forwarding → remove)

Leave any `MX`, `TXT`, or email-related records alone unless you know you don't need them.

---

## Step 2 — Add these records

### Four A records (apex domain → GitHub Pages)

| Type | Name | Value            | TTL      |
|------|------|------------------|----------|
| A    | `@`  | `185.199.108.153`| 600 secs |
| A    | `@`  | `185.199.109.153`| 600 secs |
| A    | `@`  | `185.199.110.153`| 600 secs |
| A    | `@`  | `185.199.111.153`| 600 secs |

- [ ] A `@` → `185.199.108.153`
- [ ] A `@` → `185.199.109.153`
- [ ] A `@` → `185.199.110.153`
- [ ] A `@` → `185.199.111.153`

### One CNAME record (www → your GitHub Pages host)

| Type  | Name  | Value                        | TTL      |
|-------|-------|------------------------------|----------|
| CNAME | `www` | `srhughes0730-cmyk.github.io`| 600 secs |

- [ ] CNAME `www` → `srhughes0730-cmyk.github.io`   *(note the trailing behavior — GoDaddy adds the dot automatically)*

### Optional — four AAAA records (IPv6, future-proofing)

Not required, but GitHub supports it. Add if you like:

| Type | Name | Value                    |
|------|------|--------------------------|
| AAAA | `@`  | `2606:50c0:8000::153`    |
| AAAA | `@`  | `2606:50c0:8001::153`    |
| AAAA | `@`  | `2606:50c0:8002::153`    |
| AAAA | `@`  | `2606:50c0:8003::153`    |

- [ ] (optional) the four AAAA records above

---

## Step 3 — Point GitHub at the domain

- [ ] In the site repo: **Settings → Pages → Custom domain** = `nova-guide.com` → Save
      *(the repo's `CNAME` file already sets this, but saving here kicks off the TLS cert)*
- [ ] Wait for the green check / "DNS check successful" (minutes to a few hours)
- [ ] Tick **Enforce HTTPS**

---

## Step 4 — Verify

- [ ] `https://nova-guide.com` loads the landing page
- [ ] `https://www.nova-guide.com` redirects to the apex (GitHub does this automatically)
- [ ] `https://nova-guide.com/privacy` and `/support` load
- [ ] Padlock (HTTPS) shows with no warnings

DNS changes can take from a few minutes up to ~48 hours to fully propagate, though
GitHub Pages usually validates within an hour.

---

### Quick reference — the exact values

```
A      @     185.199.108.153
A      @     185.199.109.153
A      @     185.199.110.153
A      @     185.199.111.153
CNAME  www   srhughes0730-cmyk.github.io
```
