---
name: Batch-verify robot rollouts
description: Start a multi-video / multi-attempt verification batch and poll it to completion.
api: openapi/instance-verifier-openapi.json
operations: [v1_batch_start_v1_batch_post, v1_batch_status_v1_batch__batch_id__get]
generated: '2026-07-19'
method: generated
---

# Batch-verify robot rollouts

Use this flow when you have many rollout videos, or one video holding several sequential
attempts of the same task. Base URL: `https://demo.instancelabs.ai` (unauthenticated demo).

## Steps

1. **Start the batch.** `POST /v1/batch` (operationId `v1_batch_start_v1_batch_post`) as
   `multipart/form-data`:
   - `task` (string, required) — the instruction all rollouts are judged against.
   - `mode` (string, default `single`) — use `multi` for one video containing several
     sequential attempts.
   - `files` (array of binary, optional) — the rollout videos.
   - `frames` (integer, default 8) — frames sampled per rollout.
   The 200 response is a `BatchStart`: `{ batch_id, total, status }`.
2. **Poll for results.** `GET /v1/batch/{batch_id}` (operationId
   `v1_batch_status_v1_batch__batch_id__get`) using the `batch_id` from step 1. Poll on an
   interval until `status` reports the batch is complete, then read the per-rollout verdicts.
3. **Interpret verdicts.** Each result mirrors `VerifyOut`: `verdict` is `"success"`,
   `"failure"`, or `null` (unparseable). A set `error` string is a verification-level failure.

## Rules

- Bad input returns HTTP `422` (`HTTPValidationError`). See `errors/instance-problem-types.yml`.
- Submit-then-poll: `batch_id` from `/v1/batch` is the handle for `/v1/batch/{batch_id}`.
  No webhooks/callbacks are documented — poll. See `conventions/instance-conventions.yml`.
