# Zenith Admin

A modern, opinionated admin template built on Next.js 16 (App Router), React 19, Tailwind CSS v4 and shadcn/ui — with a multi-channel CRM chat, a three-pane mail client, a calendar, a kanban board, a draggable widget dashboard, and a deep customization panel.

Forked from [arhamkhnz/next-shadcn-admin-dashboard](https://github.com/arhamkhnz/next-shadcn-admin-dashboard) and extended with the features below.

---

## Highlights

- **Customize panel (right-side Sheet)** — Theme · Color · Density · Layout · Container · Direction · Language, plus advanced controls (Font, Sidebar style, Navbar behavior, Collapse mode). Anti-flicker boot script applies preferences from cookie before hydration.
- **Soft pastel theme presets** — Neutral · Zinc · Blue · Violet · Rose · Orange. Color-only overrides (no radius/shadow changes), so swapping palettes never breaks the layout.
- **Notifications panel** — All / Unread tabs, 5 types (info, success, warning, error, mention), avatar fallback for actor messages, mark-all-as-read.
- **Multi-channel CRM chat** — WhatsApp, Telegram, Instagram, Messenger, Email, Live chat, SMS. Status (Open/Pending/Resolved), assignment, internal notes, customer side panel, quick replies.
- **Mail client** — three-pane (folders → list → viewer) with reply box, labels, attachments, search, star.
- **Calendar** — month view, event dialog, color-coded events.
- **Kanban** — DnD-kit drag&drop board.
- **Components showcase** — kitchen-sink page with every primitive plus Origin UI patterns: password strength, tag input, combobox, multi-select, OTP, rating, stepper, plan toggle, **phone input with libphonenumber-js validation**.
- **Authentication layouts** — v1 / v2 (login + register).
- **Form patterns** — `react-hook-form` + `zod` with field-level error states, including phone validation.
- **Persistent preferences** — cookie-based with full SSR consistency.

---

## Tech

- **Next.js 16** (App Router, Turbopack), **React 19**, **TypeScript**
- **Tailwind CSS v4** + **shadcn/ui** primitives
- **Zustand** (preferences store), **react-hook-form + zod**, **sonner** (toasts)
- **lucide-react** + **simple-icons** (brand logos)
- **libphonenumber-js**, **date-fns**, **@dnd-kit**
- **Biome** for lint/format · **Husky + lint-staged** on commit
- **pnpm** package manager

---

## Quick start

```bash
pnpm install
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) — you'll land on `/dashboard/default`.

Other useful commands:

```bash
pnpm build           # production build
pnpm start           # serve production build
pnpm exec tsc --noEmit          # type-check
pnpm exec biome check src/      # lint
pnpm exec biome check --write src/  # autofix
```

---

## Routes

| Path | Description |
| --- | --- |
| `/dashboard/default` | Overview dashboard |
| `/dashboard/analytics` | Analytics dashboard |
| `/dashboard/crm` | CRM dashboard |
| `/dashboard/finance` | Finance dashboard |
| `/dashboard/productivity` | Productivity dashboard |
| `/dashboard/draggable` | Draggable widget dashboard |
| `/dashboard/mail` | 3-pane mail client |
| `/dashboard/chat` | Multi-channel CRM messenger |
| `/dashboard/calendar` | Calendar with events |
| `/dashboard/kanban` | Kanban board |
| `/dashboard/components` | Component showcase + Origin UI |
| `/auth/v1/login`, `/auth/v2/login` | Login layouts |
| `/auth/v1/register`, `/auth/v2/register` | Register layouts |

---

## Project layout

```
src/
├── app/
│   ├── layout.tsx                    # Root layout — sets data-* prefs from defaults
│   ├── globals.css                   # Tailwind + theme presets + density tokens
│   └── (main)/
│       ├── auth/                     # Login / register (v1, v2)
│       └── dashboard/
│           ├── layout.tsx            # Sidebar + header (Notifications, Customize, Theme, Account)
│           ├── _components/sidebar/  # App sidebar, account switcher, search, layout-controls, notifications-sheet
│           ├── default/, analytics/, crm/, finance/, productivity/, …
│           ├── components/           # Showcase page (kitchen sink)
│           ├── mail/                 # 3-pane mail client
│           ├── chat/                 # 4-pane multi-channel CRM
│           ├── calendar/
│           ├── kanban/
│           └── draggable/
├── components/
│   ├── ui/                           # shadcn primitives (button, dialog, sheet, …)
│   ├── ui/phone-input.tsx            # Country selector + libphonenumber-js formatter
│   └── simple-icon.tsx               # simple-icons SVG renderer
├── data/                             # Sample/demo data (notifications, mail, chat, users)
├── navigation/sidebar/sidebar-items.ts
├── lib/
│   ├── preferences/                  # Layout & theme types, defaults, persistence, apply utils
│   ├── fonts/registry.ts             # Font options for the Customize panel
│   └── utils.ts                      # cn(...)
├── stores/preferences/               # Zustand store + provider
├── scripts/theme-boot.tsx            # Pre-hydration script for cookie-driven prefs
├── styles/presets/*.css              # Theme preset CSS (zinc, blue, violet, rose, orange)
├── server/server-actions.ts          # getPreference / setValueToCookie
└── config/app-config.ts              # APP_CONFIG (name, copyright, meta)
```

---

## Customization system

The Customize sheet (`/dashboard/_components/sidebar/layout-controls.tsx`) is wired through a single source of truth:

1. **Type & options** — `src/lib/preferences/layout.ts`
2. **Defaults & persistence** — `src/lib/preferences/preferences-config.ts`
3. **Zustand state** — `src/stores/preferences/preferences-store.ts`
4. **DOM apply** — `src/lib/preferences/layout-utils.ts` (sets `data-*` attributes on `<html>`)
5. **Anti-flicker** — `src/scripts/theme-boot.tsx` reads cookies and applies attributes before hydration

Adding a new preference (e.g. `density`) is the same 9-step flow each time. The full guide lives in [`CLAUDE.md`](./CLAUDE.md).

### Theme preset rule

Theme presets in `src/styles/presets/*.css` override **only color tokens** — `--primary`, `--primary-foreground`, `--ring`, `--sidebar-primary`, `--sidebar-ring`, `--chart-1`. **No radius, shadows, fonts or backgrounds.** Swapping palettes never breaks the layout.

To add a new color preset:
1. Create `src/styles/presets/<name>.css` (light + dark blocks)
2. Add `@import` in `src/app/globals.css`
3. Append entry to `THEME_PRESET_OPTIONS` in `src/lib/preferences/theme.ts`

---

## Adding a new dashboard page

1. **Sidebar entry** — `src/navigation/sidebar/sidebar-items.ts`
2. **Sample data** — `src/data/<feature>.ts`
3. **Components** — `src/app/(main)/dashboard/<feature>/_components/*.tsx` (mark client comps with `"use client"`)
4. **Page** — `src/app/(main)/dashboard/<feature>/page.tsx` (server component, exports `metadata`)

See `mail/`, `chat/`, `components/` for reference patterns. Push interactivity into a single `_components/<feature>-app.tsx` client container; keep the `page.tsx` server-side.

---

## Working with Claude Code

This repo ships with [`CLAUDE.md`](./CLAUDE.md) — conventions, architecture, where things live, and the exact rules an AI agent should follow when extending the template. Open the project in [Claude Code](https://claude.com/claude-code) and the doc is auto-loaded into every session.

Highlights:
- Server vs client component rules
- The 9-step preference-adding flow
- Theme preset constraint (colors only)
- Sheet patterns (Customize, Notifications) vs full pages (Mail, Chat)
- Form patterns (RHF + zod, phone validation)
- "Things NOT to do" list

---

## Conventions

- **No emojis** in code or commit messages unless the user asks.
- **No comments** explaining what code does — only WHY, when non-obvious.
- **TypeScript-strict** — `pnpm exec tsc --noEmit` must pass.
- **Path alias** — import from `@/` (mapped to `src/`).
- **Server vs client** — `page.tsx` is server (with `metadata`); interactivity goes into a `_components/*-app.tsx` client container.
- **No back-compat shims** when refactoring data shapes — this is an in-house template, not a library.
- Pre-commit hook (Husky + lint-staged + Biome) auto-formats and lints staged files.

---

## License

MIT — see [LICENSE](./LICENSE).
