---
generated: '2026-08-27'
method: generated
name: Invite a user into a SIMBA organisation
description: Send, track, resend, revoke and accept an organisation invitation, with roles attached.
api: openapi/simba-chain-member-service-openapi.json
operations: [get_org_scoped_roles_organisations__organisation_name__roles__get, create_organisation_invite_organisations__organisation_name__invites__post, get_organisation_invites_organisations__organisation_name__invites__get, resend_user_invite_organisations__organisation_name__invites__invite_id__resend_patch, revoke_organisation_invite_organisations__organisation_name__invites__invite_id__delete, accept_new_user_invite_invites__invite_id__accept_new_put, accept_invite_for_existing_user_invites__invite_id__accept_existing_put, get_organisation_users_organisations__organisation_name__users__get]
source: >-
  operationIds verified verbatim in openapi/simba-chain-member-service-openapi.json; organisation
  membership model documented at
  https://docs.simbachain.com/documentation/simba-build/managing-an-organization
---

# Invite a user into a SIMBA organisation

## Auth
Bearer token from `skills/simba-chain-authenticate-client-credentials.md` with invite permission in the organisation.

## Steps
1. **Choose the roles** — `get_org_scoped_roles_organisations__organisation_name__roles__get` (`GET /organisations/{organisation_name}/roles/`).
2. **Create the invite** — `create_organisation_invite_organisations__organisation_name__invites__post` (`POST /organisations/{organisation_name}/invites/`), body `CreateInviteInput`. The resulting `Invite` carries `invitee_email`, `status`, `expires_at`, `roles` and, for nested tenancy, `sub_invites`.
3. **Track it** — `get_organisation_invites_organisations__organisation_name__invites__get` (`GET /organisations/{organisation_name}/invites/`), paged. Read `status` and `expires_at` off each record rather than assuming a policy window: SIMBA publishes no invite-expiry policy, only the per-record field.
4. **Nudge or withdraw** — `resend_user_invite_organisations__organisation_name__invites__invite_id__resend_patch` (`PATCH .../invites/{invite_id}/resend`), or `revoke_organisation_invite_organisations__organisation_name__invites__invite_id__delete` (`DELETE .../invites/{invite_id}`).
5. **Acceptance (invitee side)** — `accept_invite_for_existing_user_invites__invite_id__accept_existing_put` (`PUT /invites/{invite_id}/accept-existing`) when the person already has a SIMBA account, `accept_new_user_invite_invites__invite_id__accept_new_put` (`PUT /invites/{invite_id}/accept-new`) when they do not. `reject_organisation_invite_invites__invite_id__reject_put` declines.
6. **Confirm membership** — `get_organisation_users_organisations__organisation_name__users__get` (`GET /organisations/{organisation_name}/users/`).

## Bulk
For many users at once, `create_bulk_users_import_request_bulk_users_import_requests__post` starts an asynchronous job; poll `get_bulk_users_import_request_bulk_users_import_requests__bulk_users_import_request_id__get` and read `status` and `errors`.

## Reversibility
Revoking a pending invite and removing an accepted member (`remove_user_from_organisation_organisations__organisation_name__users__user_id__delete`) are both available, with **no stated window**. Deleting a user account outright has no restore operation — treat it as final.

## Errors
`422` `HTTPValidationError` on a malformed body; `403` on insufficient permission. See `errors/simba-chain-problem-types.yml`.
