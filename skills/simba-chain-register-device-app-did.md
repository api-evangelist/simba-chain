---
generated: '2026-08-27'
method: generated
name: Register a device app and issue its DID credential
description: Enrol a device application, capture its registrant and device DIDs and public keys, and check the issued Verifiable Credential.
api: openapi/simba-chain-member-service-openapi.json
operations: [create_device_app_organisations__organisation_name__device_apps__post, update_device_app_roles_organisations__organisation_name__device_apps__device_app_name__roles__put, create_device_app_registration_organisations__organisation_name__device_apps__app_name__registrations__post, get_device_app_registrations_organisations__organisation_name__device_apps__app_name__registrations__get, get_device_app_registration_organisations__organisation_name__device_apps__app_name__registrations__registration_id__get, update_device_app_registration_organisations__organisation_name__device_apps__app_name__registrations__registration_id__put, get_vc_secure_session_oauth_vc_secure_session_post]
source: >-
  operationIds verified verbatim in openapi/simba-chain-member-service-openapi.json and
  openapi/simba-chain-member-service-validator-openapi.json; SIMBA Ensure DID/VC concepts documented
  at https://docs.simbachain.com/documentation/simba-ensure/decentralized-identifiers-dids and
  /simba-ensure/verifiable-credentials-vcs
---

# Register a device app and issue its DID credential

This is where SIMBA's decentralized-identity surface is actually reachable from the published contract: device-app registration mints DIDs and a Verifiable Credential for a registrant and a device.

## Auth
Bearer token from `skills/simba-chain-authenticate-client-credentials.md` with device-app administration permission in the organisation.

## Steps
1. **Create the device app** — `create_device_app_organisations__organisation_name__device_apps__post` (`POST /organisations/{organisation_name}/device-apps/`), body `CreateDeviceAppInput`.
2. **Grant it roles** — `update_device_app_roles_organisations__organisation_name__device_apps__device_app_name__roles__put`, or the `roles/` POST/DELETE pair to add and remove.
3. **Register a device** — `create_device_app_registration_organisations__organisation_name__device_apps__app_name__registrations__post` (`POST .../device-apps/{app_name}/registrations/`), body `CreateDeviceAppRegistration`. Supply the registrant and device public keys.
4. **Read the result** — `get_device_app_registration_organisations__organisation_name__device_apps__app_name__registrations__registration_id__get`. The `DeviceAppRegistration` carries `registrant_did` / `registrant_did_id`, `device_did` / `device_did_id`, `registrant_public_key_multicodec` and `device_public_key_multicodec` (multicodec-encoded, the DID Core representation), the hex key forms, `vc` and `vc_id` for the issued credential, plus `status` and `errors`.
5. **Amend if needed** — `update_device_app_registration_organisations__organisation_name__device_apps__app_name__registrations__registration_id__put`. Check `status` (`DeviceAppRegistrationStatus`) and `errors` before treating the registration as complete.
6. **Use the DID for a session** — `get_vc_secure_session_oauth_vc_secure_session_post` (`POST /oauth/vc-secure-session` on `/api/member-service-validator`), body `GetVcSecureParams { registrant_did_id }`. Returns a `VPChallenge` — a Verifiable Presentation challenge. An optional `dpop` request header supplies an RFC 9449 proof of possession; SIMBA publishes `@simbachain/simba-chain-dpop` and `@simbachain/simba-chain-vc` to build these client-side.

## Errors
`422` `HTTPValidationError` on a malformed body. Registration failures surface on the record's own `errors` field rather than as an HTTP error, so read the record — do not assume `201` means enrolled.

## Do not
- Do not invent DID method strings or credential shapes; read them back off the `DeviceAppRegistration` record. SIMBA does not publish the DID method or the credential schema in either OpenAPI.
