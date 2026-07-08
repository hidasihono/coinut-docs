# Swap - Iron

[Skip to main content](https://docs.iron.xyz/swap#content-area)

## Use Case

Enable users to swap **token 1 into token 2** across various chains.

Examples:

- Setup wallet address 1 which turns incoming **USDC on Solana into EURC on Base**
- Setup wallet address 2 which turns incoming **SOL on Solana into USDC on Arbitrum**

## Example Flow

1

User sets up a **wallet address 1** on Solana owned by Iron

2

Now an `autoramp` is activatedi. which turns all incoming USDC on wallet address 1 into EURC on Baseii. and delivers those into his connected wallet in **wallet address 2**

3

Now user sends 1000 USDC to **wallet address 1**

4

As soon as funds hit this address, Iron turns them into EURC on Base and delivers them to **wallet address 2**

This flow lets applications add bridging and automatic swaps without users stepping through multi-step UIs. Any wallet or frontend can integrate it with one API call.

## Prerequisites

Every step must complete before moving to the next. Sandbox-only steps are marked.

1

Customer is Active

Your customer must have `Active` status. This means they have completed [identification](https://docs.iron.xyz/onboarding) (KYC/KYB) and signed all required documents.

2

Register the recipient wallet address

Register the destination wallet via [Crypto Addresses](https://docs.iron.xyz/crypto-addresses) for [Travel Rule](https://docs.iron.xyz/travel-rule) compliance. Self-hosted wallets require a signed proof-of-ownership message; hosted wallets require the custodian’s DID.The wallet you pass in `recipient_account` must match an address you have already registered for this customer.

3

(Optional) Get a quote

If you need a locked rate (and lower-bps fee profiles), request a quote via `GET /api/autoramps/quote`. See [Quotes](https://docs.iron.xyz/quotes). Otherwise, the swap executes at the current mid-market rate.Standalone swap autoramps cannot attach quotes later. If you anticipate needing locked rates over time, start with a quote-source autoramp.

4

Create the swap autoramp

Create the autoramp via `POST /api/autoramps` using the registered wallet address as the recipient. See the Implementation example below.

5

Sandbox: approve the autoramp

In Sandbox, the autoramp starts in `Authorized` status. Approve it to enable transaction processing:

```
curl -X PUT "https://api.sandbox.iron.xyz/api/sandbox/autoramp/<autoramp_id>" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '"Approved"'
```

In production, autoramps are approved automatically after compliance review.

## Implementation

Follow these example steps to create an autoramp logic for turning USDC on Solana to EURC on Base.

### Request

```
curl -X POST "https://api.sandbox.iron.xyz/api/autoramps" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "IDEMPOTENCY-KEY: <your-idempotency-key>" \
  -H "X-API-Key: <your-api-key>" \
  -d '{
    "source_currencies": [{
      "type": "Crypto",
      "token": "USDC",
      "blockchain": "Solana"
    }],
    "destination_currency": {
      "type": "Crypto",
      "token": "EURC",
      "blockchain": "Base"
    },
    "recipient_account": {
      "type": "Crypto",
      "chain": "Base",
      "address": "0xaf77d065e77c8cC2239327C5EDb3A432268e5831"
    },
    "customer_id": "123e4567-e89b-12d3-a456-426614174000",
    "source_is_third_party": false
  }'
```

> `POST /api/autoramps` requires an `IDEMPOTENCY-KEY` header. Use a unique UUID per request to prevent duplicate autoramps.

> The request uses `chain` in `recipient_account`, but the response returns the same value as `blockchain` in `recipient`. Both are correct: map between them when comparing the request and response.

### Response

```
{
  "id": "d4e3c2b1-a9f8-7654-3210-fedcba987654",
  "customer_id": "123e4567-e89b-12d3-a456-426614174000",
  "status": "Authorized",
  "kind": "Swap",
  "source": "Standalone",
  "source_currencies": [
    {
      "type": "Crypto",
      "token": "USDC",
      "blockchain": "Solana"
    }
  ],
  "destination_currency": {
    "type": "Crypto",
    "token": "EURC",
    "blockchain": "Base"
  },
  "recipient": {
    "type": "Wallet",
    "blockchain": "Base",
    "address": "0xaf77d065e77c8cC2239327C5EDb3A432268e5831"
  },
  "is_third_party": false,
  "batch_payout": false,
  "fee_profile_id": "9b2e7c14-5f3a-4d8b-b1c6-2a4e6f8d0c12",
  "quotes": [],
  "deposit_rails": [],
  "created_at": "2025-01-20T14:23:45Z"
}
```

> `deposit_rails` is empty at `Authorized`. Poll `GET /api/autoramps/{id}` or subscribe to webhooks until `status = Approved` before sharing deposit details with end users. See [Autoramp Status](https://docs.iron.xyz/autoramp-status).

Once the autoramp reaches `Approved`, the response includes the deposit wallet address under `deposit_rails`. Share it with the user along with the supported assets for that wallet (e.g. EURC, USDC). Non-supported assets sent to the wallet will be returned to sender.

Was this page helpful?

[Previous](https://docs.iron.xyz/offramp)[Next](https://docs.iron.xyz/account)

⌘I