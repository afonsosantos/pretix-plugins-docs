# pretix-pt-invoicing

Issues AT-certified **Fatura-Recibo** (invoice-receipt) documents through a Portuguese electronic
invoicing provider, automatically once a pretix order is paid, and a **credit note** once it's fully
refunded. A Control-panel dashboard tracks every issuance and lets you retry the ones that failed.

![The order page's Invoicing (PT) panel with an issued invoice and credit note](img/order-panel.png)

- PyPI: <https://pypi.org/project/pretix-pt-invoicing/>
- Source: <https://github.com/afonsosantos/pretix-pt-invoicing>
- Issues: <https://github.com/afonsosantos/pretix-pt-invoicing/issues>
- Requires `pretix>=2024.1.0`, Python ≥ 3.11
- Documented version: **0.7.1**

New here? Follow [Getting started](getting-started.md), a step-by-step walkthrough with screenshots.

## Providers

One provider is active per event.

| Provider | Identifier | Status |
|---|---|---|
| [Fact.pt](factpt.md) | `factpt` | Verified against a real sandbox account |
| [Moloni](moloni.md) | `moloni` | Verified against a real account (invoice-receipts, credit notes, PDFs) |

!!! warning "Before production"
    Issue a few test documents in your provider's sandbox and check the amounts, client and tax
    data before turning this on for a live event. See [Known limitations](limitations.md).

## What it does

- **Invoice-receipt on payment**, credit note on full refund, both issued in the background so a
  slow provider never delays the buyer.
- **Every line of the order**: tickets, with identical ones grouped, and every fee; optionally one
  line per attendee. Net prices are computed so the document total equals what was paid.
- **The buyer's NIF** from everyone, not only businesses, through a single NIF field at checkout.
- **The buyer gets the PDF** on their order page and, if you turn it on, by e-mail.
- **A dashboard and a per-order panel** with status, document number, errors, retries, and
  confirmation pages before anything irreversible.
- **Manual-only mode**, for events where only some orders need a document.
- **Billing-change alerts** when a buyer edits their address after being invoiced; nothing is
  re-issued without staff approval.
- **Moloni**: connect with OAuth, map each pretix payment method to a Moloni one, catalog products
  created automatically.

## How it works

1. An order is paid, and pretix fires `order_paid`.
2. The plugin queues a Celery task. The provider is never called during checkout.
3. The task issues the invoice-receipt and records the result (document number, PDF link, or the
   provider's error).
4. Optionally, the PDF is e-mailed to the buyer and offered for download on their order page.

Issuance is idempotent: each order has a stable identifier, and a successful issuance is never
repeated. An order paid again after its invoice was credited (a refund, then a new payment) gets a
new invoice; see [Paid again after a refund](usage.md#paid-again-after-a-refund).

## Installation

```bash
pip install pretix-pt-invoicing
python -m pretix migrate
```

Restart pretix and enable **Portuguese invoicing** under the event's **Settings → Plugins**,
**Integrations** tab. Then [configure it](getting-started.md).
