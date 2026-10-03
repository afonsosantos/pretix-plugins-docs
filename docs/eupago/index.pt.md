# pretix-eupago

Integra a [euPago](https://eupago.pt), um gateway de pagamentos português, no pretix, acrescentando
dois métodos de pagamento:

- **Multibanco** — o comprador recebe uma entidade e referência para pagar num Multibanco ou no
  homebanking.
- **MB WAY** — o comprador aprova uma notificação no telemóvel.

Ambos ficam *pendentes* no pretix até a euPago confirmar o pagamento através de um
[webhook](webhooks.md). Só são disponibilizados em eventos cuja moeda é o euro.

- PyPI: <https://pypi.org/project/pretix-eupago/>
- Código: <https://github.com/afonsosantos/pretix-eupago>
- Problemas: <https://github.com/afonsosantos/pretix-eupago/issues>
- Requer `pretix>=2024.1.0`, Python ≥ 3.11

## Instalação

```bash
pip install pretix-eupago
```

Reinicie o pretix e ative **Pagamentos euPago** em **Configurações → Plugins** do evento. Depois
[configure-o](configuration.md).

## A página euPago

O plugin acrescenta uma entrada **euPago** à barra lateral do evento com todos os pagamentos euPago do
evento. Cada linha mostra a referência Multibanco ou a transação MB WAY, e o motivo de um pagamento ter
falhado (por exemplo, o comprador cancelou o pedido MB WAY). Pode:

- pesquisar por código do pedido ou referência Multibanco (com ou sem espaços), o que ajuda quando um
  comprador liga por causa de um pagamento;
- filtrar por método (Multibanco / MB WAY) e estado do pagamento.

Use-a para ver o que foi dado a um comprador, ou que pagamentos ainda aguardam a euPago.
