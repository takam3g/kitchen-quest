# Kitchen Quest

A web app for restaurant kitchen staff: study each menu item's ingredients and steps, then check what you remember with a game. Admins manage recipes and see who has learned what. Built for one restaurant first; designed to grow into a SaaS.

## Read before working

| Task | Read |
|---|---|
| Any feature work | `docs/requirements.md` |
| Routes, pages, navigation | `docs/screens.md` |
| Tables, migrations, RLS | `docs/data-model.md` |
| Stack, Edge Functions, environments, deploy | `docs/architecture.md` |
| Colors, fonts, theme, any styling | `docs/design-tokens.md` |
| Why something was decided | `docs/decisions.md` |

Use the terms in the Glossary at the top of `docs/requirements.md` for names and code (for example `game_attempts`, `ask_in_game`; never "test" or "quiz"). UI text follows the "UI label" column.

If a request conflicts with these docs, point it out and ask before changing behavior. When a decision changes, update the doc in the same change.

## Stack

- TypeScript, React SPA (Vite), React Router
- MUI only (including free DataGrid). Use `sx` and `Stack`. **No Tailwind**, no other styling library.
- Supabase: Postgres, Auth, Storage, RLS. Schema via Supabase CLI migrations.
- Supabase Edge Functions for admin-only operations (invite, reset email, deactivate, change display name or role)
- i18next + react-i18next (en / ja)
- Netlify hosting (SPA only)

## Commands

<!-- Fill in once the project is scaffolded -->

```bash
npm run dev            # start Vite
npm run build
npm run lint
npm run test           # Vitest
npx playwright test    # end-to-end tests
npx supabase start     # local Supabase (Docker)
npx supabase migration new <name>
npx supabase db reset  # re-apply migrations + seed locally
npx supabase gen types typescript --local > src/types/database.ts
```

## Rules

### Security

- The service role key is used **only** inside Edge Functions. Never import it or reference it in `src/`.
- Every table has RLS enabled with explicit policies. A migration that adds a table must add its policies.
- Policies call the helper functions `is_store_user(store_id)` and `is_store_admin(store_id)` instead of repeating the check (see `docs/data-model.md`).
- Edge Functions verify the caller is an active admin before doing anything.
- Route guards only hide screens; never rely on them for access control.

### Data

- Do not store derived progress (cleared, best score, not attempted). Calculate from game attempts.
- Game attempts are insert-only. Never update or delete them.
- Roles are `role` values (`admin` / `staff`), not booleans.
- Every table except `stores` has a store reference.
- After changing migrations, regenerate types.

### Edge Functions

- Keep business logic in plain TypeScript with no Deno-specific APIs. Entry points only handle request/response, so the logic stays portable if the runtime ever has to change (see `docs/architecture.md`).

### UI

- Readability first: no text below 12px (labels 13px+), sentence case, no all caps, no pixel or novelty fonts.
- Read colors, fonts and shapes from the MUI theme (`src/theme/theme.ts`, defined in `docs/design-tokens.md`). Never hard-code hex values in components.
- Never show correct/wrong or a status by color alone. Always add ✓ / ×, a label or a pattern.
- Staff screens (`/app`) are mobile portrait first. Touch targets at least 44px.
- Every UI string goes through i18next. Add keys to both `en` and `ja`.
- Admin-entered content (menu names, steps) is shown as entered; do not translate it.

### Demo data

- Only fictional home-cooking menus. Never real restaurant recipes or staff data.

## Working style

- Git flow: issue branch → feature branch (`feature/<name>`) → `main`, each step by pull request. Never commit directly to `main`. See `docs/architecture.md` (Hosting and deployment).
- Keep the MVP simple. Do not add columns, settings or UI (such as custom sort order) that the docs do not ask for. Suggest them instead, and add them only after they are agreed.
- Small, focused changes. Run lint and tests before saying something is done.
- Write tests for scoring, unlock rules and other pure logic (Vitest), and keep the core flows (sign in → game → result, admin editing a menu) covered by Playwright.
- Never commit, push or open a pull request unless asked. When work is ready, give a short summary of what was changed and why, show the changes, and suggest a commit message; the user commits. Pull requests follow `.github/pull_request_template.md`.
- Commit messages follow Conventional Commits:
  - `feat:` a new feature
  - `fix:` a bug fix
  - `refactor:` code changes that do not change behavior
  - `test:` adding or fixing tests only
  - `docs:` documentation only
  - `chore:` tooling, config or dependency updates
