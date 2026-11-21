---
name: adding-an-endpoint
description: Add or change an HTTP endpoint in the Infinity URL API, from route to OpenAPI, including authorization, validation, error codes and tests. Use for any change to the public contract.
---

# Add an endpoint

## Objective

Add an endpoint that respects the contract in `docs/10-api.md`, is covered by a
test before it is written, and appears in the generated OpenAPI.

## Procedure

### 1. Check the contract first

- Read `docs/10-api.md` §7 (endpoints) and §9 (abilities).
- If the endpoint is not listed, stop: it is a product decision, not an
  implementation detail.
- If it is listed, copy its method, path, ability, request shape and response
  shape. Do not invent a variant.

### 2. Route

- Add the route under the `/v1` prefix, in the right route file.
- Attach `auth:sanctum` **and** `abilities:<scope>`. A route without an ability is
  a bug and fails `RouteAbilitiesTest`.
- Never add a redirect-path route here: it lives in its own file, without session
  middleware.

### 3. Request and validation

- One FormRequest per write endpoint.
- Authorize explicitly: the policy decides, not the controller.
- Validate types strictly, and return `422` with the `errors` object.
- Never accept a `*_hash` column, a `workspace_id`, or a `public_id` that the
  client should not choose.

### 4. Action

- Business logic in an action class, not in the controller.
- Scope every query by the current workspace. A resource from another workspace
  answers `404`, never `403`.
- Use the domain errors from §4 of the contract: `slug_taken`, `slug_reserved`,
  `destination_blocked`, `domain_has_links`, `scope_missing`…
- Throw a dedicated exception mapped to the right `problem+json` response, rather
  than returning a hand-built array.

### 5. Test

Write the test **before** the implementation. Minimum:

- the happy path, with the exact response shape;
- one validation failure (`422`, with `errors`);
- one authorization case: wrong ability, `403`, code `scope_missing`;
- one cross-workspace case: `404`;
- one domain rule from `link-behaviour` if the endpoint touches it.

### 6. Contract

- Regenerate the spec: `php artisan scramble:export`.
- Regenerate the web types: `npm run api:types`.
- If the CI reports a spec diff without the types, regenerate both.

### 7. Definition of done

- [ ] Test written first and green.
- [ ] Ability declared; no route without one.
- [ ] Errors use a stable `code`.
- [ ] Scoped to the workspace.
- [ ] Spec and types regenerated.
- [ ] No privacy invariant touched (see the `privacy` rule).
