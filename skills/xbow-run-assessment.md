---
name: Run an XBOW assessment and collect validated findings
description: Launch an autonomous pentest against a registered asset, monitor it, and pull the validated findings and report.
api: openapi/xbow-openapi-original.json
operations:
- POST /api/v1/assets/{assetId}/assessments
- GET /api/v1/assessments/{assessmentId}
- GET /api/v1/assets/{assetId}/findings
- GET /api/v1/findings/{findingId}
- GET /api/v1/assets/{assetId}/reports
- GET /api/v1/reports/{reportId}/summary
---

# Run an XBOW assessment and collect validated findings

The XBOW spec publishes no operationIds — address operations by METHOD + path.

1. Send every request with `Authorization: Bearer <PAT>` and `X-XBOW-API-Version: 2026-07-01` (400 without the version header).
2. Request an assessment: `POST /api/v1/assets/{assetId}/assessments`. The asset must already exist (`POST /api/v1/organizations/{organizationId}/assets`).
3. Poll `GET /api/v1/assessments/{assessmentId}` until it completes. Assessments can pause (e.g. credential problems) — resume with `POST /api/v1/assessments/{assessmentId}/resume`, or `.../pause` / `.../cancel` to control the run.
4. List results with `GET /api/v1/assets/{assetId}/findings` using cursor pagination (`limit` 1-100 default 20, `after` = previous `nextCursor`; stop when `nextCursor` is null).
5. Fetch detail with `GET /api/v1/findings/{findingId}` — XBOW-owned fields: state, severity, CVSS.
6. Reports: `GET /api/v1/assets/{assetId}/reports`, then `GET /api/v1/reports/{reportId}` (PDF) or `GET /api/v1/reports/{reportId}/summary` (markdown).
7. On 429/500 retry with exponential backoff; errors arrive as `{code, error, message}`. Lightspeed org keys are read-only (GET endpoints only).
