# Writing a provider

Providers are discovered automatically: write a package under
`pretix_ptinvoicing/providers/<name>/` containing a subclass of
`pretix_ptinvoicing.providers.base.InvoiceProvider`. Nothing to register, no migration.

```
pretix_ptinvoicing/providers/acme/
├── __init__.py   # settings form + provider class
├── client.py     # thin HTTP wrapper
├── payload.py    # pretix Order → request body (pure functions)
└── urls.py       # optional: the provider's own views (e.g. an OAuth callback)
```

## The contract

| Member | Purpose |
|---|---|
| `identifier` / `verbose_name` | Stored settings value and admin-facing label. |
| `settings_form_class` | A pretix `SettingsForm`. Prefix field names by hand (`acme_api_key`) — pretix event settings are one flat namespace. |
| `deduplicates_issuance` | `True` only if the API itself rejects a second document for the same `identifier_id`. |
| `is_configured` | `False` while not set up; issuance then skips silently. |
| `issue(order, identifier_id)` | Issue the invoice-receipt; return an `IssuedDocument`. |
| `credit(order, document_id, identifier_id)` | Issue a credit note for the full document. |
| `download(document_id)` | Return the PDF bytes. Make sure they *are* a PDF — some APIs link to an HTML download page. |
| `lookups(data)` | Optional. Live dropdowns for the settings page. |
| `document_number(document_id)` | Optional. The number people read (e.g. `FR M2026/20`), built only from what the API returns. Called right after issuance, so it must **never raise**: return `None` on failure. |
| `logo` | Optional. Static path of the logo on the settings page's provider card; without one the card shows `verbose_name`. |
| `lookup_triggers` | Optional. `lookups()` fields whose value changes what `lookups()` returns (Moloni's company). Changing one re-runs the lookup. |
| `settings_template` | Optional. A template rendered at the top of the provider's settings, with the provider as `provider` — for what isn't a form field (Moloni's connect button). |
| `hidden_settings_fields()` | Optional. Settings fields to leave off the page right now. They're dropped from the form, so saving keeps their stored values. |
| `keepalive()` | Optional. Called about once a day for every event using the provider, e.g. to refresh an expiring token. Return `True` if the connection is dead, and the event's contact address is e-mailed to reconnect. |

`self.event` is the event and `self.settings` its settings store (writable).

`IssuedDocument` has `document_id`, `link`, `permanent_url` and `number`. A provider that fills
`number` itself can skip `document_number()`; the bundled ones call it from `issue()` and `credit()`.

The `identifier_id` the core passes is stable per document: a retry gets the same one. An order paid
again after a credited refund gets a new invoice, and with it a new `identifier_id` (`…-r1`, …), so
pass it through to the API's dedup field unchanged.

## Provider URLs

A provider that needs its own views (an OAuth callback, a connect button) lists them in
`providers/<name>/urls.py`. The core adds every provider's `urlpatterns` to the plugin's, so names
reverse as `plugins:pretix_ptinvoicing:<name>` and the core never names a provider. Prefix URL names
with the provider's identifier (`acme_connect`), since they share one namespace. Use the full
Control-panel path, as the core does:

```python title="providers/acme/urls.py"
from django.urls import path

from . import views

urlpatterns = [
    path(
        "control/event/<str:organizer>/<str:event>/invoicing/settings/acme/connect/",
        views.ConnectView.as_view(),
        name="acme_connect",
    ),
]
```

## A minimal provider

A made-up "Acme" API, showing every method.

```python title="providers/acme/__init__.py"
from django import forms
from django.utils.translation import gettext_lazy as _
from pretix.base.forms import SettingsForm

from ...orderdata import bare_tin, client_name
from ..base import InvoiceProvider, IssuedDocument, ProviderError
from .client import AcmeClient


class AcmeSettingsForm(SettingsForm):
    # Field names are the storage keys — namespace them by hand.
    acme_api_key = forms.CharField(
        label=_("API key"), widget=forms.PasswordInput(render_value=True)
    )
    acme_sandbox = forms.BooleanField(label=_("Use sandbox"), required=False)
    acme_tax_id = forms.IntegerField(
        label=_("VAT rate (Acme)"),
        help_text=_("Populated automatically once the API key is valid."),
    )


class AcmeProvider(InvoiceProvider):
    identifier = "acme"
    verbose_name = "Acme Invoicing"
    settings_form_class = AcmeSettingsForm
    deduplicates_issuance = True  # Acme rejects a repeated `external_id`

    @property
    def is_configured(self):
        return bool(self.settings.get("acme_api_key") and self.settings.get("acme_tax_id"))

    def _client(self):
        return AcmeClient(
            api_key=self.settings.get("acme_api_key"),
            sandbox=self.settings.get("acme_sandbox", as_type=bool, default=False),
        )

    def issue(self, order, identifier_id):
        tin = bare_tin(
            order,
            custom_field_is_nif=self.settings.get(
                "ptinvoicing_nif_custom_field", as_type=bool, default=False
            ),
        )
        result = self._client().create_invoice_receipt(
            {
                "external_id": identifier_id,  # the API's dedup field
                "customer": {"name": client_name(order), "vat": tin},
                "lines": [
                    {
                        "description": str(p.item),
                        "quantity": 1,
                        # pretix prices are gross; check what your API expects.
                        "unit_price": str(p.price - p.tax_value),
                        "tax_id": self.settings.get("acme_tax_id", as_type=int),
                    }
                    for p in order.positions.all()
                ],
            }
        )
        return IssuedDocument(
            document_id=str(result["id"]),
            link=result.get("pdf_url"),
            permanent_url=result.get("public_url"),
        )

    def credit(self, order, document_id, identifier_id):
        result = self._client().credit_document(document_id, external_id=identifier_id)
        return IssuedDocument(document_id=str(result["id"]), link=result.get("pdf_url"))

    def download(self, document_id):
        return self._client().download_pdf(document_id)

    def lookups(self, data):
        # `data` is what the admin has typed (prefix stripped), not what's saved.
        api_key = (data.get("acme_api_key") or "").strip()
        if not api_key:
            raise ProviderError(_("No API key provided."))
        client = AcmeClient(api_key=api_key, sandbox=data.get("acme_sandbox") == "true")
        return {
            "acme_tax_id": [
                {"id": t["id"], "label": f"{t['name']} ({t['rate']}%)"}
                for t in client.list_taxes()
            ]
        }
```

!!! tip "Use the shared order helpers"
    `pretix_ptinvoicing.orderdata` has the provider-independent buyer data: `bare_tin` (the NIF,
    without a `PT` prefix, honouring the custom-field setting), `client_name` (company, then name,
    then e-mail), `is_valid_pt_nif`, `one_line(value, limit)` and `FINAL_CONSUMER_NIF`. Never import
    from another provider's package.

## Errors: rejection vs. transient

The task decides what to do from the exception type alone:

| Raised | Meaning | Task behaviour |
|---|---|---|
| `ProviderError` | The provider refused the request | Marked *error*, shown to the admin, **not retried** |
| anything else | Transient failure | Marked *error*, retried 3× (2 min apart) — only if `deduplicates_issuance` |

Wrap API-level rejections in a `ProviderError` subclass, passing the per-field errors as `detail` so
the Control panel shows them:

```python title="providers/acme/client.py"
import requests
from django.utils.translation import gettext_lazy as _

from ..base import ProviderError


class AcmeAPIError(ProviderError):
    pass


class AcmeClient:
    timeout = 30

    def __init__(self, api_key, sandbox=False):
        self.api_key = api_key
        self.base_url = "https://sandbox.acme.test" if sandbox else "https://api.acme.test"

    def _request(self, method, path, payload=None):
        # Network errors are left to propagate: they are transient.
        response = requests.request(
            method,
            f"{self.base_url}{path}",
            json=payload,
            headers={"Authorization": f"Bearer {self.api_key}"},
            timeout=self.timeout,
        )
        if 400 <= response.status_code < 500:
            body = response.json()
            raise AcmeAPIError(
                body.get("message") or _("Rejected by Acme"),
                detail=body.get("errors"),  # e.g. {"customer.vat": "invalid"}
                http_status=response.status_code,
            )
        response.raise_for_status()  # 5xx → transient
        return response.json()

    def create_invoice_receipt(self, payload):
        return self._request("POST", "/invoice-receipts", payload)

    def credit_document(self, document_id, external_id):
        return self._request("POST", f"/documents/{document_id}/credit", {"external_id": external_id})

    def list_taxes(self):
        return self._request("GET", "/taxes")["data"]

    def download_pdf(self, document_id):
        try:
            response = requests.get(
                f"{self.base_url}/documents/{document_id}/pdf",
                headers={"Authorization": f"Bearer {self.api_key}"},
                timeout=self.timeout,
            )
            response.raise_for_status()
        except requests.RequestException as e:
            # download() is called from a view, which only catches ProviderError.
            raise AcmeAPIError(_("Could not download the document: %(e)s") % {"e": e})
        return response.content
```

!!! note
    The bundled Fact.pt and Moloni clients currently wrap network errors in `ProviderError` too, so
    in practice their failures are never retried automatically — only by hand.

!!! danger "`deduplicates_issuance` and retries"
    If your API has no field it dedupes on, set `deduplicates_issuance = False`. A request that timed
    out may still have created the document, and a blind retry would issue a **second official
    invoice**. With the flag off, a failure waits for a human to check and press Retry.

## Refreshing credentials

`self.settings` is writable, so a provider with expiring OAuth tokens can cache them per event
instead of logging in on every issuance:

```python
def _access_token(self):
    token = self.settings.get("acme_access_token")
    expires = self.settings.get("acme_access_expires", as_type=float, default=0)
    if token and expires > time.time() + 60:
        return token
    data = AcmeClient.login(
        self.settings.get("acme_username"), self.settings.get("acme_password")
    )
    self.settings.set("acme_access_token", data["access_token"])
    self.settings.set("acme_access_expires", time.time() + data["expires_in"])
    return data["access_token"]
```

Keep cached tokens out of `settings_form_class` so the settings page doesn't render or overwrite them.

## Testing

Put provider tests in `tests/test_<name>_*.py`, and mock HTTP with
[`responses`](https://github.com/getsentry/responses). Select the provider **and** configure its
credentials, or the task skips the order:

```python title="tests/test_acme.py"
import responses
from django_scopes import scopes_disabled

from pretix_ptinvoicing.models import IssuedInvoice
from pretix_ptinvoicing.tasks import issue_invoice


@responses.activate
def test_issue(event, order):
    event.settings.ptinvoicing_provider = "acme"
    event.settings.acme_api_key = "k"
    event.settings.acme_tax_id = 1
    responses.add(
        responses.POST,
        "https://api.acme.test/invoice-receipts",
        json={"id": 42, "pdf_url": "https://acme.test/42.pdf"},
    )

    issue_invoice.apply(kwargs={"order_pk": order.pk, "event_pk": event.pk})

    with scopes_disabled():
        invoice = IssuedInvoice.objects.get(order=order)
    assert invoice.status == IssuedInvoice.STATUS_SUCCESS
    assert invoice.document_id == "42"


@responses.activate
def test_rejection_is_terminal(event, order):
    event.settings.ptinvoicing_provider = "acme"
    event.settings.acme_api_key = "k"
    event.settings.acme_tax_id = 1
    responses.add(
        responses.POST,
        "https://api.acme.test/invoice-receipts",
        status=422,
        json={"message": "bad", "errors": {"customer.vat": "invalid"}},
    )

    issue_invoice.apply(kwargs={"order_pk": order.pk, "event_pk": event.pk})

    with scopes_disabled():
        invoice = IssuedInvoice.objects.get(order=order)
    assert invoice.status == IssuedInvoice.STATUS_ERROR
    assert invoice.attempts == 1
```
