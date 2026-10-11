# TACHYON - House Points Arena

A futuristic esports-style dashboard for running a house points competition between **ALPHA**, **INFER**, **MATRIX** and **LAYER**. Award points, watch the leaderboard re-order live, review and edit the full score history, and export everything. All data is stored in the browser, so there is no backend to set up.

Built with React 18, Vite, TypeScript, Tailwind CSS, Framer Motion, Lucide icons and Recharts.

## Run it

```bash
npm install
npm run dev        # http://localhost:5173
```

| Command | What it does |
| --- | --- |
| `npm run dev` | Start the dev server |
| `npm run build` | Type-check, then create a production build in `dist/` |
| `npm run preview` | Serve the production build locally |
| `npm run typecheck` | TypeScript only |
| `npm test` | Unit tests for the scoring logic plus UI smoke tests (Vitest, jsdom) |

Requires Node 18 or newer. Fonts (Unbounded and Manrope) load from Google Fonts and fall back to system fonts when offline.

## Features

- **Dashboard**: a hero that names the current leader, plus animated counters for total points, leading house, transaction count and the score difference between first and second place.
- **House cards**: score, rank, share of all points, a progress bar relative to the leader, the latest award, and a cursor-following glow. Each card has its own Award button.
- **Leaderboard**: sorted by descending score with animated re-ordering. Ties share a rank (1, 1, 3, 4) and are labelled as tied. A banner shows the winner, a tie, or "no leader yet".
- **Award points**: house picker, whole-number validation (1 to 10,000), quick buttons (+5, +10, +25, +50, +100 add to the amount), optional reason (80 characters), live preview of the new total, and a success toast.
- **Score history**: timestamps, search (reason, house, points, date), house filters with counts, newest/oldest sorting, edit and delete with a confirmation dialog, and "show more" paging. Totals, ranks and charts recalculate after every change.
- **Analytics**: Recharts bar chart, donut (share of points) and cumulative trend line, all in house colours.
- **Settings**: dark/light theme, JSON and CSV export, load demo data, reset all points.
- **Persistence**: LocalStorage with demo data on first launch.
- **Celebrations**: when an award or edit gives a different house the sole lead, a confetti burst in that house's colours plays and a toast announces it. The trophy-row button on the leaderboard replays it.

### House colours

| House | Colour |
| --- | --- |
| ALPHA | cyan to blue |
| INFER | purple |
| MATRIX | emerald green |
| LAYER | magenta/pink |

## Scoring rules

All rules live in `src/lib/scoring.ts` and are covered by tests.

- Points per award: whole numbers from **1 to 10,000**. Negative values, decimals, signs and letters are rejected.
- **Ranking** uses competition ranking. Equal scores share a rank and the next rank is skipped. Tied houses are listed in the fixed order ALPHA, INFER, MATRIX, LAYER so the list never flickers.
- **Leader**: the house or houses with the highest score, provided it is above zero. With several leaders the app reports a tie instead of picking a winner.
- **Score difference**: top score minus the best score below it. It is 0 when tied.
- **Zero state**: with no points, there is no leader, ranks show as unranked, progress bars are empty and charts show an empty state.

## Data and error handling

- Everything is validated when read from LocalStorage. Invalid or duplicate entries are skipped and reported in a notice.
- If the stored JSON is unreadable, the app keeps a copy under `tachyon:arena:v1:corrupt`, starts empty and tells the user.
- If LocalStorage is blocked (private mode, policy), the app keeps working in memory and warns that changes will not persist.
- A React error boundary offers Reload and Clear saved data instead of a blank page.
- CSV export neutralises spreadsheet formulas (cells starting with `=`, `+`, `-` or `@`).

## Accessibility

- Semantic landmarks and headings, a skip link, and visible keyboard focus.
- Dialogs trap focus, close on Escape, restore focus and lock page scroll. Only the top-most dialog reacts to Escape when two are open.
- The house picker is a radio group with arrow-key navigation. Chart tabs are a real tablist.
- Errors are announced with `role="alert"`, and toasts use live regions.
- `prefers-reduced-motion` is respected: Framer Motion transform animations are disabled, counters jump to their final value, confetti is skipped, and CSS animations are shortened.

## Project structure

```
src/
  types.ts                 Shared types and house ids
  lib/
    scoring.ts             Pure scoring, ranking, validation, filtering (no React)
    scoring.test.ts        Unit tests
    storage.ts             LocalStorage load/save with corruption recovery
    demoData.ts            Sample week of activity
    export.ts              JSON/CSV builders and file download
    houses.ts              House metadata and colour helpers
    format.ts              Number, date and relative-time formatting
  hooks/
    useArena.ts            Transactions reducer + persistence
    useTheme.ts            Theme state
    useToasts.ts           Notification queue
  components/              Header, Hero, StatCards, HouseCard, Leaderboard,
                           AwardModal, HistoryPanel, ChartsPanel, SettingsPanel,
                           Modal, ConfirmDialog, Toasts, Celebration, ...
  App.tsx                  Composition and user flows
  index.css                Design tokens, glass utilities, chart theming
```

## Customising

- **House names and colours**: `src/types.ts` (ids), `src/lib/houses.ts` (icons and chart colours) and the `--c-*` variables in `src/index.css` (theme-aware UI colours).
- **Limits**: `MAX_POINTS_PER_AWARD` and `MAX_REASON_LENGTH` in `src/lib/scoring.ts`.
- **Demo data**: `src/lib/demoData.ts`.
