# Webhooks & refunds

## Webhook URL

Payments are confirmed by euPago calling back into pretix. Configure **one** URL in euPago's Backoffice
(Channels → Channel Listing → "Receive notification for a URL"), for every channel you use:

```
https://<your-pretix-domain>/eupago/webhook/
```

This URL is global — not per event — and handles both euPago webhook formats:

| Format | Verified against |
|---|---|
| v1.0 (GET) | your configured API key |
| v2.0 (POST) | your webhook signature secret, if set; otherwise a weaker cross-check only |

The optional "encrypt" webhook setting in Backoffice is supported too — the payload is decrypted with
the same secret.

!!! warning
    Without a webhook, payments are never confirmed and orders stay pending until they expire.

## Cancellations and expiry

A v2.0 `Cancel` or `Expired` notification (e.g. the buyer rejected the MB WAY request, or a Multibanco
reference expired) fails the pending payment. A stray notification arriving *after* the payment was
confirmed is ignored.

## Refunds

Refunds **cannot be started from pretix** — issue them in euPago's Backoffice. When euPago then sends
a `Refund`/`Refunded` notification, the plugin records an external refund on the pretix payment.

euPago's notification carries no refund amount, so it's always recorded as a **full** refund, and only
once per payment — webhook retries don't double-refund.

If you also use [pt-invoicing](../pt-invoicing/index.md), that full refund triggers a credit note
automatically.
