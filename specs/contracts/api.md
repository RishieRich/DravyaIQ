# Contract: API

FastAPI under `ui/backend/api/`. JSON only. Routes parse, call one service, serialize. No business rule in a route.

| Method and path | Purpose | Notes |
| --- | --- | --- |
| `GET /api/health` | liveness and config status | returns `{status, reader: "live"|"unavailable", reader_reason, data: "sample"|"plant"}` and never echoes a key |
| `GET /api/doc-types` | the five document types with field keys and hints | from `data/samples/*.json` field lists |
| `GET /api/samples/{doc_type}` | sample sheet layout and prepared reading | for the demo path |
| `POST /api/captures` | start a capture | body `{doc_type, mode: "live"|"prepared"|"manual"}` plus optional multipart `image`. Live mode calls the reader; on reader failure the capture is still created in `review` with empty fields and `reader_error` set |
| `GET /api/captures/{id}` | capture with fields and last checks | |
| `PATCH /api/captures/{id}/fields` | edit or confirm fields | body `[{key, value?, confirmed?}]`; re-runs checks and returns them with the gate |
| `POST /api/captures/{id}/fix/{check_id}` | apply a check's one-click fix | re-runs checks |
| `POST /api/captures/{id}/commit` | save to the record | idempotent on capture id. 409 with fresh checks if the gate fails. 200 with `{entity, id, version, next: {screen, id}}` on success |
| `GET /api/material` | plant totals and lot rows | |
| `GET /api/shades` and `GET /api/shades/{code}` | shade list and shade memory | |
| `GET /api/dispatches` | dispatch list, newest first, with complaint flag | `?q=` filters dn, customer, shade, batch |
| `GET /api/trace/{dn}` | dispatch tree plus blast radius for the default suspect | `?lot=` picks another suspect |
| `PATCH /api/dispatches/{dn}/complaint` | record a complaint text | versioned like any record |
| `GET /api/certificates/{batch_id}` | certificate with lines, sources and `issuable` | |
| `POST /api/demo/reset` | reload fixtures | only when `APP_ENV=development` |

## Payload limits

- Image upload: JPEG, PNG or WebP, at most 8 MB, longest side downscaled to 2000 px on the server before the reader sees it. Larger files get 413 with a plain message.
- Reader call timeout 60 s. On timeout the capture moves to `review` with `reader_error: "timeout"` and the user can retry, use the prepared reading (sample sheets only) or type the values.

## Errors

`{error: {code, message}}` with a stable code. No stack traces, no provider response bodies, no keys in any error.
