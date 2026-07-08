# Offramp - Iron

[Skip to main content](https://docs.iron.xyz/offramp#content-area)

## Use Case

Enable users to offramp **stablecoins into fiat** across various chains.

This works via setting up an autoramp rule on Iron that offramps stablecoins that are received into that address to your bank account.

Examples:

- Setup **wallet address 1** which turns stablecoins (e.g. USDC) into fiat and sends them to your bank account at Deutsche Bank
- Setup **wallet address 2** which turns stablecoins (e.g. USDC) into fiat and sends them to your personal bank account at Revolut

## Example Flow

1

Max sets up a **wallet address 1** owned by Iron

2

Now an `autoramp` is activated which turns USDC to EUR and is tied to his bank account

3

Max sends 1000 USDC to **wallet address 1**

4

As soon as funds hit this address, Iron turns them into EUR and pays them to Max’s bank account

5

Max can transfer at any time. This **wallet address 1** is persistent.

6

Non-supported assets sent to **wallet address 1** will be returned to sender

## Prerequisites

Every step must complete before moving to the next. Sandbox-only steps are marked.

1

Customer is Active

Your customer must have `Active` status. This means they have completed [identification](https://docs.iron.xyz/onboarding) (KYC/KYB) and signed all required documents.

2

Register a bank account

Register the destination bank account via `POST /addresses/fiat`. See [Fiat Addresses](https://docs.iron.xyz/fiat-addresses). The account starts in `RegistrationPending` status.

3

Sandbox: approve the fiat address

In Sandbox, the fiat address stays in `RegistrationPending` until you manually advance it:

```
curl -X PUT "https://api.sandbox.iron.xyz/api/sandbox/fiat-verification/<fiat_address_id>" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '"Registered"'
```

In production, registration is handled automatically.

4

Register crypto addresses

Register your customer’s wallet addresses via [Crypto Addresses](https://docs.iron.xyz/crypto-addresses) for [Travel Rule](https://docs.iron.xyz/travel-rule) compliance. Funds from unregistered wallets may be delayed for manual review.

5

(Optional) Get a quote

If you need a locked rate, request a quote via `GET /autoramps/quote`. See [Quotes](https://docs.iron.xyz/quotes). Otherwise, the autoramp executes at the current mid-market rate.If you started with a quote, you can keep attaching further quotes after the offramp is created via `POST /autoramps/{autoramp_id}/quotes`. Standalone offramps (created without an initial quote) cannot use this and always execute at the current rate.

6

Create the offramp autoramp

Create the autoramp via `POST /autoramps` using the registered bank account’s `account_identifier`.

7

Sandbox: approve the autoramp

In Sandbox, the autoramp starts in `Authorized` status. Approve it to enable transaction processing:

```
curl -X PUT "https://api.sandbox.iron.xyz/api/sandbox/autoramp/<autoramp_id>" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '"Approved"'
```

In production, autoramps are approved automatically after compliance review.

> The `account_identifier` you provide in `recipient_account` must match a bank account you’ve already registered via the [Fiat Addresses API](https://docs.iron.xyz/fiat-addresses). Iron uses this to ensure secure and compliant payouts.

## Implementation

Create an autoramp that converts USDC on Ethereum to EUR and pays out to a SEPA bank account.

`POST` `/api/autoramps`

> This endpoint accepts an `Idempotency-Key` header. Send the same key to safely retry a request without creating a duplicate autoramp.

### Request fields

| Field | Required | Description |
| --- | --- | --- |
| `source_currencies` | Yes | Array of input currencies (Crypto or Fiat objects) |
| `destination_currency` | Yes | Output currency (Crypto or Fiat object) |
| `recipient_account` | Yes | Destination account. For fiat: `{ "type": "Fiat", "account_identifier": { ... } }`. For crypto: `{ "type": "Crypto", "chain": "...", "address": "..." }` |
| `customer_id` | Yes | The customer’s UUID |
| `source_is_third_party` | No | Whether the source allows third-party payments |
| `sepa_payment_reference` | No | Reference for outgoing SEPA transfers (max 140 chars) |
| `ach_payment_reference` | No | Reference for outgoing ACH transfers (max 10 chars) |
| `batch_payout` | No | If `true`, payouts are batched daily instead of sent individually. Fiat destinations only. |

The `account_identifier` inside `recipient_account` uses the same payment rail discriminator as [Fiat Addresses](https://docs.iron.xyz/fiat-addresses): `SEPA` (requires `iban`), `ACH` / `Wire` / `RTP` (require `routing_number` + `account_number`), `SWIFT` (requires `bic` + `account_number`), `CHAPS` / `FPS` (require `sort_code` + `account_number`).

### Request

```
curl -X POST "https://api.sandbox.iron.xyz/api/autoramps" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Idempotency-Key: <your-idempotency-key>" \
  -H "X-API-Key: <your-api-key>" \
  -d '{
    "source_currencies": [{
      "type": "Crypto",
      "token": "USDC",
      "blockchain": "Ethereum"
    }],
    "destination_currency": {
      "type": "Fiat",
      "code": "EUR"
    },
    "recipient_account": {
      "type": "Fiat",
      "account_identifier": {
        "type": "SEPA",
        "iban": "DE89370400440532013000"
      }
    },
    "customer_id": "123e4567-e89b-12d3-a456-426614174000",
    "source_is_third_party": false
  }'
```

### Response

```
{
  "id": "9a7b6c5d-e3f1-4g2h-8i7j-6k5l4m3n2o1p",
  "status": "Authorized",
  "deposit_rails": [
    {
      "type": "Crypto",
      "chain": "Ethereum",
      "address": "0xYourEthereumAddress"
    }
  ],
  "created_at": "2025-01-20T13:45:23Z"
}
```

> The example above is trimmed to the fields most integrations need. The full `Autoramp` object also returns `kind`, `source`, `recipient`, `customer_id`, `source_currencies`, `destination_currency`, `is_third_party`, `fee_profile_id`, `batch_payout`, and `quotes`. The `deposit_rails` array contains the payment details. The legacy `deposit_account` field is deprecated.

### Error response

A `422` is returned when the request fails validation, for example when the `account_identifier` does not match a registered fiat address.

```
{
  "error": {
    "code": "unprocessable_entity",
    "message": "recipient_account does not match a registered fiat address",
    "request_id": "req_1a2b3c4d5e6f7g8h"
  }
}
```

Now you are ready to show the user their custom wallet address which will be auto-connected to this offramp rule. Please also show supported assets for this wallet address (e.g. EURC, USDC). Non-supported assets will be returned to sender.

Was this page helpful?

[Previous](https://docs.iron.xyz/onramp)[Next](https://docs.iron.xyz/swap)
