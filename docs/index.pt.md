# Plugins do pretix para Portugal

Dois plugins independentes para o [pretix](https://github.com/pretix/pretix), para vender bilhetes em
Portugal.

| Plugin | O que faz | PyPI |
|---|---|---|
| [**pretix-eupago**](eupago/index.md) | Aceita pagamentos por **referência Multibanco** e **MB WAY** através da [euPago](https://eupago.pt). | [`pretix-eupago`](https://pypi.org/project/pretix-eupago/) |
| [**pretix-pt-invoicing**](pt-invoicing/index.md) | Emite **faturas-recibo** certificadas pela AT (e notas de crédito) através do [Fact.pt](https://fact.pt) ou do [Moloni](https://moloni.pt), assim que uma encomenda é paga. | [`pretix-pt-invoicing`](https://pypi.org/project/pretix-pt-invoicing/) |

Funcionam sozinhos ou em conjunto: a euPago confirma o pagamento, o pretix marca a encomenda como paga
e o pt-invoicing emite a fatura-recibo.

## Instalar um plugin

Ambos se instalam da mesma forma — no mesmo ambiente Python da sua instância do pretix:

```bash
pip install pretix-eupago pretix-pt-invoicing
```

Reinicie o pretix. Os plugins registam-se sozinhos através de um entry point do setuptools, por isso
não é preciso mexer em `INSTALLED_APPS`. Depois aplique as migrações (o pt-invoicing inclui um
modelo):

```bash
python -m pretix migrate
```

Por fim, ative cada plugin por evento em **Configurações → Plugins**.
