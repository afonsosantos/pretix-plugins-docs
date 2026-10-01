# Troubleshooting

## Both plugins

**The plugin isn't listed under Settings → Plugins.**
pretix only reads installed plugins at startup — restart it (web server *and* Celery workers) after
`pip install`. Make sure you installed into the same Python environment pretix runs from.

## euPago

**Payments stay pending forever.**
Payments are only confirmed by euPago's webhook. Check that:

- the webhook URL `https://<your-pretix-domain>/eupago/webhook/` is set in euPago's Backoffice for
  **every** channel you use ([Webhooks](eupago/webhooks.md));
- your pretix instance is reachable from the internet at that address;
- the pretix log shows `euPago webhook` lines when a payment is made — if there are none, euPago isn't
  reaching you.

**The webhook answers `invalid credentials`.**
A v1.0 notification carried an API key that doesn't match the one configured in pretix. Copy the API
key for that channel again from Backoffice → Channels → Channel Listing.

**The webhook answers `invalid signature`.**
A v2.0 notification's signature doesn't match the **Webhook signature secret**. Copy the secret again
from the channel's webhook settings in Backoffice.

**Notifications arrive, but nothing happens (encrypted webhooks).**
With euPago's "encrypt" option turned on, the pretix log shows
`could not decrypt payload with any configured webhook secret`. The notification is acknowledged but
ignored, so euPago won't resend it. Set the channel's secret as the **Webhook signature secret** in
pretix, then confirm the affected payments by hand.

**Checkout says "Could not reach the payment provider" or "The payment provider returned an error".**
The request to euPago failed. Usually a wrong API key, or **Sandbox / Test mode** not matching the
key: sandbox and production keys are different.

**"euPago sandbox mode is active" is shown at checkout.**
Turn off **Sandbox / Test mode** before selling for real — no real payment is taken in sandbox.

**The buyer didn't approve MB WAY in time.**
The request expires after 5 minutes. The buyer can retry the payment from their order page.

**A refund made in euPago doesn't show up in pretix.**
It's recorded when euPago sends a `Refund` notification — check the webhook is working. Refunds
can't be started from pretix.

## PT invoicing

Most problems show up in the **Invoicing (PT)** dashboard or the order's **Invoicing (PT)** panel, with
the provider's message. Once the cause is fixed, press **Retry**.

**A paid order has no invoice, and no row in the dashboard.**
Issuance is skipped silently when:

- no **Invoicing provider** is selected;
- the selected provider isn't set up: Fact.pt has no API token, or any required Moloni setting is
  empty;
- the plugin isn't enabled for the event, or the order isn't paid.

Fix the settings, then use **Issue invoice now** on the order page. Orders paid before the plugin was
set up need this too.

**A row stays in "Processing".**
The task was queued but never ran — pretix's Celery workers aren't running, or aren't picking up
tasks. Check them, then press **Retry**.

**`VAT mismatch: pretix charged X% on this order but the configured Fact.pt rate is Y%`**
pretix and Fact.pt disagree on the VAT rate, and the invoice would be for the wrong amount. Either fix
the event's tax rule in pretix, or pick the matching **VAT rate** in the plugin settings. See
[Fact.pt](pt-invoicing/factpt.md#the-vat-rate-must-match-pretix).

**`VAT rate N does not exist in this Fact.pt account.`**
The configured rate was deleted in Fact.pt, or the token belongs to another account (or the other
environment — check **Use sandbox environment**). Pick a rate again.

**`Multiple clients with same tin. Specify an ID.`**
Fact.pt has several clients with the buyer's NIF, and the plugin won't guess which one to invoice.
Merge or delete the duplicates in Fact.pt's Backoffice, then retry.

**`Could not reach Fact.pt: …` / `Could not reach Moloni: …`**
The provider was down or unreachable. These aren't retried automatically — press **Retry** once it's
back. With Moloni, first check in Moloni that no document was created for the order.

**`Invalid response from Fact.pt (HTTP …)`**
Often a production token used against the sandbox, or the other way round. Check **Use sandbox
environment**.

**`Moloni refused the credentials.`**
Check the developer ID, client secret, username and password. They're checked live when you fill
them in: errors show under the Moloni settings.

**`Not connected to Moloni, or the connection has expired.`**
The Moloni connection is missing or expired, and no username/password is set to fall back on. Press
**Connect to Moloni** (or **Reconnect**) in the settings, then retry. The plugin refreshes the
connection daily, so this usually means Moloni revoked it. See
[Connecting to Moloni](pt-invoicing/moloni.md#connecting-to-moloni).

**`VAT mismatch: pretix charged X% on this order but the configured Moloni rate is Y%`**
Same as for Fact.pt: fix the event's tax rule, or pick the matching **VAT rate**. Nothing was created
in Moloni. See [Taxes](pt-invoicing/moloni.md#taxes).

**`This order has 0% VAT lines, which Moloni only accepts with an exemption reason.`**
Set an **Exemption reason** (e.g. `M07`) in the Moloni settings, then retry.

**The VAT rate / company / document set dropdowns don't appear.**
They're filled from your provider account once the credentials are valid. If they stay plain number
fields, the line under the provider's settings shows why (bad token, provider unreachable).

**The invoice was made out to a final consumer, but the buyer gave a NIF.**
pretix only asks business customers for a VAT ID. Collect it from everyone with a custom invoice-address
field — see [Tax numbers (NIF)](pt-invoicing/configuration.md#tax-numbers-nif). NIFs that fail the
Portuguese check digit are ignored.

**An order was refunded, but no credit note was issued.**
One is only issued automatically when:

- the refund is marked **done** in pretix;
- refunds add up to the full amount paid — a partial refund issues nothing;
- the order's invoice was issued successfully.

Otherwise, use **Issue credit note** on the order page. See
[Credit notes](pt-invoicing/usage.md#credit-notes).

**The buyer didn't receive the document by e-mail.**
Both e-mails are **off** by default — turn them on under [E-mail](pt-invoicing/configuration.md#e-mail).
A failed e-mail doesn't mark the issuance as failed; check the pretix log.

**The PDF download returns "not found".**
The document was issued by a different provider than the one currently selected, and the old
provider's credentials aren't kept. Download it from that provider's back office.
