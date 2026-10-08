# Moloni

[Moloni](https://moloni.pt) provider — identifier `moloni`.

Issuing invoice-receipts and credit notes, and downloading their PDFs, has been verified against a
real Moloni account.

## Connecting to Moloni

At the top of the Moloni settings, **Connect to Moloni** sends you to Moloni to authorize the plugin,
then brings you back to the settings page. No Moloni password is stored, only the access tokens.

1. In Moloni's developer area, register the **redirect URI** shown under the button. It's the same
   for every event: `https://<your pretix>/control/ptinvoicing/moloni/callback/`.
2. Fill in the **Developer ID** and **Client secret**, then press **Connect to Moloni**. There's no
   need to save first.
3. Once connected, the panel shows **Connected** and offers **Reconnect** and **Disconnect**.

Moloni's refresh token expires after **14 days without use**. The plugin refreshes it daily for every
event connected to Moloni, so a quiet event never loses the connection. If Moloni refuses a refresh,
the connection is marked **expired**: the settings page says so, and the event's **contact e-mail**
(pretix **Settings → General**) gets a message with a link to reconnect.

!!! note "Username and password"
    Still accepted as an alternative to **Connect to Moloni**, and hidden while connected. With them
    set, an expired connection logs in again by itself instead of waiting for you to reconnect.

## Settings

Most fields turn into dropdowns filled from your Moloni account once you're connected. The
company-specific ones load after you pick a **Company**.

| Setting | Required | Description |
|---|---|---|
| **Developer ID (client_id)** | yes | From Moloni's developer area. |
| **Client secret** | yes | From Moloni's developer area. |
| **Moloni username / password** | no | Only without **Connect to Moloni** — see above. |
| **Company** | yes | With several companies in the account you must pick one explicitly. Each option shows the name and NIF. |
| **Document set** | yes | Series used for invoice-receipts. |
| **Credit note document set** | yes | A series created for the **Credit Note** document type. It can't be the invoice-receipt's series. |
| **VAT rate** | for taxed tickets | Must be the same rate pretix charges — see [Taxes](#taxes). |
| **Exemption reason** | for 0% tickets | Moloni's exemption codes, e.g. `M07` (Artigo 9.º do CIVA). |
| **Payment method** | yes | Invoice-receipts always carry a payment; this is the method it's recorded under. |
| **Maturity date** | yes | Set on each new Moloni customer, e.g. "Pronto pagamento". |
| **Product category** | yes | Where the plugin creates its catalog products — see [Products](#products). Only top-level categories are listed. |
| **Product type** | yes | *Service* (default) or *Product*. |
| **Unit** | yes | The products' unit of measure. |

## Products

Moloni only invoices products from its catalog, so each pretix product gets one, created automatically
on its first sale. Each is created in the chosen category, type and unit, with the reference
`pretix-item-<id>`, and reused from then on, so Moloni's sales reports stay per ticket type. Each line
on the document still shows the pretix product name and the price actually paid.

## Taxes

Each line follows the VAT pretix actually charged on it:

- **Taxed line** — sent with the configured **VAT rate**. If that rate isn't the one pretix charged,
  issuance stops with `VAT mismatch: …` *before* anything is created in Moloni. Otherwise the
  document would be for a different amount than the buyer paid.
- **0% line** — sent without tax, with the **Exemption reason** instead. Moloni requires one, so an
  order with 0% lines and no exemption reason set stops with an explicit error.

## Customers

A buyer with a NIF is matched to an existing Moloni customer when exactly one has that NIF. Otherwise a
customer is created from the order's invoice address. Buyers without a NIF are created as final
consumers (`999999990`).

## No automatic retry

Moloni has no idempotency field, so a request that timed out may or may not have created a document.
To avoid issuing a second official invoice, the plugin **never retries Moloni automatically**: a failed
attempt is left as an error for you to check in Moloni and retry by hand from the Control panel.

## Credit notes

Built from the original document's line items and associated to it, crediting it in full.
