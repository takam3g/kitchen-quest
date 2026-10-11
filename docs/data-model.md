# Kitchen Quest — Data model

Last updated: 2026-10-11

This file is the design for the database: table names, column names, types and the rules behind them. Migrations in `supabase/migrations/` implement it. If the two ever differ, fix one of them in the same change so they match.

## Conventions

- Table and column names are `snake_case`, tables in the plural (`menus`, `game_attempts`).
- Every table has `id uuid primary key default gen_random_uuid()` and `created_at timestamptz not null default now()`. Every table also has `updated_at timestamptz not null default now()`, kept current by a trigger (on insert-only tables it simply stays equal to `created_at`). These columns are not repeated in the tables below; only exceptions are listed.
- Display order columns are named `sort_order` (never `order`, which is an SQL keyword).
- "Unique per store" means a unique constraint on (`store_id`, `lower(name)`), so "Onion" and "onion" count as the same name. "Unique per menu" / "per set" are unique constraints on the parent id plus that column.
- Booleans start with `is_` or read as a verb (`is_active`, `ask_in_game`).
- Every table except `stores` has `store_id`, to prepare for SaaS. RLS checks it on every row.
- Fixed value sets are Postgres enums:
  - `user_role`: `admin`, `staff`
  - `menu_type`: `single`, `set`
  - `kitchen_status`: `no_experience`, `with_assistance`, `independent`

## Principles

- The only progress data stored is game attempts. Whether something is cleared, the best score and "not attempted" are all calculated from `game_attempts`; no progress status column is stored.
- Kitchen status is the exception: it cannot be calculated, so it is stored, with a change log.
- Recipes for prepped items are never stored. A prepped item is just a row in `ingredients`.

## Tables

### `stores`

| Column | Type | Notes |
|---|---|---|
| `name` | text, not null | Not unique: different stores can share a name. The MVP has one store |
| `slug` | text, not null, unique | Short readable identifier for people, for example `sakura-vancouver`. Lowercase letters, digits and hyphens, 3–40 characters (check constraint). Can be used in URLs later. The `id` stays the internal key and is never shown to users |

### `profiles`

One row per user, linked to Supabase Auth. Email and password live in `auth.users` only and are not copied here.

| Column | Type | Notes |
|---|---|---|
| `id` | uuid, primary key → `auth.users.id` | No default. Set to the id that Supabase Auth gave the user, so the profile and the sign-in account share one id |
| `store_id` | uuid, not null → `stores` | |
| `display_name` | text, not null | No full name needed |
| `language` | text, not null, default `en` | `en` or `ja` (check constraint; extended when a language is added) |
| `role` | `user_role`, not null, default `staff` | |
| `is_active` | boolean, not null, default true | Deactivated users cannot sign in. Never deleted |

### `ingredient_categories` / `menu_categories`

Same shape, two tables.

| Column | Type | Notes |
|---|---|---|
| `store_id` | uuid, not null → `stores` | |
| `name` | text, not null | Unique per store |
| `sort_order` | integer, not null | Order on the Categories screen, menu list sections and game tabs |

### `ingredients`

| Column | Type | Notes |
|---|---|---|
| `store_id` | uuid, not null → `stores` | |
| `category_id` | uuid, not null → `ingredient_categories` | |
| `name` | text, not null | Unique per store |
| `is_active` | boolean, not null, default true | Inactive ingredients are not game choices |

### `menus`

| Column | Type | Notes |
|---|---|---|
| `store_id` | uuid, not null → `stores` | |
| `category_id` | uuid, not null → `menu_categories` | |
| `name` | text, not null | Unique per store |
| `type` | `menu_type`, not null | |
| `is_set_only` | boolean, not null, default false | Only allowed when `type = 'single'` (check constraint). Shown as "Part of a set" |
| `target_minutes` | integer, null | Target cooking time |
| `is_active` | boolean, not null, default true | Inactive menus are hidden from staff; attempts are kept |

Recipes assume one serving; there is no servings column.

### `menu_photos`

| Column | Type | Notes |
|---|---|---|
| `store_id` | uuid, not null → `stores` | |
| `menu_id` | uuid, not null → `menus` | |
| `storage_path` | text, not null | Path in Supabase Storage |

At most two photos per menu, enforced by a trigger. Photos are shown in the order they were added (`created_at`); there is no order to set.

### `menu_ingredients` (single menus only)

| Column | Type | Notes |
|---|---|---|
| `store_id` | uuid, not null → `stores` | |
| `menu_id` | uuid, not null → `menus` | |
| `ingredient_id` | uuid, not null → `ingredients` | Unique per menu |
| `amount_value` | numeric, null | |
| `unit` | text, null | For example `g`, `pcs`, `tbsp` |
| `amount_note` | text, null | For amounts that are not numbers, such as "to taste" |
| `ask_in_game` | boolean, not null, default false | If true, `amount_value` and `unit` must not be null (check constraint) |

Ingredients are listed in ingredient category order (`sort_order`), then by name. There is no separate order per menu.

### `menu_steps` (single menus only)

| Column | Type | Notes |
|---|---|---|
| `store_id` | uuid, not null → `stores` | |
| `menu_id` | uuid, not null → `menus` | |
| `step_number` | integer, not null | 1, 2, 3… Unique per menu. This is the correct answer in the game |
| `body` | text, not null | |
| `tip` | text, null | |

### `set_components` (set menus only)

| Column | Type | Notes |
|---|---|---|
| `store_id` | uuid, not null → `stores` | |
| `set_menu_id` | uuid, not null → `menus` | Must be a `set` |
| `component_menu_id` | uuid, not null → `menus` | Must be a `single`. Unique per set |
| `amount_value` | numeric, null | The portion in this set, independent of the single item's recipe |
| `unit` | text, null | |
| `amount_note` | text, null | |
| `ask_in_game` | boolean, not null, default false | Same check constraint as `menu_ingredients` |

Components are listed in menu category order (`sort_order`), then by name. There is no separate order per set.

The "must be a set / single" rules are checked by a trigger, because a check constraint cannot look at another row.

### `game_attempts`

One row per game. Never updated or deleted.

| Column | Type | Notes |
|---|---|---|
| `store_id` | uuid, not null → `stores` | |
| `user_id` | uuid, not null → `profiles` | |
| `menu_id` | uuid, not null → `menus` | |
| `score` | smallint, not null | 0 to 100. An attempt is cleared when `score = 100`; this is not stored separately |
| `items_correct`, `items_total` | smallint, not null | Ingredients for a single menu, components for a set |
| `amounts_correct`, `amounts_total` | smallint, not null | `0` / `0` when there are no amount questions |
| `steps_correct`, `steps_total` | smallint, null | Null for sets |
| `answers` | jsonb, not null | What the user picked, entered and ordered |
| `answer_key` | jsonb, not null | The correct answers at the time, so editing a recipe never changes past results |

`created_at` is the time the game was played.

### `kitchen_statuses`

| Column | Type | Notes |
|---|---|---|
| `store_id` | uuid, not null → `stores` | |
| `user_id` | uuid, not null → `profiles` | |
| `menu_id` | uuid, not null → `menus` | |
| `status` | `kitchen_status`, not null | |
| `updated_by` | uuid, not null → `profiles` | An admin |

A row is created the first time an admin sets a status for that user and menu. Until then there is no row, and the app shows Not yet. If an admin later sets it back to Not yet, the row stays and `status` becomes `no_experience`, so the log can record the change. The app treats "no row" and `no_experience` the same.

(`user_id`, `menu_id`) is unique (unique constraint): one status per user and menu.

### `kitchen_status_logs`

One row per change, written by a trigger on `kitchen_statuses`. Never updated or deleted.

| Column | Type | Notes |
|---|---|---|
| `store_id` | uuid, not null → `stores` | |
| `user_id` | uuid, not null → `profiles` | |
| `menu_id` | uuid, not null → `menus` | |
| `from_status` | `kitchen_status`, null | Null for the first change (there was no row yet) |
| `to_status` | `kitchen_status`, not null | |
| `changed_by` | uuid, not null → `profiles` | |

## Rules

- Menu, ingredient and category names are unique within a store (case-insensitive).
- Categories, ingredients and menus that are in use cannot be deleted (foreign keys without cascade). They are renamed or deactivated instead.
- Roles are `user_role` values, not a boolean such as `is_admin`, so more roles can be added later.
- The store is the tenant: every row belongs to a store, and RLS keeps stores apart. This is in place from the MVP because adding `store_id` to every table later would be close to a rebuild.
- In the MVP, each user has one store and one role directly on `profiles`. For SaaS, this moves to a `store_users` table (user, store, role) so one user can belong to several stores. A store only ever sees its own `store_users` rows, so it cannot tell where else a user works.
- There is no `organizations` table yet. For chains, an `organizations` table can be added above `stores` later. Whether menus, ingredients and categories are then shared per organization is decided at that point, based on what chains actually need.
- Filter and sort choices on My progress and Team progress are kept in the browser (local storage), not in the database.

## Access rules (RLS) — outline

Policies do not repeat the store check. They call two SQL helper functions, `is_store_user(store_id)` (any user of that store, admin or staff) and `is_store_admin(store_id)`, so that moving to `store_users` later changes only these functions.

| Tables | Staff | Admin | Demo view-only admin | Demo guest staff |
|---|---|---|---|---|
| `menus`, `menu_*`, `set_components`, `ingredients`, `*_categories` | Read active rows in own store | Read and write in own store | Read | Read active rows |
| `profiles` | Read own row | Read rows in own store; changes via Edge Functions | Read | Read own row |
| `game_attempts` | Insert and read own | Read all in own store | Read | Insert (rate-limited) and read own |
| `kitchen_statuses`, `kitchen_status_logs` | Read own | Read all in own store; write `kitchen_statuses` | Read | Read own |

- Admins see users' emails through an Edge Function that reads `auth.users`, because emails are not in `profiles`.
- Demo accounts use addresses such as `guest-staff@example.com`, so nothing real is exposed in demo mode.

Details of the demo accounts are in [architecture.md](architecture.md#guest-sign-in-demo-mode-only).
