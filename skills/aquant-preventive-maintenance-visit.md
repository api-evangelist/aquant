---
name: Run a preventive-maintenance visit and file the summary
description: >-
  Use the Aquant MCP Server to generate a preventive-maintenance checklist for an asset,
  walk it as a structured form, then produce the service summary report.
api: openapi/aquant-mcp-server-openapi.json
mcp: https://mcp.aquant.ai/mcp
operations: [health_check, preventive_maintenance_check_list, summary_report]
generated: '2026-07-31'
method: generated
source: openapi/aquant-mcp-server-openapi.json + mcp/aquant-mcp-tools.json
---

# Run a preventive-maintenance visit and file the summary

## Steps

1. **Check the service is up.**
   Call `health_check` (no arguments). It is the only `GET` operation on the API.

2. **Generate the checklist.**
   Call `preventive_maintenance_check_list` with `asset_id` and/or `model` — both are
   optional, but calling with neither gives you an ungrounded checklist, so pass at least one.

   The result is a versioned form: `{schema_version, agent_id, steps[], status, message}`.
   `steps[]` is polymorphic — walk it by the `type` discriminator:
   - **`NumberStep`** — `{id, title, required, label, placeholder, key, unit, value}`.
     Always render `unit`; a reading without its unit is a data-quality defect.
   - **`RadioStep`** — `{id, title, required, options[], value, blocking_rule}`.
     Honour `blocking_rule` as `{if_value, block}`: when the chosen value matches `if_value`,
     you must **block** the named downstream step rather than continuing past it.
   - **`SelectStep`** — `{id, title, required, key, options[], value}`.
   - **`FormStep`** — `{id, title, required, fields[], values}`, where each `FormField` is
     `{key, type, disabled_if, placeholder}`. Respect `disabled_if`.

   Carry `schema_version` through to whatever you persist — it is how Aquant versions the
   checklist shape, and it is not the API version.

3. **Write the summary.**
   Call `summary_report` with `chat_history`. The result is
   `{summary_version, site, agents_used[], status, message}` where `site` is `{name, address}`
   and each `agents_used` entry is `{id, name, status, result, highlights}` — the AI agents
   that contributed, with their individual outcomes.

4. **Attribute the summary.**
   Surface `agents_used[].highlights` alongside the summary. Presenting the report without the
   contributing agents strips the provenance Aquant deliberately returns.

## Do not

- Do not flatten `steps[]` into plain text. The `blocking_rule` and `disabled_if` semantics are
  the safety logic of the checklist; discarding them changes what the technician is told to do.
- Do not mark a `required: true` step complete without a value.
