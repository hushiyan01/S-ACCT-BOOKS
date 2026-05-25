# UI Design Guide for S-ACCT-BOOKS (Claude-driven)

Instead of producing Figma mockups, we generate and iterate on React + Material UI components directly using Claude (via Claude Code). The "design" deliverable is working code in the web app, reviewed in a running dev server.

## Workflow

1. **Design tokens first.** Colors, typography, spacing, radii, and elevation are defined as MUI theme constants in `src/theme/`. This is the canonical "design system."
2. **Component library, iteratively.** Ask Claude to scaffold each reusable component (Button variants, `TransactionCard`, `BalanceCard`, etc.) directly in React/MUI. Iterate by running the dev server and asking for adjustments.
3. **Screen-by-screen build.** Work through screens in the order defined in `PRODUCT_SPEC.md`. For each screen, prompt Claude with the relevant spec section, current theme, and any neighboring already-built screens (for consistency).
4. **States and edge cases.** For every screen, also generate loading, empty, and error states.
5. **Responsive check.** Verify at 375 px (mobile web), 768 px (tablet), 1440 px (desktop). Use MUI's `sx` breakpoints and `Grid` / `Stack` utilities.

## Design Tokens

These live in source as MUI theme overrides under `src/theme/`. Reference them in every prompt to keep visual consistency.

### Color Palette

**Primary**
- `primary.main` — `#1976D2`
- `primary.light` — `#42A5F5`
- `primary.dark` — `#1565C0`
- `primary.contrastText` — `#FFFFFF`

**Secondary**
- `secondary.main` — `#424242`
- `secondary.light` — `#616161`
- `secondary.dark` — `#212121`

**Semantic**
- `success.main` (income) — `#4CAF50`
- `error.main` (expense) — `#F44336`
- `warning.main` (budget alert) — `#FF9800`
- `info.main` — `#2196F3`

**Neutral / Surface**
- `background.default` — `#FAFAFA`
- `background.paper` — `#FFFFFF`
- `text.primary` — `#212121`
- `text.secondary` — `#757575`
- `divider` — `#E0E0E0`

> Dark mode is enabled in MVP. Define a parallel `darkPalette` in the theme; respect `prefers-color-scheme` and expose a manual toggle in Settings.

### Typography

Font: **Inter** (web). Use MUI's `Typography` variants.

| Variant | Size | Weight | Line height |
|---|---|---|---|
| h1 | 32 px | 700 | 40 px |
| h2 | 24 px | 600 | 32 px |
| h3 | 20 px | 600 | 28 px |
| h4 | 18 px | 500 | 24 px |
| body1 | 16 px | 400 | 24 px |
| body2 | 14 px | 400 | 20 px |
| caption | 12 px | 400 | 16 px |
| button | 14 px | 500 | uppercase, letter-spacing 0.5 px |

**Amount** styles (custom variants) for money use `font-variant-numeric: tabular-nums` so columns align.

### Spacing — 8 px grid via `theme.spacing(n)`

| Token | px |
|---|---|
| `theme.spacing(0.5)` | 4 |
| `theme.spacing(1)` | 8 |
| `theme.spacing(1.5)` | 12 |
| `theme.spacing(2)` | 16 |
| `theme.spacing(3)` | 24 |
| `theme.spacing(4)` | 32 |
| `theme.spacing(6)` | 48 |
| `theme.spacing(8)` | 64 |

### Border Radius

- `shape.borderRadius` — 8 px (default)
- Small — 4 px
- Large — 16 px
- Pill — 999 px

### Elevation (MUI `Paper.elevation`)

- `1` — cards
- `2` — raised cards
- `4` — modals
- `8` — drawers / menus

### Breakpoints (MUI defaults)

| Key | min-width |
|---|---|
| `xs` | 0 |
| `sm` | 600 |
| `md` | 900 |
| `lg` | 1200 |
| `xl` | 1536 |

Primary target widths: 375 (mobile), 768 (tablet), 1440 (desktop).

## Components to Build

Each lives under `src/components/`. Build in this order:

1. **Theme + global styles** — `src/theme/index.ts`
2. **Layout primitives** — `AppShell` (header + sidebar/bottom-nav), `PageContainer`
3. **Form primitives** — `TextField` wrappers, `NumberInput` (for amount), `Select`, `DatePicker`
4. **Money components** — `AmountDisplay` (color-coded, tabular nums), `CurrencyBadge`
5. **TransactionCard** — variants: compact / default / detailed; expense / income; shared / personal badge
6. **BalanceCard** — large amount, trend indicator, period selector
7. **CategoryChip / CategoryIconButton** — for predefined categories
8. **EmptyState / ErrorState / LoadingSkeleton**
9. **Modal/Dialog wrappers** — built on MUI `Dialog`
10. **Charts** (Phase 4) — `Recharts` wrappers for pie / line / bar

## Screens to Build (in order, matching `PRODUCT_SPEC.md`)

1. Auth — Login, Register, Create-Ledger post-registration step
2. Dashboard
3. Transaction List (with filters)
4. Add/Edit Transaction (modal)
5. Settings / Profile
6. Reports (Phase 4)
7. Ledger Management (Phase 3)

For each screen also build:
- Loading state (skeleton)
- Empty state
- Error state
- Layouts at 375 / 768 / 1440 px

## Prompting Claude for UI Work

Effective prompts include:
- Path to the current theme file and at least one already-built neighboring component (for consistency)
- Target breakpoint(s)
- Reference to the screen section in `PRODUCT_SPEC.md`
- Reference to Material Design 3 patterns where applicable

Iterate quickly in conversation: "Make the amount larger and right-aligned." "Move the category chip to the left of the description." "Add a skeleton state for when transactions are loading."

## Accessibility (enforced)

- WCAG 2.1 AA contrast ratios (≥ 4.5:1 for text)
- Keyboard navigation: tab order + focus rings (MUI handles via theme)
- Screen-reader labels on icon-only buttons (`aria-label`)
- Touch targets ≥ 44 × 44 px on mobile breakpoints
- Form fields have visible labels (no placeholder-only labels)

## Definition of Done (per screen)

- [ ] Implements all fields/sections from `PRODUCT_SPEC.md` for that screen
- [ ] Uses only theme tokens (no hard-coded colors, sizes, or spacing)
- [ ] Loading, empty, and error states implemented
- [ ] Verified visually at 375 / 768 / 1440 px
- [ ] Keyboard-navigable
- [ ] Contrast passes axe-core or Lighthouse audit
- [ ] No React or browser console warnings

## Design System Export (for Android in Phase 2)

The MUI theme is the source of truth. To share with Android in Phase 2, export tokens as JSON from `src/theme/tokens.ts` and generate a matching Kotlin `Tokens.kt` so the two platforms stay in sync.

```json
{
  "colors": {
    "primary": { "main": "#1976D2", "light": "#42A5F5", "dark": "#1565C0" },
    "semantic": { "income": "#4CAF50", "expense": "#F44336" }
  },
  "typography": { "fontFamily": "Inter", "h1": { "fontSize": "32px", "fontWeight": "700", "lineHeight": "40px" } },
  "spacing": { "xs": "8px", "sm": "12px", "md": "16px", "lg": "24px" }
}
```

## Reference Apps for Inspiration

- **Splitwise** — shared expense tracking patterns
- **Money Lover** / **Spendee** — category visualization
- **Wallet by BudgetBakers** — multi-currency
- **Material Design 3 showcase** — https://m3.material.io

## Out of Scope (vs the old Figma workflow)

- No `.fig` deliverables, no Figma component library, no Figma prototype links
- No separate designer ↔ developer handoff step — the design *is* the code
- Visual review happens in the running app, not in Figma
