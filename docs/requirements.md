# Kitchen Quest — Requirements

Last updated: 2026-10-11

This file covers **what** Kitchen Quest does and **why**. Related documents:

| Document | Contents |
|---|---|
| [screens.md](screens.md) | Screens, URLs, route guards, admin sidebar |
| [data-model.md](data-model.md) | Tables, relationships, data rules |
| [architecture.md](architecture.md) | Tech stack, security layers, deployment modes, hosting |
| [design-tokens.md](design-tokens.md) | Colors, fonts, shapes, MUI theme |
| [decisions.md](decisions.md) | Key technical decisions and why |

## Overview

Kitchen Quest is a web app where restaurant staff learn the ingredients and steps of each menu item, then check what they remember through a game. The MVP is built for one restaurant first, with a design that can grow into a SaaS later.

- **Background**: Staff are expected to keep their own notes on ingredients and steps, but many don't. Putting everything in one app removes the excuse not to study.
- **Goal 1**: Staff can study menu items and check what they remember through the game, anytime, anywhere.
- **Goal 2**: Admins can see at a glance who has learned which menu items, and how well.

## Glossary

Use these words consistently in the UI, docs and code. Do not use "test", "quiz" or "exam" for the game.

| Term | Meaning | UI label | In code |
|---|---|---|---|
| Study | Reading a menu item's ingredients, amounts and steps | Study | `study` |
| Game | The feature where staff pick ingredients, answer amount questions and order steps | Game | `game` |
| Game attempt | One play of the game for one menu item, saved with its score | (not shown as a word) | `game_attempts` table |
| Score | The result of an attempt, out of 100 | Score | `score` |
| Cleared | Scored 100 at least once on a menu item | Cleared | calculated, not stored |
| Amount question | An amount that must be entered in the game. Set by an admin on the menu edit screen | Admin checkbox: **Ask in game** | `ask_in_game` |
| Kitchen status | Whether a user can make an item in the real kitchen | Kitchen: Not yet / With assistance / Independent | `kitchen_statuses` table; values `no_experience` / `with_assistance` / `independent` |

"Test" is only used for automated software tests (Vitest, Playwright).

People: "user" means anyone who can sign in. A user's role is `admin` or `staff`. Docs use only these words for people: "user" for anyone, "admin" and "staff" for the roles. There is no player, trainee, trainer or member concept.

## Users and roles

There are two roles. Admins also review everyone's progress.

| Role | Who | Can do |
|---|---|---|
| Admin | Owner, training lead | Manage users; register menus, ingredients and steps; view everyone's progress |
| Staff | New and current kitchen staff | Study recipes, play the game, view their own progress |

- The role is stored as `role` with the values `admin` and `staff`.
- People cannot sign up on their own. Admins create accounts.
- One account per user.

## Features: staff

The core of the staff side is **Study** and **Game**. After signing in, staff only see active menu items.

### Study

- Each menu item shows its target cooking time (the time staff should aim to finish in), ingredients (name and amount) and steps (text plus an optional tip).
- Photos (if any) are shown at the bottom of the Study screen, full width, one under the other.
- **Hide & practice** mode blurs amounts and steps. Tapping an item reveals it. A **Reveal all** button reveals everything at once.
- Study views and time spent are not tracked.

### Game

1. Choose a menu item.
2. **Pick ingredients**: From all active ingredients, pick the ones this menu item uses. The prompt states how many to pick (for example "Pick the 7 ingredients you need"), and staff cannot pick more than that. A counter such as "5/7 picked" is shown, and other ingredients become unselectable once the limit is reached. Staff can check answers with fewer than the limit. Category tabs and search narrow the list. Picked ingredients are listed at the top.
3. **Check answers**: Correct, missed and not-used ingredients are shown in different styles.
4. **Amounts**: For each ingredient whose amount is marked as an amount question, ask a question such as "How many grams of chicken thigh?" The unit is shown in the question, and staff enter the number only. Decimals are allowed. An answer is correct only if it exactly matches. Menu items with no amount questions skip this step.
5. **Order the steps**: Tap the shuffled step cards in order, starting from the first step. Placed steps can be reordered by dragging the handle (≡) on the left, and × returns a card to the pool. Only the handle is draggable, not the whole card, so dragging does not conflict with scrolling on phones.
6. **Result**: Show right and wrong for ingredients, amounts and steps, and a total score out of 100. Each attempt is saved. Menu photos are not shown during the game, because the look of the dish would give away the ingredients.

**Scoring**

- Number of questions = number of ingredients + number of amount questions + number of steps. Every question has the same weight.
- Correct ingredients = number of correctly picked ingredients. Because picks are capped, a wrong pick takes the place of a correct one, so there is no penalty subtraction.
- Correct amounts = number of amount questions whose answer exactly matches.
- Correct steps = number of steps placed in the correct position.
- Total score = (correct ingredients + correct amounts + correct steps) ÷ number of questions × 100, rounded to an integer.
- Cleared = 100 (all correct).
- Result screen example: "60 / 100 (9/15 correct)", followed by "Items 4/7", "Amounts 1/2", "Steps 4/6".
- Progress tables show the total score. The admin's "best score" is the attempt with the highest total score.

### Set menus

- A set is a combination of single menu items only (for example a set meal or a bento). Sides such as small dishes or pickles are registered as single menu items too. A part that is never sold on its own but has its own recipe (for example the teriyaki plate in "Chicken Teriyaki" = teriyaki plate + salad) is registered as a single item marked **Set only**. Set-only items can be studied and played like any single item, and are labeled "Part of a set" in the menu list.
- Set components are always menu items, never ingredients. Garnishes and add-ons that are only placed on the plate (for example the lemon with karaage) are ingredients of the single item they come with, so a set lists only the dishes. Sets have no ingredients or steps of their own, but each component has its own amount in the set (for example "Karaage 3 pcs" in a bento, even if the Karaage plate uses 5). Component amounts work exactly like ingredient amounts: "number + unit" or a note, and each can be marked "Ask in game".
- **Study**: Shows the set's components with their amounts. Hide & practice works the same as for single items. Tapping a component opens that item's recipe. When opened from a set, the recipe shows a note with the amount in that set (for example "In Karaage Bento: 3 pcs"), because the recipe's own amounts are for the single item. Everything else in the recipe, such as garnishes (for example the lemon), is the same in the set.
- **Unlock rule**: A set's game unlocks once the user has cleared (scored 100 at least once) every single item in the set. This is calculated from game attempts every time. Inactive single items are left out of the unlock rule, so that retiring a seasonal item never makes a set impossible to unlock.
- Locked sets show a lock icon and the unlock condition: which items still need to be cleared (for example "Clear Teriyaki plate and Salad to unlock"). Items already cleared are marked as done. Where there is no room for the full text (such as a narrow list row), an info button shows it on tap. The Study screen of a locked set shows the condition in full.
- **Game**: (1) Pick the set's components. As with ingredients, the prompt states how many to pick and picks are capped. The choices are all active single menu items (not the ingredient list). (2) Amount questions for components marked "Ask in game" (for example "How many pcs of karaage?"), answered the same way as ingredient amount questions. Sets with no amount questions skip this step. There is no step ordering.
- **Scoring**: Total score = (correct components + correct amounts) ÷ (number of components + number of amount questions) × 100, rounded to an integer. The result screen shows something like "Components 3/4", "Amounts 1/2".
- Only the game is locked. The Study screen is always available, even while a set is locked.

### My progress

- Menu items run down the side. Each item's game scores run across, newest first, so every row shows its latest score at the left without scrolling. The latest score is highlighted.
- When the row gets long, it scrolls horizontally to show older scores. The menu name column stays fixed.
- Menu items cleared at least once are highlighted. The kitchen status is shown next to the menu name (view only).
- Items not yet attempted show "-".
- **Filter** by menu category, kitchen status (Not yet / With assistance / Independent) and game result (Cleared / Not cleared / Not attempted). Filters can be combined.
- **Sort** by menu name (A to Z; the default), latest game score (low to high, to find weak items; not attempted first) or kitchen status (Not yet → With assistance → Independent).
- The chosen filters and sort are remembered per user on that device.

## Features: admin

Admins manage user accounts, recipe data and everyone's progress.

### User management

- Invite a user with a display name and email. The user receives an invite email and sets their own password from the link. The display name is only used to tell people apart in progress tables, so a full name is not needed (a first name or nickname is fine).
- Send a password reset email. Users can also reset their own password from the sign-in screen.
- Deactivate and reactivate accounts. Past game attempts are kept when an account is deactivated.
- Deactivated users cannot sign in.

### Ingredients

- Add and edit ingredients (name, category).
- Each store defines its own ingredient categories on the Categories screen. The app ships with no fixed list; demo mode loads sample categories as part of its seed data.
- Prepped items (for example a house soup stock) can be registered as a single ingredient without revealing what is in them. Recipes for prepped items are intentionally never stored in the app, now or later, because they are often the store's most confidential know-how.
- Ingredients can be deactivated. Inactive ingredients do not appear as choices in the game.

### Menus

- Add and edit menu items (name, menu category, target cooking time, dish photos). Each menu item can have up to two photos. What they show is up to each store (for example dine-in and take-out, which often look different). Photos are optional and resized for display when uploaded.
- Pick ingredients from the ingredient list and enter amounts. An amount is either "number + unit" (for example 2 pcs, 150 g) or a note for amounts that are not numbers (for example "to taste"). Common units can be picked from suggestions. Each ingredient's amount can be marked as an amount question (checkbox labeled "Ask in game"); an amount question requires both a number and a unit.
- Enter and reorder steps. Each step can have an optional tip. Menu items are either **Single** or **Set**. A single item can be marked **Set only** when it is never sold on its own. Sets have no ingredients or steps; instead, admins pick the single items that make up the set and enter each one's amount in the set, with the same "Ask in game" checkbox as ingredients.
- Menu items can be deactivated. Inactive items are hidden from staff, but past game attempts are kept.
- The add/edit screen always shows this notice: "Everything you enter is shown on staff phones and could spread through screenshots. Do not enter confidential recipes or secret blends."

### Categories

- Add, rename and reorder menu categories (for example Rice bowls, Mains, Soups & sides, Sets) and ingredient categories.
- Categories in use cannot be deleted, only renamed.
- The order is used for menu list sections and for the ingredient tabs in the game (for example, the same order as the restaurant's menu board, or an order that matches how the kitchen thinks).
- Managed on a dedicated Categories screen, with tabs for menu categories and ingredient categories.

### Kitchen status

Separate from game results, admins record whether a user can actually make each item in the kitchen. This cannot be calculated, so it is stored as data.

- One status per user × menu item (single and set): **Not yet** (has not made it in the kitchen yet) / **With assistance** (needs a supervisor) / **Independent** (can be left to make it alone).
- Only admins can change it. Staff can only view their own.
- Every change is logged with who changed it, when, and from which status to which.
- It is not tied to game results. It can be set even if the user has not cleared the game, because people often learn a dish on their first day by working on it.
- Set menus have a kitchen status too.

### Team progress

- Menu items run down the side and users run across. Users include admins as well as staff (each can be hidden). Each user's column is split into **Game** and **Kitchen** (grouped column headers).
- Game cells: cleared at least once → ✓; tried but not cleared → best score; not attempted → -. Kitchen cells: the kitchen status, which admins can change right in the cell.
- A toggle switches between "Active users" and "Include inactive users". Individual users can also be hidden (hiding a user hides both of their columns). The chosen view is remembered in the browser.
- The menu name column stays fixed, and the table scrolls horizontally.

## Non-functional requirements

Kitchen Quest is a web app that staff can use from their own phones, anywhere.

- **Platform**: A web app in the browser. Not distributed through app stores.
- **Screens**: Staff screens (`/app`) prioritize phones in portrait. Admin screens work on both desktop and phone.
- **Language**: The UI (buttons, headings, messages) supports English (`en`) and Japanese (`ja`) from the MVP (i18n). UI strings live in one file per language, not in the code, so more languages can be added later. Each user can choose a language from the account menu in the header. The default is English. Content entered by admins (menu names, ingredients, steps) can be in any language and is stored, shown and searched as entered. The demo environment uses English.
- **Authentication**: Sign-in is required for every screen. Sign-in uses email and password. Forgotten passwords are reset from the sign-in screen by email.
- **Session**: Users can stay signed in on their own phone.
- **Information handling**: Content is limited to what staff would be allowed to write in their own notes. Screenshots cannot be prevented technically, so sign-in exists to keep unrelated people out and to record who has learned what. Confidential recipes are kept out by policy, reinforced by the notice on the admin edit screen.
- **Access control**: Staff cannot reach admin screens or admin data. Deactivated users lose access immediately. See [architecture.md](architecture.md#layers-of-protection).
- **Personal data**: Only a display name and email are collected. Staff cannot see other people's emails, and admins only see users in their own store. On request, a user's email can be deleted while their game attempts remain under their display name.
- **Readability first**: This is a training tool, so game-like styling must never make text harder to read. No text below 12px (labels 13px or more), sentence case instead of all caps, no pixel or novelty fonts. See [design-tokens.md](design-tokens.md).

## Out of scope for the MVP and future ideas

None of these are in the MVP. Because every game attempt is stored, any of them can be added later.

| Item | Reason / note |
|---|---|
| Study time tracking | It grows even if the screen is left open, so it is not meaningful |
| Recipe versions and retakes | Small changes to a dish people already know can be passed on in the kitchen |
| Beginner / advanced modes | The game uses one mode: picking from all ingredients |
| A third role (for example one that can only view progress) | Admins review progress in the MVP |
| Multi-store management, store sign-up, billing | Considered at the SaaS stage, after it proves useful in one restaurant |
| Translating recipe content | Storing menu names and steps per language. The MVP translates the UI only |
| Allergen information | Not discussed yet. Left out of the MVP for now |
| Ingredients as set components | For items that are only placed in a set and belong to no dish. Not needed now, because garnishes are part of each single item's recipe. If added, the set game's choices must keep dishes and ingredients apart (for example separate tabs), so the same thing cannot be both a menu choice and an ingredient choice |
