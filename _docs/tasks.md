# OurHome — Task backlog

Each task is sized for one working session and written so it can be handed over without reading the other tasks. Where a task needs something that another task produces, its description says what it assumes to exist.

Reference documents (read only the parts a task names): [plan.md](plan.md) is the product scope, [stack.md](stack.md) is the tech stack and architecture rules, and [data-model.md](data-model.md) is the data model. All user-facing text is in Portuguese (pt-PT).

---

## A. Foundation

### 1. Set up an empty project with a passing test
**Goal:** A new project that installs, builds, and runs one passing test.
**Description:** Create the Next.js (App Router) + TypeScript project described in `stack.md`, with Tailwind and Vitest configured. Add one trivial unit test and the npm scripts `dev`, `build`, `typecheck`, `lint`, and `test`. Done when `npm run test` passes on a fresh clone and `npm run build` succeeds.

### 2. Add continuous integration
**Goal:** Every push runs typecheck, lint, tests, and build automatically.
**Description:** Assuming a Next.js project with `typecheck`, `lint`, `test`, and `build` scripts, add a CI workflow (GitHub Actions) that runs them on pushes and pull requests. Cache dependencies, pin the Node version, and document the workflow in the README. Done when a pull request shows the checks passing.

### 3. Connect PostgreSQL with Drizzle and migrations
**Goal:** The app can talk to Postgres, and schema migrations run locally and in tests.
**Description:** Add Drizzle ORM and a Postgres connection configured by environment variables, plus a `docker-compose` file for local Postgres and a separate test database. Provide scripts to generate and apply migrations, and one test that applies all migrations to an empty test database and runs a simple query. Document setup in the README.

### 4. Build the responsive app shell with Portuguese i18n
**Goal:** A navigable, accessible layout in pt-PT that works on phone and desktop.
**Description:** Set up `next-intl` with pt-PT as the only locale, and Tailwind + shadcn/ui. Build a shell with a top bar and navigation (mobile bottom nav or drawer, desktop sidebar) with placeholder pages for Painel, Calendário, Compras, Tarefas, Finanças, and Definições. It must be keyboard accessible, have visible focus states, and meet contrast guidelines.

### 5. Add sign-up, login, and logout
**Goal:** A person can create an account and sign in and out with email and password.
**Description:** Integrate Better Auth with the Drizzle database (see `stack.md`), with sign-up, login, logout, and a protected-route helper that redirects anonymous visitors to login. Forms are in Portuguese, validated with Zod, and show clear error messages. Include tests for the happy path and for wrong credentials.

---

## B. Households, access, and isolation

### 6. Create the household and membership schema
**Goal:** The database models households and memberships with the right constraints.
**Description:** Add the `household` and `membership` tables from `data-model.md` §1, including a unique `user_id` (one household per account) and a partial unique index so a household has exactly one active admin. Add the finance permission flags and soft removal. Write migration tests that prove a second household per user and a second admin are rejected.

### 7. Add the "create household" onboarding flow
**Goal:** A newly registered user can create a household and becomes its admin.
**Description:** Assuming authentication and the household/membership tables exist, add a screen where a signed-in user without a household enters a name and chooses a base currency, then creates it as admin. Users who already belong to a household are redirected away. Add tests for creation and for the redirect.

### 8. Build the tenant-scoped data-access layer
**Goal:** One typed entry point that injects the household scope and checks permissions for every query.
**Description:** Implement the pattern from `stack.md` architecture rules 1 to 3: a function that resolves the authenticated member (user, household, role, finance flags) and returns repositories that always filter by that household. Include a permission-check helper (`requireAdmin`, `requireFinancePermission(kind)`). Cover it with unit tests, including "no household" and "removed member" cases.

### 9. Create a cross-household isolation test harness
**Goal:** A reusable test helper that proves one household can never touch another's data.
**Description:** Build test utilities that seed two households with members, and a pattern for asserting that every data-access function called as household A returns nothing from, and cannot modify, household B. Apply it to the household and membership repositories as the first examples. Document the pattern in the README so later features reuse it.

### 10. Invite members by email
**Goal:** An admin can invite someone by email and the invitee can join the household.
**Description:** Implement the `invite` table (`data-model.md` §1) for the email type, with a hashed token, an expiry, and a role and permission preset, and send the invitation through Resend. Add the accept page: a signed-in user without a household who opens a valid link joins as a member. Test expired, revoked, reused, and already-in-a-household cases.

### 11. Invite members with a shareable code
**Goal:** An admin can generate a join code and a user can join by entering it.
**Description:** Extend the `invite` model with the code type (hashed code, configurable max uses, expiry), with an admin screen to create and revoke codes and a join-by-code form for users without a household. Rate-limit failed code attempts per user and per IP. Test valid, expired, revoked, exhausted, and rate-limited cases.

### 12. Build the members page with permission editing
**Goal:** The admin can see members, edit their finance permissions, and remove them.
**Description:** Add a members screen that lists active members and lets the admin toggle `can_register_finance`, `can_edit_finance`, and `can_manage_finance_rules`, or remove a member (soft removal that keeps authorship). The admin cannot be removed here, and non-admins cannot open the page or call the actions, which is enforced on the server. Test both the server checks and the UI.

### 13. Transfer the admin role
**Goal:** The admin can hand the admin role to another member safely.
**Description:** Add an action that, in one transaction, makes the chosen active member the admin and the current admin a regular member, keeping the "exactly one admin" invariant. Block the admin from leaving, being removed, or deleting their account until the role is transferred. Test the transaction, the invariant, and the blocked cases.

### 14. Add the append-only audit log
**Goal:** A shared helper that records who changed what, and when.
**Description:** Create the `audit_log` table from `data-model.md` §7 (entity, action, actor, before and after as JSON) and a helper that writes an entry inside the caller's transaction. Add a read function scoped to a household that hides entries about secret ledger records from everyone except their owner, plus a basic history view component. Test that entries cannot be updated or deleted through the app layer.

---

## C. Shopping and tasks

### 15. Manage shopping lists
**Goal:** Household members can create, rename, archive, and view shopping lists.
**Description:** Add the `shopping_list` table (`data-model.md` §3) with repository functions scoped by household, and a Compras page listing active lists with create, rename, and archive actions. All members can use it. Include isolation tests using the harness pattern from the repo.

### 16. Manage shopping list items
**Goal:** Members can add items with a quantity, tick them off, and keep a bought history.
**Description:** Assuming shopping lists exist, add the `shopping_item` table and a list detail page for adding an item (name and quantity, default 1), toggling bought (recording who and when), and showing bought items in a separate history section. Quantity must be a positive number. Test the bought toggle and isolation.

### 17. Manage household tasks
**Goal:** Members can create tasks, assign them to nobody or one member, and complete them.
**Description:** Add the `task` table (`data-model.md` §3) and a Tarefas page with create, edit, delete, assign (optional, to an active member of the same household), and mark done or reopen. Record who completed a task and when. Test that an assignee from another household is rejected.

---

## D. Calendar

### 18. Create and manage one-off events
**Goal:** Members can create, view, edit, and delete non-recurring events.
**Description:** Add the `event` table (`data-model.md` §2) with title, description, location, start and end, all-day flag, and time zone, plus repository functions and a form with Zod validation. Show authorship (created and last changed by). Test validation (end after start) and household isolation.

### 19. Build the recurrence expansion module
**Goal:** A tested function that turns a recurring event into occurrences for a date range.
**Description:** Using the `rrule` library, write a pure module that takes an event (start, end, time zone, RRULE, until) and a date window and returns the occurrences, respecting time zones and daylight saving changes. Support a simple recurrence builder (daily, weekly, monthly, yearly, with an end) that produces valid RRULEs. Cover edge cases such as the 31st of the month and DST changes in unit tests.

### 20. Edit or cancel a single occurrence
**Goal:** Members can change or skip one occurrence without affecting the series.
**Description:** Assuming events and an expansion function exist, add the `event_exception` table (`data-model.md` §2) and merge exceptions (cancelled, or overridden title, time, and description) into the expanded occurrences. Offer "this occurrence" versus "all occurrences" when editing or deleting a recurring event. Test the merge logic and both choices.

### 21. Add one reminder per event, shown in the app
**Goal:** An event can have a single reminder that appears in the app when due.
**Description:** Add a nullable `reminder_offset_minutes` to events and a function that returns the occurrences whose reminder time has passed but whose start is still ahead. Add an optional per-user dismissal so a member can hide a reminder for one occurrence. There are no emails or push notifications. Test the due-reminder function across recurring events.

### 22. Build the calendar views
**Goal:** Members can browse events in a month grid and an agenda list.
**Description:** Build the Calendário page with a month view and an agenda view, with navigation between periods and a way to create an event from a day. Occurrences come from the expansion function. The page must work with a keyboard and on a small screen. Add an end-to-end test that creates an event and sees it in both views.

---

## E. Currencies and exchange rates

### 23. Store exchange rates and fetch daily rates
**Goal:** The system keeps daily exchange rates and updates them automatically.
**Description:** Create the global `fx_rate` table (`data-model.md` §4) and a client for a historical-capable provider such as Frankfurter, plus a protected cron route that fetches the latest day. Make the fetch idempotent and store rates against a single reference currency, so cross rates can be computed. Test the client with recorded responses, not live calls.

### 24. Backfill history and apply rate corrections
**Goal:** The rate table can fill past dates on demand and absorbs corrected rates.
**Description:** Assuming the `fx_rate` table and fetch client exist, add a backfill function for a date range and extend the daily job to re-fetch a recent window (for example, 30 days) and overwrite changed values, setting `updated_at`. Optionally log each change. Test overwriting, idempotency, and the weekend fallback inputs.

### 25. Build the currency conversion service
**Goal:** A pure, tested function that converts amounts with the correct rate and explains which rate it used.
**Description:** Implement conversion from `data-model.md` §4 "Currency conversion rules": paid entries use the rate for their date (or the latest earlier date), planned entries and balances use the current rate, and identical currencies use 1. It returns the converted amount, the rate, and the rate date, and flags a missing rate. Test cross rates, weekend fallback, and rounding in integer minor units.

### 26. Restrict currencies to those with rates
**Goal:** Only currencies the rate source supports can be chosen anywhere in the app.
**Description:** Add a module that exposes the list of supported currencies (from the stored rates) and a Zod validator and currency picker component that accept only those codes. Used by ledger entries, goals, budgets, and the household base currency. Test that unsupported codes are rejected on the server and absent from the picker.

---

## F. Finance core

### 27. Manage income and expense categories
**Goal:** Authorised members can create, rename, and archive categories, and mark expense categories as essential.
**Description:** Add the `category` table (`data-model.md` §4) with kind (income or expense), `is_essential` (expense only), and archiving, plus a settings page. Require `can_manage_finance_rules`. Test uniqueness of names within a kind, the essential flag limits, and permission checks.

### 28. Record shared income and expenses
**Goal:** Members with permission can add, edit, and delete shared ledger entries.
**Description:** Add the `ledger_entry` table (`data-model.md` §4) with kind, status (planned or paid), amount in minor units, original currency, date, category, and description, defaulting to shared. Enforce `can_register_finance` to create and `can_edit_finance` to change other people's entries, show authorship, and write audit log entries for changes. Test the permission matrix and isolation.

### 29. Add secret expenses with owner-only visibility
**Goal:** An expense can be marked secret so only its owner can see its details.
**Description:** Assuming ledger entries exist, add the `secret` visibility (expenses only, enforced by a database constraint) and make every ledger read function return secret rows only to their owner, admins included. Let the owner switch an entry between shared and secret, with the audit entry visible only to the owner. Test with three members that no one else can list or open a secret row.

### 30. Compute the secret-expense aggregate
**Goal:** The household sees one total of secret spending without any details.
**Description:** Add a dedicated function that returns the monthly total of secret expenses in the base currency (using the conversion service) and nothing else: no rows, descriptions, categories, or per-member splits. There is no minimum number of members. Test that the total includes all members' secret expenses and that nothing identifying is returned.

### 31. Build the ledger list and entry form UI
**Goal:** Members can browse the month's entries and add or edit one on any device.
**Description:** Build the Finanças page with a month selector, a list of shared entries plus the user's own secret entries, and the secret aggregate as a separate line. The form has kind, amount, currency (supported only), date, category, a shared/secret toggle for expenses, and a planned/paid status. Add an end-to-end test for adding an entry.

### 32. Define recurring finance rules and generate planned entries
**Goal:** A recurring income or expense produces planned entries ahead of time.
**Description:** Add the `recurring_rule` table (`data-model.md` §4) with weekly, monthly, or yearly frequency, interval, start and end, and a job that creates planned entries for a rolling 12-month window. Generation is idempotent, handles short months, and editing a rule changes only future planned entries, never paid ones. Test idempotency and rule edits.

### 33. Confirm planned entries as paid
**Goal:** A member can confirm that a planned entry was actually paid.
**Description:** Add a confirm action on planned entries that sets status to paid, records who confirmed and when, and lets the member adjust the amount or date at confirmation. It requires the same permission as editing and writes an audit entry. Test that confirmation is limited to planned entries and respects visibility.

### 34. Compute monthly summaries
**Goal:** The month shows income, expenses, balance, and a per-category breakdown.
**Description:** Write a function that, for a month, returns income paid, expenses paid, the balance, and spending by category, all in the base currency using the conversion service. The secret aggregate appears as one separate line and never in category breakdowns, and each result carries its formula and reference date. Test with mixed currencies, planned versus paid entries, and secret expenses.

### 35. Manage budgets and show spending
**Goal:** The household can set a total monthly limit and optional category limits and see what is left.
**Description:** Add the `budget_limit` table (`data-model.md` §4) with versioning by `effective_from` and its own currency, and a function returning spent and remaining per limit. The total limit includes the secret aggregate, while category limits use shared entries only. Require `can_manage_finance_rules` to edit. Test version selection by month and the currency conversion.

---

## G. Goals and autonomy

### 36. Manage goals and their contribution history
**Goal:** Members can create goals and record contributions and withdrawals with date and author.
**Description:** Add the `goal` and `goal_contribution` tables (`data-model.md` §5), with name, optional target and deadline, and signed append-only contributions (opening balance, contribution, withdrawal). Progress is the sum converted at the current rate, and corrections are new rows. Require `can_manage_finance_rules` for goals and test that rows cannot be edited.

### 37. Set up the emergency fund goal
**Goal:** The household has exactly one emergency fund goal with an opening balance.
**Description:** Assuming goals and contributions exist, allow one goal of kind `emergency_fund` per household, prompt for an opening balance when it is created, and compute its balance from the contribution history. Let the admin flag other goals as counting toward the fund, which is off by default. Test the one-per-household rule and the balance calculation.

### 38. Calculate months of financial autonomy
**Goal:** The app shows emergency fund balance divided by average essential monthly expense, with explanations and warnings.
**Description:** Implement the calculation from `plan.md` "Resumo financeiro e autonomia": the average of paid expenses in essential categories (including secret ones, computed server-side) over the household's lookback window (default 6 months). Return the value, formula, period, reference date, months actually used, and warnings (short history, no essential categories, missing rate, no fund). Add the lookback and target settings, and test partial history and the warnings.

---

## H. Surplus and allocation

### 39. Calculate the monthly surplus with a manual override
**Goal:** The app shows the month's surplus and lets a member replace it with a manual value.
**Description:** Compute surplus as income received minus expenses paid minus planned commitments, in the base currency, and add the `allocation_period` table (`data-model.md` §6) with an optional manual surplus. Return the formula and inputs with the result. Test planned commitments, mixed currencies, and the override.

### 40. Manage versioned allocation rules
**Goal:** The household can configure the 70% and 5% rule and keep the history of changes.
**Description:** Add the append-only `allocation_rule_version` table with the savings percentage, high-risk percentage, and whether the high-risk share is included in or additional to the savings share, plus a settings page and a history view. Require `can_manage_finance_rules`. Test that edits create new versions and the current version is the latest.

### 41. Generate allocation suggestions
**Goal:** The app suggests how to split the month's savings, without moving any money.
**Description:** Assuming surplus and allocation rules exist, write a pure function that splits the surplus per the current rule: before the emergency goal is complete the savings share goes to it, afterwards it is spread across active goals, with the high-risk share per the household's choice. Persist suggestions as `allocation_line` rows with suggested, planned (manual), and recorded amounts. Test both modes and the included versus additional case.

### 42. Build the allocation page
**Goal:** Members can review suggestions, adjust them, and record what was actually allocated.
**Description:** Build a page for the selected month showing the surplus, the suggestions, editable planned amounts, and the recorded amounts, with a clear note that the app does not move money. Closed periods keep their currency and are shown converted. Add an end-to-end test for adjusting and recording.

---

## I. Dashboard, settings, and data rights

### 43. Build the dashboard
**Goal:** A lean home screen shows upcoming events, pending items, the month's balance, and months of autonomy.
**Description:** Assuming the calendar, shopping, tasks, monthly summary, and autonomy functions exist, build the Painel page showing upcoming events and due reminders, a count and list of unbought items and open tasks, the month's balance, and months of autonomy with its warnings. Each indicator links to its explanation. Add an end-to-end test of the loaded dashboard.

### 44. Let the admin change the base currency
**Goal:** The admin can switch the household's base currency and all totals recompute.
**Description:** Add an admin setting to change `base_currency` to a supported currency, with a confirmation explaining that totals will be recalculated from original amounts and historical rates, and an audit entry. Budget limits and closed allocation periods keep their own currency and display converted. Test that totals change correctly and no stored entry is modified.

### 45. Export household and personal data
**Goal:** The admin can download the household's data and each member can download their own.
**Description:** Add an export that produces a JSON (and CSV for the ledger) file of the household's shared data for the admin, and a per-user export of a member's own records, including their secret entries. Secret entries never appear in anyone else's export. Test the contents and the secret-entry exclusion.

### 46. Delete accounts and households
**Goal:** A member can delete their account, and the admin can delete the household, following the plan's rules.
**Description:** Implement user deletion (authorship becomes "former member", the user's secret entries are deleted, shared entries stay, and the admin must transfer the role first) and household deletion with a confirmation and grace period before the data is removed. Add a correction path for the user's own profile data. Test the deletions and the cascade.

---

## J. Quality and release

### 47. Add accessibility and end-to-end smoke tests
**Goal:** Key flows are tested end to end and checked for basic accessibility.
**Description:** Add Playwright tests for sign-up, creating a household, adding an event, a shopping item, and a ledger entry, and viewing the dashboard, plus automated accessibility checks (for example axe) on the main pages. Verify keyboard navigation and contrast on mobile and desktop viewports. Run the suite in CI.

### 48. Deploy to production in the EU
**Goal:** The app runs on a public URL with a managed database and scheduled jobs.
**Description:** Deploy to Vercel with a Neon (or equivalent) Postgres in an EU region, configure environment variables, apply migrations, and configure the cron jobs for the FX fetch and recurring-entry generation using a shared secret. Set up the Resend sender domain for invitations. Document the deployment and rollback steps in the README.
