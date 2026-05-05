# CLAUDE.md

Guide for Claude when extending this admin template. Read this fully before adding features.

## Stack

- **Next.js 16** (App Router, Turbopack), React 19, TypeScript
- **Tailwind CSS v4** + **shadcn/ui** primitives in `src/components/ui/*`
- **Zustand** for client state (preferences store), **react-hook-form + zod** for forms
- **lucide-react** for icons; **simple-icons** for brand logos (WhatsApp, Telegram, etc.)
- **libphonenumber-js** for phone validation, **date-fns** for dates, **sonner** for toasts
- **Biome** for lint/format. **Husky + lint-staged** on commit.
- Package manager: **pnpm**. Dev server: `pnpm dev` (port 3000).

## Project layout

```
src/
├── app/
│   ├── layout.tsx                    # Root layout — sets data-* attrs from PREFERENCE_DEFAULTS
│   ├── globals.css                   # Tailwind + theme presets + density tokens
│   └── (main)/
│       ├── auth/                     # /auth/* (v1, v2)
│       └── dashboard/
│           ├── layout.tsx            # Sidebar + header (Notifications, Customize, Theme, Account)
│           ├── _components/sidebar/  # Sidebar, account switcher, search, layout-controls, notifications
│           ├── default/, analytics/, crm/, finance/, productivity/, ...
│           ├── components/           # Showcase page — kitchen sink
│           ├── mail/                 # 3-pane mail client
│           └── chat/                 # 4-pane multi-channel CRM
├── components/
│   ├── ui/                           # shadcn primitives (button, dialog, sheet, ...)
│   ├── ui/phone-input.tsx            # Custom: country selector + format-on-type
│   └── simple-icon.tsx               # Renders simple-icons SVG path
├── data/                             # Sample/demo data (notifications, mail, chat, users)
├── navigation/sidebar/sidebar-items.ts  # Sidebar config — add new pages here
├── lib/
│   ├── preferences/
│   │   ├── layout.ts                 # Type+option lists for each preference
│   │   ├── theme.ts                  # THEME_PRESET_OPTIONS (Neutral/Zinc/Blue/...)
│   │   ├── preferences-config.ts     # PREFERENCE_DEFAULTS + persistence map
│   │   ├── layout-utils.ts           # apply* functions (set data-* attrs on <html>)
│   │   ├── theme-utils.ts            # applyThemeMode / applyThemePreset
│   │   └── preferences-storage.ts    # persistPreference (cookie / localStorage)
│   └── utils.ts                      # cn(...)
├── stores/preferences/               # Zustand store + provider for prefs
├── scripts/theme-boot.tsx            # Pre-hydration script — applies prefs from cookie before render
├── styles/presets/*.css              # Theme preset CSS files (zinc, blue, violet, rose, orange)
├── server/server-actions.ts          # `getPreference`, `setValueToCookie`
└── config/app-config.ts              # APP_CONFIG (name, copyright, meta)
```

## Adding a new dashboard page

Pattern is consistent across `mail/`, `chat/`, `components/`. To add `/dashboard/foo`:

1. **Sidebar entry** — `src/navigation/sidebar/sidebar-items.ts`. Pick the right group (Dashboards / Pages / Misc), add `{ title, url: "/dashboard/foo", icon, isNew?: true }`. Import the icon from `lucide-react`.
2. **Sample data** — `src/data/foo.ts` with typed entities. Export the data and any tints/labels maps next to the types.
3. **Components** — `src/app/(main)/dashboard/foo/_components/*.tsx` (one component per file). Mark client components with `"use client"`.
4. **Page** — `src/app/(main)/dashboard/foo/page.tsx`. Server component by default. Sets `metadata`. Renders header + the client app.
5. **Verify** — `curl -s -o /dev/null -w "%{http_code}\n" http://localhost:3000/dashboard/foo` should return 200, plus `pnpm exec tsc --noEmit`.

Server pages don't need `"use client"`. Push `useState` and event handlers down into a single client `*-app.tsx` container under `_components/`.

## Preferences system (Customize panel)

The Customize sheet (`_components/sidebar/layout-controls.tsx`) controls layout + theme via data attributes on `<html>`. To add a new preference:

1. Add type + options to `src/lib/preferences/layout.ts` (e.g. `Density`).
2. Add key to `PreferenceValueMap`, default to `PREFERENCE_DEFAULTS`, and persistence to `PREFERENCE_PERSISTENCE` in `preferences-config.ts`.
3. Add field + setter to the Zustand store (`stores/preferences/preferences-store.ts`).
4. Read DOM attribute in `preferences-provider.tsx` `readDomState()`.
5. Add `applyXxx(value)` in `layout-utils.ts` (sets `data-*` attribute on `<html>`).
6. Read the new key in `scripts/theme-boot.tsx` and apply before hydration (anti-flicker).
7. Add `data-xxx={value}` on the `<html>` element in `app/layout.tsx`.
8. Wire UI in `layout-controls.tsx` (use the `IconTile` helper for grid sections).
9. (CSS) If the preference affects styling, target `html[data-xxx="..."]` in `globals.css`.

Skip steps 6–7 only if the preference is purely client-state and never touches SSR.

## Theme presets

Files live in `src/styles/presets/*.css` and are imported in `globals.css`. **Constraint:** presets must override **only color tokens** (`--primary`, `--primary-foreground`, `--ring`, `--sidebar-primary`, `--sidebar-ring`, `--chart-1`). Do NOT change `--radius`, shadows, fonts, or background — that's reserved for the global theme. Soft pastel colors only.

`THEME_PRESET_OPTIONS` in `lib/preferences/theme.ts` is the source of truth — its `primary.light` / `primary.dark` swatches drive the bubbles in the Customize panel.

To add a new color preset:
1. Create `src/styles/presets/<name>.css` (light + dark blocks).
2. Add `@import` in `globals.css`.
3. Append entry to `THEME_PRESET_OPTIONS` in `theme.ts` with matching swatches.

## Sheet patterns

Three Sheet panels are wired in the dashboard header (`(main)/dashboard/layout.tsx`):

- **Customize** — global preferences (`layout-controls.tsx`)
- **Notifications** — message inbox (`notifications-sheet.tsx`)
- *(extend the same way: trigger button + Sheet content + state in client component)*

Sheet conventions:
- Use `<Sheet><SheetTrigger asChild>…<SheetContent side="right" className="…p-0 sm:max-w-md">…</SheetContent></Sheet>`
- Layout: `SheetHeader` (border-b, padded), scrollable middle (`flex-1 overflow-y-auto`), `SheetFooter` (border-t).
- For multi-panel apps (Mail, Chat) use **dedicated route + 3-pane flex layout inside a bordered card**, not a Sheet.

## Forms

- `react-hook-form` + `zodResolver`. Schema with `z.object({...})`. Use `z.email()`, `z.literal(true)`, `z.enum`.
- Phone validation: `z.string().min(1).refine(v => isValidPhoneNumber(v), { message: "..." })` — pair with `<PhoneInput value={watch('phone')} onChange={(v) => setValue('phone', v, { shouldValidate: true, shouldTouch: true })} />`.
- Show errors as `<p className="text-xs text-destructive">{errors.field?.message}</p>` under the input.
- On submit: `toast.success("...")` and `form.reset(EMPTY_VALUES)`.

## Origin UI patterns

Composed components inspired by [originui.com](https://originui.com) live in `(main)/dashboard/components/_components/origin-ui-section.tsx`. They reuse shadcn primitives but compose richer UX (password strength, tag input, multi-select, OTP, plan toggle, rating, stepper, phone input).

When the user asks for an "Origin UI"-style component, prefer composing from existing `components/ui/*` over installing new packages. Only install when the primitive truly doesn't exist (e.g., `libphonenumber-js`).

## Multi-channel CRM (chat)

`src/app/(main)/dashboard/chat/` — reference for any "messenger" / "support inbox" feature.

- **Channels** are typed enum in `src/data/chat.ts`: `whatsapp | telegram | instagram | messenger | email | livechat | sms`.
- Brand icons via `simple-icons` (`siWhatsapp`, etc.) wrapped in `SimpleIcon`. Email/Live chat/SMS use lucide.
- **Status**: `open | pending | resolved` with color tints in `STATUS_TINT`.
- 4-pane layout: rail filters → conversation list → thread → customer panel (last hidden below `xl`).
- Composer has `Reply / Internal note` tabs; internal notes have an amber background.

If the user adds a new channel, extend `Channel` union, add label/tint to `CHANNEL_LABEL` / `CHANNEL_TINT`, add a case in `ChannelIcon`, optionally add a brand icon.

## Notifications

`src/data/notifications.ts` defines 5 types: `info | success | warning | error | mention`. Each has a tint class in `TYPE_TINT` and a lucide icon in `TYPE_ICON` (defined in `notifications-sheet.tsx`). Avatars override the icon when `actor` is set.

## Conventions

- **No emojis in code or commit messages** unless the user asks. Rare exception: data files (e.g., chat sample messages can include realistic emoji).
- **No comments** explaining what code does. Add a one-line `// ...` only for non-obvious WHY.
- **TypeScript-strict.** Run `pnpm exec tsc --noEmit` before reporting done. Ignore the pre-existing `draggable/_components/dashboard-widget.tsx` error (missing `@dnd-kit/utilities` — not in scope).
- **Don't introduce backwards-compat shims** when refactoring preferences / data shapes.
- **Path aliases**: import from `@/` (mapped to `src/`).
- **Server vs client**: keep `page.tsx` as a server component (`export const metadata`); push interactivity into `_components/<x>-app.tsx` (`"use client"`).

## Verifying changes

```bash
# typecheck (ignore the pre-existing draggable error)
pnpm exec tsc --noEmit

# dev server (already running on :3000 — don't double-start)
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:3000/dashboard/<route>

# lint
pnpm exec biome check src/
```

For UI work, also open the page in a browser and click through. Type-checks don't catch CSS/UX issues.

## Where features live (quick map)

| Feature | Entry |
| --- | --- |
| Customize panel | `src/app/(main)/dashboard/_components/sidebar/layout-controls.tsx` |
| Notifications panel | `src/app/(main)/dashboard/_components/sidebar/notifications-sheet.tsx` |
| Theme presets | `src/styles/presets/*.css` + `src/lib/preferences/theme.ts` |
| Components showcase | `src/app/(main)/dashboard/components/` |
| Mail client | `src/app/(main)/dashboard/mail/` |
| Chat CRM | `src/app/(main)/dashboard/chat/` |
| Phone input | `src/components/ui/phone-input.tsx` |
| Sidebar nav | `src/navigation/sidebar/sidebar-items.ts` |
| App config (name/copyright/meta) | `src/config/app-config.ts` |

## Things NOT to do

- Don't change theme `--radius` / `--shadow-*` per preset. Colors only.
- Don't replace the existing Customize controls (Font, Sidebar Style, Navbar, Collapse Mode) when adding new sections — keep them under **Advanced**.
- Don't put the dev server in `run_in_background` more than once. Reuse the running one on port 3000.
- Don't add a new dependency for what existing primitives already cover (e.g., don't install a multi-select package — compose with `Popover` + `Checkbox`).
- Don't modify `package.json` versions casually.
