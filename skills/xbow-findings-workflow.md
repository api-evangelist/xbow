---
name: Sync XBOW findings into an external ticketing workflow
description: Track finding remediation in your own system using workflow metadata and verify fixes with a retest.
api: openapi/xbow-openapi-original.json
operations:
- GET /api/v1/assets/{assetId}/findings
- PATCH /api/v1/findings/{findingId}
- POST /api/v1/findings/{findingId}/verify-fix
- GET /api/v1/findings/{findingId}
---

# Sync XBOW findings into an external ticketing workflow

The XBOW spec publishes no operationIds — address operations by METHOD + path. Requires API version 2026-07-01+ (workflow metadata fields).

1. Pull open findings with `GET /api/v1/assets/{assetId}/findings` (cursor pagination: `limit`/`after`, stop when `nextCursor` is null).
2. For each finding you ticket externally, record your state on the finding itself: `PATCH /api/v1/findings/{findingId}` with `externalWorkflowState` (your tracking state) and `externalTicketReference` (may only be present alongside an externalWorkflowState). Omitted fields are unchanged; send null to clear. These fields are customer-controlled and independent of XBOW-owned state/severity/CVSS.
3. Subscribe to `finding.changed` webhooks (see the webhook skill) to catch XBOW-side state transitions like open -> fixed without polling.
4. After deploying a fix, trigger `POST /api/v1/findings/{findingId}/verify-fix` so XBOW re-tests and validates the remediation.
5. Confirm via `GET /api/v1/findings/{findingId}` — workflow metadata also appears in finding.changed payloads for subscriptions pinned to 2026-07-01.
