# Known limitations

**One VAT rate per event.**
Every line is invoiced at the single configured rate. An event selling items at different rates fails
issuance (visibly) rather than mis-invoicing. A per-rate mapping is not implemented yet. Moloni can mix
in 0% lines, which it invoices with the exemption reason instead.

**Moloni product categories: top level only.**
The **Product category** dropdown lists only Moloni's top-level categories.

**Partial refunds have no credit note.**
Both providers can only credit a document's full value. A partial refund issues nothing until refunds
reach the full amount; you can issue a full credit note by hand.

**Duplicate client records stop issuance (Fact.pt).**
If a NIF matches several clients in Fact.pt, clean them up in the Backoffice and retry.

**Fact.pt client data is overwritten.**
When a client is created/updated from an order, the order's address replaces the one on file for that
NIF.

**Switching providers mid-event doesn't re-issue.**
Old documents stay with the provider that issued them, and can no longer be downloaded through pretix.

**No bulk issuance.**
Issuance by hand is one order at a time, on purpose. For a backlog, loop the task from a shell:

```python
# python -m pretix shell
from django_scopes import scopes_disabled
from pretix.base.models import Order
from pretix_ptinvoicing.tasks import issue_invoice

with scopes_disabled():
    for order in Order.objects.filter(event__slug="my-event", status=Order.STATUS_PAID):
        issue_invoice.apply(kwargs={"order_pk": order.pk, "event_pk": order.event.pk})
```

Orders already issued successfully are skipped.
