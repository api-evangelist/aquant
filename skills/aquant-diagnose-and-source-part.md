---
name: Diagnose an asset and source the right replacement part
description: >-
  Use the Aquant MCP Server to confirm a reported symptom, predict the next most likely
  symptom for an asset, then look up, detail, and source the replacement part.
api: openapi/aquant-mcp-server-openapi.json
mcp: https://mcp.aquant.ai/mcp
operations: [observation_existence, next_symptom, part_catalog_lookup, part_info, part_sources]
generated: '2026-07-31'
method: generated
source: openapi/aquant-mcp-server-openapi.json + mcp/aquant-mcp-tools.json
---

# Diagnose an asset and source the right replacement part

The Aquant MCP Server exposes this flow as five tools. The same five exist as REST
`POST` operations with identical names on `https://mcp.aquant.ai`.

## Before you start

- **Transport.** If you are calling MCP: `POST https://mcp.aquant.ai/mcp` with
  `Accept: application/json, text/event-stream`. Call `initialize` first, capture the
  `mcp-session-id` response header, and send it on every later request — omitting it returns
  JSON-RPC `-32600 "Bad Request: Missing session ID"`.
- **Tenancy.** `part_catalog_lookup`, `part_info` and `part_sources` all require a
  `tenant_id`. Aquant documents it as `[TEMPORARY]`, so treat it as a value that will change.
- **Response shape.** Every result carries a required `status` string and an optional
  `message`, wrapping the domain payload. Check `status` before reading the payload.
- **Errors.** Validation failures come back as FastAPI `{"detail":[{"loc":…,"msg":…,"type":…}]}`
  (HTTP 422). There is no RFC 9457 problem detail. See `errors/aquant-problem-types.yml`.
- **No idempotency, no pagination.** Every operation is `POST`, including reads. There is no
  `Idempotency-Key` and no cursor — do not assume a retry is safe, and do not expect to page.

## Steps

1. **Confirm the reported symptom is one Aquant knows.**
   Call `observation_existence` with `observation_name` (the symptom as reported).
   Read `exists`. If `false`, do not fabricate a diagnosis — fall back to asking the
   technician to restate the symptom.

2. **Predict what to check next.**
   Call `next_symptom` with `asset_id` and, optionally, `top_n` to bound the list.
   You get back `predictions[]` of `{rank, obs_predict, confidence, how_to_check}`.
   Surface `how_to_check` to the technician verbatim — it is the actionable instruction.

3. **Find candidate parts.**
   Call `part_catalog_lookup` with `tenant_id`, `triage_product_id`, and `question`
   (the technician's natural-language ask). You get `parts[]` of
   `{part_id, part_name, part_description, part_image_link, part_url_link}`.

4. **Get detail on the chosen part.**
   Call `part_info` with `tenant_id` and `catalog_data` — pass through the catalog entry from
   step 3 rather than re-deriving it.

5. **Find where to get it.**
   Call `part_sources` with `tenant_id`, `catalog_data`, `model`, and `question`.
   Read `sources` from the result.

## Do not

- Do not invent a `part_id` that did not come back from `part_catalog_lookup`.
- Do not skip step 1. `observation_existence` is the guard against acting on a symptom the
  model has no grounding for.
- Do not retry a failed `POST` blindly — Aquant publishes no idempotency contract.
