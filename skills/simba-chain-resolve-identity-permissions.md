---
generated: '2026-08-27'
method: generated
name: Resolve what an identity is actually allowed to do
description: Look up the effective role and permission set behind a SIMBA identity before acting on its behalf.
api: openapi/simba-chain-member-service-openapi.json
operations: [whoami_user_accounts_whoami__get, get_identity_permissions_by_simba_id_identity__simba_id__permissions__get, get_permissions_permissions__get, get_roles_roles__get, get_org_scoped_roles_organisations__organisation_name__roles__get]
source: >-
  operationIds verified verbatim in openapi/simba-chain-member-service-openapi.json and
  openapi/simba-chain-member-service-validator-openapi.json; authorization model documented at
  https://docs.simbachain.com/documentation/simba-build/managing-an-organization/user-roles-and-permissions
---

# Resolve what an identity is actually allowed to do

SIMBA does not express API authorization in OAuth scopes — the token's scope set is only `openid`, `email`, `profile`. Everything real lives in roles and permissions, enforced by OPA middleware. So an agent cannot infer its own capability from the token; it has to ask.

## Auth
Bearer token from `skills/simba-chain-authenticate-client-credentials.md`.

## Steps
1. **Find your `simba_id`** — `whoami_user_accounts_whoami__get` (`GET /user_accounts/whoami/`). The `simba_id` claim is also present in the token itself (`claims_supported` includes `simba_id` and `identity_urn`).
2. **Resolve effective permissions** — `get_identity_permissions_by_simba_id_identity__simba_id__permissions__get` (`GET /identity/{simba_id}/permissions/`). Returns `IdentityPermissionItem` entries.
3. **Read a permission's meaning** — `get_permissions_permissions__get` (`GET /permissions/`, paged) or `get_permission_permissions__permission_id__get`. A `Permission` is `service` + `resource` + `action` with an `effect` (allow/deny), optional `resource_attributes`, and the `urls` it governs. Match `service`/`resource`/`action` against the operation you intend to call.
4. **Understand the grant path** — `get_roles_roles__get` (`GET /roles/`) for global roles, `get_org_scoped_roles_organisations__organisation_name__roles__get` for organisation-scoped ones. Roles support `inherited_roles`, so the effective set is the transitive closure, not the direct assignment.

## Second authorization layer
The dynamic contract API adds its own per-application permission check on top of this one. Denials there come back as codes `2001`–`2010` (`NO_PERMISSIONS`, `NO_USER`, `METHOD_ACCESS_DENIED`, `WRITE_BLOCKCHAIN_DENIED`, …). A caller that passes the control-plane check can still be refused at the contract-method level. See `errors/simba-chain-problem-types.yml`.

## Pagination
All collection reads use `page` / `size` (max 100) and return `{items,total,page,size,pages}` — see `conventions/simba-chain-conventions.yml`.
