# Mocks (frontend-first)

This folder contains mock data to unblock a frontend build (e.g. in Lovable) while the backend is implemented.

- `mocks/db.json`: JSON-server–style dataset (entities + join tables).
- `mocks/orient_examples.json`: example inputs/outputs for the AI “orientation” search.

Notes:
- Visitors/users have **no accounts** in the MVP. Anything “user-auth” is intentionally absent.
- Verification documents in the MVP: `professional_id` and `national_id`.
- Languages included in text fields: `es`, `ca`, `en`, `de`.

