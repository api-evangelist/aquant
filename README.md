# Aquant

Aquant is an agentic AI platform for organizations that build, sell, and service complex
equipment — industrial equipment, medical devices, manufacturing plants, food equipment,
high-tech and electronics, and printing. It turns a company's service content and its
technicians' tacit expertise into role-specific AI agents delivered over web, mobile, voice,
offline, and API channels.

- Website: https://www.aquant.ai/
- Platform: https://www.aquant.ai/platform
- Trust center: https://security.aquant.ai/
- Support: https://support.aquant.ai/

## Public API surfaces

| API | Base URL | Contract |
|---|---|---|
| Aquant MCP Server | `https://mcp.aquant.ai` | [OpenAPI 3.1.0](openapi/aquant-mcp-server-openapi.json) (11 ops) + live MCP `tools/list` (11 tools) |
| Aquant Conversation Platform (VoiceAI) | `https://voiceai-api.aquant.ai` | [OpenAPI 3.1.0](openapi/aquant-voiceai-api-openapi.json) (health only) + `@aquantinc/acp-web-sdk` |

The MCP server is the substantive surface. It publishes eleven service-intelligence tools —
parts lookup/detail/sourcing, technician data and proximity, asset location, observation and
next-symptom diagnostics, preventive-maintenance checklists, summary reporting, health — twice
over: as MCP tools on `POST /mcp` (JSON-RPC Streamable HTTP, protocol `2025-06-18`, server
`1.28.1`) and as REST `POST` operations described by `/openapi.json`. `tools/list` answers
anonymously; tool invocation is tenant-scoped. See [mcp/aquant-tool-crosswalk.yml](mcp/aquant-tool-crosswalk.yml)
for the one-to-one binding.

The Conversation Platform powers Voice AI and web chat. `POST /acp/token` exchanges an API key
and secret for a one-hour bearer token; `POST /vss/web-chat` streams replies as Server-Sent
Events; voice runs over WebRTC. Those routes are live but absent from the served OpenAPI —
they are documented in Aquant's own browser SDK and recorded in
[overlays/aquant-voiceai-api-overlay.yaml](overlays/aquant-voiceai-api-overlay.yaml).

## Notable gaps

- No A2A agent card on any host (`/.well-known/agent-card.json` and `/.well-known/agent.json`
  are 404 everywhere) — nothing is written to `a2a/`.
- No `securitySchemes` in either OpenAPI; no `/.well-known/oauth-protected-resource` on the MCP
  server, so MCP clients cannot discover how to authenticate tool calls.
- No RFC 9457 problem details — three incompatible error envelopes are in use.
- No idempotency contract, no pagination, no rate-limit signalling, no public status page, and
  no deprecation policy.
- No AsyncAPI and no webhook catalog; the event surface is client-pull SSE only.
- The legacy `aquant.io` domain (including `docs.aquant.io`) returns HTTP 521 — dead, not
  redirected.

## Artifacts

`openapi/` `mcp/` `arazzo/` `skills/` `overlays/` `json-schema/` `examples/` `data-model/`
`errors/` `conventions/` `lifecycle/` `changelog/` `conformance/` `authentication/` `scopes/`
`packages/` `well-known/` `security/` `agentic-access/` `llms/`

Provenance is stamped in the frontmatter of every artifact (`generated`, `method`, `source`).
