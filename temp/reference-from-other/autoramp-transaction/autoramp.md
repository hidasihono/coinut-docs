# Autoramp - Iron

[Skip to main content](https://docs.iron.xyz/autoramp#content-area)

Use Autoramp to convert funds between rails: fiat, crypto, or across blockchains.

When you create an Autoramp, you define a source asset (for example, EUR in a bank account) and a destination asset (for example, USDC on Solana). You then send funds to the designated source address, and Autoramp automatically exchanges them into the target asset, delivering them to your self-custody wallet or specified bank account.

You can create and manage Autoramps using our API or through the Partner Dashboard. By using this conversion “primitive,” you can build the ideal product experience for your customers, eliminating manual steps and simplifying on-ramping, off-ramping, or cross-chain swaps.

## How Autoramp Works

1

Create a rule

Specify the source and destination assets for your conversion. For example, convert EUR to USDC, or USDC to EUR.

2

Share the address

Provide the associated payment instructions (like an IBAN or wallet address) to your customers or teams.

3

Send funds

When funds arrive in the source asset, Autoramp automatically converts them into the destination asset and forwards them to the specified account or wallet.

You can treat each Autoramp as a “standing order.” After setup, Autoramp continuously monitors all incoming transactions for the specified source asset and performs the conversion without additional steps.

## Common Use Cases

- **On-ramp:** Receive EUR in a bank account and convert it instantly to crypto (for example, USDC on Solana).
- **Off-ramp:** Convert crypto assets (for example, USDC or other stablecoins) directly to fiat and deposit the resulting funds into a linked bank account.
- **Cross-chain swap:** Convert an incoming stablecoin on one chain into another stablecoin or token on a different chain.
- **Automatic payouts:** Provide a single wallet or payment address. Autoramp handles the conversions automatically, so you don’t need to track or execute manual swaps each time.

## Create an Autoramp

You can create an Autoramp in your **Partner Dashboard** or by using our **API**.

1

Go to Autoramp in the Dashboard

Navigate to the Autoramp section. Alternatively, use the API endpoint to create and manage Autoramps programmatically.

2

Select New Autoramp

Choose your source and destination assets.

3

Enter source details

Choose your fiat currency or crypto asset, and provide the bank account information or wallet address where incoming funds will arrive.

4

Enter destination details

Choose your target asset and specify the bank account or wallet address where you want to receive converted funds.

5

Save and retrieve your address

If you selected a fiat source, copy the IBAN or payment instructions. If you selected a crypto source, copy the unique deposit address.

6

Test your Autoramp

Send a small test transaction to confirm that funds are converted and delivered as expected.

> You can create multiple Autoramps for different assets. Each Autoramp has its own unique source information (IBAN or crypto address) and destination.

### API: optional pay by reference

Set `pay_by_reference: true` on `POST /autoramps` (either standard parameters or a signed quote from `GET /autoramps/quote`) to share a single virtual account across multiple Autoramps for the same customer. Each PBR Autoramp returns an `inbound_payment_reference` of the form `IR-XXXXXX` that senders must include verbatim on every inbound transfer for it to route correctly.

PBR is currently supported only when all of the following hold:

- **Kind**: `Onramp` or `Mint`
- **Deposit account type**: `Iban`, `AchWire`, `Swift`, or `ChapsFps` (Pix, SPEI, African bank transfer, and mobile money are not supported yet)
- **Source currencies**: include at least one fiat currency

A PBR-enabled virtual account is exclusive on the inbound side: every deposit must carry a matching reference, otherwise it is returned to sender. When you create a PBR Autoramp, Iron will not reuse a virtual account that has a live non-PBR Autoramp; you’ll either get a fresh VA or share one with another PBR Autoramp for the same customer.

### API: attach quotes over time

A quote-source Autoramp can hold multiple active quotes at once. After creation, request a new signed rate quote with `GET /autoramps/{autoramp_id}/quote` and attach it via `POST /autoramps/{autoramp_id}/quotes`. Inbound deposits match the newest active quote whose `amount_in` value and source currency match the deposit. See [Quotes](https://docs.iron.xyz/quotes#attach-a-new-quote-to-an-existing-autoramp).

## Manage Your Autoramps

View, edit, or disable Autoramps in the Dashboard. Each Autoramp entry shows:

- **Source asset and address** (IBAN or crypto wallet)
- **Destination asset and address**
- **Recent conversion activity**
- **Status** (active or disabled)

> Disable an Autoramp if you temporarily don’t want to accept and convert funds. You can enable it again at any time.

## Important Considerations

- **Wallet registration required:** Every wallet address used as a recipient in an Autoramp must be [registered beforehand](https://docs.iron.xyz/crypto-addresses) for [Travel Rule](https://docs.iron.xyz/travel-rule) compliance. This applies to all flows: onramp, offramp, and swap.
- **Self-custody:** You remain in control of your wallets and funds. Autoramp doesn’t hold your crypto or fiat; it only routes the conversion.
- **Supported assets:** Confirm that the assets you want to convert are supported by your banking or custody providers.
- **Transaction fees:** Autoramp includes any applicable network and processing fees. Verify fees before sending high-volume transactions.
- **Compliance checks:** We perform KYC/AML checks to ensure that source and destination comply with regulations.
- **Timely settlements:** Funds settle in the destination asset shortly after arrival, reducing operational delays.

Was this page helpful?

[Previous](https://docs.iron.xyz/travel-rule)[Next](https://docs.iron.xyz/autoramp-status)
