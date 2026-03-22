# Mallorca Holística

Single-repo setup for:
- Product contract (`openapi.yaml`) + mocks (`mocks/`) to unblock frontend (Lovable) and backend in parallel.
- Backend API implementation (you).
- Frontend export/import workflow (Lovable collaborator).

## Repository layout

- `openapi.yaml` — API contract (source of truth for request/response shapes)
- `mocks/` — mock dataset + AI orientation examples used by the frontend until the API is ready
- `apps/backend/` — backend API + DB migrations/seeds (implementation)
- `apps/frontend/` — Lovable export drop (frontend code lives here once exported)
- `packages/shared/` — optional shared types/schemas (later)
- `infra/` — deployment/infra notes/scripts (later)
- `PLAN.md` — phased delivery plan

## Frontend workflow (Lovable)

Lovable may not “respect” a repo structure while *generating* the app, and that’s fine.

Recommended approach:
1) In Lovable, build the frontend using the contract + mocks:
   - Contract: `openapi.yaml`
   - Mocks: `mocks/db.json`, `mocks/orient_examples.json`
2) Export/download the project from Lovable.
3) Place the exported code inside `apps/frontend/` (keep whatever internal structure Lovable generates).
4) Keep a small adapter layer (API client) that can switch between:
   - Mock mode (read from `mocks/`)
   - Real API mode (call backend endpoints from `openapi.yaml`)

This repo structure is mainly for *us* (versioning, handoff, CI/CD). Lovable can still generate code however it wants; we just “dock” the export under `apps/frontend/`.

## Backend workflow

Implement endpoints per `openapi.yaml` and use `mocks/db.json` as a seed/reference dataset.
