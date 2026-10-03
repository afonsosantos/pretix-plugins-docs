# Webhooks & refunds

## Webhook URL

Payments are confirmed by euPago calling back into pretix. Configure **one** URL in euPago's Backoffice
(Channels → Channel Listing → "Receive notification for a URL") for every channel you use, and choose
**webhook 2.0**:

```
https://<your-pretix-domain>/eupago/webhook/
```

The exact URL is shown on the event's **Settings → euPago** page. It's global, not per event, and
handles both euPago webhook formats:

| Format | Verified against |
|---|---|
| v1.0 (GET) | your configured API key |
| v2.0 (POST) | your **webhook key** (signature, or decryption if encryption is on) |

Copy the channel's webhook key from Backoffice into the **Webhook key** setting. Without it, v2.0
"paid" and "refunded" notifications are **refused**. Otherwise anyone who knows their own order code
could mark it as paid. Cancellations and expiries are still accepted without the key, since at worst
they fail a payment the buyer can retry.

The optional "encrypt" webhook setting in Backoffice is supported too. The payload is decrypted with
the same key, and only accepted for payments of the event that key belongs to.

!!! warning
    Without a working webhook, payments are never confirmed and orders stay pending until they
    expire.

## Payments made after a cancellation

If a buyer switches to another payment method, pretix cancels the pending Multibanco payment. The
reference itself stays payable at euPago until it expires. If the buyer pays it anyway, the payment is
still confirmed in pretix, so the money is never lost track of.

If the order expired and its tickets were sold in the meantime, pretix can't mark it paid. The payment
is recorded and the pretix log shows a `quota is exceeded` error, so you can sort it out with the
buyer.

## Cancellations, errors and expiry

A v2.0 `Cancel`, `Error` or `Expired` notification fails the pending payment. For example, the buyer
rejected the MB WAY request, the request failed, or a Multibanco reference expired. The buyer can then
pay again from their order page. A stray notification arriving *after* the payment was confirmed is
ignored.

## Refunds

Refunds **cannot be started from pretix**. Issue them in euPago's Backoffice. When euPago then sends a
`Refund`/`Refunded` notification, the plugin records an external refund on the pretix payment.

euPago's notification carries no refund amount, so it's always recorded as a **full** refund, and only
once per payment. Webhook retries don't double-refund. A refund notification that arrives before the
payment's own confirmation is answered with a retry request, so euPago resends it afterwards.

If you also use [pt-invoicing](../pt-invoicing/index.md), that full refund triggers a credit note
automatically.
