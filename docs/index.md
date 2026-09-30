# pretix plugins for Portugal

Two independent [pretix](https://github.com/pretix/pretix) plugins for selling tickets in Portugal.

| Plugin | What it does | PyPI |
|---|---|---|
| [**pretix-eupago**](eupago/index.md) | Accept **Multibanco** references and **MB WAY** payments through [euPago](https://eupago.pt). | [`pretix-eupago`](https://pypi.org/project/pretix-eupago/) |
| [**pretix-pt-invoicing**](pt-invoicing/index.md) | Issue AT-certified **Fatura-Recibo** invoices (and credit notes) through [Fact.pt](https://fact.pt) or [Moloni](https://moloni.pt) once an order is paid. | [`pretix-pt-invoicing`](https://pypi.org/project/pretix-pt-invoicing/) |

They work on their own or together: euPago confirms the payment, pretix marks the order paid, and
pt-invoicing issues the invoice-receipt.

## Installing a plugin

Both install the same way — into the same Python environment as your pretix instance:

```bash
pip install pretix-eupago pretix-pt-invoicing
```

Restart pretix. Plugins register themselves through a setuptools entry point, so there's nothing to
add to `INSTALLED_APPS`. Then run migrations (pt-invoicing ships a model):

```bash
python -m pretix migrate
```

Finally, enable each plugin per event under **Settings → Plugins**.
