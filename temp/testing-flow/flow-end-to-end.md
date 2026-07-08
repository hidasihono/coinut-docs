# End-to-End Endpoint Testing Checklist

Source: `openapi.json` (54 endpoints, current as of this file's creation). Ordered by a realistic
integration flow rather than by OpenAPI tag, so you can work top to bottom. Check the box once
you've manually tested that endpoint against sandbox, and add a note (result, gotcha, blocker,
whatever's useful) even if it passed.

---

## Phase 0 — Setup & Environment

- [ ] `GET /health` — service health check
  - Notes:
- [ ] `GET /basic-info` — partner basic info
  - Notes:
- [ ] `GET /balance` — partner balance per currency
  - Notes:
- [ ] `GET /callback-url` — read registered webhook callback URL
  - Notes:
- [ ] `POST /callback-url` — register/update webhook callback URL
  - Notes:
- [ ] `GET /subscribed-events` — read subscribed webhook event types
  - Notes:
- [ ] `POST /subscribed-events` — update subscribed webhook event types
  - Notes:
- [ ] `GET /ip-white-list` — read IP allowlist
  - Notes:
- [ ] `POST /ip-white-list` — update IP allowlist (full replacement)
  - Notes:
- [ ] `GET /deposit-address` — legacy/common shared deposit address lookup by currency+network (not the per-customer `/deposit-address/*` API)
  - Notes:

## Phase 1 — Customer Onboarding

- [ ] `POST /customer/create` — create customer, triggers KYC
  - Notes:
- [ ] `POST /customer/createWithFiles` — create customer with file upload (KYC docs)
  - Notes:
- [ ] `GET /customer/detail` — fetch by `id`, `email`, or `externalId`
  - Notes:
- [ ] `GET /customer/list` — paginated customer list
  - Notes:

## Phase 2 — Fiat Rails (Virtual Accounts & Bank Accounts)

- [ ] `POST /virtual-account/create` — create VA (AUD/EUR/GBP/SGD/USD)
  - Notes:
- [ ] `GET /virtual-account/list` — paginated VA list
  - Notes:
- [ ] `GET /virtual-account/detail` — VA banking details (`payId`/`iban`/`bicSwift`/`accountNumber`)
  - Notes:
- [ ] `POST /virtual-account/create-aud-whitelist` — AUD-only sender whitelist
  - Notes:
- [ ] `POST /virtual-account/update-aud-whitelist` — update AUD whitelist entry
  - Notes:
- [ ] `GET /virtual-account/get-aud-whitelist` — read AUD whitelist
  - Notes:
- [ ] `POST /bank-account/add` — add customer payout bank account
  - Notes:
- [ ] `GET /bank-account/list` — paginated bank account list
  - Notes:
- [ ] `GET /bank-account/detail` — bank account detail by `id`
  - Notes:

## Phase 3 — Crypto Rails (Crypto Addresses & Deposit Addresses)

- [ ] `POST /crypto-address/add` — register customer's external wallet (onramp settlement target)
  - Notes:
- [ ] `GET /crypto-address/list` — paginated crypto address list
  - Notes:
- [ ] `GET /crypto-address/detail` — crypto address detail by `id`
  - Notes:
- [ ] `POST /deposit-address/create` — Coinut-controlled deposit address (offramp entry)
  - Notes:
- [ ] `GET /deposit-address/list` — paginated deposit address list
  - Notes:
- [ ] `GET /deposit-address/detail` — poll until `status` leaves `PENDING` / `address` populated
  - Notes:

## Phase 4 — Autoramp Flows (Onramp / Offramp)

- [ ] `POST /autoramp-flow/onramp` — bundled onramp flow (creates/reuses VA automatically)
  - Notes:
- [ ] `POST /autoramp-flow/offramp` — bundled offramp flow (creates/reuses deposit address automatically)
  - Notes:
- [ ] `POST /autoramp-flow/create` — generic flow creation from an existing entry resource `id`
  - Notes:
- [ ] `GET /autoramp-flow/list` — filter by `type`/`status`
  - Notes:
- [ ] `GET /autoramp-flow/detail` — full entry/exit objects (`fromVirtualAccount`/`fromDepositAddress`, `toBankAccount`/`toCryptoAddress`)
  - Notes:
- [ ] `GET /autoramp-flow/trade-preview` — indicative rate for a currency pair + amount
  - Notes:
- [ ] `POST /autoramp-flow/delete` — delete a flow
  - Notes:

## Phase 5 — Payments

- [ ] `GET /payment/estimate` — payout estimate
  - Notes:
- [ ] `POST /payment/create` — payout from partner balance (`deductCurrency`: USDT/USDC)
  - Notes:
- [ ] `GET /payment/detail` — payment detail by `id`
  - Notes:
- [ ] `GET /payment/list` — paginated payment list
  - Notes:

## Phase 6 — Monitoring & Reconciliation

- [ ] `GET /deposit/list` — filter by `customerId`/`currency`/`status`
  - Notes:
- [ ] `GET /deposit/detail` — single deposit by `id`
  - Notes:
- [ ] `POST /deposit/submitAttachmentFile` — attach compliance doc for `ACTION_REQUIRED` deposit
  - Notes:
- [ ] `POST /deposit/mock` — sandbox-only simulated deposit (accepts EUR/GBP/SGD/USD/AUD/USDT/USDC)
  - Notes:
- [ ] `GET /legacy-deposit/list` — old-flow deposit records (numeric IDs)
  - Notes:
- [ ] `GET /legacy-deposit/detail` — old-flow deposit detail
  - Notes:
- [ ] `GET /trade/list` — OTC trade list
  - Notes:
- [ ] `GET /trade/detail` — trade detail by `id`
  - Notes:

## Phase 7 — Old API (Deprecated)

Only test these if you're still integrated against the legacy module — new integrations should
use the phases above instead.

- [ ] `POST /create-user` — legacy customer/party creation
  - Notes:
- [ ] `GET /get-user` — legacy user/VA lookup
  - Notes:
- [ ] `POST /create-va` — legacy VA creation (requires wallet-ownership `signature` for crypto)
  - Notes:
- [ ] `POST /create-whitelist` — legacy whitelist creation
  - Notes:
- [ ] `POST /update-whitelist` — legacy whitelist update
  - Notes:
- [ ] `GET /get-whitelist` — legacy whitelist read
  - Notes:

---

**Total: 54 endpoints** (10 Common/Setup + 4 Customer + 9 Virtual/Bank Account + 6 Crypto/Deposit
Address + 7 Autoramp Flow + 4 Payment + 8 Deposit/Trade monitoring + 6 Old API).
