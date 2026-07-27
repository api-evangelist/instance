---
name: Verify a robot rollout
description: Judge whether a single robot-rollout video completed its task, using Instance's verifier.
api: openapi/instance-verifier-openapi.json
operations: [v1_verify_v1_verify_post, v1_verify_frames_v1_verify_frames_post]
generated: '2026-07-19'
method: generated
---

# Verify a robot rollout

Use Instance's Robot Rollout Verifier to get a `success` / `failure` verdict for one
robot-rollout video against a task instruction. Base URL: `https://demo.instancelabs.ai`.
The demo backend is unauthenticated — no API key or token is required.

## Steps

1. **Submit the video.** `POST /v1/verify` (operationId `v1_verify_v1_verify_post`) as
   `multipart/form-data` with:
   - `task` (string, required) — the instruction the robot was supposed to perform.
   - `file` (binary, required) — the rollout video.
   - `frames` (integer, optional, default 8) — frames sampled from the video; the server
     clamps this to 8–32.
2. **Read the verdict.** The 200 response is a `VerifyOut`:
   - `verdict` — `"success"`, `"failure"`, or `null` when the model returned no parseable
     answer. Treat `null` as "no judgement", not as failure.
   - `error` — a non-null string here means a verification-level failure even on HTTP 200.
3. **Frames-only alternative.** If you already have sampled frames instead of a video, call
   `POST /v1/verify_frames` (operationId `v1_verify_frames_v1_verify_frames_post`) with
   `task`, `frames` (array of frame strings) and optional `times` (per-frame timestamps).
4. **Multiple attempts.** A video containing several sequential attempts should NOT go to
   `/v1/verify` — use the batch skill (`POST /v1/batch` with `mode=multi`) instead.

## Rules

- Validation failures return HTTP `422` with a FastAPI `HTTPValidationError` envelope
  (`detail[].loc/msg/type`). See `errors/instance-problem-types.yml`.
- No idempotency key is supported; a retry re-runs verification. See
  `conventions/instance-conventions.yml`.
