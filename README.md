# MOLOCO

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
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
