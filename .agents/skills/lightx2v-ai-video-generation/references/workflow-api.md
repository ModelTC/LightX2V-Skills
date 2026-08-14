# Saved Workflow API

Use this reference when an existing saved workflow should be discovered, inspected, run, monitored, or cancelled through the LightX2V API. For creating or editing workflow graphs, use `workflow-nodes.md` instead.

## Scope And Ownership

- API keys can run workflows owned by the authenticated account.
- A public/community workflow must first be copied or recreated in that account.
- A missing workflow and a workflow owned by another user both return `404`.
- Run-time values belong in `inputs`; do not edit stored Input node values for a one-off run.
- Use `save_as_default=true` only when the user explicitly wants to update workflow defaults.

## Recommended Sequence

1. List owned workflows.
2. Read the selected workflow if graph details are needed.
3. Discover required inputs for the intended run scope.
4. Start the run with values keyed by Input node id.
5. Poll status or listen to SSE events.
6. Read final outputs after a terminal status.

CLI:

```bash
lightx2v workflow list --json
lightx2v workflow inputs WORKFLOW_ID --json
lightx2v workflow run WORKFLOW_ID --inputs @inputs.json --json
lightx2v workflow stream WORKFLOW_ID RUN_ID --json
lightx2v workflow outputs WORKFLOW_ID RUN_ID --json
```

HTTP equivalents:

```text
GET  /api/v1/workflow/list
GET  /api/v1/workflow/{workflow_id}
GET  /api/v1/workflow/{workflow_id}/inputs
POST /api/v1/workflow/{workflow_id}/runs
GET  /api/v1/workflow/{workflow_id}/runs
GET  /api/v1/workflow/{workflow_id}/runs/{run_id}
GET  /api/v1/workflow/{workflow_id}/runs/{run_id}/stream
GET  /api/v1/workflow/{workflow_id}/runs/{run_id}/outputs
POST /api/v1/workflow/{workflow_id}/runs/{run_id}/cancel
POST /api/v1/workflow/{workflow_id}/runs/{run_id}/cancel-node
```

Authenticate direct requests with `Authorization: Bearer apikey_...`. Never print or commit the key.

## Discover Inputs

Input requirements depend on the selected run scope:

```bash
lightx2v workflow inputs WORKFLOW_ID \
  --mode downstream \
  --node-id TARGET_NODE_ID \
  --json
```

Modes:

| Mode | Meaning |
| --- | --- |
| `full` | All processor nodes |
| `single` | Only the listed processor nodes |
| `downstream` | The selected node and its descendants |
| `upstream` | The selected nodes and processor ancestors |

By default, required upstream dependencies are included. Add `--no-include-upstream` only when intentionally excluding them.

Each returned input includes `node_id`, `type`, `required`, `default_preview`, and accepted `value_formats`. Use the exact `node_id` as the key in the run body:

```json
{
  "inputs": {
    "prompt_input": "A clean studio product reveal",
    "reference_image": {
      "type": "url",
      "data": "https://example.com/product.jpg"
    }
  }
}
```

With the CLI, local media can be bound without manually creating base64:

```bash
lightx2v workflow run WORKFLOW_ID \
  --inputs '{"prompt_input":"A clean studio product reveal"}' \
  --input-file reference_image=./product.jpg \
  --poll \
  --json
```

## Partial Runs

Use the same `mode`, `node_ids`, and `include_upstream` values for both input discovery and run submission. Otherwise the discovered required inputs may not match the actual run.

```bash
lightx2v workflow run WORKFLOW_ID \
  --mode downstream \
  --node-id TARGET_NODE_ID \
  --inputs @inputs.json \
  --poll
```

## Monitor And Read Outputs

Choose one monitoring strategy:

```bash
# Poll a run
lightx2v workflow status WORKFLOW_ID RUN_ID --json

# Stream run_status and run_outputs events
lightx2v workflow stream WORKFLOW_ID RUN_ID --json

# List active runs after reconnecting
lightx2v workflow runs WORKFLOW_ID --status running --json
```

Terminal statuses are `succeeded`, `failed`, and `cancelled`. Calling `outputs` before then is valid and returns `pending: true` with an empty output list.

Final output items identify the source `node_id`, `port_id`, and `kind`. Media outputs may contain `url` or `urls`; text outputs contain `text`. Preserve signed URLs only as long as needed and do not expose them in shared logs.

## Cancellation

Cancel a whole run:

```bash
lightx2v workflow cancel WORKFLOW_ID RUN_ID --json
```

Cancel one node and dependent downstream work:

```bash
lightx2v workflow cancel-node WORKFLOW_ID RUN_ID NODE_ID --json
```

The single-node response contains `cancelled_node_ids`, which can include dependent nodes in addition to the requested node.

## Error Handling

- `400`: invalid mode/scope, missing required input, or a node cannot be cancelled.
- `401` / `403`: missing, expired, or unauthorized API key.
- `404`: workflow or run is unavailable to this account.
- `409`: another process claimed the same run.
- `429`: account concurrency limit reached; retry with backoff.
- `503` on outputs: output storage is temporarily unavailable; retry without starting a duplicate run.

Keep `workflow_id` and `run_id` after a timeout. Reconnect with `runs`, `status`, or `stream` instead of resubmitting blindly.
