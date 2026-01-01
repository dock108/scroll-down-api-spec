# AGENTS.md — Scroll Down API Spec

> This file provides context for AI agents (Codex, Cursor, Copilot) working on this codebase.

## Quick Context

**What is this?** OpenAPI specification defining the contract between Scroll Down backend and clients.

**Tech Stack:** OpenAPI 3.0, YAML

**Key Files:**
- `spec/openapi.yml` — The API specification
- `examples/` — Example request/response payloads
- `docs/` — Contract documentation

## Core Principle

**The spec is the source of truth.** Implementation follows spec, not vice versa.

## Consumers

This spec is consumed by:
- `scroll-down-app` (iOS) — Generates Swift models
- `scroll-down-sports-ui` (Web) — Generates TypeScript types
- `sports-data-admin` (Backend) — Implements this spec

## Coding Standards

See `.cursorrules` for complete standards. Key points:

1. **Consistent Naming** — `camelCase` for properties, `PascalCase` for schemas
2. **Reusable Components** — Define schemas in `components/schemas`
3. **Include Examples** — Every response type needs examples
4. **Document Everything** — All fields need descriptions

## Breaking Change Policy

Breaking changes affect multiple repos. Before making breaking changes:

1. Document in `docs/CHANGELOG.md`
2. Coordinate with consumer repos
3. Prefer additive changes (new optional fields) over modifications

## Do NOT

- Make breaking changes without documentation
- Remove or rename existing fields without coordination
- Add required fields to existing schemas (breaks clients)

## Validation

```bash
# Validate spec
npx @redocly/cli lint spec/openapi.yml
```

