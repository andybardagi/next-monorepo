# Architecture

## Overview

pnpm workspace + Turborepo monorepo. Two workspace groups (`pnpm-workspace.yaml`):

```
apps/
  web                      → Next.js 16 app (App Router, React 19)
  storybook                → @workspace/storybook — Storybook docs/preview for @workspace/ui (Vite builder)
packages/
  ui                       → @workspace/ui — shared component library (shadcn-style, Base UI, Tailwind v4)
  typescript-config        → @workspace/typescript-config — shared tsconfig presets
```

## Dependency direction

```
apps/web ────────► @workspace/ui ──► @base-ui/react, cva, tailwind-merge
apps/storybook ──►      │
    │                   │
    └───────────────────┴──► @workspace/typescript-config (dev-only)
```

- Apps depend on packages. **Packages never import from apps.**
- `@workspace/ui` has no dependency on other workspace packages except the tsconfig preset.
- Cross-package imports go through the `exports` map of `@workspace/ui` (`./components/*`, `./hooks/*`, `./lib/*`, `./globals.css`, `./postcss.config`) — never deep-import via relative paths across packages.

## apps/web layout

```
app/          → routes (App Router: layout.tsx, page.tsx)
components/   → app-specific components (e.g. theme-provider.tsx)
hooks/        → app-specific hooks
lib/          → app-specific utilities
```

Path alias `@/*` resolves to the app root (see `apps/web/tsconfig.json`).

## apps/storybook layout

```
.storybook/   → main.ts (framework, Vite/Tailwind config), preview.ts (globals.css import, dark-mode decorator)
stories/      → *.stories.tsx, one per @workspace/ui component, imported via @workspace/ui/components/*
```

Every component in `packages/ui/src/components/` must have a matching story here (see `docs/conventions.md`).

## packages/ui layout

```
src/components/   → shared components (one file per component, e.g. button.tsx)
src/hooks/        → shared hooks
src/lib/          → shared utilities (utils.ts exports cn())
src/styles/       → globals.css (Tailwind v4 theme, design tokens)
```

## Component placement rule

- Reusable across apps or design-system material → `packages/ui/src/components/`.
- Specific to one app (wiring, providers, page sections) → `apps/web/components/`.
- When in doubt, start app-local; promote to `@workspace/ui` only when a second consumer appears.

## Styling & theming

- Tailwind CSS v4 via `@tailwindcss/postcss`. No `tailwind.config.js` — theme lives in `packages/ui/src/styles/globals.css` (imported once in `apps/web/app/layout.tsx` as `@workspace/ui/globals.css`).
- Design tokens are CSS variables (`--primary`, `--muted`, `--radius-*`, …); components reference them through Tailwind utilities (`bg-primary`, etc.).
- Dark mode via `next-themes` (`ThemeProvider` in `apps/web/components/theme-provider.tsx`, `suppressHydrationWarning` on `<html>`).

## Build orchestration

Turborepo (`turbo.json`): `build` and `typecheck` fan out per package with `^build`/`^typecheck` dependencies; `lint`/`format` are root-level tasks (`//#lint`, `//#format`, aggregated as `//#quality`).

`storybook` (persistent, uncached) and `build-storybook` (cacheable, outputs `storybook-static/**`) are separate task keys scoped to `apps/storybook` — deliberately **not** part of the `build` task, so `pnpm build`/the `pnpm lint && pnpm typecheck && pnpm build` verification bar does not cover Storybook's own build. Only `typecheck`/`lint` reach `apps/storybook`; run `pnpm build-storybook` explicitly to verify the Storybook build itself.

## Next.js caveat

This is Next.js 16 — newer than most training data. Before writing routing, caching, or data-fetching code, read the relevant guide in `node_modules/next/dist/docs/` and heed deprecation notices.
