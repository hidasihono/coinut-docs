# Limits and Minimums - Iron

[Skip to main content](https://docs.iron.xyz/limits-and-minimum#content-area)

## Spending Limits

Individual customer limits are calculated dynamically from the income and wealth information collected during KYC (the wealth questionnaire). There is no single fixed limit. The wealth questionnaire is what derives each Tier 1 customer’s personalized cap, so higher declared income produces a higher limit with fewer manual reviews. The limit can be increased to Tier 2 (unlimited) by completing additional due diligence (compliance questionnaire and Source of Funds proof).

Tier 1 limits apply **per direction**, tracked separately for fiat onramp and crypto offramp, measured on a 52-week rolling window from the customer’s most recent transaction in that direction. Business customers (KYB) have no enforced limit.

| Risk Category | Tier (Access Level) | Limit per Direction (52-week rolling) | Required Verification |
| --- | --- | --- | --- |
| Low and medium risk | Tier 1 | Dynamic (based on declared income) | Government-issued ID + Liveness check + KYC Questionnaire + Proof of address (conditional) |
|  | Tier 2 | Unlimited | Government-issued ID + Liveness check + Proof of address + KYC Questionnaire + SoF proof |
| High risk | N/A | Unlimited | Government-issued ID + Liveness check + Proof of address + KYC Questionnaire + SoF proof |

> Proof of address is conditional at Tier 1. Iron requests it only when the customer’s ID document country differs from their residence country (except EU citizens), their residence permit lacks or mismatches address data, or they reside in or connect from a high-risk jurisdiction. At Tier 2 and for high-risk customers, proof of address is always required. See [Proof of Address Rules](https://docs.iron.xyz/kyc#proof-of-address-rules).

> Because Tier 1 limits are personalized per customer, do not hard-code a threshold. Read the live value from [Get customer spending limit usage](https://docs.iron.xyz/reference-sandbox/customer/get-customer-spending-limit-usage) (`threshold_eur` per direction). The per-direction limit is **not** a combined cap: a Tier 1 customer can spend up to their limit on fiat onramp **and** up to their limit on crypto offramp within the same 52-week window before either direction triggers a tier upgrade.

## Checking Remaining Spending Limit

> Poll [Get customer spending limit usage](https://docs.iron.xyz/reference-sandbox/customer/get-customer-spending-limit-usage) to read a customer’s exact `used_eur` and `remaining_eur` per direction. Use it to trigger proactive EDD before they hit a threshold, instead of waiting for a paused transaction.

To read a customer’s live usage, call [Get customer spending limit usage](https://docs.iron.xyz/reference-sandbox/customer/get-customer-spending-limit-usage). The response returns `threshold_eur`, `used_eur`, and `remaining_eur` per direction (fiat on-ramp and crypto off-ramp), in EUR cents as strings. `threshold_eur` and `remaining_eur` are `null` for Business customers and Regulated persons (no enforced cap).

## Proactively Increasing Customer Limits

Partners can proactively upgrade a customer to Tier 2 (unlimited) by initiating Enhanced Due Diligence (EDD) upfront, before the customer hits any spending threshold. This avoids transaction pauses and provides a smoother experience for high-volume customers.

#### How to Trigger EDD

When creating an identification via the [Create Customer Identification V2](https://docs.iron.xyz/reference-sandbox/customer/create-customer-identification-v2) endpoint, you can trigger EDD depending on the identification type:

**Link flow.** Set `with_edd` to `true`:

```
{
  "type": "Link",
  "with_edd": true
}
```

**Token flow.** Include `edd_questionnaire` in the request body. See [SumSub Token Sharing](https://docs.iron.xyz/reliance-kyc-token-sharing#with-kyc--edd-questionnaire) for the full example and field reference.

**Person flow (Outsourcing).** Include `edd_questionnaire` in the Person params. See [Outsourcing](https://docs.iron.xyz/outsourcing#triggering-edd) for details.

#### How It Works

- If the customer already has an approved KYC, Iron will reuse it and only prompt for the additional EDD steps (no duplicate verification required).
- If the customer has not yet completed KYC, the full KYC + EDD flow will be initiated.
- The same identification URL used for regular KYC is used for KYC + EDD, so partners can copy the link from the Partner Dashboard as usual.

Once the identification is created, the `with_edd` field on the [Identification response](https://docs.iron.xyz/onboarding#tracking-edd-status) indicates whether EDD was applied. This field is also set automatically when Iron’s AML checks determine EDD is required.

> Proactive EDD is only available for individual customers. Business customers (KYB) are not supported.

## Handling Transactions Paused Due to Limit Thresholds

Iron sets spending limits per customer to support its risk-based approach to transactions oversight. When a customer reaches their threshold, transactions may be temporarily blocked until further verification is completed.

#### How It Works

When a customer hits their transaction or balance limit:

- The transaction is **paused**.
- The customer’s status moves to `IdentificationRequired`.
- Iron triggers a **tier upgrade**, requiring additional identity verification (including Source of Funds, “SoF”).
- The partner is responsible for providing the customer with the identity link for the tier upgrade.

#### Partner Responsibilities

- Providing the customer with the identity link for the tier upgrade.
- or contacting the end customer and collecting the required documents (e.g., Source of Funds, Source of Wealth) and passing it to Iron compliance team

#### Timing matters:

- If the customer submits the documents **within 24 hours**, Iron reviews, clears the transaction, and may **raise the customer’s limit**.
- If the customer fails to provide the required documents in time, the transaction is **rejected and returned**.

#### Bank-Triggered Blocks

In some cases, a **banking partner** may block a transaction due to compliance concerns. Iron’s compliance team will:

- Contact the partner and request additional customer documentation (typically SoF).
- Optionally, set the customer status to `IdentificationRequired`.
- The partner should assist the customer in providing the documents promptly to avoid transaction rejection.

#### Important Notes

- Iron’s compliance team may apply new limit thresholds after reviewing the submitted documentation.
- Even if the document arrives **after** the 24-hour window, limits may still be raised, but the original transaction will not automatically resume. It must be retried.

## Source of Funds (SoF) Document Requirements

Every document must include: **Customer Name**, **Current Address** (matching onboarding), **Issue Date**, and **Financial Value**.

| Source of Funds | Accepted Documents | Validity |
| --- | --- | --- |
| Salary | Bank statement (2 payments) or payslip | < 3 months old |
| Savings | Bank statement from the holding institution | < 3 months old |
| Inheritance | Signed letter from a licensed solicitor or estate trustee | No expiry |
| Crypto | Wallet/Account screenshot (with name/address); add bank statement for DEX users | < 3 months old |
| Investments | Copy of investment portfolio statement | < 3 months old |
| Company Profits | Dividend distribution certificate | < 12 months old |
| Sole Trader | Bank statement (2 payments) or latest tax return | 3 months for bank / 12 months for tax |
| Pension | Pension statement or bank statement (2 payments) | < 12 months old |
| Rental Income | Signed Rental Agreement | Must be valid for 3+ months |
| Property Sale | Copy of the signed sale contract | No expiry |
| Company Sale | Sale contract + screenshot of Registry (showing change of ownership) | No expiry |
| Benefits | Government document + bank statement (2 deposits) | < 3 months old |

## Minimum Transaction Amount

The minimum supported amount depends on the payout method. Any request below the applicable threshold is rejected and not executed, because processing and network costs would exceed the transferred value.

| Transaction type | Minimum amount |
| --- | --- |
| Standard onramp and offramp | **1** unit of the payout currency (e.g. 1 EUR, 1 USD) |
| SWIFT / wire payouts | **5** units of the payout currency |
| Stablecoin mint and redeem | **10** units of the input currency |

When a deposit arrives but the resulting payout would fall below the applicable minimum, the deposit is returned to the customer.

> Incoming crypto deposits worth less than `1 EUR` (or equivalent) are treated as dust. They are not processed and are not returned.

We recommend that partners clearly communicate these minimums to their end users, especially during onboarding or when guiding them through their first test transaction. This helps avoid confusion and unnecessary troubleshooting.

Was this page helpful?

[Previous](https://docs.iron.xyz/transaction-status)[Next](https://docs.iron.xyz/webhooks)