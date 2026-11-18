# OurHome — Tech stack

**Status:** decided (Option B: Next.js full-stack + Postgres)
**Related:** [plan.md](plan.md), [data-model.md](data-model.md)

## Why this option

The domain has many custom rules (per-member finance permissions, secret expenses that count in a household total without exposing details, historical-rate currency conversion with a changeable base currency, allocation logic, explainable financial formulas). These are easiest to build and test in a single typed codebase, with no BaaS lock-in.

## Decisions

| Layer | Choice | Notes |
|---|---|---|
| Framework | Next.js (App Router) + TypeScript | Full-stack; server actions for mutations. |
| Database | PostgreSQL (Neon or any Postgres) | Exact `NUMERIC`/integer money, transactions. RLS available as a second isolation layer. |
| ORM | Drizzle | SQL-close, typed queries, simple migrations. Prisma is the fallback. |
| Auth | Better Auth | Email + password; household membership, roles and invites modelled in our own tables. |
| UI | Tailwind + shadcn/ui (Radix) | Accessible, keyboard-friendly, responsive. |
| i18n | `next-intl`, default pt-PT | Date, number and currency formatting. |
| Validation | Zod | Shared by forms and server actions. |
| Recurrence | `rrule` (calendar) + simple custom rule (finance) | Finance recurrence is intentionally simple (weekly/monthly/yearly). |
| Scheduled jobs | Host cron (e.g. Vercel Cron) → protected route | Daily FX fetch plus re-fetch of a recent window (e.g. 30 days) to apply rate corrections; rolling generation of planned entries. |
| FX source | Frankfurter (ECB) or equivalent with historical data | Daily and historical rates for every supported currency, stored locally; on-demand backfill for older dates. Unsupported currencies are blocked (no manual rates). Confirm the provider's currency list covers the family's currencies before committing. |
| Email | Resend | Household invites only (no notifications in MVP). |
| Testing | Vitest (logic) + Playwright (key flows) | Priority: tenant isolation, permissions, autonomy and surplus formulas. |
| Hosting | Vercel + Neon, **EU region** | A small EU VPS with Docker is an alternative. |

## Architectural rules

1. **Single data-access layer.** Feature code never queries the DB directly. Every call receives the authenticated member (user, household, role, permissions); the layer adds the `household_id` scope and runs the permission check.
2. **Server-side authorization only.** The client never decides what is allowed; UI hiding is cosmetic.
3. **Tenant safety in the schema.** Every tenant table carries `household_id`; cross-table references use composite keys so a row cannot point at another household's row. Optionally add Postgres RLS as defence in depth.
4. **Money.** Integer minor units + ISO 4217 currency code. Never floats. Original currency is always preserved. Converted amounts are derived at read time (paid entries at the historical rate of their date, everything else at the current daily rate), never stored on the entry, so the base currency can change freely.
5. **Explainable indicators.** Financial indicators return value + formula + period + reference date + warnings, not just a number.
6. **Audit trail.** Append-only history for financial records, goals and allocation rules.
7. **EU data residency** for app and database.
8. **Secret expenses go through one dedicated aggregate function.** Secret rows are readable only by their owner (admins included). Household indicators receive only the aggregate total, never rows, categories, or per-member splits. No other code path may read secret rows on behalf of someone else.

## Testing priorities

- A member of household A can never read or write household B data (automated test over every data-access function).
- Permission matrix per role and financial permission.
- Shared vs secret expense visibility: owner sees own secret rows; no other member (admin included) can list or open them; the aggregate total includes them and exposes nothing else.
- Autonomy, surplus and 70%/5% allocation calculations, including insufficient-data warnings and the 6-month default lookback with partial history.
- Currency conversion: historical vs current rate, base-currency change, missing-rate fallback to the latest earlier date, rate corrections overwriting stored values.
- Recurrence expansion and planned → paid confirmation.
- Single-admin invariant and admin transfer.
