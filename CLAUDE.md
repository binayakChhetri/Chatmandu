# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Chatmandu is an AI-powered SaaS platform for small online sellers. It pulls customer inquiries from WhatsApp, Instagram, and Facebook Messenger into a single inbox and drafts replies using the seller's own products, prices, and policies.

## Structure

pnpm workspace monorepo with two packages, wired together via `pnpm-workspace.yaml`:

- `client/` — Vite + React 19 + TypeScript frontend, styled with Tailwind CSS 3.
- `server/` — NestJS 12 (on Express) backend, tested with Vitest (not Jest).

One root `pnpm-lock.yaml` covers both packages — always run `pnpm install` from the repo root, never inside `client/` or `server/` directly.

## Commands

Run from the repo root unless noted.

```bash
pnpm install              # install deps for both packages (root-only — do not run inside client/ or server/)

pnpm dev                  # run client + server dev servers together
pnpm dev:client           # client only (vite)
pnpm dev:server           # server only (nest start --watch)

pnpm build                # build both packages (pnpm -r build)
pnpm lint                 # lint both packages (pnpm -r lint)
pnpm format                # prettier --write . across the whole repo
pnpm format:check          # prettier --check .

pnpm --filter server start        # nest start (no watch)
pnpm --filter server start:debug  # nest start --debug --watch
pnpm --filter server test         # vitest run (unit tests)
pnpm --filter server test:watch   # vitest, watch mode
pnpm --filter server test:cov     # vitest run --coverage
pnpm --filter server test:e2e     # vitest run --config ./vitest.config.e2e.ts
pnpm --filter server test <path>  # run a single spec, e.g. pnpm --filter server test src/app.controller.spec.ts

pnpm --filter client preview      # preview a production build
```

Linting uses **oxlint**, not ESLint, in both packages (`server` runs it type-aware: `oxlint --type-aware src/ test/`). Formatting is Prettier (`singleQuote: true, trailingComma: "all"`), configured at the repo root and applied across both packages.

## Architecture notes

- Both packages declare `"type": "module"` — ESM throughout, no CommonJS.
- `server` uses Vitest instead of Nest's default Jest setup: unit specs matched by `**/*.spec.ts` (`vitest.config.ts`), e2e specs by `**/*.e2e-spec.ts` (`vitest.config.e2e.ts`, separate config/command). Both configs load `vite-tsconfig-paths` so TS path aliases (e.g. from `nest g library`) resolve automatically — define new path aliases in `server/tsconfig.json`, not in the vitest configs.
- `server/tsconfig.json` sets `module`/`moduleResolution` to `nodenext` and `rootDir` to `.` (not `./src`) — build output mirrors the full package structure under `dist/`.
- `client` has no test setup — there is no `test` script in `client/package.json`.
- Currently both packages are close to framework-scaffold defaults (`AppController`/`AppService` on the server, the default Vite template on the client) — no established feature-module conventions yet to follow.
