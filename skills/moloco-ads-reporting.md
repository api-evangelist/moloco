---
name: moloco-ads-reporting
description: Pull Moloco Ads campaign performance — choose between the synchronous analytics endpoints and the asynchronous Report API, poll a report to completion, and export impression/click/conversion logs.
api: openapi/moloco-ads-campaign-management-openapi.yml
base_url: https://api.moloco.cloud
operations:
  - DspApi_CreateToken
  - DspApi_QueryAnalyticsOverview
  - DspApi_QueryAnalyticsDetail
  - DspApi_QueryAnalyticsSKAdNetwork
  - DspApi_ListReportFilters
  - DspApi_ListReportActionEvents
  - DspApi_CreateReport
  - DspApi_ReadReportStatus
  - DspApi_ReadReport
  - DspApi_ListReports
  - DspApi_CreateReportExport
  - DspApi_CreateLog
  - DspApi_ReadLogStatus
  - DspApi_ReadCohortSummary
generated: '2026-07-31'
method: generated
source: openapi/moloco-ads-campaign-management-openapi.yml + https://developer.moloco.cloud/docs/report-api
---

# Pull Moloco Ads performance data

Two reporting surfaces with very different quotas. Pick deliberately — the wrong one exhausts the daily
budget in minutes.

## Choose the surface

| Need | Operation | Quota | Output | Row limit |
|---|---|---|---|---|
| Dashboard / ad-hoc, filtered | `DspApi_QueryAnalyticsDetail` | 60 / hour | JSON | 10,000 |
| Headline numbers | `DspApi_QueryAnalyticsOverview` | 60 / minute | JSON | — |
| SKAdNetwork | `DspApi_QueryAnalyticsSKAdNetwork` | 60 / hour | JSON | — |
| Scheduled bulk extract | `DspApi_CreateReport` | 30/10/5 per day by date range | CSV or JSON | 4,000,000 |

The Report API quota tiers on the requested range: 0–1 day → 30/day, 2–7 days → 10/day, 8–31 days → 5/day.
Full table in `rate-limits/moloco-rate-limits.yml`.

## Synchronous path

1. `DspApi_CreateToken` for a bearer token (16-hour lifetime).
2. `DspApi_ListReportFilters` and `DspApi_ListReportActionEvents` to discover the filters and action events
   available for the ad account — do not hard-code them.
3. `DspApi_QueryAnalyticsDetail` (`POST /cm/v1/analytics-detail`) with your dimensions, metrics and filters.

## Asynchronous path

1. `DspApi_CreateReport` (`POST /cm/v1/reports`) with `ad_account_id`, `date_range.start`, `date_range.end`
   and `dimensions[]`. Available dimensions: `DATE`, `APP_OR_SITE`, `CAMPAIGN`, `AD_GROUP`,
   `CREATIVE_GROUP`, `CREATIVE`, `EXCHANGE`, `SUB_PUBLISHER`, `TRAFFIC`, `SKAN`. Optional metrics:
   `VIDEO_PLAY_PROGRESS`, `ENGAGED_VIEWS`.
2. The response returns `id`, `href` and `status` URLs.
3. Poll `DspApi_ReadReportStatus` (`GET /cm/v1/reports/{report_id}/status`) until the status is `READY`.
   Statuses are `ACCEPTED`, `READY`, `FAILED`; on `FAILED` the cause is in `errors`.
4. Download the file URLs the status resource publishes. No filters are available on this path.
5. `DspApi_CreateReportExport` covers the recurring-export variant.

Poll with backoff. Status reads count against the 300 / 5 min ad-account limit.

## Log export

`DspApi_CreateLog` with `ad_account_id`, `type` (`IMP`, `CLICK`, `CONVERSION`, `SKAN_CONVERSION`,
`ENGAGED_VIEW`), `format` (`CSV` or `AVRO`) and `date`, then poll `DspApi_ReadLogStatus`. The Log API is
**disabled by default** and must be enabled by a Moloco representative; `GET /logs` is capped at 30 / day.

## Cohort summaries

`DspApi_ReadCohortSummary` is quota-split by the `type` property: 60/minute for ROAS types, 60/hour for KPI
action types. Branch your throttling on the type you are requesting.

## Error handling

A quota breach returns gRPC code 8: `{"code": 8, "message": "API quota exceeded. Please try again later.
code = ResourceExhausted"}`. Read `X-Rate-Limit-Quota`, `X-Rate-Limit-Remaining` and `X-Rate-Limit-Reset`
from every response and schedule against them rather than retrying blind.
