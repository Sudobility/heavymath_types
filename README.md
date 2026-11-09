# @sudobility/heavymath_types

Zero-dependency TypeScript types for the Heavymath prediction market platform.

Pure type definitions and small runtime helpers (enum value arrays, type
guards, condition encode/decode, sports/game-status classifiers) shared across
the Heavymath packages. No runtime dependencies; `viem` is an optional peer
(only for structural compatibility of the `Hex`/`Address` types in `events.ts`).
Published to **GitHub Packages** under the `@sudobility` scope. ESM only.

## Installation

```bash
bun add @sudobility/heavymath_types
```

The `@sudobility` scope resolves to GitHub Packages (see `.npmrc`), so consumers
need a GitHub token with `read:packages` configured in their own `~/.npmrc`.

## Usage

```typescript
import type {
  MarketEntity, PredictionEntity, ClaimEntity,
  MarketData, PredictionData, DealerNftData,
  MarketStatus, ClaimType, WithdrawalType,
} from '@sudobility/heavymath_types';

import { OracleType } from '@sudobility/heavymath_types';
```

## Types

### Entity Types (`entities.ts`)

Database-level types using `bigint` fields:

- `MarketEntity`, `PredictionEntity`, `ClaimEntity`
- `OracleEntity`, `DealerNftEntity`, `DealerPermissionEntity`
- `WalletFavoriteEntity`, `FeeWithdrawalEntity`, `OracleRequestEntity`
- `MarketStateHistoryEntity`

**Enums:** `MarketStatus` (Active/Cancelled/Resolved/Abandoned), `ClaimType` (winnings/refund), `WithdrawalType` (dealer/system)

### API Types (`api.ts`)

Serialized types for API responses (strings instead of bigints):

- `MarketData`, `PredictionData`, `DealerNftData`, `DealerPermissionData`
- `FeeWithdrawalData`, `OracleRequestData`, `MarketStateHistoryData`
- `MarketStatsData`, `HealthData`, `SSEStatsData`
- `WalletFavoriteData`, `CreateFavoriteRequest`, `PaginationMeta`

### Event Types (`events.ts`)

Blockchain event argument interfaces for PredictionMarket, DealerNFT, and OracleResolver contracts. Includes `OracleType` constant (`Manual: 0`, `PriceFeed: 1`, `CustomData: 2`).

### Common Types (`common.ts`)

- `Optional<T>`, `ChainPrefixedId`, `PredictionId`, `TxLogId`

### Key Distinction

- **Entity types** use `bigint` -- for indexer/database code
- **Data types** use `string` -- for API responses and client code

## Development

Requires [Bun](https://bun.sh). Install dependencies with `bun install` first.

```bash
bun run build        # Build ESM to dist/ (tsc + fix-esm-imports.mjs)
bun run test:unit    # Run Vitest once (bun run test:watch for watch mode)
bun run typecheck    # TypeScript validation (--noEmit)
bun run lint         # ESLint check (bun run lint:fix to auto-fix)
bun run format       # Prettier (bun run format:check to verify)
bun run verify       # typecheck + lint + build (run before committing)
```

The build emits ESM `.js` + `.d.ts` to `dist/`; `scripts/fix-esm-imports.mjs`
rewrites relative import specifiers to Node-resolvable paths after `tsc`.

## Related Packages

- `@sudobility/heavymath_contracts` -- smart contracts and TypeScript SDK
- `@sudobility/heavymath_indexer_client` -- indexer API client
- `@sudobility/heavymath_lib` -- business logic hooks
- `heavymath_app` -- frontend web application

## License

BUSL-1.1
