# Kitchen Quest Design Tokens

Theme: **paper & stainless steel**. Content sits on warm paper-colored cards on a stainless counter background, with heat-lamp orange for the main actions.

Readability comes first. This is a training tool, so any game feel must never make text harder to read.

## Colors

### Steel (structure)

| Token | Hex | Use |
|---|---|---|
| `steel-900` | `#2B3134` | Header, bottom bar, admin sidebar, "Independent" status, correct answers |
| `steel-700` | `#4A5357` | Finished game steps, disabled primary button, sidebar divider |
| `steel-500` | `#8D969A` | Dashed borders ("Not yet", missed items, empty slots), drag handles |
| `steel-300` | `#B9C1C5` | Input and chip borders |
| `steel-200` | `#C9CFD2` | Dividers, card shadow line, photo placeholder |
| `steel-150` | `#D6DBDD` | Dotted row lines |
| `steel-100` | `#E3E7E9` | Neutral badge background, blurred practice text |
| `counter` | `#E8ECEE` → `#DCE1E4` | Page background (vertical gradient, top to bottom) |
| `field` | `#F6F8F9` | Search fields, unplaced step cards |

### Paper (content)

| Token | Hex | Use |
|---|---|---|
| `paper` | `#FFFDF7` | Cards, tables, inputs |
| `paper-head` | `#F6F3EA` | Table header row |

### Text

| Token | Hex | Use |
|---|---|---|
| `ink` | `#1E2326` | Main text |
| `ink-muted` | `#5E666B` | Secondary text on paper |
| `ink-muted-steel` | `#4F575C` | Secondary text on the steel background |
| `on-steel` | `#F3F5F6` | Text on `steel-900` |
| `on-steel-muted` | `#C8CED1` | Secondary text on `steel-900` |
| `on-steel-faint` | `#9AA2A6` | Tertiary text on `steel-900` |

### Accent (heat-lamp orange)

| Token | Hex | Use |
|---|---|---|
| `orange` | `#F08A3C` | Primary buttons, selected chips, current game step, active nav marker. **Text on it is always `ink`, never white.** |
| `orange-pressed` | `#C9692A` | 3px inner bottom shadow on primary buttons |
| `orange-soft` | `#FDE4D0` | "With assistance" status, best-score badge, editing row, warning banner |
| `orange-line` | `#F5C59E` | Borders on `orange-soft` areas |
| `orange-text` | `#B84A14` | Links and small accent text on paper |
| `orange-text-hover` | `#8F3810` | Link hover |
| `warning-text` | `#6A2A0C` | Text in the warning banner |

### Feedback

| Token | Hex | Use |
|---|---|---|
| `error` | `#9B2C2C` | Wrong answers. Always shown with × and/or strikethrough, never by color alone |
| `error-soft` | `#F5DEDA` | Background behind wrong answers |

Correct answers use `steel-900` with a ✓, not green. Do not use ○ for correct: it reads as "correct" in Japan but not in North America. Use a drawn check icon or the ✓ text glyph, not a color emoji (✅), so it renders the same on every device and stays in the palette.

### Contrast checks

| Text / background | Ratio |
|---|---|
| `ink` on `paper` | 15.6 : 1 |
| `ink-muted` on `paper` | 5.8 : 1 |
| `ink-muted-steel` on `steel-100` | 5.9 : 1 |
| white on `steel-900` | 13.2 : 1 |
| `on-steel-muted` on `steel-900` | 8.3 : 1 |
| `ink` on `orange` | 6.4 : 1 |
| `ink` on `orange-soft` | 13.0 : 1 |
| `orange-text` on `paper` | 5.1 : 1 |
| `error` on `error-soft` | 5.9 : 1 |
| `orange` on `steel-900` (active bottom-nav item) | 5.3 : 1 |

White text on `orange` is only 2.5 : 1, so do not use it.

## Status styles

### Kitchen status (set by admins)

| Status | Label | Style |
|---|---|---|
| Not yet | `Not yet` | Transparent, 1px dashed `steel-500` border, `ink-muted-steel` text |
| With assistance | `With assistance` | `orange-soft` background, 1.5px `orange` border, `ink` text |
| Independent | `Independent` | `steel-900` background, white text |

Lightness rises step by step (outline → light fill → dark fill), so the three states can be told apart without relying on hue.

### Game results

| State | Staff view | Admin table |
|---|---|---|
| Cleared (100) | `100 ✓` on `steel-900`, row gets a 4px `steel-900` left bar | `✓` in `ink` |
| Tried, not cleared | Score in `IBM Plex Mono` | Best score |
| Not attempted | `-` in `ink-muted` | `-` |
| Set locked | Lock icon + `Locked` | Lock icon |

### Game answers

| State | Style |
|---|---|
| Correct | `steel-900` fill, white text, ✓ |
| Missed (should have picked) | 1.5px dashed `steel-900` border |
| Wrong (not used) | `error-soft` background, `error` text, ×, strikethrough |

## Typography

All three families are on Google Fonts.

| Role | Family | Weights | Use |
|---|---|---|---|
| Heading | IBM Plex Sans Condensed | 600, 700 | Page titles, menu names, app name, big score |
| Body | IBM Plex Sans | 400, 500, 700 | All running text, buttons, form labels |
| Data | IBM Plex Mono | 500, 600 | Numbers, scores, amounts, step numbers, small labels |

```html
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans+Condensed:wght@600;700&family=IBM+Plex+Sans:wght@400;500;700&family=IBM+Plex+Mono:wght@500;600&display=swap" rel="stylesheet">
```

| Style | Size / weight | Example |
|---|---|---|
| Display score | Condensed 700, 68px | Result score "60" |
| Page title | Condensed 700, 26–32px | "Menu", "Team progress" |
| Question | Condensed 700, 32px | "How many grams of chicken thigh?" |
| Section title | Sans 700, 17–18px | "Pick the 7 ingredients you need" |
| Body | Sans 400, 15px, line-height 1.55 | Recipe steps |
| Button | Sans 700, 16–17px | "Check answers" |
| Small / label | Sans or Mono, **13px minimum** | "Target 5 min", "Cleared 2/7" |

Rules:

- No text below 12px. Keep labels at 13px or more.
- Use sentence case. Avoid all-caps labels, because they are harder to read.
- Avoid wide letter-spacing. Keep it at 0.02em or less.
- No pixel or novelty fonts.

## Shape and depth

| Token | Value | Use |
|---|---|---|
| `radius-sm` | 2–3px | Badges, table panels |
| `radius-md` | 4px | Chips, inputs, cards |
| `radius-lg` | 6px | Primary buttons |
| `shadow-card` | `0 1px 0 #C9CFD2, 0 10px 18px -14px rgba(30,35,38,0.6)` | Paper cards on the counter |
| `shadow-button` | `inset 0 -3px 0 #C9692A` | Primary orange button |
| Touch target | 44px minimum (primary buttons 52–54px) | All tappable elements |

Cards are plain MUI `Paper` with `radius-md` corners and `shadow-card` (set once in the theme).

## MUI theme

This is the single source of truth for colors, fonts and shapes in code. The tables above are the reference for people; if a value changes, change it here and in the table together. Do not hard-code hex values in components.

`cssVariables: true` makes MUI expose the theme as CSS variables (for example `var(--mui-palette-steel-900)`), so plain CSS can use the same values.

```ts
// src/theme/theme.ts
import { createTheme } from "@mui/material/styles";

const fonts = {
  heading: '"IBM Plex Sans Condensed", "IBM Plex Sans", system-ui, sans-serif',
  body: '"IBM Plex Sans", system-ui, sans-serif',
  mono: '"IBM Plex Mono", ui-monospace, monospace',
};

export const theme = createTheme({
  cssVariables: true,
  palette: {
    mode: "light",
    primary: { main: "#F08A3C", dark: "#C9692A", light: "#FDE4D0", contrastText: "#1E2326" },
    secondary: { main: "#2B3134", light: "#4A5357", contrastText: "#FFFFFF" },
    error: { main: "#9B2C2C", light: "#F5DEDA", contrastText: "#FFFFFF" },
    background: { default: "#E8ECEE", paper: "#FFFDF7" },
    text: { primary: "#1E2326", secondary: "#5E666B" },
    divider: "#C9CFD2",
    steel: { 900: "#2B3134", 700: "#4A5357", 500: "#8D969A", 300: "#B9C1C5", 200: "#C9CFD2", 150: "#D6DBDD", 100: "#E3E7E9" },
    counter: { top: "#E8ECEE", bottom: "#DCE1E4" },
    field: "#F6F8F9",
    paperHead: "#F6F3EA",
    inkMutedSteel: "#4F575C",
    onSteel: { main: "#F3F5F6", muted: "#C8CED1", faint: "#9AA2A6" },
    accent: { line: "#F5C59E", text: "#B84A14", textHover: "#8F3810", warningText: "#6A2A0C" },
  },
  shape: { borderRadius: 4 },
  typography: {
    fontFamily: fonts.body,
    h1: { fontFamily: fonts.heading, fontWeight: 700, fontSize: "2rem", lineHeight: 1.2 },
    h2: { fontFamily: fonts.heading, fontWeight: 700, fontSize: "1.625rem", lineHeight: 1.2 },
    h3: { fontWeight: 700, fontSize: "1.125rem" },
    body1: { fontSize: "0.9375rem", lineHeight: 1.55 },
    body2: { fontSize: "0.8125rem" },
    button: { fontWeight: 700, textTransform: "none" },
    caption: { fontSize: "0.8125rem" },
    data: { fontFamily: fonts.mono, fontWeight: 600, fontSize: "0.875rem", letterSpacing: "0.02em" },
  },
  components: {
    MuiCssBaseline: {
      styleOverrides: {
        body: { background: "linear-gradient(180deg, #E8ECEE 0%, #DCE1E4 100%)", minHeight: "100vh" },
      },
    },
    MuiButton: {
      defaultProps: { disableElevation: true },
      styleOverrides: {
        root: { minHeight: 44 },
        containedPrimary: { boxShadow: "inset 0 -3px 0 #C9692A", minHeight: 52, borderRadius: 6 },
      },
    },
    MuiPaper: {
      defaultProps: { elevation: 0 },
      styleOverrides: { root: { boxShadow: "0 1px 0 #C9CFD2, 0 10px 18px -14px rgba(30,35,38,0.6)" } },
    },
    MuiLink: { styleOverrides: { root: { color: "#B84A14", "&:hover": { color: "#8F3810" } } } },
  },
});
```

The custom palette keys (`steel`, `counter`, `field`, `paperHead`, `inkMutedSteel`, `onSteel`, `accent`) and the `data` typography variant need TypeScript module augmentation:

```ts
// src/theme/mui.d.ts
import type { CSSProperties } from "react";
import "@mui/material/styles";

declare module "@mui/material/styles" {
  type Steel = Record<900 | 700 | 500 | 300 | 200 | 150 | 100, string>;
  interface Palette {
    steel: Steel;
    counter: { top: string; bottom: string };
    field: string;
    paperHead: string;
    inkMutedSteel: string;
    onSteel: { main: string; muted: string; faint: string };
    accent: { line: string; text: string; textHover: string; warningText: string };
  }
  interface PaletteOptions extends Partial<Palette> {}
  interface TypographyVariants { data: CSSProperties }
  interface TypographyVariantsOptions { data?: CSSProperties }
}

declare module "@mui/material/Typography" {
  interface TypographyPropsVariantOverrides { data: true }
}
```
