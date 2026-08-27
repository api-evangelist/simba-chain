---
generated: '2026-08-27'
method: generated
name: Get a SIMBA access token with client credentials
description: Exchange a SIMBA Secret Key Pair for a bearer token and confirm which identity it represents.
api: openapi/simba-chain-member-service-validator-openapi.json
operations: [get_access_token_oauth_token_post, whoami_user_accounts_whoami__get, openid_configuration_oauth__well_known_openid_configuration_get, get_jwks_oauth__well_known_jwks_get]
source: >-
  operationIds verified verbatim in openapi/simba-chain-member-service-validator-openapi.json and
  openapi/simba-chain-member-service-openapi.json; flow documented at
  https://docs.simbachain.com/documentation/getting-started/obtaining-api-keys and
  /developer-resources/environmental-variables
---

# Get a SIMBA access token with client credentials

Every other SIMBA skill starts here. `client_credentials` is the only auth flow SIMBA's own SDK docs say is supported for non-interactive callers.

## Before you start
- A **Secret Key Pair** — `client_id` (the "API Secret Key") and `client_secret` (the "API Client Secret") — created in SIMBA Build under organisation → application → **Secrets**, or user Profile → **Secrets** → **New Secret**. The secret is displayed exactly once; if it is lost, rotate rather than recover.
- The Blocks instance base URL. SIMBA Blocks is deployed per customer as well as hosted: set `SIMBA_API_BASE_URL` (and `SIMBA_AUTH_BASE_URL`) to your instance. The SIMBA-hosted instance is `https://blocks.simbachain.com`.

## Steps
1. **(Optional) Read the discovery document** — `openid_configuration_oauth__well_known_openid_configuration_get` (`GET /oauth/.well-known/openid-configuration` on `/api/member-service-validator`). Confirms `token_endpoint`, `grant_types_supported` and `jwks_uri` for the instance you are pointed at. Cached copy: `well-known/simba-chain-openid-configuration.json`.
2. **Request the token** — `get_access_token_oauth_token_post` (`POST /oauth/token`). Body schema `ClientCredentialParams`: `grant_type` = `client_credentials`, `client_id` (UUID), `client_secret`. Response schema `TokenResponse`.
3. **(Optional) Verify the token signature** — `get_jwks_oauth__well_known_jwks_get` (`GET /oauth/.well-known/jwks`). One RSA key, `RS256`. Cached copy: `well-known/simba-chain-jwks.json`.
4. **Confirm the identity** — `whoami_user_accounts_whoami__get` (`GET /user_accounts/whoami/` on `/api/member-service`) with `Authorization: Bearer <token>`. Returns the `UserAccount`, including `simba_id` — the join key you need for the permissions lookup.

## Auth notes
- `token_endpoint_auth_methods_supported` is `client_secret_post`, `client_secret_basic`, `none`. Use `client_secret_post` to match the `ClientCredentialParams` body schema.
- The issuer is `simba://authservice`, **not** an https URL. If your OIDC client library validates `iss` strictly, this will fail — see `authentication/simba-chain-authentication.yml`.
- SIMBA can also be fronted by Keycloak (`SIMBA_AUTH_PROVIDER=KC` with `SIMBA_AUTH_REALM`); in that case the token endpoint is Keycloak's, not SIMBA's.

## Errors
- `401` returns `{"detail":"missing-auth-header"}` when the `Authorization` header is absent. Full taxonomy: `errors/simba-chain-problem-types.yml`.
- Validation failures return `422` with `HTTPValidationError` (`detail[].loc/msg/type`).

## Do not
- Do not retry a failed write after a timeout expecting idempotency — SIMBA has no `Idempotency-Key`. See `conventions/simba-chain-conventions.yml`.
