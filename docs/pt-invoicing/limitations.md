# Known limitations

**One VAT rate per event.**
Every taxed line is invoiced at the single configured rate. An event selling items at different
rates fails issuance (visibly, with a *VAT mismatch* error) rather than mis-invoicing. A per-rate
mapping is not implemented yet. Moloni can mix in 0% lines, which it invoices with the exemption
reason instead.

**Partial refunds have no credit note.**
Both providers can only credit a document's full value. A partial refund issues nothing until refunds
reach the full amount; you can issue a full credit note by hand from the order page.

**Fact.pt needs an 8-decimal series.**
Net unit prices are sent with several decimals so that documents add up to the cent. A series with
2 decimal places rejects them, and rounding on Fact.pt's side would make totals drift by cents.
Assign an 8-decimal series to the API key.

**Moloni prints unit prices to 5 decimals.**
A tax-inclusive €35.00 at 23% shows a unit price of 34,99999 on the PDF; the incidence, VAT and
total are right. Only whole-cent net prices (pretix's "price includes tax" off) print clean.

**Moloni product categories: top level only.**
The **Product category** dropdown lists only Moloni's top-level categories.

**Duplicate client records stop issuance.**
If a NIF matches several clients in Fact.pt, or several customers in Moloni, issuance stops with an
explicit error. Merge them in the provider's back office and retry.

**Fact.pt client data is overwritten.**
When a client is created or updated from an order, the order's address replaces the one on file for
that NIF. Moloni customers are never overwritten; the document carries an alternate address instead.

**A Fact.pt document created by a call that timed out isn't linked back.**
The automatic retry is rejected as a duplicate, so no second invoice is issued, but the row stays in
*Error* without a document number. Find the document in Fact.pt's Backoffice.

**Billing-change warnings only cover documents issued since 0.7.**
Older rows have no snapshot of the billing data, so an edit after them never warns.

**Switching providers mid-event doesn't re-issue.**
Old documents stay with the provider that issued them and can no longer be downloaded through pretix,
since the old provider's credentials aren't kept.

**Requiring the VAT ID breaks the single NIF field.**
With **The custom invoice-address field holds the buyer's tax number** on, pretix's own VAT ID field
is hidden for Portuguese addresses, so pretix's **Require VAT ID** option would reject Portuguese
businesses. Leave it off.

**No bulk issuance.**
Issuance by hand is one order at a time, on purpose. For a backlog, loop the task from a shell; see
[Bulk issuance](usage.md#bulk-issuance).
