---
name: Subscribe to XBOW webhooks with verified signatures
description: Create a webhook subscription, verify Ed25519-signed deliveries, and keep subscriptions on supported API versions.
api: openapi/xbow-openapi-original.json
operations:
- POST /api/v1/organizations/{organizationId}/webhooks
- GET /api/v1/meta/webhooks-signing-keys
- POST /api/v1/webhooks/{webhookId}/ping
- GET /api/v1/webhooks/{webhookId}/deliveries
- PATCH /api/v1/webhooks/{webhookId}
---

# Subscribe to XBOW webhooks with verified signatures

The XBOW spec publishes no operationIds — address operations by METHOD + path.

1. Create the subscription: `POST /api/v1/organizations/{organizationId}/webhooks` with an HTTPS `targetUrl`. Events: `asset.changed`, `assessment.changed`, `finding.changed`, `challenge.changed`, `ping` (`target.changed` is deprecated).
2. On setup XBOW sends two `ping` events — one validly signed, one signed with an invalid key — to prove your verification works. Respond 2xx to valid signatures, 401 to invalid ones.
3. Verify every delivery: fetch the public key from `GET /api/v1/meta/webhooks-signing-keys` (cache it), then check the `X-Signature-Ed25519` hex signature over `X-Signature-Timestamp` + raw body, and reject timestamps outside ~±5 minutes.
4. Delivery is best-effort with NO retries — poll `GET /api/v1/webhooks/{webhookId}/deliveries` to reconcile misses; test manually with `POST /api/v1/webhooks/{webhookId}/ping`.
5. Payloads follow the API version pinned on the subscription. Before a version's EOL, bump with `PATCH /api/v1/webhooks/{webhookId}` `{"apiVersion": "<new>"}` — an apiVersion-only PATCH does not re-validate the endpoint; changing targetUrl re-sends the validation pings.
