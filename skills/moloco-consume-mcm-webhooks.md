---
name: moloco-consume-mcm-webhooks
description: Receive Moloco Commerce Media webhooks safely — verify the HMAC signature against the raw body, dedupe on the delivery id, and acknowledge with a 2xx before doing work.
api: openapi/moloco-commerce-media-webhooks-openapi.yml
operations:
  - receiveAdAccountReEngageChurned
  - receiveCampaignBudgetHighUpsell
generated: '2026-07-31'
method: generated
source: openapi/moloco-commerce-media-webhooks-openapi.yml
---

# Consume Moloco Commerce Media webhooks

MCM POSTs signed JSON to the HTTPS endpoint configured for your platform. Two event types are published
today; both are recommendation feeds, not state-change notifications.

| Event | Payload | Batching |
|---|---|---|
| `ad_account.re_engage.churned` | campaign recommendations for re-engaging churned ad accounts | ≤ 20 ad accounts, ≤ 20 campaigns total |
| `campaign.budget.high_upsell` | budget-increase recommendations for high-utilization campaigns | campaigns from ≤ 20 ad accounts |

## Envelope

```json
{
  "id": "<delivery id>",
  "event": "campaign.budget.high_upsell",
  "test": false,
  "event_at": "<RFC 3339>",
  "platform_id": "<your platform>",
  "data": { }
}
```

Headers on every delivery:

- `x-moloco-webhook-signature` — required
- `x-moloco-webhook-id` — required; identical to the body `id`
- `User-Agent: Moloco-Webhook/1` — required

## 1. Verify the signature — on the RAW body

The signature has the form `t=<unix>,v1=<hex>` and matches
`^t=[0-9]+,v1=[0-9a-f]{64}(,v1=[0-9a-f]{64})*$`.

Compute `HMAC-SHA256(signing_secret, "{t}.{raw_request_body}")` and compare the lowercase hex digest against
a `v1` value using a constant-time comparison.

Two rules the spec states outright:

- **Do not parse or re-serialize the body before verification.** Frameworks that JSON-decode and re-encode
  will change bytes and break the digest. Capture the raw bytes.
- **More than one `v1` value can be present during signing-key rotation.** Accept the delivery if *any*
  `v1` matches, otherwise rotation causes an outage.

Reject the request if verification fails. Also bound `t` against your clock to reject replays.

## 2. Dedupe on `x-moloco-webhook-id`

The header is the idempotency key and equals the body `id`. Store processed ids and ignore duplicates —
Moloco retries failed attempts, so the same delivery will arrive more than once.

## 3. Acknowledge fast

Any 2xx acknowledges the delivery. A non-2xx is treated as a failed attempt and may be retried. Persist the
payload, return 2xx, and process asynchronously — do not do the recommendation work inline.

## 4. Honour the `test` flag

`test: true` means the delivery was initiated as a test. Route it away from production side-effects.

## 5. There is no other push surface

Moloco Ads (DSP) publishes no webhooks. Report and log generation there are asynchronous **jobs you poll**
(`DspApi_ReadReportStatus`, `DspApi_ReadLogStatus`), not pushed events. See
`asyncapi/moloco-commerce-media-webhooks.yml`.
