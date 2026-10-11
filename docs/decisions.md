# Kitchen Quest — Decisions

Last updated: 2026-10-11

Key technical decisions and why they were made. When a decision changes, update its section and note the date.

## React SPA instead of Next.js

**Context**

The UI needs routing, sign-in and role checks, and a few admin-only operations that must use a secret key. Both a React single-page app (SPA) and Next.js can do this.

**Decision**

Build the UI as a React SPA with Vite and React Router. Admin-only operations run in Supabase Edge Functions (see the next section).

**Reasons**

- Almost every screen is behind sign-in, so server rendering and SEO bring no benefit.
- A static SPA uses almost none of Netlify's free compute. A Next.js app would run server functions on many requests.
- There is no server rendering layer to configure, secure and keep patched. Next.js has had several security releases, including a critical one in 2025.

**Consequences**

- Admin-only operations need their own server-side place (Edge Functions).
- If a public page that needs SEO is added later (for example a marketing page), it can be built as a separate static page.

## Supabase Edge Functions for admin-only operations

**Context**

Inviting users, sending reset emails, deactivating users and changing roles need the Supabase service role key. That key bypasses RLS, so it must never reach the browser. The operations need to run on a server. The options were Supabase Edge Functions and Netlify Functions.

**Decision**

Use Supabase Edge Functions. Keep the business logic in plain TypeScript with no Deno-specific code, and keep the Deno entry points thin.

**Reasons**

- The functions live in the same Supabase project as the database and Auth, so the service role key stays inside Supabase and is not copied to another service.
- They run in the local Supabase stack (Docker), so they can be developed and tested offline with everything else.
- They do not use Netlify's monthly credits.

**Risks**

Edge Functions run on the Deno runtime. On 2026-10-09 Deno announced that its team is joining Cloudflare and that its own development of the runtime will end after one more year of fixes ([Deno blog](https://deno.com/blog/cloudflare)). Supabase runs Edge Functions on its own runtime, so the effect on Supabase is not certain, but the long-term outlook is less clear. To be ready if it does affect Supabase:

- Business logic is plain TypeScript, so only the small entry points would need rewriting.
- Netlify Functions remain the fallback.

**Consequences**

- Every function checks that the caller is an active admin before doing anything.
- Admins read users' emails through a function, because emails are not copied into `profiles`.

## MUI only, no Tailwind

**Context**

The app needs accessible form controls, dialogs and tables, and a custom look (paper and stainless style, see [design-tokens.md](design-tokens.md)). Options were MUI alone, Tailwind alone, or both together.

**Decision**

Use MUI only, including the free DataGrid. Layout and spacing use `sx` and `Stack`.

**Reasons**

- MUI provides accessible, tested components (dialogs, menus, selects), which a training tool used by many people needs.
- The free DataGrid covers the admin tables.
- The design tokens map directly to one MUI theme (`createTheme`), so colors and type live in one place.
- Using MUI and Tailwind together means two styling systems, with overlapping rules and style-order conflicts. One system is easier to keep consistent, for people and for AI-generated code.

**Consequences**

- Features that only exist in MUI X Pro (such as pinned columns) are not used. Fixed columns are done with sticky CSS instead.
- Small lists that need drag to reorder (categories, steps) use dnd-kit instead of the DataGrid.

## Supabase (Postgres) instead of Firebase

**Context**

The app needs a database, sign-in, photo storage and access rules. The two main backend-as-a-service options were Supabase (PostgreSQL) and Firebase (Firestore).

**Decision**

Use Supabase: PostgreSQL, Auth, Storage and Row Level Security (RLS).

**Reasons**

- The data is relational: menus, ingredients, set components, game attempts and kitchen statuses all point to each other. SQL joins and aggregates (for example the team progress table) are simple in Postgres and awkward in a document database.
- Rules can be enforced by the database itself: unique names per store, "an amount question needs a number and a unit", "at most two photos".
- Access rules are written as RLS policies in SQL and kept in migrations in the repository, so every environment gets the same rules.
- Postgres is standard and portable, so the data is not locked to one vendor.

**Consequences**

- Schema changes are made with Supabase CLI migrations, and TypeScript types are generated from the schema.
- Local development runs the Supabase stack in Docker.
