---
name: spl1t-deploy
description: >-
  Deploy spl1t to Cloudflare Workers with Wrangler OAuth, separately from
  GitHub. Use when the user asks to deploy, ship to CF, npm run deploy,
  wrangler deploy, update spl1t.pages.dev, or when GitHub Actions Deploy is
  mentioned. GitHub is not linked to CF anymore.
---

# spl1t GitHub vs Cloudflare deploy

update github on one hand and deploy on CF on the other hand (Wrangler OAuth login.)
github is not linked to CF anymore

These are **two independent actions**. A git push does not go live. `npm run deploy` does not push git.

## Do not

- Do not run GitHub Actions **Deploy** (`gh workflow run Deploy`). The workflow token is not linked to Cloudflare.
- Do not treat CI on push as a production deploy. CI is types/lint/format/tests only.
- Do not deploy Prisma/Postgres PR branches (`pr/*` based on upstream). Production is **`main`** (OpenNext + D1).

## GitHub (source)

Commit and push as the user already instructed (`git push origin <branch>` after approval when required). That updates GitHub only.

## Cloudflare (live)

Live site: [https://spl1t.pages.dev](https://spl1t.pages.dev) (Pages proxy → Worker `spl1t`).

1. Be on **`main`** with the Worker/D1 tree (`wrangler.jsonc`, `open-next.config.ts`, `npm run deploy`).
2. Confirm Wrangler OAuth: `npx wrangler whoami` (email `contti@gmail.com`, account `6a37a5e62c84bd5b0442c4a8136134c8`).
3. If not logged in, open a real Brave window (working directory `D:\SOFTWARE\Brave`) and run `npx wrangler login`. Do not use Edge/Chrome/`start`.
4. `npm run deploy` (`opennextjs-cloudflare build` then Wrangler upload). Uses local OAuth, not `CLOUDFLARE_API_TOKEN`.
5. If `migrations/` changed, also `npm run db:migrate:remote`.

### Windows ARM64 (this workstation)

`workerd` has no ARM64 Windows build. Wrangler still needs it. Force the x64 binary, do **not** commit it:

```
npm install @cloudflare/workerd-windows-64 --no-save --force
npx wrangler whoami
npm run deploy
```

If that fails with auth error 10000, Wrangler is not logged in — run `npx wrangler login` in Brave, not GitHub secrets.

## Optional: auto-deploy from Git

Not configured. Reconnecting GitHub in the Cloudflare dashboard (Workers Builds) would deploy on push and needs the user to authorize GitHub in dash.cloudflare.com. Do not set that up unless asked.
