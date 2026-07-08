# Autoramp Status - Iron

[Skip to main content](https://docs.iron.xyz/autoramp-status#content-area)

## Autoramp Status Mapping

| State Type | API Status | Description |
| --- | --- | --- |
| Blocking | `Created` | The autoramp has been created and is waiting for authorization via Two-Factor Authentication. Autoramps that require 2fa for creation (disabled by default) start in this state. |
| Authorized | `Authorized` | The autoramp has been authorized and is waiting for the deposit account to be created and verified. By default (2fa for creation disabled), autoramps start in this state. |
| Blocking | `EditPending` | The autoramp has pending edits that require authorization |
| Progress | `DepositAccountAdded` | A deposit account has been added to the autoramp |
| Progress | `Approved` | The autoramp has been approved and deposit account verified |
| Terminal | `Rejected` | The autoramp has been rejected |
| Terminal | `Cancelled` | The autoramp has been cancelled |

`deposit_rails` is populated once a deposit account is attached, at `DepositAccountAdded`. Earlier states return `deposit_rails: []`. The account isn’t verified until `Approved`, so don’t share deposit details with end users until the autoramp reaches `Approved`. Poll `GET /api/autoramps/{autoramp_id}` or subscribe to webhooks to detect the transition.

Was this page helpful?

[Previous](https://docs.iron.xyz/autoramp)[Next](https://docs.iron.xyz/quotes)
