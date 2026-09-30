# pretix-eupago

Integrates [euPago](https://eupago.pt), a Portuguese payment gateway, into pretix, adding two payment
methods:

- **Multibanco** — the buyer gets an entity/reference pair to pay at an ATM or in home banking.
- **MB WAY** — the buyer approves a push notification on their phone.

Both stay *pending* in pretix until euPago confirms the payment through a [webhook](webhooks.md).

- PyPI: <https://pypi.org/project/pretix-eupago/>
- Source: <https://github.com/afonsosantos/pretix-eupago>
- Issues: <https://github.com/afonsosantos/pretix-eupago/issues>
- Requires `pretix>=4.0.0`

## Installation

```bash
pip install pretix-eupago
```

Restart pretix and enable **euPago** under the event's **Settings → Plugins**. Then
[configure it](configuration.md).
