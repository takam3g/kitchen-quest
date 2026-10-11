# Kitchen Quest — Architecture

Last updated: 2026-10-11

## Overview

The UI is a React single-page app, data and auth are Supabase, and only operations that need admin privileges run in Supabase Edge Functions.

## Tech stack

| Area | Choice | Status |
|---|---|---|
| Language | TypeScript | Decided |
| UI | React (SPA built with Vite) | Decided |
| Routing | React Router (`/app` and `/admin` as separate route trees with sign-in and role checks) | Decided |
| Component library | MUI only (including the free DataGrid). Layout and spacing use `sx` and `Stack`; the theme is defined in [design-tokens.md](design-tokens.md) | Decided |
| Data, auth, images | Supabase (PostgreSQL, Auth, Storage, RLS) | Decided |
| Admin-only operations | Supabase Edge Functions (invite, send reset email, deactivate, change display name or role) | Decided |
| Schema management | Supabase CLI migrations; types generated with `supabase gen types` | Decided |
| i18n | i18next + react-i18next (en / ja) | Decided |
| Hosting | Netlify (one site per deployment; serves the SPA only) | Decided |
| Local development | Supabase CLI local stack on Docker | Decided |
| Data fetching | TanStack Query | Decided |
| Forms | React Hook Form + Zod | Decided |
| Drag and drop | dnd-kit (steps and categories) | Decided |
| Photo resizing | Resize in the browser before upload with `browser-image-compression` (long edge up to 1200px, JPEG or WebP, about 0.5MB at most) | Decided |
| Testing | Vitest (scoring, unlock rules and other logic), Playwright (end-to-end flows such as sign in → game → result). React Testing Library is not used in the MVP; it can be added on top of Vitest later | Decided |
| Quality | ESLint + Prettier + GitHub Actions; Dependabot for dependency updates | Decided |
| Daily demo reset | Supabase pg_cron (SQL only) | Decided |

## Layers of protection

- RLS on every table, with the rules in [data-model.md](data-model.md#access-rules-rls--outline). Route guards only shape the UI.
- The service role key is used only inside Edge Functions, and every function checks that the caller is an active admin.
- Public sign-up is turned off in Supabase Auth (`enable_signup = false` locally and in every hosted project). Accounts are created only by admin invites.

## Edge Functions and portability

Supabase Edge Functions run on the Deno runtime. On 2026-10-09, Deno announced that its team is joining Cloudflare: the runtime will get bug fixes and security updates for one more year, after which Deno's own development of it will end ([Deno blog](https://deno.com/blog/cloudflare)).

Supabase runs Edge Functions on its own runtime, so this does not necessarily affect Supabase directly or soon. To be ready if it does, the business logic lives in plain TypeScript functions with no Deno-specific code. Only the thin entry points would need rewriting if the functions move to Netlify Functions or elsewhere. Revisit this section when Supabase announces its plans.

## Deployment modes

The same code runs in two modes. The code is identical; what differs is the settings and the data loaded into each deployment. Each deployment has its own Supabase project and Netlify site, with its own URL and keys.

| | Demo mode | Standard mode |
|---|---|---|
| Purpose | Public showcase anyone can try | Real use by one store |
| `VITE_DEMO_MODE` | `true` | `false` or unset |
| Initial data | Fictional sample menus and the guest accounts (demo seed) | Nothing. Only the first admin, created by a script |
| Data afterwards | No real recipes or staff data | The store's own menus and staff |
| Guest sign-in buttons | Shown | Hidden |
| Top page | Intro page (after the MVP) | Redirects to `/login` or `/app` |
| Daily data reset | Runs: game attempts made by the guest staff account are cleared. Everything else is sample data that guests cannot change | Not set up |

- The environment variable only changes what the UI shows (guest buttons, intro page). It is not a security boundary.
- Both modes apply the same migrations, including the RLS rules for guest accounts. The guest accounts exist only in the demo database, so in standard mode those rules match nobody.

### Guest sign-in (demo mode only)

- The sign-in screen has two buttons, **Try as staff** and **Try as an admin (view only)**, which sign in to shared demo accounts.
- The view-only admin can open every admin screen and form, and can type into forms, but nothing is saved. This includes menus, ingredients, categories, users, photos and kitchen statuses. RLS blocks every write; the UI does not rely on hiding buttons.
- In demo mode, a banner at the top of every admin screen says "View-only demo: changes are not saved". Pressing a save button shows the same message instead of an error.
- The guest staff account can only write game attempts, with a rate limit to prevent bulk writes.
- The demo stays on free plans, so overuse cannot create a bill. If paid-per-call features (such as AI) are added later, they are disabled in demo mode.

### Reproducible setup

So that a new deployment can be set up with the same steps:

- The schema and access rules (RLS) are managed as migration files in the repository.
- A script creates the first admin (admins can only be invited by admins, so the first one is created this way).
- All required settings (connection details, keys, demo mode and so on) are listed in `.env.example`.
- The README explains how to set up a new deployment, including turning off public sign-up in Supabase Auth.

## Hosting and deployment

- Every deployment uses Netlify. Its free plan allows commercial use.
- **Branches**: `main` is always releasable. Each feature (for example menu management) gets a feature branch, `feature/<name>`, created from `main`. Each GitHub issue gets its own branch from the feature branch and is merged back into it by pull request. When the whole feature works, the feature branch is merged into `main` by pull request.
- **CI**: GitHub Actions runs lint, type check and tests on every pull request (into feature branches and into `main`). This runs on GitHub, so it does not use Netlify credits. `main` is protected: it only accepts pull requests that pass CI.
- **CD**: Netlify deploys automatically when `main` changes. Netlify is connected once the core staff flow (menu list → study → game) works locally; until then, merges into `main` only run CI.
- The Netlify free plan shares a monthly credit pool across deploys, bandwidth and compute, and all sites on the account pause when it runs out. To save credits, deploy previews and branch deploys are turned off, so only merges to `main` trigger a deploy. Because features reach `main` only when complete, deploys happen once per feature, not once per issue.
