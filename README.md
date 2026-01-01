# Scroll Down API Specification

**Version:** 1.0.0

This repository is the **single source of truth** for the API contract used by:
- Scroll Down Web (scroll-down-sports-ui)
- Scroll Down iOS App (future)
- Any future integrations

## Purpose

This spec defines:
- Endpoints (URL paths, methods, query params)
- Payload shapes (request/response models)
- Enums and constants
- Error models
- Versioning policies

**No UI logic lives here.**

---

## Repository Structure

```
scroll-down-api-spec/
├── spec/
│   └── openapi.yml          # Main OpenAPI 3.0 contract
├── examples/
│   ├── game-response.json   # Example game detail response
│   ├── game-list-response.json
│   ├── social-posts-response.json
│   └── pbp-response.json
├── docs/
│   ├── README.md            # This file
│   ├── CHANGELOG.md         # Version history
│   ├── implicit-contracts.md # Discovered assumptions from UI
│   └── spec-vs-implementation.md # Validation report
└── tools/
    └── generator-configs/   # Code generator configs (future)
```

---

## Versioning Policy

We follow **Semantic Versioning (SemVer)**:

| Change Type | Version Bump | Example |
|-------------|--------------|---------|
| **PATCH** | 0.0.x | Typo fix, doc correction, example update |
| **MINOR** | 0.x.0 | Add new optional field, add new endpoint |
| **MAJOR** | x.0.0 | Remove/rename field, change existing behavior |

### Rules

1. **Backends + clients should pin to MAJOR version**
2. Breaking changes require a new MAJOR version
3. New optional fields are backwards-compatible (MINOR)
4. Bug fixes in spec accuracy are PATCH

---

## How to Update the Spec

### Spec-First Workflow

All API changes require a spec update **before** implementation:

1. **Propose** - Open a PR with the spec change
2. **Discuss** - Review with backend + frontend stakeholders
3. **Approve** - Get sign-off from API owners
4. **Merge** - Merge the spec update
5. **Implement** - Update backend to match spec
6. **Update Clients** - Update UI/mobile clients

This prevents silent breakage and ensures all consumers agree on contracts.

### PR Requirements

- [ ] Update `spec/openapi.yml`
- [ ] Add example payloads if new models
- [ ] Update `docs/CHANGELOG.md`
- [ ] Run validation if tooling is set up

---

## Who Approves Changes?

- **Backend Lead** - Confirms implementation feasibility
- **Frontend Lead** - Confirms UI compatibility
- **API Owner** - Final approval on spec changes

---

## Validation

To validate the spec matches reality:

```bash
# Future: automated validation scripts
# For now, manual comparison
```

See `docs/spec-vs-implementation.md` for the current validation report.

---

## Quick Start

1. View the spec: `spec/openapi.yml`
2. See examples: `examples/*.json`
3. Check assumptions: `docs/implicit-contracts.md`
4. Review validation: `docs/spec-vs-implementation.md`

---

## Future Enhancements

- [ ] Auto-generate TypeScript types
- [ ] Auto-generate Swift structs
- [ ] CI validation on PR
- [ ] Mock server from spec
- [ ] SDK generation

---

## Related Repositories

- [sports-data-admin](../sports-data-admin) - Backend API implementation
- [scroll-down-sports-ui](../scroll-down-sports-ui) - Web frontend

