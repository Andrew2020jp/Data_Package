# 3. Postgres + Kysely data stack on Deno

Date: 2026-06-09

## Status

Accepted — **except the runtime**: the Deno / `npm:`-compat parts are superseded by
[ADR-0004](0004-switch-runtime-to-node.md) (runtime is now Node + pnpm). Postgres, Kysely,
the migrator, kysely-codegen, and raw-SQL-as-escape-hatch all still stand.

## Context

The merge model (ADR-0002) is relational — joins, `COUNT(DISTINCT identity)`, ranking by
derived vote counts. We need a persistence layer that fits that, stays legible to AFK agents,
and respects the project's supply-chain stance (Deno chosen to avoid npm `postinstall` risk;
deps gated by `docs/approved-deps.md`).

## Decision

- **Postgres**, run locally via **docker-compose**. Chosen over SQLite/Deno-KV: the owner is
  fluent in Postgres and fine running a compose stack, and it's a more realistic substrate for
  a public portfolio repo. Over Deno-KV because KV would force hand-rolled aggregation against
  a relational model.
- **Runtime stays Deno**, consuming the data libraries via `npm:` specifiers. Deno does **not**
  execute `postinstall`/lifecycle scripts even for npm packages, so the supply-chain protection
  that motivated choosing Deno still applies to these deps — consuming npm under Deno is safer
  than the same packages under Node. (We deliberately did *not* switch to Node + pnpm: that
  would reopen the locked Deno/TS decision and discard the no-postinstall protection.)
- **Driver: `npm:pg`** behind Kysely's official `PostgresDialect`. Kysely's first-class support
  targets node-postgres; pairing it with JSR-native deno-postgres needs a lightly-maintained
  community dialect — a fragility risk in an AFK build — so we take the documented `pg` path.
- **Query layer: Kysely** — type-safe builder, no ORM. The data layer is a handful of queries;
  raw SQL is used only as a `` sql`...` `` escape hatch for Postgres-specific DDL/predicates.
- **Migrations: Kysely's built-in migrator** — TS migration files, no separate migration tool.
- **Types: `kysely-codegen`** introspects the migrated database and generates the Kysely
  `Database` type, so query types are derived from the real schema and can't drift. It is a
  **dev-time tool only** — run via `node`/`npx` (not Deno) to sidestep any Deno-compat rough
  edges in the CLI; it emits a `.ts` file and the app never depends on it at runtime.

## Consequences

- Tests need a real Postgres, not an in-process file (see Testing Decisions). Designed around
  an ephemeral DB rather than a throwaway file.
- Three deps enter `approved-deps`: `pg` + `kysely` (runtime), `kysely-codegen` (dev-only).
- Schema authoring is all-TypeScript: DDL via the Kysely migrator, types via codegen, queries
  via Kysely — raw SQL is the exception, not the default.
- The codegen workflow has an ordering rule: **migrate, then regenerate types.** A `deno task`
  should wrap this so the AFK agent doesn't run them out of order.
