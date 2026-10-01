# Fact.pt

[Fact.pt](https://fact.pt) provider — identifier `factpt`.

## Settings

| Setting | Description |
|---|---|
| **API token (x-auth-token)** | Your Fact.pt API token. |
| **Use sandbox environment** | Issue against `api.sandbox.fact.pt`. Sandbox and production tokens are different. |
| **VAT rate ID** | Dropdown of your account's active tax rates, loaded live once the token is valid. Applied to every line. |
| **Unit** | Units / Meters / Boxes / Kilograms / Liters. |
| **Item type** | `service` or `product`. |
| **Send the buyer's e-mail address to Fact.pt** | Off by default. Stores the e-mail on the Fact.pt client, which lets Fact.pt e-mail the document itself. |

## The VAT rate must match pretix

Fact.pt receives each line's **net** price and adds its configured rate on top. If that rate differs
from what pretix charged, the document would be for an amount the buyer never paid (e.g. a €15.00
ticket with no tax rule in pretix, invoiced at 23%, becomes €18.45).

So before every issuance the plugin checks the configured rate against each line's pretix tax rate and
**refuses with an explicit error** on a mismatch. Fix the tax rule or the setting, then retry.

!!! note
    One VAT rate per event: an event selling items at different rates can't be invoiced yet. See
    [Known limitations](limitations.md).

## Clients

- **With a NIF** — an existing Fact.pt client with exactly that NIF is reused. Otherwise one is
  created (or updated, if Fact.pt already has that NIF) from the order's invoice address.
- **Without a NIF** — an existing final-consumer client with the same name is reused; otherwise Fact.pt
  creates one under `999999990`.

!!! warning "Duplicate clients"
    If several Fact.pt clients share the buyer's NIF, issuance stops with
    `Multiple clients with same tin. Specify an ID.` The plugin won't guess which one to invoice —
    merge the duplicates in Fact.pt's Backoffice and retry.

A client created or updated from pretix overwrites that NIF's record in Fact.pt with the order's
address data.

## Duplicate protection

Each document carries an `identifierId` (`pretix-<event>-<order code>`), and Fact.pt rejects a second
document with the same one, so pressing Retry after an ambiguous failure can never issue twice.

An order [paid again after a refund](usage.md#paid-again-after-a-refund) gets a new one per new
invoice (`…-r1`, `…-r2`), and each credit note uses its invoice's plus `-credit`.

## Credit notes

Issued with Fact.pt's credit endpoint, crediting the original document in full. Fact.pt allows exactly
one credit note per document, for its full amount.
