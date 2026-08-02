---
name: Upload source code to guide gray-box testing
description: Upload a source archive via the multipart S3 resource flow so XBOW can run gray-box assessments.
api: openapi/xbow-openapi-original.json
operations:
- POST /api/v1/organizations/{organizationId}/resources
- POST /api/v1/resources/{resourceId}/parts
- POST /api/v1/resources/{resourceId}/commit
- GET /api/v1/resources/{resourceId}
- DELETE /api/v1/resources/{resourceId}
---

# Upload source code to guide gray-box testing

The XBOW spec publishes no operationIds — address operations by METHOD + path.

1. Create the resource: `POST /api/v1/organizations/{organizationId}/resources` with `{name, fileName, type: "source"}` — status starts `initiated`.
2. Split the file into parts (>= 5 MiB each except the last, <= 5 GiB) and request presigned URLs: `POST /api/v1/resources/{resourceId}/parts` with the part numbers.
3. `PUT` each part directly to its presigned S3 URL and save the `ETag` response header from each part.
4. Commit: `POST /api/v1/resources/{resourceId}/commit` with `{parts: [{partNumber, eTag}...], sha256}` — the sha256 of the whole file is optional but recommended for integrity verification. Status moves to `processing`.
5. Poll `GET /api/v1/resources/{resourceId}` until status is `ready` (or `failed` — see `statusMessage`). Failed parts can be retried individually instead of restarting the upload.
6. Delete when no longer needed: `DELETE /api/v1/resources/{resourceId}`.
