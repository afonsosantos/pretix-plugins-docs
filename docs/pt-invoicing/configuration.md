# Configuration

Everything lives on one page: the event's **Settings → Invoicing (PT)**. (It's labelled "(PT)" to
tell it apart from pretix's own "Invoicing" entry, which configures pretix's internal invoices.) For
a guided first setup, start with [Getting started](getting-started.md); this page is the reference
for each option.

![The Invoicing (PT) settings page: provider cards, general options and the E-mail section](img/settings-general.png)

## General

| Setting | Default | Description |
|---|---|---|
| **Invoicing provider** | none | The provider cards: **No invoicing**, [Fact.pt](factpt.md) or [Moloni](moloni.md). With none, nothing is issued. Only the selected provider's section is shown, saved and used. |
| **Issue documents automatically** | on | Issues the invoice-receipt once an order is paid and the credit note once it's fully refunded. Turn it off for a manual-only mode: nothing is issued unless you use the buttons on an order's page. See [Manual-only issuance](#manual-only-issuance). |
| **The custom invoice-address field holds the buyer's tax number** | off | Collects a NIF from every buyer, not only businesses. See [Tax numbers (NIF)](#tax-numbers-nif). |
| **Let buyers itemize the invoice by attendee** | off | Adds a checkbox to the checkout's contact form. When the buyer ticks it, every ticket gets its own invoice line carrying the attendee's name. See [Invoice lines](#invoice-lines). |
| **Show the invoice on the buyer's order page** | on | Adds download buttons for the invoice and any credit note beside the ticket downloads. The PDF is proxied through pretix, so your provider credentials never reach the browser. |

## E-mail

pretix's own order e-mails can't attach these documents (they only attach pretix's own invoices), so
the plugin sends its own, one per document.

| Setting | Default | Description |
|---|---|---|
| **E-mail the invoice-receipt to the buyer** | off | Sends the PDF once the invoice-receipt is issued. |
| **E-mail the credit note to the buyer** | off | Sends the PDF once the credit note is issued; the text names the invoice it cancels. |

Issuing by hand from the Control panel sends the e-mail too. Whether each document went out is
shown on its row in the order's **Invoicing (PT)** panel (*E-mailed to the buyer* / *Could not be
e-mailed to the buyer*). A failed e-mail is logged and never marks the issuance as failed: the
document already exists at the provider.

## Provider settings

Below the E-mail section comes the selected provider's own section. Each has its own page:

- [Fact.pt](factpt.md#settings): API token, sandbox switch, VAT rate, unit and item type.
- [Moloni](moloni.md#settings): the connection, company, document series, VAT, payment methods and
  product defaults.

Some fields (VAT rate, company, document set…) turn into dropdowns populated live from your provider
account as soon as valid credentials are typed, with no need to save first. Fields that depend on
another one (Moloni's per-company settings) reload when it changes. If a lookup fails, the line
under the provider's section says why, and the fields stay plain inputs holding their saved values.

!!! info "Not in use yet"
    Picking a provider card is not enough. Until the page is saved with everything the provider
    needs, its section shows a blue *Not in use yet* notice and nothing is issued through it. The
    notice disappears once the provider is both selected and fully configured.

The settings fields tell browsers and password managers (Bitwarden, 1Password, LastPass, Dashlane)
not to autofill them, so your pretix login never ends up in a provider's credentials.

## Tax numbers (NIF)

By default the buyer's tax number is read from pretix's **VAT ID** field on the invoice address.
pretix only offers that field to business customers, so individuals have nowhere to enter a NIF
and are invoiced as final consumers.

To collect it from everyone:

1. Under pretix's **Settings → Invoicing**, open the **Address form** tab and fill in **Custom
   recipient field label**, e.g. `NIF`. An invoice address has to be asked for (**Ask for invoice
   address**) for the form to appear at all.

    ![pretix's invoice settings with the custom recipient field labelled NIF](img/pretix-invoice-custom-field.png)

2. Turn on **The custom invoice-address field holds the buyer's tax number** here.

The custom field is then used whenever the VAT ID field is empty, and it becomes the **one tax
number field** at checkout:

![The checkout's invoice address form showing a single NIF field](img/checkout-nif.png)

- pretix's *internal reference* field, which pretix's own Portuguese translation labels "NIF
  (Número de Contribuinte)" for business buyers, is hidden so there is no second "NIF" box.
- For Portuguese addresses pretix's **VAT ID** field is hidden too, businesses included. A buyer
  from another EU country still sees it, since reverse-charge handling needs it.

Numbers are accepted with spaces, dots or dashes and a `PT` prefix (`PT 237.892.294` is fine).
Portuguese numbers that fail their check digit are ignored rather than sent, so a typo turns the
document into a final-consumer one instead of a rejected one.

Without a tax number, the document is issued to a **final consumer** (`999999990`). A buyer who
gave neither a name nor a company is invoiced as "Consumidor Final", never under their e-mail
address.

!!! warning "Don't require the VAT ID"
    With the custom field as the NIF, pretix's own **Require VAT ID** option would reject Portuguese
    business buyers, since the field they'd have to fill is hidden. Leave it off.

## Invoice lines

Every document carries every non-cancelled position **and every fee** of the order (payment,
service, shipping fees…), so its total is the order total.

- Identical tickets (same product, price and rate) share one line with a quantity.
- With **Let buyers itemize the invoice by attendee** on and the checkbox ticked at checkout, each
  ticket gets its own line, named `<product> — <attendee name>`.

    ![The checkout's contact form with the "Itemize the invoice by attendee" checkbox](img/checkout-itemize.png)

- Each line shows the pretix product name and the price actually paid.

Providers receive **net** unit prices and add their configured VAT on top. pretix prices are gross,
so the net is derived to 8 decimal places and chosen so that the document's total equals what the
buyer paid, cent for cent, even across several lines at the same rate.

## Manual-only issuance

With **Issue documents automatically** off, paying an order or completing a refund issues nothing.
Documents come only from the buttons on the order's **Invoicing (PT)** panel: **Issue invoice now**
and **Issue credit note**. Everything else works the same, including e-mails and the buyer's
downloads.

Use it for an event where only some orders need an official document, or while you're still
checking the provider's output.

## Billing changes after invoicing

Each issued document records the billing data it was made out with (name, NIF, address). If the
buyer later edits their invoice address on their order page, pretix's **order modified** hook
compares the two and, when they differ:

- the order's **Invoicing (PT)** panel shows a warning naming the invoice;
- the event's **contact e-mail** (pretix **Settings → General**) receives a message with a link
  to the order.

Nothing is credited or re-issued on its own. To accept the change, press **Issue credit note**,
then **Issue invoice now** on the order page; the new invoice uses the new data. The e-mail repeats
on every later save while the data still differs. Documents issued by plugin versions before 0.7
carry no snapshot and never warn.
