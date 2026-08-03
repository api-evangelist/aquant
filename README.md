# Aquant

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
