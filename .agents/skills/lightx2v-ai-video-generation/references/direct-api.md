# Direct HTTP API

Use this reference when the user requests raw HTTP, the CLI is unavailable, or exact endpoint behavior matters.

## Source Order

1. Read `https://x2v.light-ai.top/llms.txt` and the relevant Markdown topic.
2. Query `GET /api/v1/model/list`, then `GET /api/v1/model/specs?task=TASK`.
3. Read `GET /openapi.json` for precise fields and response shapes.
4. Quote, submit, and poll only after the live contract is known.

Do not copy mutable model limits into a long-lived script. Resolve model names, media requirements, dimensions, duration, and conditional parameters at runtime.

## Authentication Boundary

Public:

```text
GET /llms.txt
GET /api-docs.md
GET /en/api-docs.md
GET /openapi.json
GET /api/v1/model/list
GET /api/v1/model/specs?task=...
```

Bearer API Key required:

```text
POST /api/v1/billing/quote
POST /api/v1/task/submit
GET  /api/v1/task/query?task_id=...
GET  /api/v1/task/result_url?task_id=...&name=output_image|output_video
GET/POST workflow discovery and run endpoints
```

Pass `Authorization: Bearer apikey_...`. Read the value from a secret store or environment at execution time. Never generate, print, log, commit, or save a user's key in task output.

## Reliable Task Sequence

Set placeholders without echoing the secret:

```bash
API_BASE="${LIGHTX2V_BASE_URL:-https://x2v.light-ai.top}"

curl -sS "$API_BASE/api/v1/model/list"
curl -sS "$API_BASE/api/v1/model/specs?task=t2i"
```

Select `model_cls` from the responses. For media jobs, use the selected spec and OpenAPI `input_meta` schema to quote without uploading or decoding the source file.

```bash
curl -sS -X POST "$API_BASE/api/v1/billing/quote" \
  -H "Authorization: Bearer $LIGHTX2V_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"task":"t2i","model_cls":"MODEL_FROM_SPECS","stage":"single_stage","prompt":"A paper-cut city at sunrise"}'

curl -sS -X POST "$API_BASE/api/v1/task/submit" \
  -H "Authorization: Bearer $LIGHTX2V_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"task":"t2i","model_cls":"MODEL_FROM_SPECS","stage":"single_stage","prompt":"A paper-cut city at sunrise"}'
```

Keep quote and submit model, media, duration, resolution, aspect ratio, and other billable parameters identical. Save the returned `task_id`. Use an idempotency or client request identifier only when the live OpenAPI explicitly documents one.

```bash
curl -sS -H "Authorization: Bearer $LIGHTX2V_API_KEY" \
  "$API_BASE/api/v1/task/query?task_id=TASK_ID"

curl -sS -H "Authorization: Bearer $LIGHTX2V_API_KEY" \
  "$API_BASE/api/v1/task/result_url?task_id=TASK_ID&name=output_image"
```

Poll at bounded intervals until `SUCCEED`, `FAILED`, or `CANCEL`. Use `output_video` for video tasks. Download the signed result promptly; it is not a permanent asset URL.

## Media Inputs

Use the OpenAPI `MediaInput` schema:

```json
{"type":"url","data":"https://example.com/input.png"}
```

or:

```json
{"type":"base64","data":"BASE64_WITHOUT_A_DATA_URL_PREFIX"}
```

Some model fields accept lists, and multi-role S2V accepts a directory map. The selected live model spec decides which field and shape are valid.

## Retry And Recovery

- `429`: wait for `Retry-After` before a bounded retry.
- Transient `5xx` or connection failure before submission: use limited exponential backoff with jitter.
- Timeout after submission may be an unknown outcome. Query the known `task_id`. If no `task_id` was received, use a documented idempotency or reconciliation mechanism when the live OpenAPI provides one; otherwise do not automatically resubmit, and report the unknown outcome for manual verification.
- Validation, parameter, balance, permission, and content-safety failures require correction or user action; do not retry the same request.
- Keep `task_id` after disconnects. A signed result URL can be requested again after success if the previous one expires.

## Workflows

For saved workflows, read `workflow-api.md`. Discover inputs for the same run scope, save `run_id`, and reconnect to status or stream after a timeout instead of starting a duplicate run.
