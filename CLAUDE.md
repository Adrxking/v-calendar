# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

v-calendar is a calendar and date picker plugin for Vue 3. It provides `Calendar`, `DatePicker`, `Popover`, and `PopoverRow` components, along with composables (`createCalendar`, `createDatePicker`) for programmatic use. The library supports daily/weekly/monthly views, date ranges, time selection, timezone handling, and an attribute-based styling system.

## Commands

```bash
yarn build          # Build library (types via vue-tsc, then ES/MJS/CJS/IIFE via Vite)
yarn test           # Run tests with Vitest (interactive watch mode by default)
yarn test --run     # Run tests once without watch mode
yarn lint           # ESLint on .js and .vue files
yarn format         # Prettier on src/**/*.{js,jsx,ts,tsx,json,vue}
```

To run a single test file:
```bash
npx vitest tests/unit/specs/Calendar.spec.ts
```

Node version: 18.14.2 (see .nvmrc)

## Architecture

### Source Layout (`src/`)

- **`index.ts`** — Plugin entry point. Exports the Vue plugin (`install`), all components, `setupCalendar` (global defaults), composables, and `popoverDirective`.
- **`components/`** — Vue SFC components: `Calendar`, `DatePicker`, `CalendarGrid`, `Popover`, `BaseSelect`, `BaseIcon`. DatePicker is a thin wrapper around DatePickerBase/DatePickerPopover.
- **`use/`** — Vue 3 composables. Core logic lives here, not in the components. `calendar.ts` (`createCalendar`/`useCalendar`) and `datePicker.ts` (`createDatePicker`/`useDatePicker`) are the main ones.
- **`utils/`** — Utility modules:
  - `date/helpers.ts` — Core date manipulation (wraps date-fns)
  - `date/range.ts` — Date range management
  - `date/rules.ts` — Date validation rules
  - `locale.ts` — Locale/i18n handling
  - `attribute.ts` — The attribute system (core customization concept)
  - `defaults/` — Global config and defaults (locales, masks, touch)
  - `popovers.ts` — Popover state management
  - `theme.ts` — Theme utilities
- **`styles/`** — PostCSS stylesheets (uses postcss-nested, postcss-simple-vars, postcss-inline-svg)

### Build System

Build is orchestrated by `build/build.ts` using tsx. It first generates TypeScript declarations via vue-tsc, then builds four formats sequentially using per-format Vite configs in `build/configs/`:
- `vite.es.ts` → `dist/es/` (ES modules)
- `vite.mjs.ts` → `dist/mjs/` (.mjs files for Node ESM)
- `vite.cjs.ts` → `dist/cjs/` (CommonJS)
- `vite.iife.ts` → `dist/iife/` (browser bundle, minified with Terser)

Common Vite config is in `build/configs/vite.common.ts`. Vue and @popperjs/core are externalized.

### Testing

Vitest with jsdom environment. Globals mode enabled (no imports needed for `describe`/`it`/`expect`). Setup file at `tests/unit/setup.ts` provides a ResizeObserver polyfill. Test specs are in `tests/unit/specs/`. Test helpers are in `tests/unit/specs/navigation.ts`, `slots.ts`, and `utils.ts`.

### Key Design Patterns

- **Composable-first**: Components are thin wrappers; logic lives in `src/use/` composables with `create*` factory functions.
- **Attribute system**: The core customization mechanism. Prefer extending attributes over adding new component props.
- **Provide/Inject**: Used for page context (CalendarPageProvider), slot rendering (CalendarSlot), and theme/config injection.
- **Plugin with prefix**: Global install registers components with a configurable prefix (default: 'V'), e.g., `VCalendar`, `VDatePicker`.

### Path Alias

`@/*` maps to `src/*` (configured in tsconfig.json and Vite).

## Code Style

- TypeScript with strict mode. `no-explicit-any` is allowed (ESLint rule off).
- Vue 3 Composition API with `<script setup>` or setup functions.
- Props defined via `propsDef` objects, emits via `emits` objects.
- Prettier: single quotes, trailing commas, no parens on single arrow params, 80-char width.
- Console/debugger statements only allowed in development.
