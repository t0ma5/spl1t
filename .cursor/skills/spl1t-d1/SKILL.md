---
name: spl1t-d1
description: spl1t data layer, PIN auth, and Cloudflare D1 conventions. Use when editing groups, expenses, tRPC procedures, wrangler bindings, migrations, cron jobs, or anything that reads/writes group data.
---

# spl1t + Cloudflare D1

## Database

- Primary store is **Cloudflare D1**, binding `DATABASE`. Schema lives in `migrations/`.
- Application access: `getRepository()` in `src/lib/db/index.ts`. Tests inject `createMemoryRepository()`.
- Do not add Prisma, Postgres, or write new `group:{id}` KV documents.
- Mutations go through `withGroupWrite` / `repo.save(group, expectedVersion)` so concurrent edits retry instead of last-write-wins.
- Amounts are **integer minor units**. Validate with `assertIntegerMinorUnits` on write.

## PIN

- `assertGroupUnlocked(groupId)` on every group-scoped tRPC procedure and both export routes.
- Unlock cookie is HTTP-only HMAC (`PIN_SECRET`). Do not trust `sessionStorage`.
- Hash new PINs with PBKDF2 (`hashGroupPin`). Legacy SHA-256 hashes still verify and upgrade on success.

## Cron

- HTTP endpoints under `src/app/api/cron/` require `Authorization: Bearer $CRON_SECRET` (`src/lib/cron-auth.ts`).
- Do not schedule cleanup until inactivity uses `lastSeenAt` (already wired). Recurring generation: `/api/cron/recurring`.

## Feature flags

- Keep `NEXT_PUBLIC_ENABLE_EXPENSE_DOCUMENTS`, `RECEIPT_EXTRACT`, and `CATEGORY_EXTRACT` false (`src/lib/env.ts` refuses boot otherwise).
