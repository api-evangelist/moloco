---
name: moloco-manage-mcm-budget
description: Fund and govern a Moloco Commerce Media ad account — read wallets and spending limits, top up or withdraw with a replay-safe request id, set spend caps, and reconcile from transaction history.
api: openapi/moloco-commerce-media-management-openapi.yml
base_url: https://sandbox-mgmt.mcm-api.moloco.com
operations:
  - RmpManagementApi_ListWallets
  - RmpManagementApi_TopUpWallet
  - RmpManagementApi_WithdrawWallet
  - RmpManagementApi_QueryWalletTransactionHistory
  - RmpManagementApi_QueryWalletAccountHistory
  - RmpManagementApi_ListSpendingLimits
  - RmpManagementApi_ReadSpendingLimit
  - RmpManagementApi_UpdateSpendingLimit
  - RmpManagementApi_ListCredits
  - RmpManagementApi_TopUpCredit
  - RmpManagementApi_WithdrawCredit
  - RmpManagementApi_QuerySpendingLimitCreditHistory
  - RmpManagementApi_ListSpendCaps
  - RmpManagementApi_CreateSpendCap
  - RmpManagementApi_UpdateSpendCap
  - RmpManagementApi_QueryWallets
  - RmpManagementApi_QuerySpendingLimits
generated: '2026-07-31'
method: generated
source: >-
  openapi/moloco-commerce-media-management-openapi.yml, https://mcm-docs.moloco.com/docs/billing,
  https://mcm-docs.moloco.com/docs/prepaid-wallet, https://mcm-docs.moloco.com/docs/postpaid-spending-limit-control
---

# Manage Moloco Commerce Media budget

Two mutually exclusive funding models per ad account, plus an optional spend cap on top.

- **Wallet** — prepaid. Balance is topped up before spend.
- **Spending limit + credits** — postpaid. Credit is extended and drawn down.
- **Spend cap** — a ceiling applied independently of either.

Today an ad account has at most one wallet and at most one spending limit.

## Auth

`x-api-key: $API_KEY` on every call (the `Authorization` bearer scheme is documented as **deprecated** in the
Management API spec). `platform_id` and `ad_account_id` are path parameters.

## Read the current position

- `RmpManagementApi_ListWallets` — `GET .../ad-accounts/{ad_account_id}/wallets`
- `RmpManagementApi_ListSpendingLimits` — `GET .../ad-accounts/{ad_account_id}/spending-limits`
- `RmpManagementApi_ListCredits` — `GET .../ad-accounts/{ad_account_id}/credits`
- `RmpManagementApi_ListSpendCaps` — `GET .../ad-accounts/{ad_account_id}/spend-caps`

Platform-wide rollups: `RmpManagementApi_QueryWallets`, `RmpManagementApi_QuerySpendingLimits`,
`RmpManagementApi_QueryCredits`.

## Move money — always with a request id

The money-moving operations carry a **`request_id` path parameter that is an idempotency key**:

```
PUT .../wallets/{wallet_id}/top-up/{request_id}
PUT .../wallets/{wallet_id}/withdraw/{request_id}
PUT .../credits/{credit_id}/top-up/{request_id}
PUT .../credits/{credit_id}/withdraw/{request_id}
```

The spec is explicit: *"For retries, the same request_id should be used again to prevent duplicate
requests."* Constraints: alphanumeric, underscore and hyphen only, 8–64 characters.

Generate the `request_id` **before** the first attempt, persist it with your intent record, and reuse it for
every retry of that same intent. Never generate a fresh id on retry — that is how a top-up gets applied
twice.

Amounts are **VAT-exclusive**.

Known failures: `INSUFFICIENT_WALLET_BALANCE` (3002) on an over-withdrawal, `INSUFFICIENT_SPENDING_LIMIT`
(3000) when an account suspended at its limit is reactivated without headroom. Full registry in
`errors/moloco-error-codes.yml`.

## Adjust the controls

- `RmpManagementApi_UpdateSpendingLimit` — `PUT .../spending-limits/{spending_limit_id}`
- `RmpManagementApi_CreateSpendCap` / `RmpManagementApi_UpdateSpendCap`

## Reconcile

- `RmpManagementApi_QueryWalletTransactionHistory` and `RmpManagementApi_QueryWalletAccountHistory`
- `RmpManagementApi_QuerySpendingLimitCreditHistory` and `RmpManagementApi_QueryCreditHistory`

List responses paginate with `page_info` (`page_index`, `page_size`, `pages_total`) — walk to
`pages_total` before treating a reconciliation as complete.

## Watch for the concurrency error

`TIMESTAMP_MISMATCH` (100) means the entity moved underneath you: *"The timestamp provided doesn't match the
latest value."* Re-read the entity, take the fresh timestamp, and re-apply — do not blind-retry the same body.
