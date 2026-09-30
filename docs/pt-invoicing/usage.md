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

## The order page panel

Every order in the Control panel gets an **Invoicing (PT)** panel, with one row per document:

- **Issue invoice now / Retry issuance** — for a paid order without a successful invoice. Use it for
  orders paid before the plugin was configured, orders marked paid by hand, or after fixing whatever
  the provider rejected.
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

## Buyer-facing

- **Order page** — a download button beside the ticket downloads, once the document exists (if
  enabled). Authenticated by the order's secret link, like pretix's own downloads.
- **E-mail** — the PDF as an attachment, if enabled in [settings](configuration.md#e-mail). Issuing
  by hand from the Control panel sends it too.
