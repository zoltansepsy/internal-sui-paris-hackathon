# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

24-hour hackathon dApp on SUI. Turborepo monorepo with pnpm workspaces.

## Commands

| Command | What it does |
|---------|-------------|
| `pnpm dev` | Next.js dev server at localhost:3000 |
| `pnpm build` | Build all packages (Turbo-cached) |
| `pnpm type-check` | Type-check all packages |
| `pnpm lint` | Lint all packages |
| `pnpm format` | Format all files with Prettier |
| `pnpm check:patterns` | Validate architecture patterns (blocks commits) |
| `pnpm --filter @hack/blockchain build:contracts` | Build Move contracts (`sui move build`) |
| `pnpm --filter @hack/blockchain test:contracts` | Test Move contracts (`sui move test`) |

Deploy contracts: `cd packages/blockchain/contracts && sui client publish --gas-budget 100000000`. After publish, add Package ID and object IDs to `packages/blockchain/sdk/networkConfig.ts`.

## Architecture

**Monorepo layout:**
- `apps/dapp/` — Next.js (App Router, React 19) frontend with `@mysten/dapp-kit` wallet integration
- `packages/blockchain/contracts/` — SUI Move smart contracts (package name: `hack`)
- `packages/blockchain/sdk/` — TypeScript SDK: network config, wallet helpers, contract interaction
- `packages/types/` — Shared TypeScript types (`@hack/types`)
- `packages/ui/` — Shared UI components: Button, Card (`@hack/ui`)
- `packages/config/typescript/` — Shared tsconfig (strict mode, `noUncheckedIndexedAccess`)
- `tools/scripts/check-patterns.ts` — Architecture pattern enforcer

**dApp wiring:** `apps/dapp/app/providers.tsx` wraps the app with `SuiClientProvider` + `WalletProvider` + `QueryClientProvider`. Network config comes from `@hack/blockchain`. Default network is testnet.

**Path aliases in dApp:** `@/*` maps to `apps/dapp/`, `@hack/types`, `@hack/ui`, `@hack/blockchain` map to their packages.

## Architecture Rules (enforced by pre-commit)

These rules are checked by `pnpm check:patterns` and block commits on violation:

1. **Services export functions**, not classes — put them in `apps/dapp/lib/services/<feature>/`
2. **API routes** (`apps/dapp/app/api/`) must call service functions from `@/lib/services/`, never direct DB/Supabase calls
3. **Shared types** go in `@hack/types` or `apps/dapp/lib/shared/types/`, not inline in service files
4. **No `any` types** — use `unknown` or proper types. Allow exceptions with `// ALLOWED` comment on the same or next line
5. **Error handling** — async service functions must use try/catch or throw custom errors
6. **Function size** — service functions must be ≤100 lines
7. **Feature structure** — prefer `lib/services/<feature>/index.ts`

## Pre-commit Hooks

Husky runs three checks in order: **lint-staged** (ESLint + Prettier on staged files) → **type-check** → **check:patterns**. Do not use `--no-verify`.

## Move Contracts

Contracts live in `packages/blockchain/contracts/sources/`. Follow the patterns in `docs/MOVE-PATTERNS.md`:
- **Capability pattern** — access control via capability objects (`AdminCap` etc.)
- **Hot Potato pattern** — one-time consumption, struct has no `key`/`store`/`drop`
- **Witness / One-Time Witness** — one-time module init

Naming: modules are one-per-feature, structs are PascalCase (suffixed `Cap`, `Potato`), functions are `snake_case`. Prefer `public entry` for tx entry points.

Always run `build:contracts` and `test:contracts` before committing Move changes.

## Key Documentation

- `docs/AGENT-HANDOFF.md` — primary onboarding doc for agents/devs (stack, structure, rules)
- `docs/MOVE-PATTERNS.md` — required Move patterns with code snippets
- `docs/PRE-COMMIT-CHECKS.md` — details on pre-commit hooks
- `docs/HACKATHON-SUGGESTIONS.md` — tips for the 24h sprint
