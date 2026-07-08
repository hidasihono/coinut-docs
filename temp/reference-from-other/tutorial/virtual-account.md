# Virtual Accounts - Iron

[Skip to main content](https://docs.iron.xyz/account#content-area)

Spin up **named virtual fiat bank accounts in USD, EUR, and GBP** for your users in minutes.

All incoming funds are immediately **auto-minted to stablecoins** (USDC, EURC, etc.) and delivered directly to users’ self-custodial wallets linked to Iron. No customer funds ever sit idle in a bank.

**Typical use cases:**

- Freelancers in emerging markets receiving USD or EUR payments
- Banking apps issuing multi-currency accounts
- Crypto/DeFi wallets offering named accounts with local payment rails

## Named Accounts

Each virtual account is issued in the **user’s own name**, not a pooled or generic account. Incoming payments clear faster, pass compliance checks more reliably, and feel like a real bank account to the sender.

**Example: a USD account issued for Jane Doe:**

| Field | Value |
| --- | --- |
| Account holder | Jane Doe |
| Account number | xxxxxxx |
| Routing number | xxxxxxx |
| Account type | Checking |
| Bank name | Banking Circle |

A sender wiring money to Jane uses these details as they would any US bank account. Once received, Iron auto-mints the funds to USDC and delivers them to Jane’s self-custodial wallet.

## How It Works

1

User creates a virtual USD, EUR, or GBP account on Iron

2

User receives their account credentials (account number, sort code/IBAN, etc.)

3

User receives fiat from their own bank transfer or from a third party

4

Iron auto-mints the fiat into USDC or another stablecoin and delivers it to the user’s self-custodial wallet

> A user can hold European bank credentials (IBAN) while having all funds auto-ramped into a USD-denominated stablecoin like USDC. No manual conversion required.

## Key Concepts

Virtual Accounts are **payment routing targets**, not holding accounts. They’re designed for efficient payment ingestion and reconciliation.

- **Payment routing only**: Virtual Accounts don’t hold funds. Every received payment is immediately processed per your configured autoramp settings.
- **Open to any sender**: The account works like a normal bank account. Anyone can send fiat to it using the account credentials. No sender pre-approval is required.
- **Named to your customer**: Each account is issued in the customer’s verified name, so incoming payments clear faster and pass compliance checks more reliably.
- **Automatic returns**: Payments that fail compliance checks or arrive with incorrect details are returned to the sender.

## Quick Start

This tutorial walks through the full flow in Sandbox: create a customer, complete identification, handle signings, register a wallet, create an autoramp, and receive your customer’s virtual bank account credentials.

**Prerequisites:**

- A Sandbox API key from the [Partner Dashboard](https://app.sandbox.iron.xyz/dashboard/keys)
- An Ethereum wallet with signing capability (e.g. [ethers.js](https://docs.ethers.org/), [viem](https://viem.sh/), or a similar library) — required in Step 5 to sign a proof-of-ownership message

1

Create a customer

`POST /customers`Create a customer record with `customer_type` set to `Person` (individual) or `Business` (company). The customer starts in `IdentificationRequired` status.**API Example**

```
curl -X POST "https://api.sandbox.iron.xyz/api/customers" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "IDEMPOTENCY-KEY: <your-idempotency-key>" \
  -H "X-API-Key: <your-api-key>" \
  -d '{
    "customer_type": "Person",
    "name": "jane_doe",
    "email": "jane@example.com"
  }'
```

**Example response:**

```
{
  "id": "b2c3d4e5-f6a7-8901-bcde-f23456789012",
  "customer_type": "Person",
  "status": "IdentificationRequired",
  "created_at": "2026-01-15T10:00:00Z",
  "updated_at": "2026-01-15T10:00:00Z"
}
```Save the `id` from the response. You need it in every subsequent step.

2

Create a Link identification

`POST /customers/{id}/identifications/v2`Create an identification with `type: "Link"`. Iron returns a URL where your customer completes KYC (individuals) or KYB (businesses) through Iron’s hosted verification interface.**API Example**

```
curl -X POST "https://api.sandbox.iron.xyz/api/customers/<customer_id>/identifications/v2" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "IDEMPOTENCY-KEY: <your-idempotency-key>" \
  -H "X-API-Key: <your-api-key>" \
  -d '{"type": "Link"}'
```

**Example response:**

```
{
  "id": "c3d4e5f6-a7b8-9012-cdef-345678901234",
  "status": "Pending",
  "url": "https://verify.iron.xyz/start?token=..."
}
```In production, pass the `url` to your customer to complete verification. Save the identification `id` for the next step.

3

Approve the identification (Sandbox only)

`POST /sandbox/identification/{id}`In Sandbox, you control whether an identification is approved or rejected. Approve it to move the customer forward.**API Example**

```
curl -X POST "https://api.sandbox.iron.xyz/api/sandbox/identification/<identification_id>" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "IDEMPOTENCY-KEY: <your-idempotency-key>" \
  -H "X-API-Key: <your-api-key>" \
  -d '{"approved": true}'
```After approval, the customer status transitions to `SigningsRequired`. In production, Iron’s compliance team handles this review (typically 24-48 hours).

4

Retrieve and complete signings

`GET /customers/{id}/required-signings`Retrieve the documents your customer must sign (e.g. Terms and Conditions), present each document to them, then mark it as signed.**API Example**

```
curl -X GET "https://api.sandbox.iron.xyz/api/customers/<customer_id>/required-signings" \
  -H "X-API-Key: <your-api-key>"
```

**Example response from required-signings:**

```
[
  {
    "display_name": "Terms and Conditions",
    "id": "019ababb-ddd6-7f02-8f5d-7469a0e8afb6",
    "url": "https://example.com/terms-and-conditions"
  }
]
```

The signing request takes `content_id` (the `id` from `required-signings`) and `signed: true`.Loop through every entry in the array and sign each one. After all signings are complete, the customer status becomes `Active`.

5

Register a crypto address

`POST /addresses/crypto/selfhosted`

Register the wallet address where stablecoins will be delivered. This is required for [Travel Rule](https://docs.iron.xyz/travel-rule) compliance before creating any autoramp.

For self-hosted wallets, sign a proof-of-ownership message with the wallet’s private key. See [Crypto Addresses](https://docs.iron.xyz/crypto-addresses) for the full signing process and hosted wallet registration.

**API Example**

```
curl -X POST "https://api.sandbox.iron.xyz/api/addresses/crypto/selfhosted" \
  -H "Content-Type: application/json" \
  -H "IDEMPOTENCY-KEY: <your-idempotency-key>" \
  -H "X-API-Key: <your-api-key>" \
  -d '{
    "customer_id": "<customer_id>",
    "address": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
    "message": "I am verifying ownership of the wallet address 0x742d35Cc6634C0532925a3b844Bc454e4438f44e as customer <customer_id>. This message was signed on 15/01/2026 to confirm my control over this wallet.",
    "signature": "<wallet-signature>",
    "blockchain": "Ethereum"
  }'
```

> The proof message must include today’s date in DD/MM/YYYY format (UTC). You can also register a [hosted wallet](https://docs.iron.xyz/crypto-addresses#register-a-hosted-wallet) if the wallet is custodied by another VASP.

6

Create an autoramp

`POST /autoramps`Create an autoramp that converts incoming fiat to stablecoins. The `recipient_account.address` must match a wallet you registered in the previous step.This example converts EUR to EURC on Ethereum:**API Example**

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
      "token": "EURC",
      "blockchain": "Ethereum"
    },
    "recipient_account": {
      "type": "Crypto",
      "chain": "Ethereum",
      "address": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e"
    },
    "customer_id": "<customer_id>"
  }'
```

**Example response:**

```
{
  "id": "d3c2b1a4-e5f6-78a9-0123-fedcba654321",
  "status": "Authorized",
  "deposit_rails": [
    {
      "type": "Iban",
      "id": "xyz98765-lkjh-4321-mnop-0987654321ab",
      "iban": "DE89370400440532013000",
      "name": "Jane Doe",
      "bic": "COBADEFFXXX",
      "bank_name": "Good Bank",
      "bank_country": "Germany",
      "currency": {
        "type": "Fiat",
        "code": "EUR"
      }
    }
  ],
  "created_at": "2026-01-15T10:05:00Z"
}
```

7

View deposit rails

The autoramp response includes a `deposit_rails` array with the customer’s virtual bank account credentials. Display these to your customer so they (or third parties) can send fiat payments.

| Field | Description |
| --- | --- |
| `type` | Rail type: `Iban` (EUR), `UsAccount` (USD), or `SortCode` (GBP) |
| `iban` | IBAN number (EUR accounts) |
| `name` | Account holder name, matching the customer’s verified identity |
| `bic` | Bank identifier code |
| `bank_name` | Name of the issuing bank |
| `bank_country` | Country of the issuing bank |
| `currency` | The fiat currency for this rail |

Virtual accounts are named. No payment references or special deposit instructions are required. Any fiat sent to this account auto-converts to the destination stablecoin and delivers to the registered wallet.

> The `deposit_rails` array contains the payment details. The legacy `deposit_account` field is deprecated.

8

Test with a sandbox transaction (Sandbox only)

`POST /sandbox/transaction`Simulate an incoming fiat payment to verify the full end-to-end flow. Pass the autoramp ID and a deposit amount.**API Example**

```
curl -X POST "https://api.sandbox.iron.xyz/api/sandbox/transaction" \
  -H "Content-Type: application/json" \
  -H "IDEMPOTENCY-KEY: <your-idempotency-key>" \
  -H "X-API-Key: <your-api-key>" \
  -d '{
    "autoramp_id": "<your-autoramp-id>",
    "amount": "100"
  }'
```The sandbox processes the simulated payment through the autoramp. Monitor the resulting transaction via [webhooks](https://docs.iron.xyz/webhooks) or the [Partner Dashboard](https://app.sandbox.iron.xyz/).

## Monitoring Payments

Webhooks are the recommended way to track incoming payments on a Virtual Account. Without them, you would need to poll the API for transaction updates.

### Setup

Register a webhook endpoint in the [Partner Dashboard](https://app.sandbox.iron.xyz/) under the Webhooks section, or use the [Webhooks API](https://docs.iron.xyz/webhooks). Subscribe to at least the `transaction` and `transaction_status` topics.

### Relevant Events

When a payment arrives at a Virtual Account, Iron fires the following events in order:

| Event | Type | When it fires |
| --- | --- | --- |
| New transaction | `transaction` | Fiat payment is received at the vIBAN |
| Status update | `transaction_status` | Each time the transaction progresses (`FundsReviewInProgress`, `ConversionInProgress`, `PayoutInProgress`, `Completed`) |
| Autoramp status | `register_autoramp_status` | Autoramp state changes (e.g. `Authorized`, `Approved`) |

The `transaction` event tells you a payment arrived. The `transaction_status` events tell you where it is in the conversion pipeline.

### Example: Transaction Received

When fiat lands on the vIBAN, you receive a `transaction` event with the `customer_id` and a transaction `id` you can use to query details:

```
{
  "type": "transaction",
  "timestamp": "2026-01-15T10:30:00+00:00",
  "data": {
    "customer_id": "b2c3d4e5-f6a7-8901-bcde-f23456789012",
    "message": {
      "Event": {
        "id": "7d834f68-cea8-496a-8eae-bb0772365028",
        "kind": "Transaction"
      }
    }
  }
}
```

### Example: Transaction Status Update

As the payment moves through conversion and payout, you receive `transaction_status` events. Use `transaction_status` (not the deprecated `status` field) for the current state:

```
{
  "type": "transaction_status",
  "timestamp": "2026-01-15T10:30:05+00:00",
  "data": {
    "customer_id": "b2c3d4e5-f6a7-8901-bcde-f23456789012",
    "message": {
      "TransactionStatus": {
        "id": "7d834f68-cea8-496a-8eae-bb0772365028",
        "status": "Pending",
        "transaction_status": "ConversionInProgress",
        "transaction_hash": null
      }
    }
  }
}
```

Once the stablecoin is delivered to the wallet, `transaction_status` becomes `Completed` and `transaction_hash` contains the on-chain transaction hash.

### Transaction Status Reference

| `transaction_status` | Description |
| --- | --- |
| `FundsReviewInProgress` | Deposit received, undergoing AML/compliance check |
| `ConversionInProgress` | Fiat is being converted to the destination stablecoin |
| `PayoutInProgress` | Stablecoin sent to blockchain, awaiting confirmation |
| `Completed` | Funds delivered to the destination wallet |
| `Failed` | Processing error |
| `RejectedAml` | Rejected by compliance checks |
| `RejectedFraud` | Rejected by fraud review |
| `RejectedMinAmount` | Amount below the minimum threshold |

> Your webhook endpoint must return `200 OK` to acknowledge receipt. Non-2xx responses trigger retries with exponential backoff. See [Webhooks](https://docs.iron.xyz/webhooks) for signature verification and full implementation details.

## Related Guides

Was this page helpful?

[Previous](https://docs.iron.xyz/swap)[Next](https://docs.iron.xyz/issuance/create-your-stablecoin)

⌘I