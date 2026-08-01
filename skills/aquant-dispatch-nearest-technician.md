---
name: Locate an asset and dispatch the nearest qualified technician
description: >-
  Use the Aquant MCP Server to resolve an asset's location, pull technician proficiency data,
  and rank the available technicians by proximity to that asset.
api: openapi/aquant-mcp-server-openapi.json
mcp: https://mcp.aquant.ai/mcp
operations: [asset_location, agent_data, technician_proximity]
generated: '2026-07-31'
method: generated
source: openapi/aquant-mcp-server-openapi.json + mcp/aquant-mcp-tools.json
---

# Locate an asset and dispatch the nearest qualified technician

In Aquant's vocabulary a field technician is an **agent** (`event_agent_name`) — not an AI
agent. Read `agent_data` as "technician data".

## Before you start

Same transport and envelope rules as every Aquant MCP flow — see
`conventions/aquant-conventions.yml`. None of these three tools takes a `tenant_id`.

## Steps

1. **Resolve where the asset is.**
   Call `asset_location` with `asset_id`. The result carries `asset_location` as
   `{asset_id, location}`. If `status` is not a success value, stop — you cannot compute
   proximity without a location.

2. **Pull the technicians and their proficiency.**
   Call `agent_data` with `observation_name` — the symptom or observation the job is about.
   The result is `agents[]` of `{event_agent_name, proficiency_score, location}`, scoped to
   technicians relevant to that observation.

3. **Rank by distance.**
   Call `technician_proximity` with:
   - `asset_location` — the location string from step 1
   - `technicians` — an array of `{event_agent_name, location}` built from step 2
   The result is `technician_proximity` entries of
   `{event_agent_name, location, proximity_km}`.

4. **Choose, but show your work.**
   Rank on `proximity_km` from step 3 *and* `proficiency_score` from step 2. Present both to
   the dispatcher. Aquant publishes no combined ranking, so do not present a single opaque
   "best" without the two inputs.

## Do not

- Do not treat `technician_proximity` as a dispatch action. It is a read: nothing in the
  Aquant public API assigns, schedules, or notifies a technician.
- Do not pass technicians into step 3 that did not come back from step 2 — you would be
  ranking people the model has no proficiency signal for.
