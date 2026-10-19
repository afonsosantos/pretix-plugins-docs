# Moloni

[Moloni](https://moloni.pt) provider — identifier `moloni`. Issuing invoice-receipts and credit
notes, and downloading their PDFs, has been verified against a real Moloni account.

## Setting up Moloni

### In Moloni

1. In Moloni's developer area, create an application. Note its **Developer ID** (client_id) and
   **Client secret**.
2. Register the plugin's **redirect URI** on that application. It is shown (with a **Copy** button)
   on the plugin's settings page and is the same for every event:
   `https://<your pretix domain>/control/ptinvoicing/moloni/callback/`.
3. In the company you'll invoice from, make sure you have a document series for
   **invoice-receipts** and a separate one for **credit notes** (a Moloni series is bound to one
   document type), plus at least one payment method and one maturity date ("Pronto pagamento").

### In pretix

Open the event's **Settings → Invoicing (PT)** and click the **Moloni** card.

![The top of the Moloni section: connection state, redirect URI, developer ID and client secret](img/settings-moloni.png)

1. Fill in the **Developer ID (client_id)** and **Client secret**.
2. Press **Connect to Moloni**. You're sent to Moloni to authorize the application, then brought
   back to this page. Whatever you had typed is saved on the way; there's no need to save first.
   Once back, the panel shows **Connected** with **Reconnect to Moloni** and **Disconnect** buttons.
3. Pick the **Company**. With several companies in the account you must pick one explicitly; each
   option shows the name and NIF. The remaining dropdowns load for that company.
4. Pick the **Document set** for invoice-receipts and the **Credit note document set**. They must
   be different series, the latter created for the *Credit Note* document type.
5. Pick the **VAT rate** pretix charges on the tickets, and an **Exemption reason** if any ticket
   is sold at 0%. See [Taxes](#taxes).
6. Pick the default **Payment method**, a **Maturity date**, and the **Product category**,
   **Product type** and **Unit** that the plugin's catalog products are created with.

    ![The Moloni section's dropdowns, from the document sets to the maturity date](img/settings-moloni-fields.png)

7. Optionally map each pretix payment method to a Moloni one in the **Payment methods** table. See
   [Payment methods](#payment-methods).

    ![The payment methods table at the bottom of the Moloni section, with the Save button](img/settings-moloni-payment-methods.png)

8. Press **Save**. Until you do, the section says *Not in use yet* and nothing is issued.

## The connection

**Connect to Moloni** uses Moloni's authorization-code flow: no Moloni password is stored, only the
access and refresh tokens. The refresh token expires after **14 days without use**, so the plugin
refreshes it daily for every event connected to Moloni, and a quiet event never loses the
connection.

If Moloni refuses a refresh, the connection is marked **expired**: the settings page says so, the
order panels warn that nothing can be issued, and the event's **contact e-mail** (pretix
**Settings → General**) gets a message with a link to reconnect. Press **Reconnect to Moloni**.

**Disconnect** asks for confirmation first, since without a connection no document can be issued
for the event.

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
| **Moloni username / password** | no | Only without **Connect to Moloni**; see above. |
| **Company** | yes | With several companies in the account you must pick one explicitly. Each option shows the name and NIF. |
| **Document set** | yes | Series used for invoice-receipts. |
| **Credit note document set** | yes | A series created for the **Credit Note** document type. It can't be the invoice-receipt's series. |
| **VAT rate** | for taxed tickets | Must be the same rate pretix charges; see [Taxes](#taxes). |
| **Exemption reason** | for 0% tickets | Moloni's exemption codes, e.g. `M07` (Artigo 9.º do CIVA). |
| **Payment method** | yes | Invoice-receipts always carry a payment; this is the method it's recorded under when a pretix payment method has no mapping of its own. |
| **Product category** | yes | Where the plugin creates its catalog products; see [Products](#products). Only top-level categories are listed. |
| **Product type** | yes | *Service* (default) or *Product*. |
| **Unit** | yes | The products' unit of measure. |
| **Maturity date** | yes | Set on each new Moloni customer, e.g. "Pronto pagamento". |
| **Payment methods** (table) | no | One row per payment method the event accepts; see [Payment methods](#payment-methods). |

## Payment methods

Moloni records a payment on every invoice-receipt. The **Payment methods** table lists each payment
method the event accepts (Multibanco, MB WAY, bank transfer, box office…) and lets you pick the
Moloni payment method it is recorded under. A row left at *default* uses the **Payment method**
setting above.

An order paid through several methods gets one payment entry per method, with its amount. An order
marked as paid by hand in the Control panel is recorded under the method of its latest other payment
attempt (a Multibanco reference that was paid but never confirmed automatically counts as
Multibanco).

## Products

Moloni only invoices products from its catalog, so each pretix product gets one, created
automatically on its first sale: in the chosen category, type and unit, with the reference
`pretix-item-<id>`. Each fee type (payment fee, shipping…) gets one too, `pretix-fee-<type>`. They
are reused from then on, so Moloni's sales reports stay per ticket type. Each line on the document
still shows the pretix product name and the price actually paid.

## Taxes

Each line follows the VAT pretix actually charged on it:

- **Taxed line** — sent with the configured **VAT rate**. If that rate isn't the one pretix charged,
  issuance stops with `VAT mismatch: …` *before* anything is created in Moloni. Otherwise the
  document would be for a different amount than the buyer paid.
- **0% line** — sent without tax, with the **Exemption reason** instead. Moloni requires one, so an
  order with 0% lines and no exemption reason set stops with an explicit error.

!!! note "Unit prices on the PDF"
    Moloni stores unit prices to 5 decimals and prints the gross unit price from them. A
    tax-inclusive €35.00 at 23% therefore prints as 34,99999 at best, because 35 / 1.23 never
    ends. The incidence, VAT and total are right (28,46 + 6,54 = 35,00, the same split pretix's own
    invoice shows). Only prices whose net is whole cents (pretix's "price includes tax" off) print
    clean.

## Customers

A buyer with a NIF is matched to an existing Moloni customer when exactly one has that NIF. If
several do, issuance stops (`Moloni has N customers with tax number X. Merge them in Moloni, then
retry.`). Otherwise a customer is created from the order's invoice address. Buyers without a NIF
share one final-consumer customer (`999999990`), matched by name.

A reused customer is **never overwritten**. When the order's address differs from the one Moloni
has on file, the document is issued with an alternate address instead, so the PDF shows the address
the buyer gave on this order. Countries are looked up in Moloni by ISO code.

## No automatic retry

Moloni has no idempotency field, so a request that timed out may or may not have created a document.
To avoid issuing a second official invoice, the plugin **never retries Moloni automatically**: a
failed attempt is left as an error reading *the document may or may not have been created. Check in
Moloni before retrying*. Look for the order in Moloni; if nothing was created, press **Retry**.

## Credit notes

Built from the original document's line items and associated to it, crediting it in full. They use
the **Credit note document set**.

## Documents and numbers

The number shown in pretix (e.g. `FR M2026/20`, `NC M2026/6`) is read back from Moloni right after
issuance: the document type's SAF-T code, the series name and the number.
