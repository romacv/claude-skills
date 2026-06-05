---
name: backend
description: Use when writing, editing, or reviewing backend code — Cloudflare Workers, Hono, TypeScript, Drizzle/D1, KV, Wrangler, or non-edge Python/Django/Rust services.
---

# Platform: Backend

## Overview
An opinionated house style for backend code. Load alongside the public stack — cloudflare plugin (`wrangler`, `workers-best-practices`, `durable-objects`, …). These conventions are the judgment layer the public tooling does not carry.

## Tools
- Use Cloudflare skills and wrangler for Cloudflare Workers work.
- Use wrangler CLI for deploy, dev, tail, and secret management.

## Code
- Primary runtime: Cloudflare Workers.
- HTTP framework: Hono.
- Language: TypeScript for Workers and Hono services. Python/Django for non-edge services. Rust where performance or WASM compilation is required.
- ORM: Drizzle for D1 / relational access. No raw SQL strings at call sites — use Drizzle query builder.
- One module/service per file. No bundling unrelated handlers into a single file.
- Secrets and tokens injected via Wrangler secrets or CI secret bindings — never hardcoded in source.

## Rules
- All Workers must handle the `fetch` event and return a `Response`.
- Never store user data in Workers KV without a defined TTL or explicit eviction strategy.
- D1 schema changes require a migration file — no ad-hoc `CREATE TABLE` or `ALTER TABLE` in handler code.
- Environment variables accessed only through the `Env` type binding — no `process.env` in Workers.
- Log structured JSON only — no unstructured string logging.
