# Onramp - Iron

[Skip to main content](https://docs.iron.xyz/onramp#content-area)

## Use Case

Enable users to onramp **fiat into stablecoins** on various chains.

This works via spinning up virtual IBANs in the names of customers which are tied to automatic onramping rules.

Examples:

- Create a **vIBAN 1** which turns EUR to USDC on Solana address 1
- Create a **vIBAN 2** which turns EUR to USDC on Arbitrum address 1
- Create a **vIBAN 3** which turns EUR to EURC on Arbitrum address 2

## Example Flow

1

Max sets up a vIBAN 1 with an `autoramp` which turns EUR to USDC on Solana address 3DkN…BV4Lm

2

Max triggers a EUR 1000 transfer from their banking app

3

Iron will monitor the vIBAN 1; as soon as funds arrive, 1100 USDC are delivered to 3DkN…BV4Lm

4

Max can at any time transfer more funds into this vIBAN 1 and they autoramp to USDC. It is persistent.

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

If you need a locked rate (and lower-bps fee profiles), request a quote via `GET /autoramps/quote`. See [Quotes](https://docs.iron.xyz/quotes). Otherwise, the onramp executes at the current mid-market rate.Standalone onramp autoramps cannot attach quotes later. If you anticipate needing locked rates over time, start with a quote-source autoramp.

4

Create the onramp autoramp

Create the autoramp via `POST /autoramps` using the registered wallet address as the recipient. See the Implementation example below.

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

Follow these example steps to create an autoramp logic for EUR to USDC on Ethereum.

### Request

```
curl -X POST "https://api.sandbox.iron.xyz/api/autoramps" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "IDEMPOTENCY-KEY: <your-idempotency-key>" \
  -H "X-API-Key: <your-api-key>" \
  -d '{
    "source_currencies": [{
      "type": "Fiat",
      "code": "EUR"
    }],
    "destination_currency": {
      "type": "Crypto",
      "token": "USDC",
      "blockchain": "Ethereum"
    },
    "recipient_account": {
      "type": "Crypto",
      "chain": "Ethereum",
      "address": "myholdings.eth"
    },
    "customer_id": "123e4567-e89b-12d3-a456-426614174000",
    "source_is_third_party": false
  }'
```

### Response

```
{
  "id": "f8f9a8c7-d8b1-4f1a-8f15-7e6c8a1a8e50",
  "status": "Authorized",
  "deposit_rails": [],
  "created_at": "2025-01-20T12:34:56Z"
}
```

> `deposit_rails` is empty at `Authorized`. Poll `GET /autoramps/{id}` or subscribe to webhooks until `status = Approved` before sharing deposit details with end users. See [Autoramp Status](https://docs.iron.xyz/autoramp-status).

Once the autoramp reaches `Approved`, the response includes the deposit vIBAN under `deposit_rails`. Share it with the user. vIBANs are named; no specific deposit instructions are required.

Was this page helpful?

[Previous](https://docs.iron.xyz/end-of-day-settlement)[Next](https://docs.iron.xyz/offramp)

⌘I