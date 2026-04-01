# Choice Properties — Developer Reference

---

## ⛔ MANDATORY — AI AGENTS READ THIS FIRST

**This file is loaded automatically into every Replit Agent session.**

This project runs on **Replit** using `server.js` (Express) to serve static files and generate `config.js` from environment variables. The backend is **Supabase cloud** — no local database.

### How it runs on Replit:
```
node server.js → serves static HTML/JS on port 5000 → Replit proxies to browser
```
### Deployment pipeline (separate from Replit):
```
Edit files in Replit → Push to GitHub → Cloudflare Pages auto-deploys → Live site
```

### You are PROHIBITED from:

| Action | Why |
|---|---|
| Installing any ORM (Drizzle, Prisma, Sequelize) | Backend is Supabase cloud. No local database. No migrations. |
| Running `db:push`, `db:migrate` | Same reason. Replit DB env vars (DATABASE_URL, PGHOST etc.) are unused ghost variables — ignore them. |
| Running `wrangler` or any Cloudflare CLI | Deployment is automatic via GitHub push. |
| Running `git push` or `git commit` | Owner handles all git operations. |
| Modifying `generate-config.js`, `_headers`, `SETUP.sql`, `js/cp-api.js`, `js/apply*.js` | Protected files. Owner approval required. |
| Committing `config.js` | Generated at runtime by server.js — gitignored. Never commit it. |

### You ARE allowed to:
- Edit `.html`, `.css`, and `.js` files in the frontend
- Edit Supabase Edge Functions in `supabase/functions/` (Deno TypeScript)
- Read any file in the project

**Full rule set:** `.agents/instructions.md`

---

## What This Project Is

**Choice Properties** is a static property rental marketplace.

| Item | Value |
|---|---|
| Frontend | Static HTML + Vanilla JS (no framework, no build tools) |
| Backend | Supabase cloud (PostgreSQL, Auth, Edge Functions, Storage) |
| Email | Google Apps Script relay |
| Images | ImageKit CDN |
| Hosting | Cloudflare Pages (auto-deploys on GitHub push) |
| Editor | Replit (code editor only — not a deployment target) |

---

## Key Files

| File | Purpose |
|---|---|
| `config.js` / `config.example.js` | All env vars and feature flags |
| `generate-config.js` | Build script — runs `node generate-config.js` at deploy |
| `js/cp-api.js` | Shared API client — all methods return `{ ok, data, error }` |
| `js/apply.js` | Rental application core (multi-step form) |
| `js/imagekit.js` | ImageKit upload client |
| `landlord/new-listing.html` | 6-step property listing wizard |
| `apply.html` | 7-step rental application form |
| `SETUP.sql` | Complete database schema (one file, one run — idempotent) |
| `ISSUES.md` | Issue registry (0 open issues) |
| `ARCHITECTURE.md` | Full system architecture |
| `.agents/instructions.md` | Mandatory AI agent rules |

---

## Supabase Secrets (set in Supabase Dashboard → Edge Functions → Secrets)

| Secret | Purpose |
|---|---|
| `GAS_EMAIL_URL` | Google Apps Script relay URL |
| `GAS_RELAY_SECRET` | HMAC secret for email relay |
| `IMAGEKIT_PRIVATE_KEY` | ImageKit private key (server-side uploads only) |
| `DASHBOARD_URL` | Public site root URL — used in lease signing links |
| `ADMIN_EMAIL` | Admin notification email from process-application |
| `FRONTEND_ORIGIN` | Exact site origin (no trailing slash) for CORS |
| `IMAGEKIT_URL_ENDPOINT` | ImageKit URL endpoint (e.g. `https://ik.imagekit.io/yourID`) |

**Note:** `DASHBOARD_URL` and `ADMIN_EMAIL` are Supabase secrets (server-side).
`ADMIN_EMAILS` (plural) is a Cloudflare Pages env var (UI display only — not security).

---

## Database Functions (Supabase)

| Function | Caller | Purpose |
|---|---|---|
| `get_application_status(app_id)` | public | Returns full tenant status for dashboard |
| `get_lease_financials(app_id, last_name)` | public (gated) | Returns financial data for lease signing |
| `claim_application(app_id, email)` | authenticated applicant | Links legacy app to OTP-auth account |
| `get_my_applications()` | authenticated applicant | Returns all apps linked to caller |
| `sign_lease_applicant(app_id, sig)` | token-gated | Records primary applicant signature |
| `sign_lease_co_applicant(app_id, sig)` | token-gated | Records co-applicant signature |
| `mark_expired_leases()` | admin or cron | Bulk-marks stale leases as expired |
| `generate_property_id()` | authenticated | Generates PROP-XXXXXXXX IDs |
| `generate_app_id()` | authenticated | Generates CP-YYYYMMDD-XXXXXXNNN IDs |
| `increment_counter(table, id, col)` | anon/auth | Increments property view counts |

---

## CSS Architecture

All styles split by concern, loaded in this order on every page:

| File | Scope | Version |
|---|---|---|
| `css/main.css` | Design tokens, base resets, shared components | v16 |
| `css/mobile.css` | Responsive layer (loaded last) | v16 |
| `css/listings.css` | Homepage hero, property grid, filters | v16 |
| `css/property.css` | Gallery mosaic, lightbox, detail layout | v16 |
| `css/apply.css` | Multi-step application form wizard | v16 |
| `css/admin.css` | Dark-themed admin dashboard | v16 |
| `css/landlord.css` | Landlord portal | v16 |

**Cache-busting rule:** Cache busting is now **automated**. HTML files use `?v=__BUILD_VERSION__` tokens that `generate-config.js` replaces with `Date.now()` at every Cloudflare Pages build. Do NOT manually edit `?v=` strings — they will be overwritten on the next deploy. Do NOT replace `__BUILD_VERSION__` with a hardcoded number.

---

## JavaScript Rules

- Vanilla JS only — no frameworks, no bundlers, no ES module imports on public pages
- All Supabase calls go through `cp-api.js` — never call `supabase` directly in page scripts
- No `process.env` in frontend code — all config comes from `config.js` loaded in `<head>`
- All images via `CONFIG.img(url, preset)` — never raw Supabase storage URLs
- Use `CP.UI.toast()` for notifications, `CP.UI.cpConfirm()` for confirm dialogs
- `window.confirm()` and `alert()` are banned

---

## CRITICAL — Do Not Touch

- **Payment flow** — `apply.html` payment copy, `mark-paid` Edge Function, `cp-api.js markPaid()`, `payment_status` logic in `admin/applications.html`. Owner-protected. Never change.
- **`apply.html` color scheme** — do not change.
- **`build.js`** — deleted intentionally in Session 019. Do not recreate.
