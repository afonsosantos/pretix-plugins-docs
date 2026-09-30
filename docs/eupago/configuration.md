# Configuration

## Shared settings

Under **Settings → Payment → euPago**. These are shared by every euPago payment method, so you set
them once:

| Setting | Description |
|---|---|
| **API Key** | Your euPago API key, from Backoffice → Channels → Channel Listing. |
| **Sandbox / Test mode** | Use the euPago sandbox instead of production. |
| **Webhook signature secret** | Optional, strongly recommended. The encryption key generated for this channel's webhook in Backoffice → Channels → Channel Listing → "Receive notification for a URL". When set, v2.0 notifications are only accepted if their signature matches. |

## Payment methods

Enable and configure each one individually under **Settings → Payment**.

### Multibanco

| Setting | Description |
|---|---|
| **Multibanco reference validity (days)** | How many days a generated reference stays valid (1–365). |

The reference is e-mailed to the buyer right after it's generated, in a dedicated follow-up e-mail.
pretix's own "order placed" e-mail goes out *before* the reference exists, so it can't carry it.

### MB WAY

No method-specific settings. The buyer enters their phone number at checkout and has **5 minutes** to
approve the payment in the MB WAY app. If the request expires, they can retry the payment from their
order page.

!!! note
    The buyer's phone number is redacted from the payment record when pretix shreds personal data.
