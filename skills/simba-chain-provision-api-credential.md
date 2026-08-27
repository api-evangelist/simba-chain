---
generated: '2026-08-27'
method: generated
name: Provision and rotate an organisation API credential
description: Mint a Secret Key Pair for an organisation, grant it roles, then rotate or revoke it.
api: openapi/simba-chain-member-service-openapi.json
operations: [create_client_credential_organisations__organisation_name__client_credentials__post, get_client_credentials_organisations__organisation_name__client_credentials__get, get_org_scoped_roles_organisations__organisation_name__roles__get, update_client_credential_roles_organisations__organisation_name__client_credentials__client_id__roles__put, refresh_client_credential_secret_organisations__organisation_name__client_credentials__client_id__refresh_put, revoke_client_credential_organisations__organisation_name__client_credentials__client_id__delete]
source: >-
  operationIds verified verbatim in openapi/simba-chain-member-service-openapi.json; credential
  lifecycle documented at https://docs.simbachain.com/documentation/getting-started/obtaining-api-keys
---

# Provision and rotate an organisation API credential

The credential an SDK, CLI or CI job authenticates with. This is the closest thing SIMBA has to an API-key management flow.

## Auth
Bearer token from `skills/simba-chain-authenticate-client-credentials.md`, held by an identity with credential-management permission in the target organisation.

## Steps
1. **List the roles available in the organisation** — `get_org_scoped_roles_organisations__organisation_name__roles__get` (`GET /organisations/{organisation_name}/roles/`). Paged: `page`, `size` (max 100), `order_by`. Pick the least-privileged role that covers the job.
2. **Create the credential** — `create_client_credential_organisations__organisation_name__client_credentials__post` (`POST /organisations/{organisation_name}/client_credentials/`), body `CreateClientCredentialInput`. The response is a `FreshClientCredential` and is **the only time `secret` is returned**. Persist it to your secret store in this step or you will have to rotate.
3. **Attach roles** — `update_client_credential_roles_organisations__organisation_name__client_credentials__client_id__roles__put` (`PUT .../client_credentials/{client_id}/roles/`) to set the full role set, or the `roles/add/` POST and `roles/remove/` DELETE operations to adjust incrementally.
4. **Verify** — `get_client_credentials_organisations__organisation_name__client_credentials__get` (`GET /organisations/{organisation_name}/client_credentials/`) and check `revoked`, `expire_at`, `last_used` and the attached roles.

## Rotation and revocation (reversibility)
- **Rotate**: `refresh_client_credential_secret_organisations__organisation_name__client_credentials__client_id__refresh_put` (`PUT .../client_credentials/{client_id}/refresh`) issues a new secret for the same `client_id`. The old secret stops working — deploy the new one first.
- **Revoke**: `revoke_client_credential_organisations__organisation_name__client_credentials__client_id__delete` (`DELETE .../client_credentials/{client_id}`).
- SIMBA publishes **no grace period or retention window** for either operation. Treat revocation as immediate and irreversible. See the `reversibility` block in `conventions/simba-chain-conventions.yml`.

## Errors
- `422` `HTTPValidationError` on a malformed body. `403` when the caller lacks the permission — SIMBA's platform-side permission denials use codes `2001`–`2010` (`errors/simba-chain-problem-types.yml`).

## Do not
- Do not create an impersonation credential (`create_impersonate_user_client_credentials_user_accounts__user_account_id__client_credentials__post`) as a substitute for role assignment; it makes the credential act as a named user and widens blast radius.
