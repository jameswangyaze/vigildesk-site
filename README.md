# vigildesk-site

Public marketing + compliance site for VigilDesk LLC, served at https://vigildesk.ai.

## Stack
- Static HTML
- Tailwind CSS via CDN (no build step)
- Hosted on Cloudflare Pages (auto-deploys on push to `main`)

## Pages
- `index.html` — landing page (hero, how it works, pricing, FAQ)
- `privacy.html` — Privacy Policy (CCPA compliant, sub-processor table)
- `sms-terms.html` — SMS Messaging Policy (Twilio TF-verification compliant)
- `terms.html` — Public Terms of Service

## Local preview
```
python3 -m http.server 8000
```
Visit http://localhost:8000.

## Deploy
Pushing to `main` triggers a Cloudflare Pages build. Custom domains: `vigildesk.ai`, `www.vigildesk.ai`.
