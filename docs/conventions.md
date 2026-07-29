# Conventions

## Language & modules

- TypeScript everywhere, `"type": "module"` (ESM). No `require`.
- Explicit types at module boundaries; rely on inference inside function bodies.
- Named exports for components and utilities (`export { Button, buttonVariants }`); `export default` only where a framework requires it (Next.js pages/layouts).

## Lint & format

- **oxlint** and **oxfmt** — not ESLint, not Prettier. Do not add ESLint/Prettier configs or disable-comments in their syntax.
- Commands (root, apply everywhere):
  - `pnpm lint` / `pnpm lint:fix`
  - `pnpm format` (check) / `pnpm format:fix`
  - `pnpm typecheck`
  - `pnpm build`
- Fix pipeline in one shot: `pnpm turbo //#quality:fix`.

## Verification bar

A change is not finished until `pnpm lint && pnpm typecheck && pnpm build` passes from the repo root.

**Testing:** no test runner is configured yet. The first feature that needs tests must add Vitest (workspace-level) as part of its scope — flag this to the leader rather than skipping tests silently.

## Components (shadcn / Base UI pattern)

Follow the existing pattern in `packages/ui/src/components/button.tsx`:

- Build on **Base UI** primitives (`@base-ui/react/*`), not raw HTML elements, when a primitive exists.
- Variants with **cva** (`class-variance-authority`); export both the component and its `*Variants` object.
- Merge classes with `cn()` from `@workspace/ui/lib/utils` — never string concatenation.
- Set `data-slot="<component-name>"` on the rendered element.
- Plain `function` declarations for components (no `React.FC`, no `forwardRef` unless a ref is actually consumed).
- Spread `...props` last; accept `className` and merge it via `cn()`.
- **Every component added to `packages/ui/src/components/` must have a matching story** in `apps/storybook/stories/<component>.stories.tsx` — the `reviewer` checks for this pairing before approving.

## Files & naming

- One component per file, **kebab-case filenames** (`theme-provider.tsx`, `button.tsx`); PascalCase component names.
- Hooks in `hooks/`, prefixed `use`. Utilities in `lib/`.
- Feature plan files live in `docs/PRDs/`, named `NNN-<plan-slug>.md` (zero-padded sequence, kebab-case slug — e.g. `001-db-infrastructure.md`), and are referenced from `feature_list.json` via the `plan` field.
- Imports: external packages first, then workspace packages (`@workspace/ui/...`), then app-local (`@/...`) — matching existing files.

## Styling

- Tailwind utilities only; no CSS modules, no inline `style` unless dynamic.
- Use design-token utilities (`bg-primary`, `text-muted-foreground`, …), never hard-coded colors.
- Both light and dark mode must work — use token pairs and `dark:` variants as the existing components do.

## Workspace hygiene

- Add dependencies to the package that uses them (`pnpm add <pkg> --filter web`), not to the root, unless they're repo-wide tooling.
- Internal deps use `workspace:*` protocol.
- Respect the dependency direction in `docs/architecture.md`.
