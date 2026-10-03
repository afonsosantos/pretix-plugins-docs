# pretix-eupago

Integrates [euPago](https://eupago.pt), a Portuguese payment gateway, into pretix, adding two payment
methods:

- **Multibanco** — the buyer gets an entity/reference pair to pay at an ATM or in home banking.
- **MB WAY** — the buyer approves a push notification on their phone.

Both stay *pending* in pretix until euPago confirms the payment through a [webhook](webhooks.md).
Both are only offered for events whose currency is the euro.

- PyPI: <https://pypi.org/project/pretix-eupago/>
- Source: <https://github.com/afonsosantos/pretix-eupago>
- Issues: <https://github.com/afonsosantos/pretix-eupago/issues>
- Requires `pretix>=2024.1.0`, Python ≥ 3.11

## Installation

```bash
pip install pretix-eupago
```

Restart pretix and enable **euPago Payments** under the event's **Settings → Plugins**. Then
[configure it](configuration.md).

## The euPago page

The plugin adds a **euPago** entry to the event's sidebar listing every euPago payment for the event.
Each row shows the Multibanco reference or the MB WAY transaction, and why a payment failed (e.g. the
buyer canceled the MB WAY request). You can:

- search by order code or Multibanco reference (with or without the spaces), which helps when a
  buyer calls about a payment;
- filter by method (Multibanco / MB WAY) and payment status.

Use it to check what a buyer was given, or which payments are still waiting on euPago.
