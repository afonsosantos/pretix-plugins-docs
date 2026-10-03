# Configuration

## Shared settings

Open the event's **Settings → euPago**. These settings are shared by every euPago payment method, so
you set them once:

| Setting | Description |
|---|---|
| **API key** | Your euPago API key, from Backoffice → Channels → Channel Listing. |
| **Sandbox mode** | Use the euPago sandbox instead of production. Needs a sandbox API key from sandbox.eupago.pt, because production keys don't work there. |
| **Webhook URL** | Read-only: the address to paste into euPago's Backoffice. See [Webhooks](webhooks.md). |
| **Webhook key** | The key euPago generates for this channel's webhook in Backoffice → Channels → Channel Listing → "Receive notification for a URL". **Required** for webhook 2.0: without it, payments aren't confirmed through it. It also decrypts notifications if you turn on euPago's webhook encryption. |

The API key and webhook key are masked once saved. To change one, type the new value over it.

The page warns you when:

- no webhook key is set;
- sandbox mode is on but the event isn't in test mode;
- the event's currency isn't the euro.

!!! warning "Sandbox mode only works in test mode"
    Sandbox payments are fake. To stop a forgotten sandbox setting from handing out real tickets for
    free, Multibanco and MB WAY are only offered at checkout while the event is in **test mode**.
    Turn sandbox mode off before going live.

## Payment methods

Enable and configure each one individually under **Settings → Payment**.

### Multibanco

| Setting | Description |
|---|---|
| **Multibanco reference validity (days)** | How many days a generated reference stays valid (1–365). Never longer than the order's own payment deadline. |

The reference is capped at the order's payment deadline. Otherwise a buyer could pay an old reference
after the order had expired and its tickets were sold to someone else.

The reference is e-mailed to the buyer right after it's generated, in a dedicated follow-up e-mail in
the buyer's language. That e-mail also links back to the order page. pretix's own "order placed" e-mail
goes out *before* the reference exists, so it can't carry it.

### MB WAY

No method-specific settings. At checkout the buyer enters a Portuguese mobile number: 9 digits starting
with 9, with or without `+351`. Foreign numbers are rejected. The buyer then has **5 minutes** to
approve the payment in the MB WAY app.

If euPago refuses the request, e.g. because the number isn't registered with MB WAY, the buyer is told
right away and can try another number or method. If the request expires or is rejected in the app, the
buyer can pay again from their order page.

!!! note
    The buyer's phone number is redacted from the payment record when pretix shreds personal data.
