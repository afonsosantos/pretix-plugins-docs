# pretix-pt-invoicing

Issues AT-certified **Fatura-Recibo** (invoice-receipt) documents through a Portuguese electronic
invoicing provider, automatically once a pretix order is paid — and a **credit note** once it's fully
refunded. A Control-panel dashboard tracks every issuance and lets you retry the ones that failed.

- PyPI: <https://pypi.org/project/pretix-pt-invoicing/>
- Source: <https://github.com/afonsosantos/pretix-pt-invoicing>
- Issues: <https://github.com/afonsosantos/pretix-pt-invoicing/issues>
- Requires `pretix>=2024.1.0`, Python ≥ 3.11

## Providers

One provider is active per event.

| Provider | Identifier | Status |
|---|---|---|
| [Fact.pt](factpt.md) | `factpt` | Verified against a real sandbox account |
| [Moloni](moloni.md) | `moloni` | Built from the published API docs and the official Moloni plugin's source; **not yet run against a real account** |

!!! warning "Before production"
    Issue a few test documents in your provider's sandbox and check the amounts, client and tax
    data before turning this on for a live event. See [Known limitations](limitations.md).

## How it works

1. An order is paid → pretix fires `order_paid`.
2. The plugin queues a Celery task — the provider is never called during checkout, so a slow provider
   can't delay the buyer.
3. The task issues the invoice-receipt and records the result (document number, PDF link, or the
   provider's error).
4. Optionally, the PDF is e-mailed to the buyer and offered for download on their order page.

Issuance is idempotent: each order has a stable identifier, and a successful issuance is never
repeated.

## Installation

```bash
pip install pretix-pt-invoicing
python -m pretix migrate
```

Restart pretix and enable **Portuguese invoicing** under the event's **Settings → Plugins**. Then
[configure it](configuration.md).
