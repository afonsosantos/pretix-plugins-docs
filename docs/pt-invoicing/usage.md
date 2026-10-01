# Issuing & credit notes

## Automatic issuance

Once the plugin is configured, every order that becomes **paid** gets an invoice-receipt, issued in the
background. Unpaid orders are never issued, whatever triggers it.

!!! tip
    Issuance runs on pretix's Celery workers. If Celery isn't running, tasks are only queued.

## Dashboard

The **Invoicing (PT)** entry in the event's sidebar lists every issuance attempt with its kind
(invoice / credit note), provider, status, document number and error, filterable by status. Each
successful row has a PDF download; each failed one has a **Retry** action.

The document number is the one printed on the document, e.g. `FR M2026/20` for Moloni or `2025QG/9`
for Fact.pt (shown as Fact.pt returns it).

## The order page panel

Every order in the Control panel gets an **Invoicing (PT)** panel, with one row per document — every
invoice and credit note the order ever had, oldest first:

- **Issue invoice now / Retry issuance** — for a paid order without a successful invoice. Use it for
  orders paid before the plugin was configured, orders marked paid by hand, after fixing whatever
  the provider rejected, or for an order [paid again after a refund](#paid-again-after-a-refund).
- **Issue credit note / Retry credit note** — available once the invoice was issued successfully.

## Two kinds of failure

| Failure | Example | What happens |
|---|---|---|
| **Rejected by the provider** | invalid NIF, VAT mismatch, bad token | Marked *error* with the provider's message. Not retried — fix the data or settings, then press Retry. |
| **Network / provider down** | timeout, unreachable | Also marked *error*. The bundled providers report these like rejections, so they're not retried automatically either — press Retry once the provider is back. |

## Credit notes

A credit note is issued **automatically** once an order's refunds add up to the full amount paid —
whether that takes one refund or several. It always credits the original invoice **in full**.

A **partial** refund that doesn't bring the order to zero issues nothing, because neither provider can
credit less than a whole document. You decide: issue a full credit note by hand from the order page,
or leave it. See [Known limitations](limitations.md).

## Paid again after a refund

An order can be paid, refunded and then paid again. Once the first invoice has been credited, the new
payment gets a **new invoice**, issued automatically like the first. A later full refund credits that
new invoice, never one that's already credited. The order panel keeps all of them, oldest first.

If the new payment arrived before the credit note was issued, nothing is issued for it on its own.
Use **Issue invoice now** on the order page once the credit note exists.

## Buyer-facing

- **Order page** — a download button beside the ticket downloads, once the document exists (if
  enabled). Authenticated by the order's secret link, like pretix's own downloads.
- **E-mail** — the PDF as an attachment, if enabled in [settings](configuration.md#e-mail). Issuing
  by hand from the Control panel sends it too.
