# Transaction status - Iron

[Skip to main content](https://docs.iron.xyz/transaction-status#content-area)

## Transaction Status Mapping

| **State Type** | **API Status** | **Dashboard Status** | **Description** |
| --- | --- | --- | --- |
| Progress | `FundsReviewInProgress` | **Reviewing Funds** | The deposit has been received and is undergoing a brief compliance and eligibility check (e.g., AML screening). This status is rare and typically resolves quickly. |
| Progress | `ConversionInProgress` | **Exchanging Funds** | The deposit has cleared review and is being converted to the configured destination asset. |
| Progress | `PayoutInProgress` | **Processing Payout** | Iron has sent the funds to the connected banking provider or blockchain. This step can take time depending on external networks. |
| Terminal | `Completed` | **Transaction Complete** | The funds have been fully processed and handed off to the destination. No further actions will occur. |
| Terminal | `Failed` | **Deposit Failed** | The deposit or transfer failed due to a general processing error. |
| Terminal | `RejectedAml` | **Rejected by Compliance** | The deposit failed AML or compliance checks and has been rejected. |
| Terminal | `RejectedFraud` | **Rejected by Fraud** | The deposit failed anti-fraud review and has been rejected. |
| Terminal | `RejectedMinAmount` | **Deposit Below Minimum** | The amount was too low to process and has been rejected. |

## Settlement Times by Payment Method

The time it takes for a transaction to complete depends on the payment rail used. Below are typical settlement times from initiation to completion:

| **Payment Method** | **Typical Settlement Time** | **Notes** |
| --- | --- | --- |
| **SEPA (Instant)** | Near-instant to 10 seconds | For transactions under €100k within the EEA |
| **SEPA (Credit)** | 1-2 business days | Standard for transactions over €100k; usually settles within 24 hours |
| **ACH** | 2-3 business days | Common for US domestic transfers; funds typically available on the third business day |
| **Wire** | Same day to 1 business day | Faster option for US domestic transfers; often completes within hours if initiated early in the business day |
| **CHAPS** | Same business day | UK domestic transfers; must be initiated before the daily cutoff time |
| **FPS** | Near-instant to 2 hours | UK Faster Payments; typically completes within minutes |

> Settlement times are estimates based on typical payment rail processing. Actual times vary depending on:
>
> - Banking partner processing schedules
> - Compliance and AML review requirements (see `FundsReviewInProgress` status)
> - Time of day and business day cutoffs
> - External network delays
>
> For real-time status updates, monitor the transaction status via the API or webhook notifications.

## API Endpoint

**GET** `/api/autoramp-transactions/ids`

Use this endpoint to retrieve the latest status of specific transactions (up to 100 at a time). Useful for syncing or polling real-time transaction states. At least one transaction ID is required.

## Webhook Notifications

Refer to the [Iron Webhooks Guide](https://docs.iron.xyz/webhooks) for status-based notifications.

Was this page helpful?

[Previous](https://docs.iron.xyz/quotes)[Next](https://docs.iron.xyz/limits-and-minimum)
