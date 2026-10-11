# Kitchen Quest — Screens and URLs

Last updated: 2026-10-11

Screen designs are in [screens/](screens/) (images exported from the design canvas). Feature details are in [requirements.md](requirements.md).

## URL groups

URLs fall into three groups: public, signed in, and admin only. Everyone (including admins) can use `/app`; only admins can use `/admin`. Staff screens (`/app`) are designed for phones in portrait first; admin screens also work well on desktop.

## Screen list

| URL | Screen | Who | Main content | Design |
|---|---|---|---|---|
| `/` | Top | Anyone | Demo mode: intro page (what the app is, buttons to try the demo). Standard mode: redirects to `/login` or `/app`. The intro page comes after the MVP |  |
| `/login` | Sign in | Anyone | Email and password |  |
| `/reset-password` | Reset password | Anyone | Send a reset email; set a new password |  |
| `/set-password` | First-time password | Invited users | Opened from the invite email; set a password |  |
| `/app` | Menu list | Signed in | Active menu items, each leading to Study or Game. Set-only items are labeled "Part of a set". Locked sets show a lock icon and the unlock condition (or an info button that shows it) | [menu-list.png](screens/menu-list.png) |
| `/app/menus/[id]` | Study | Signed in | Ingredients and amounts, steps, hide & practice, then up to two photos at the bottom. Sets show their components and amounts | [study.png](screens/study.png) |
| `/app/menus/[id]/game` | Game | Signed in | Ingredients → amounts → steps → result, all on one screen. Sets: components → amounts → result | [game-1-ingredients.png](screens/game-1-ingredients.png), [game-2-amounts.png](screens/game-2-amounts.png), [game-3-steps.png](screens/game-3-steps.png), [game-4-result.png](screens/game-4-result.png) |
| `/app/progress` | My progress | Signed in | Menu × game scores, cleared highlight, kitchen status. Filter by category, kitchen status and game result; sort by menu name (default), latest game score or kitchen status | [my-progress.png](screens/my-progress.png) |
| `/admin` | Team progress | Admin | Menu × user (Game / Kitchen) table, choose which users to show | [admin-team-progress.png](screens/admin-team-progress.png) |
| `/admin/menus` | Menus | Admin | Single and set items. The list is read only. Clicking a menu name opens the edit screen; the Add button at the top right creates a new item | [admin-menus.png](screens/admin-menus.png) |
| `/admin/menus/new`, `/admin/menus/[id]` | Edit menu | Admin | See [Edit menu form](#edit-menu-form) | [admin-edit-menu.png](screens/admin-edit-menu.png) |
| `/admin/ingredients` | Ingredients | Admin | Ingredient list. The Add button at the top right adds a row; name, category and active/inactive are edited directly in the list (no separate edit screen) | [admin-ingredients.png](screens/admin-ingredients.png) |
| `/admin/categories` | Categories | Admin | Tabs for menu and ingredient categories; add, rename, reorder. Because there are few categories, this is a simple drag-to-reorder list rather than a DataGrid | [admin-categories.png](screens/admin-categories.png) |
| `/admin/users` | Users | Admin | List and invite. Each row's ⋯ menu has Send reset email, Edit name or role, and Deactivate; for inactive users it shows Reactivate instead of Deactivate. Deactivate and Reactivate are not shown as row buttons, to avoid accidental taps, and both ask for confirmation | [admin-users.png](screens/admin-users.png) |

## Edit menu form

- A form filled with the current values: name, type (Single / Set, with a "Set only" checkbox under it that appears only for Single), category, target cooking time, photos (up to two), active/inactive, ingredients and amounts, amount questions ("Ask in game" checkbox), steps, set components and their amounts ("Ask in game" checkbox), and the confidentiality notice.
- Creating a new item uses the same form, empty.
- The Update button is enabled only when something has changed. It sits at the bottom of the form (not fixed to the screen) so admins review everything before updating. A Discard changes button sits next to it.
- Leaving with unsaved changes asks for confirmation.

## Navigation and route guards

- After sign-in, admins go to `/admin` and everyone else goes to `/app`. The admin header has a switch between the admin screens and the practice screens.
- Everything under `/app` and `/admin` requires sign-in. `/admin` also requires the admin role; others are sent back to `/app`.
- Games that admins play under `/app` are saved as game attempts like anyone else's.

**Account menu**: The header on every signed-in screen (`/app` and `/admin`) has an account menu showing the display name, a language switch (English / 日本語) and Sign out. The language is saved to the user's profile and applies right away.

**Admin sidebar**: Progress (home), Menus, Ingredients, Categories, Users, in that order — the same order as the `/admin` rows above. On phones it becomes a collapsible menu.
