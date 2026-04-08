# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

BIR1 is a TypeScript ESM library for querying the Polish GUS REGON API (BIR 1.1 and BIR 1.2). It provides a SOAP client for searching entities by NIP/REGON/KRS and retrieving detailed reports. Supports PKD 2025 classification via BIR12 report types. Pure ESM, Node ≥18.

## Commands

- **Build**: `npm run build` (cleans `dist/`, runs `tsc`)
- **Dev**: `npm run dev` (tsc watch mode)
- **Test**: `npm test` (uses `tap`, coverage disabled via `.taprc`)
- **Format**: `npm run pretty` (prettier on `src/`)

No linter configured. No single-test runner — tap runs all tests in `test/`.

## Architecture

The library follows a SOAP request pipeline: **Template → HTTP Client → XML Extract → Optional Normalize**.

- **`src/bir.ts`** — Main `Bir` class. Session-based: auto-logs-in on first API call, stores session ID. Methods: `login()`, `search()`, `report()`, `summary()`, `value()`, `logout()`.
- **`src/templates/`** — SOAP XML envelope builders for each GUS API action.
- **`src/client.ts`** — HTTP fetch wrapper targeting GUS production/test SOAP endpoints.
- **`src/extract.ts`** — Unwraps SOAP response XML, parses inner XML to JSON via `fast-xml-parser`.
- **`src/normalize.ts`** — Exported separately (`bir1/normalize`). Three modes: raw, legacy (v2 compat), modern (strips prefixes, camelCase, nullifies empties).
- **`src/error.ts`** — `BirError` with static `assert()` that checks GUS error codes in parsed responses.
- **`src/types.ts`** — String literal union types for API option values (report names, diagnostic keys).

## Conventions

- ESM-only (`"type": "module"`, module: `node16`)
- TypeScript strict mode, target `es2022`
- No semicolons, single quotes, trailing commas (prettier config)
- Two package exports: `.` (main) and `./normalize`
- Tests in `test/` are integration tests hitting the GUS test API (require network)
- `test/legacy.ts` compares v4 output against `bir1-legacy` (v2) for backward compatibility
