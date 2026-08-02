# MOLOCO

Moloco is a machine-learning advertising company operating three developer-facing platforms: **Moloco Ads**
(a performance demand-side platform for app marketers), **Moloco Commerce Media (MCM)** (a retail-media
platform for marketplaces and retailers), and Moloco Streaming Monetization.

- Website: https://www.moloco.com/
- Moloco Ads developer hub: https://developer.moloco.cloud/
- Moloco Commerce Media developer hub: https://mcm-docs.moloco.com/
- Trust center: https://trust.moloco.com/

## APIs

| API | Base URL | Operations | Spec |
|---|---|---|---|
| Moloco Ads Campaign Management API (v1.10) | `https://api.moloco.cloud` | 96 | `openapi/moloco-ads-campaign-management-openapi.yml` |
| MOLOCO Cloud Auth API | `https://api.moloco.cloud` | 28 | `openapi/moloco-cloud-auth-openapi.yml` |
| MCM Management API | `https://<platform>-mgmt.mcm-api.moloco.com` | 74 | `openapi/moloco-commerce-media-management-openapi.yml` |
| MCM Decision API | `https://sandbox-dcsn.mcm-api.moloco.com` | 4 | `openapi/moloco-commerce-media-decision-openapi.yml` |
| MCM Event API | `https://sandbox-evt.mcm-api.moloco.com` | 1 | `openapi/moloco-commerce-media-event-openapi.yml` |
| MCM Webhooks (OpenAPI 3.1 webhooks) | — | 2 events | `openapi/moloco-commerce-media-webhooks-openapi.yml` |

Specs were harvested verbatim from the ReadMe api-registry backing both developer portals; the originals are
kept under `openapi/_original/`.

## Agent surfaces

- **MCP** — three remote servers. `https://mcm-docs.moloco.com/mcp` answers `tools/list` anonymously (7 tools,
  captured in `mcp/moloco-mcm-docs-mcp-tools.json`). `https://mcp.moloco.cloud/mcp` is OAuth-protected
  (scope `cloudapi.read`). `https://developer.moloco.cloud/mcp` requires authorization.
- **OAuth** — `api.moloco.cloud` publishes RFC 8414 authorization-server metadata with PKCE S256 and
  client-id-metadata-document support; `mcp.moloco.cloud` publishes RFC 9728 protected-resource metadata.
- **Agent Skills** — Moloco publishes a Claude Code skill marketplace at
  https://github.com/moloco-mcm/agent-skills; `skills/moloco-ad-preview-template.md` is that skill verbatim.
- **A2A** — no agent card. `/.well-known/agent-card.json` and `/.well-known/agent.json` return 404 on every
  Moloco host probed.

## Artifacts

`openapi/` `overlays/` `authentication/` `scopes/` `well-known/` `mcp/` `skills/` `llms/` `conventions/`
`errors/` `lifecycle/` `changelog/` `rate-limits/` `conformance/` `security/` `sandbox/` `components/`
`cli/` `packages/` `data-model/` `asyncapi/` `agentic-access/`
