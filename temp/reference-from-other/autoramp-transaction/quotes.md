# Quotes - Iron

[Skip to main content](https://docs.iron.xyz/quotes#content-area)

Request a signed quote to lock a conversion rate for a defined window. An Autoramp can hold multiple active quotes at once, so partners can keep a single ramp running over time without re-creating it.

## How It Works

A quote binds a conversion rate between two currencies for a limited window. You request a quote with `GET /autoramps/quote`, then pass the signed result to `POST /autoramps` to create an Autoramp that executes at the locked rate.

Once the Autoramp exists, you can keep attaching new quotes to it via `POST /autoramps/{autoramp_id}/quotes`. Each attached quote stays active until its own `valid_until` timestamp passes.

When a deposit arrives, Iron picks the **newest active quote** whose `amount_in` value and source currency match the deposit. Deposits that don’t match any quote, or only match an expired quote, are returned to sender. When a deposit matches an active quote whose rate-lock has expired (but the quote itself is still within `valid_until`), behavior depends on the quote’s `rate_expiry_policy` (see [Expiry policies](https://docs.iron.xyz/quotes#expiry-policies)).

> Without a quote, conversions execute at the current mid-market rate. With one or more active quotes, the deposit executes at the locked rate of the matching quote.

### Key parameters

**Rate lock duration**: time window where the quoted rate and fee are fully guaranteed. Deposits that match within this window execute at the quoted price.

**Validity period**: total time the quote remains usable. Deposits arriving after `valid_until` will not match this quote.

### Rate lock duration caps

The `rate_lock_duration_minutes` parameter is optional. If omitted, the quote uses the maximum allowed for the pair. Larger values are silently clamped to the cap.

| Currency pair | Max rate lock |
| --- | --- |
| Pegged pair (EUR ↔ EUR-stablecoin, USD ↔ USD-stablecoin) | 2 days |
| Other fiat ↔ stablecoin (e.g. EUR ↔ USDC) | 10 minutes |
| Stablecoin ↔ stablecoin (e.g. USDC ↔ USDT) | 10 minutes |

### Expiry policies

What happens when funds arrive after the rate lock expires depends on the `rate_expiry_policy` you set.

- Return
- Slippage

Validity period equals rate lock duration. Funds arriving after the lock expires are refunded.

**Example:**

- Quote: 100 EUR for 108 USDC
- Rate lock: 10 minutes, validity: 10 minutes
- Funds arrive at 12 minutes
- **Result**: refunded (both windows expired)

## Prerequisites

Complete these before requesting a quote:

1

Create and verify a customer

Register a customer via `POST /customers` and complete identity verification. The customer must reach `Active` status before they can transact.See [Onboarding Lifecycle](https://docs.iron.xyz/onboarding) for the full flow.

2

Register a recipient address

The `recipient_account_id` in your quote request must reference a registered and verified address.

- **Fiat offramps**: register a bank account (SEPA, ACH, Wire, or SWIFT) via `POST /addresses/fiat`. The address must be verified before use. See [Fiat Addresses](https://docs.iron.xyz/fiat-addresses).
- **Crypto swaps**: register the destination wallet via `POST /addresses/crypto`. Self-hosted wallets require a signed proof-of-ownership message; hosted wallets require the custodian’s DID. See [Crypto Addresses](https://docs.iron.xyz/crypto-addresses).

3

Authenticate your requests

Include your API key in the `X-API-Key` header on every request. See [Authentication](https://docs.iron.xyz/authentication).

> Quote requests fail if the `recipient_account_id` references an address that has not been registered or has not passed verification.

## Create an Autoramp with a quote

1

Create a quote

`GET /autoramps/quote`Request a signed quote that defines the conversion rate, lock duration, validity period, and fees. The quote must be passed unchanged when creating an Autoramp.**API Example**

```
curl -X GET "https://api.sandbox.iron.xyz/api/autoramps/quote?customer_id=<your-customer-id>&source_currency_code=USDC&source_currency_chain=Ethereum&destination_currency_code=EUR&recipient_account_id=<your-fiat-address-id>&amount_out=100&rate_lock_duration_minutes=10&rate_expiry_policy=Return&expiry_in_hours=1&is_third_party=false" \
  -H "Accept: application/json; charset=utf-8" \
  -H "X-API-Key: <your-api-key>"
```

**Example response:**

```
{
  "amount_in": {
    "amount": "111.19863",
    "currency": { "blockchain": "Ethereum", "token": "USDC", "type": "Crypto" }
  },
  "amount_out": {
    "amount": "100",
    "currency": { "code": "EUR", "type": "Fiat" }
  },
  "customer_id": "<your-customer-id>",
  "deposit_account_type": "Iban",
  "destination_currency": { "code": "EUR", "type": "Fiat" },
  "fee": {
    "banking_fee": { "amount": "0.326434", "currency": { "blockchain": "Ethereum", "token": "USDC", "type": "Crypto" } },
    "iron_fee": { "amount": "0.326434", "currency": { "blockchain": "Ethereum", "token": "USDC", "type": "Crypto" } },
    "network_fee": { "amount": "1.734570", "currency": { "blockchain": "Ethereum", "token": "USDC", "type": "Crypto" } },
    "partner_fee": { "amount": "0", "currency": { "blockchain": "Ethereum", "token": "USDC", "type": "Crypto" } },
    "total_fee": { "amount": "2.387438", "currency": { "blockchain": "Ethereum", "token": "USDC", "type": "Crypto" } }
  },
  "fee_settlement": "deductedImmediately",
  "is_third_party": false,
  "quote_id": "e8cd73a4-54cc-4a34-916d-238c5ce5f898",
  "rate": "0.919023105",
  "rate_expiry_policy": "Return",
  "rate_lock_valid_until": "2025-03-14T11:37:24.250798+00:00",
  "recipient_account": {
    "type": "Fiat",
    "account_identifier": { "type": "SEPA", "iban": "DE89370400440532013000" }
  },
  "signature": "36665918454471c078724046bcb82698eac2fb6f5f91c58a0f1a3923c5d95a28",
  "slippage_tolerance_in_bips": null,
  "source_currency": { "blockchain": "Ethereum", "token": "USDC", "type": "Crypto" },
  "valid_until": "2025-03-14T11:37:24.250798+00:00",
  "additional_in_currencies": null,
  "pay_by_reference": null
}
```

2

Create an Autoramp with the quote

`POST /autoramps`Submit the signed quote payload to create an Autoramp. The Autoramp matches inbound deposits against the active quotes attached to it.**API Example**

```
curl -X POST "https://api.sandbox.iron.xyz/api/autoramps" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "IDEMPOTENCY-KEY: <your-idempotency-key>" \
  -H "X-API-Key: <your-api-key>" \
  -d '<signed-quote-payload-from-step-1>'
```

**Example response:**

```
{
  "id": "f8f9a8c7-d8b1-4f1a-8f15-7e6c8a1a8e50",
  "status": "Created",
  "deposit_rails": [
    {
      "type": "Crypto",
      "chain": "Ethereum",
      "address": "0x1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b"
    }
  ],
  "created_at": "2025-01-20T12:34:56Z",
  "source": "Quote",
  "quotes": [ { "quote_id": "e8cd73a4-54cc-4a34-916d-238c5ce5f898", "...": "..." } ]
}
```

The `quotes` array lists every currently-active quote on the Autoramp. The legacy `quote` field is deprecated; read `quotes` instead.

> The quote is digitally signed. Pass it verbatim into the `POST /autoramps` call. Any modification (changing values, omitting fields) causes the request to fail.

## Attach a new quote to an existing Autoramp

After the Autoramp exists, you can attach further quotes to keep it executing at locked rates over time. This works only on quote-source Autoramps (`source = "Quote"`).

> Use the right quote object. This endpoint takes the `SignedAutorampRateQuote` returned by `GET /autoramps/{autoramp_id}/quote`, not the `SignedAutorampQuote` returned by `GET /autoramps/quote`. The creation quote has no `autoramp_id` field, so submitting it here fails with `Expected input type "string_uuid", found null`.

1

Get a signed rate quote

`GET /autoramps/{autoramp_id}/quote`Returns a `SignedAutorampRateQuote`. The Autoramp’s recipient, fee profile, and destination currency are reused automatically, so this endpoint has a smaller parameter surface than the initial quote endpoint.The `source_currency_code` must be one of the Autoramp’s allowed input currencies (the original `source_currency` plus any entries you registered in `additional_in_currencies` at creation time).**API Example**

```
curl -X GET "https://api.sandbox.iron.xyz/api/autoramps/<autoramp-id>/quote?source_currency_code=USDC&source_currency_chain=Ethereum&amount_out=100&rate_lock_duration_minutes=10&rate_expiry_policy=Return&expiry_in_hours=1" \
  -H "Accept: application/json; charset=utf-8" \
  -H "X-API-Key: <your-api-key>"
```

2

Attach the signed quote

`POST /autoramps/{autoramp_id}/quotes`Submit the signed rate quote returned in the previous step. Requires an `IDEMPOTENCY-KEY` header. Returns `201` with the updated Autoramp object (its `quotes` array now includes the newly attached quote).**API Example**

```
curl -X POST "https://api.sandbox.iron.xyz/api/autoramps/<autoramp-id>/quotes" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "IDEMPOTENCY-KEY: <your-idempotency-key>" \
  -H "X-API-Key: <your-api-key>" \
  -d '<signed-rate-quote-payload>'
```

> Attaching a new quote that shares `amount_in` value and source currency with an existing active quote supersedes the older one for incoming deposits. The older quote stays in `quotes` until its own `valid_until` passes; nothing is deleted retroactively.

> **Staged setup pattern.** You can create the Autoramp with any initial quote to surface the deposit wallet (so the payer can whitelist it or finish their onboarding), then attach the production quote later via `POST /autoramps/{autoramp_id}/quotes` once the real amount is known. The initial quote simply expires unused if no matching deposit arrives.

## Multi-currency input

By default, an Autoramp only accepts deposits in its original `source_currency`. To let it accept additional input currencies (and request quotes for them later), pass `additional_in_currency_codes` and `additional_in_currency_chains` when you call `GET /autoramps/quote`.

The two parameters are index-aligned arrays of up to 16 entries each. The chain at position `i` describes the code at position `i`. The list must not include the `source_currency` itself and must not contain duplicates.

After Autoramp creation, request a quote for any of the additional currencies via `GET /autoramps/{autoramp_id}/quote` and attach it via `POST /autoramps/{autoramp_id}/quotes`.

**API Example**

```
curl -X GET "https://api.sandbox.iron.xyz/api/autoramps/quote?customer_id=<your-customer-id>&source_currency_code=USDC&source_currency_chain=Ethereum&destination_currency_code=EUR&recipient_account_id=<your-fiat-address-id>&amount_out=100&rate_expiry_policy=Return&expiry_in_hours=1&is_third_party=false&additional_in_currency_codes=USDT&additional_in_currency_chains=Ethereum&additional_in_currency_codes=USDC&additional_in_currency_chains=Polygon" \
  -H "Accept: application/json; charset=utf-8" \
  -H "X-API-Key: <your-api-key>"
```

### Query parameters: `GET /autoramps/quote`

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `customer_id` | string (UUID) | Yes | Unique identifier for the customer. |
| `source_currency_code` | string | Yes | Currency or token being sent (e.g. `USDC`, `USD`). |
| `source_currency_chain` | string | Conditional | Blockchain for the source currency. Required for crypto source currencies. |
| `destination_currency_code` | string | Yes | Currency or token being received (e.g. `EUR`, `GBP`). |
| `destination_currency_chain` | string | Conditional | Blockchain for the destination currency. Required for crypto destination currencies. |
| `recipient_account_id` | string (UUID) | Yes | UUID of a registered recipient account (`fiat_address` or `verified_crypto_address`). |
| `recipient_account` | string | No | **Deprecated.** Use `recipient_account_id` instead. Ignored when `recipient_account_id` is provided. |
| `amount_out` | string | Conditional | Desired output amount in the destination currency. Set either `amount_out` or `amount_in`, not both. |
| `amount_in` | string | Conditional | Desired input amount in the source currency. Set either `amount_in` or `amount_out`, not both. |
| `rate_lock_duration_minutes` | integer | No | Duration the rate is locked. Defaults to the pair’s maximum; values above the cap are clamped. See the rate-lock cap table above. |
| `rate_expiry_policy` | string | Yes | What happens when the rate lock expires. One of `Return`, `Slippage`. |
| `slippage_tolerance_in_bips` | string | Conditional | Allowable deviation from quoted rate (1 bip = 0.01%). Required when `rate_expiry_policy` is `Slippage`. |
| `is_third_party` | boolean | Yes | Whether this is a third-party payment Autoramp. Set to `false` for standard customer-owned accounts. |
| `expiry_in_hours` | integer | Yes | Duration the quote is valid. Max 24. Capped at `rate_lock_duration_minutes` when `rate_expiry_policy` is `Return`. |
| `additional_in_currency_codes` | string array | No | Additional input currencies to allow on the Autoramp beyond `source_currency`. Up to 16, must not duplicate or include `source_currency`. Index-aligned with `additional_in_currency_chains`. |
| `additional_in_currency_chains` | string array | No | Chains for each entry in `additional_in_currency_codes`. Same length, index-aligned. |
| `pay_by_reference` | boolean | No | If `true`, the Autoramp uses pay-by-reference. See [Autoramp → pay by reference](https://docs.iron.xyz/autoramp#api-optional-pay-by-reference) for eligibility rules and the `inbound_payment_reference` response field. Defaults to `false`. |

### Query parameters: `GET /autoramps/{autoramp_id}/quote`

The Autoramp’s recipient, destination currency, and fee profile are reused, so only pricing-related parameters are needed.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `source_currency_code` | string | Yes | Currency or token being sent. Must be one of the Autoramp’s allowed input currencies. |
| `source_currency_chain` | string | Conditional | Blockchain for the source currency. Required for crypto source currencies. |
| `amount_out` | string | Conditional | Desired output amount. Set either `amount_out` or `amount_in`, not both. |
| `amount_in` | string | Conditional | Desired input amount. Set either `amount_in` or `amount_out`, not both. |
| `rate_lock_duration_minutes` | integer | No | Optional. Defaults and clamps as on the initial endpoint. |
| `rate_expiry_policy` | string | Yes | One of `Return`, `Slippage`. |
| `slippage_tolerance_in_bips` | string | Conditional | Required when `rate_expiry_policy` is `Slippage`. |
| `expiry_in_hours` | integer | Yes | Max 24. |

> This endpoint returns `400 Bad Request` if called on a standalone (non-quote-source) Autoramp, and `400` if `source_currency_code` is not one of the Autoramp’s allowed input currencies.

> - Check `valid_until` before submitting. Expired quotes return `400 Bad Request`.
> - Use idempotency keys for `POST /autoramps` and `POST /autoramps/{id}/quotes`.
> - Sandbox rates are simulated but signatures work identically to Production.

## Error Handling

| Status | Cause |
| --- | --- |
| `400 Bad Request` | Quote expired (`valid_until` passed), signed payload modified, source currency not allowed on the Autoramp, or `GET /autoramps/{id}/quote` called on a standalone Autoramp. |
| `401 Unauthorized` | Missing or invalid API key. |
| `404 Not Found` | `recipient_account_id` does not reference a registered address, or `autoramp_id` not found. |
| `409 Conflict` | Quote already attached (only on `POST /autoramps/{autoramp_id}/quotes`). |
| `422 Unprocessable Entity` | Required parameter missing or invalid value. A common cause on `POST /autoramps/{id}/quotes` is submitting the `SignedAutorampQuote` from `GET /autoramps/quote` instead of the `SignedAutorampRateQuote` from `GET /autoramps/{autoramp_id}/quote`; the former has no `autoramp_id` and fails with `Expected input type "string_uuid", found null`. |

Was this page helpful?

[Previous](https://docs.iron.xyz/autoramp-status)[Next](https://docs.iron.xyz/transaction-status)
