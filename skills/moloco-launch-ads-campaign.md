---
name: moloco-launch-ads-campaign
description: Launch a Moloco Ads (Cloud DSP) user-acquisition campaign end to end — mint an access token, create the ad account, product, tracking link, creatives and creative group, then create the campaign and its ad group.
api: openapi/moloco-ads-campaign-management-openapi.yml
base_url: https://api.moloco.cloud
operations:
  - DspApi_CreateToken
  - DspApi_CreateAdAccount
  - DspApi_CreateProduct
  - DspApi_CreateTrackingLink
  - DspApi_ValidateTrackingLink
  - DspApi_CreateAssetUploadSession
  - DspApi_CreateCreative
  - DspApi_CreateCreativeGroup
  - DspApi_CreateCampaign
  - DspApi_CreateAdGroup
  - DspApi_ReadCampaign
generated: '2026-07-31'
method: generated
source: openapi/moloco-ads-campaign-management-openapi.yml + https://developer.moloco.cloud/docs/campaign-management-api
---

# Launch a Moloco Ads campaign

Operating instructions for creating a Moloco Ads (Cloud DSP) campaign through the Campaign Management API.
Every operationId below exists verbatim in `openapi/moloco-ads-campaign-management-openapi.yml`.

## Before you start

- You need a Moloco Ads account provisioned by a Moloco representative, and an API key created in the portal
  under **User settings → API Access**. One non-expiring key per workplace.
- TLS 1.2 or later is required.
- There is no test mode and no sandbox host for Moloco Ads. **Everything you create here is live.** Create
  campaigns in a paused state first if you are experimenting.

## 1. Get an access token

`DspApi_CreateToken` — `POST /cm/v1/auth/tokens` with `{"api_key": "$API_KEY"}`.

The response carries `token`. Send it as `Authorization: Bearer $TOKEN` on every subsequent call. **Tokens
expire after 16 hours** — mint a new one rather than retrying a 401 in a loop.

Pin the contract version on every request:

```
Moloco-Cloud-Api-Version: v1.10
```

Omitting the header binds you to whatever is current, which will silently change at the next release.

## 2. Create the entities, in order

The entity graph must be built bottom-up (see `data-model/moloco-data-model.yml`):

1. `DspApi_CreateAdAccount` — the advertiser.
2. `DspApi_CreateProduct` — the app or website being advertised. Requires `ad_account_id`.
3. `DspApi_CreateTrackingLink` — requires `ad_account_id` and `product_id`. Validate it first with
   `DspApi_ValidateTrackingLink` (`POST /cm/v1/tracking-links/validate`); an invalid link surfaces in
   `validation_errors` rather than as a hard failure at campaign time.
4. `DspApi_CreateAssetUploadSession` — returns a URL to upload the actual creative asset content. Upload the
   file to that URL **before** creating the creative (v1.7+ flow).
5. `DspApi_CreateCreative` — one and only one of the creative sources may be specified.
6. `DspApi_CreateCreativeGroup` — groups `creative_ids`; may carry its own `tracking_link_id`.
7. `DspApi_CreateCampaign` — requires `ad_account_id`, `product_id` and `tracking_link_id`.
8. `DspApi_CreateAdGroup` — requires `campaign_id` and references `creative_group_ids`.

Confirm the result with `DspApi_ReadCampaign`, or `DspApi_QueryCampaignOverviews` to pull the campaign plus
every related entity in one call.

## 3. Rules that will bite you

- **No idempotency key on these writes.** The Campaign Management API declares no `Idempotency-Key` header.
  A timed-out POST may have succeeded — always `DspApi_ListCampaigns` (or the matching list operation,
  scoped by `ad_account_id`) before retrying a create, or you will duplicate entities.
- **Only never-activated campaigns can be deleted** (`DspApi_DeleteCampaign`). Pause instead.
- **Update semantics are narrow.** `DspApi_UpdateCreative` only accepts `title`, `custom_key_values` and
  `enabling_state` (plus `RICH_CUSTOM_HTML` as an exception).
- **Rate limits.** 300 requests / 5 minutes per ad account across all endpoints. See
  `rate-limits/moloco-rate-limits.yml`.
- **Errors are gRPC-shaped**, not RFC 9457: `{code, message, details[]}`. A 403 carries an
  `api.adcloud.common.APIErrorInfo` detail with an `error_log_id` — quote it to support. See
  `errors/moloco-problem-types.yml`.

## 4. Entity quotas

`DspApi_ReadEntityCount` returns the current entity counts and the limits under each ancestor type. Check it
before a bulk build rather than discovering the ceiling mid-run.
