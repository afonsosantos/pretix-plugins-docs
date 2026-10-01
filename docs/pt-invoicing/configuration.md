# Configuration

Open the event's **Settings → Invoicing (PT)**. (It's labelled "(PT)" to tell it apart from pretix's
own "Invoicing" entry.)

## General

| Setting | Default | Description |
|---|---|---|
| **Invoicing provider** | none | Pick a provider card (Fact.pt, Moloni) or **No invoicing**. With none, nothing is issued. Only the selected provider's fields are saved. |
| **The custom invoice-address field holds the buyer's tax number** | off | See [Tax numbers (NIF)](#tax-numbers-nif). |
| **Show the invoice on the buyer's order page** | on | Adds a download button next to the ticket downloads. The PDF is proxied through pretix, so your provider credentials never reach the browser. |

## E-mail

pretix's own order e-mails can't attach these documents (they only attach pretix's own invoices), so
the plugin sends its own.

| Setting | Default | Description |
|---|---|---|
| **E-mail the invoice-receipt to the buyer** | off | Sends the PDF once it's issued. |
| **E-mail the credit note to the buyer** | off | Sends the credit note's PDF once it's issued. |

A failed e-mail is logged and never marks the issuance as failed — the document already exists.

## Provider settings

Below the general settings, fill in the selected provider's fields:

- [Fact.pt](factpt.md#settings)
- [Moloni](moloni.md#settings)

Some fields (VAT rate, company, document set…) turn into dropdowns populated live from your provider
account as soon as valid credentials are typed — no need to save first. Fields that depend on another
(Moloni's per-company settings) reload when it changes.

The settings fields tell browsers and password managers (Bitwarden, 1Password, LastPass, Dashlane)
not to autofill them, so your pretix login never ends up in a provider's credentials.

## Tax numbers (NIF)

By default the buyer's tax number is read from pretix's **VAT ID** field on the invoice address. pretix
only offers that field to business customers, so individuals have nowhere to enter a NIF.

To collect it from everyone:

1. Under pretix's **Settings → Invoicing**, add a custom recipient field labelled e.g. "NIF".
2. Turn on **The custom invoice-address field holds the buyer's tax number** here.

The custom field is then used when the VAT ID is empty. Portuguese numbers that fail their check digit
are ignored rather than sent.

Without a tax number, the document is issued to a **final consumer** (`999999990`).
