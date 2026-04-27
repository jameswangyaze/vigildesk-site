# vigildesk.ai — Site Runbook

Operational reference for the public marketing site. Read this first whenever something goes wrong with the site.

---

## What This Site Is

Static 4-page compliance + marketing site at `https://vigildesk.ai`.
- Source: `github.com/jameswangyaze/vigildesk-site` (public)
- Hosted on **Cloudflare Pages** (free tier)
- DNS on **Cloudflare** (zone `vigildesk.ai`, expires 2028-04-20)
- SSL: **Cloudflare Universal SSL** via Google Trust Services (auto-renewing, 90-day certs)

## Architecture

```
       (you edit locally)
                ↓
         git push origin main
                ↓
     ┌──────────────────────────┐
     │  GitHub                  │
     │  jameswangyaze/          │
     │  vigildesk-site          │
     └─────────────┬────────────┘
                   │ webhook
                   ↓
     ┌──────────────────────────┐
     │  Cloudflare Pages        │
     │  Project: vigildesk-site │
     │  Build: (none, static)   │
     └─────────────┬────────────┘
                   │ deploy
                   ↓
     ┌──────────────────────────┐
     │  Cloudflare global edge  │
     │  vigildesk.ai            │
     │  www.vigildesk.ai        │
     └──────────────────────────┘
```

---

## Deploying a Change

```bash
cd ~/.openclaw/workspace/business-plans/vigil-desk/website
# edit any .html file
git add -A
git commit -m "describe the change"
git push origin main
# wait ~30-60 sec, refresh https://vigildesk.ai
```

That's the whole process. Cloudflare Pages auto-builds on push to `main`.

**Watch a deploy live:** https://dash.cloudflare.com → Workers & Pages → `vigildesk-site` → Deployments

---

## Preview Deploys (for risky changes)

Push to any branch other than `main` to get a preview URL like `<branch>.vigildesk-site.pages.dev`.

```bash
git checkout -b experiment-foo
# edit
git push -u origin experiment-foo
# preview URL appears in Cloudflare Pages dashboard within 1 min
```

When ready, merge to `main`:
```bash
git checkout main
git merge experiment-foo
git push origin main
```

---

## Rollback

If a deploy breaks something:
1. Cloudflare dashboard → `vigildesk-site` → **Deployments** tab
2. Find the last good deploy (status: Active or Success)
3. Click `⋯` next to it → **Rollback to this deployment**
4. Live within seconds; no git changes needed

After rollback, fix the issue locally, re-commit, re-push.

---

## SSL Certificate Auto-Renewal

### How it works
- Cloudflare auto-issues + auto-renews **Universal SSL** certs for both `vigildesk.ai` and `www.vigildesk.ai`
- Certs are issued for **90 days** at a time
- Cloudflare renews them **~30 days before expiry**, with zero downtime
- Continues as long as: domain stays "Active" in Pages, DNS records remain in place, account is in good standing

### Current cert
- **Issued:** 2026-04-27
- **Expires:** 2026-07-26 (90 days)
- **Issuer:** Google Trust Services / WE1
- **Expected next renewal:** ~2026-06-26

### Renewal verification calendar
After each renewal, re-arm the next check.

| Cert # | Issued | Expires | Verify by | Status |
|---|---|---|---|---|
| 1 | 2026-04-27 | 2026-07-26 | 2026-07-10 | Pending verify |
| 2 | (auto)     | (auto)     | +90d       | — |
| 3 | (auto)     | (auto)     | +90d       | — |

### How to verify a renewal completed
```bash
echo | openssl s_client -servername vigildesk.ai -connect vigildesk.ai:443 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates
```

The `notAfter` date should have rolled forward by ~90 days from the previous value.
Then update the table above.

### What to do if a renewal does NOT happen
1. Check Cloudflare → Notifications → SSL/TLS → Universal SSL alerts (should be enabled)
2. Confirm `vigildesk.ai` is still "Active" in Pages → Custom domains
3. Confirm DNS records still present (check **DNS** tab on the zone)
4. If anything is missing, re-add the custom domain in Pages (it auto-recreates DNS + cert)
5. Worst case: file a ticket with Cloudflare support — Universal SSL renewal failures are rare and they fix them within hours

### Backup monitoring (optional)
Set up at https://uptimerobot.com (free 50 monitors):
- Add `https://vigildesk.ai` as an HTTPS monitor
- Enable "SSL certificate monitoring" with 30-day expiry alert
- Sends email if cert expiry < 30 days OR any HTTPS request fails

---

## DNS Records (read-only reference)

In Cloudflare DNS for zone `vigildesk.ai`:

| Type | Name | Content | Proxy |
|---|---|---|---|
| CNAME | @ | vigildesk-site.pages.dev | Proxied |
| CNAME | www | vigildesk-site.pages.dev | Proxied |
| (NS records) | — | jim.ns.cloudflare.com, elinore.ns.cloudflare.com | — |

**Don't edit these by hand.** They're managed by the Cloudflare Pages "Custom domains" UI. Editing manually can break the integration.

---

## Email (`hello@vigildesk.ai`)

Currently **not yet configured.** Site advertises this address but emails will bounce.

To set up:
1. Cloudflare dashboard → `vigildesk.ai` zone → **Email** → **Email Routing**
2. Enable Email Routing → Cloudflare adds MX + SPF records automatically
3. Add destination address: `jameswangyaze@gmail.com` (verify via the email Cloudflare sends)
4. Add custom address rule: `hello@vigildesk.ai` → forward to `jameswangyaze@gmail.com`

Free, ~5 min, no ongoing cost.

---

## Common Issues

| Symptom | Likely cause | Fix |
|---|---|---|
| `vigildesk.ai` returns NXDOMAIN | Custom domain removed from Pages | Re-add in Pages → Custom domains |
| `vigildesk.ai` returns 522 / 525 / 526 | SSL handshake issue | Wait 5 min for cert provision; if persists, re-add custom domain |
| New commit not deploying | GitHub webhook broken | Pages → Settings → Builds → "Retry build" or reconnect repo |
| Wrong content live | Wrong branch deployed | Pages → Settings → Production branch should be `main` |
| All HTTPS requests fail | Cloudflare zone paused | Cloudflare zone overview → "Resume" |

---

## Backups

The site is its own backup — repo is in GitHub, deploy artifacts are in Cloudflare Pages history (last 100+ deploys retained). Even if Cloudflare Pages were lost entirely:
1. Pull the repo: `git clone git@github.com:jameswangyaze/vigildesk-site.git`
2. Re-deploy to any static host (Netlify, Vercel, GitHub Pages, S3) — it's just HTML

---

## Cost

Currently **$0/month** (Cloudflare Pages + DNS + SSL all free tier).

Cost would only ever appear if:
- Bandwidth exceeds 100 GB/month (Pages free tier limit) — extremely unlikely for a static site this small
- We add a Pro plan feature (Workers, Stream, Images) — not needed for this site
