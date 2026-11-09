# CLAUDE.md - Heavymath Types

> **Git policy — never auto-commit or auto-push.** Leave your work in the working
> tree. Run `git commit`, `git push`, or `gh pr create` **only when the user
> explicitly asks in that turn**. Approval for an earlier change does not carry
> forward, and finishing a task is not permission to commit it.
> Do not use feature branches; stay on the current branch.

## What this is

Zero-dependency TypeScript types package (`@sudobility/heavymath_types`) for the
Heavymath prediction market platform. Provides shared interfaces, enums, and
small runtime helpers consumed by the indexer, API client, lib, contracts, and
app packages.

- **Stack**: TypeScript 5.9, Vitest 4.1, ESLint 9 (flat), Prettier 3
- **Package manager**: Bun (run `bun install` before any script)
- **Output**: ESM only — `tsc -p tsconfig.esm.json` then `node scripts/fix-esm-imports.mjs` → `dist/`
- **License**: BUSL-1.1
- **Optional peer**: `viem` ^2.0.0 (for structural compat of `Hex`/`Address` in `events.ts`)

## Commands

```bash
bun run build          # Build ESM to dist/ (tsc + fix-esm-imports.mjs)
bun run clean          # Remove dist/
bun run dev            # tsc --watch
bun run typecheck      # tsc --noEmit
bun run test:unit      # Vitest, single run   (NOTE: there is no `test` script)
bun run test:watch     # Vitest watch mode
bun run lint           # ESLint check
bun run lint:fix       # ESLint auto-fix
bun run format         # Prettier write
bun run format:check   # Prettier check
bun run verify         # typecheck + lint + build (run before committing)
bun run prepublishOnly # clean + verify
```

All of the above are verified working (155 tests pass, lint/build clean).

## Layout

```
src/
├── index.ts            # re-exports ./types/index.js
└── types/
    ├── index.ts        # barrel export of everything public
    ├── common.ts       # Optional<T>, ChainPrefixedId, PredictionId, TxLogId + guards/parsers
    ├── entities.ts     # DB entity interfaces (bigint fields) + MarketStatus/ClaimType/WithdrawalType enums
    ├── api.ts          # API/serialized types (string fields) — *Data types
    ├── events.ts       # blockchain event arg interfaces + OracleType const + Hex/Address
    ├── condition.ts    # market condition bytes32 encode/decode + ConditionType/ScoreType/... enums
    ├── sports.ts       # per-sport league/season/country metadata types
    ├── game-status.ts  # GameState + per-sport status codes & classifier fns (10 sports)
    ├── discussion.ts   # discussion/comment entities + SIWE auth types
    ├── datetime.ts     # getCurrentDatetime()
    └── *.test.ts       # co-located Vitest tests
scripts/fix-esm-imports.mjs  # post-build import-specifier rewrite
```

Detailed type inventories live in the source and `src/types/index.ts`; do not
re-list them here (they drift). `plans/IMPROVEMENTS.md` tracks proposed work.

## Key convention: Entity vs Data types

- **Entity types** (`MarketEntity`, …) use `bigint` numeric fields — for indexer/database code.
- **Data types** (`MarketData`, …) use `string` numeric fields — for API responses and clients.

Enums are string literal types backed by runtime `*Values` const arrays plus
`is*` type guards (e.g. `MarketStatusValues`, `isMarketStatus`). Add new types to
the right file, export from `src/types/index.ts`, add a co-located `.test.ts`,
then `bun run verify`.

## Gotchas (non-obvious)

- **No `test` script.** Use `bun run test:unit` / `bun run test:watch`.
- **Two ESLint configs exist; only `eslint.config.js` is active.** ESLint flat
  config resolves `eslint.config.js` before `eslint.config.mjs`, so the `.mjs`
  file is stale/unused — edit `eslint.config.js`. It sets `no-unused-vars` off in
  favor of the TS rule (ignores `^_`), `no-console` warn (allows warn/error),
  and `prefer-const` error. (The old CLAUDE.md described the inactive `.mjs`.)
- **Prettier**: semi, es5 trailing commas, single quotes, 80 cols, 2-space indent.
- **`events.ts` `Hex`/`Address`** are locally defined `0x${string}` template
  types, always available regardless of whether the optional `viem` peer is
  installed; structurally compatible with viem's when present.
- **`fix-esm-imports.mjs` is required after `tsc`.** The build uses
  `moduleResolution: bundler`, so tsc emits extensionless/`.ts` relative imports
  that Node's ESM loader rejects; the script rewrites them to `.js`/`/index.js`.
  Unresolvable specifiers are warned, never fatal. It is idempotent.
- **`src/types/index.ts` imports `discussion` with a `.ts` extension** (not
  `.js` like the others) — intentional; the fixup script maps `.ts`→`.js`.
- Tests import from source, not `dist/`; Vitest only includes `src/**/*.test.ts`.

## CI/CD & publishing

- `.github/workflows/ci-cd.yml` calls the reusable
  `johnqh/workflows/.github/workflows/unified-cicd.yml@main` with
  `publish-target: github` → publishes to **GitHub Packages** (not npmjs).
- `.npmrc` routes the `@sudobility` scope to `https://npm.pkg.github.com`.
- Runs on push/PR to `main` and `develop`.

## Consumers

`heavymath_indexer` (entity types), `heavymath_indexer_client` (re-exports Data
types), `heavymath_lib`, `heavymath_contracts`, `heavymath_ui`, and the apps
(`heavymath_app`, `wcprediction_app`, indirectly via indexer_client).
