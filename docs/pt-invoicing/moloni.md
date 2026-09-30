# Moloni

[Moloni](https://moloni.pt) provider — identifier `moloni`.

!!! warning "Not yet verified against a real account"
    This provider was built from Moloni's published API docs, cross-checked against the official
    Moloni WooCommerce plugin's source, and tested against mocks only. In particular, whether Moloni's
    `price` is net or gross is unconfirmed. **Issue test documents and check the totals before using
    it for a live event.**

## Settings

Credentials come from Moloni's developer area.

| Setting | Required | Description |
|---|---|---|
| **Developer ID (client_id)** | yes | |
| **Client secret** | yes | |
| **Moloni username** | yes | |
| **Moloni password** | yes | |
| **Company** | yes | Dropdown, loaded once the credentials are valid. |
| **Document set** | yes | Series used for invoice-receipts. |
| **Credit note document set** | yes | A series created for the **Credit Note** document type. It can't be the invoice-receipt's series. |
| **VAT rate** | no | Dropdown of your account's taxes. |
| **Exemption reason** | no | Required on 0% lines, e.g. `M07` (Artigo 9.º do CIVA). |
| **Payment method** | yes | Invoice-receipts always carry a payment; this is the method it's recorded under. |

The access token is cached in the event settings and refreshed automatically.

## No automatic retry

Moloni has no idempotency field, so a request that timed out may or may not have created a document.
To avoid issuing a second official invoice, the plugin **never retries Moloni automatically**: a failed
attempt is left as an error for you to check in Moloni and retry by hand from the Control panel.

## Credit notes

Built from the original document's line items and associated to it, crediting it in full.
