# pretix-eupago

Integra a [euPago](https://eupago.pt), um gateway de pagamentos português, no pretix, acrescentando
dois métodos de pagamento:

- **Multibanco** — o comprador recebe uma entidade e referência para pagar num Multibanco ou no
  homebanking.
- **MB WAY** — o comprador aprova uma notificação no telemóvel.

Ambos ficam *pendentes* no pretix até a euPago confirmar o pagamento através de um
[webhook](webhooks.md).

- PyPI: <https://pypi.org/project/pretix-eupago/>
- Código: <https://github.com/afonsosantos/pretix-eupago>
- Problemas: <https://github.com/afonsosantos/pretix-eupago/issues>
- Requer `pretix>=4.0.0`

## Instalação

```bash
pip install pretix-eupago
```

Reinicie o pretix e ative **Pagamentos euPago** em **Configurações → Plugins** do evento. Depois
[configure-o](configuration.md).

## A página Pedidos euPago

O plugin acrescenta uma entrada **Pedidos euPago** à barra lateral do evento: todos os pagamentos
euPago do evento, filtráveis por método (Multibanco / MB WAY) e estado, com a referência Multibanco
ou o telemóvel MB WAY de cada um. Use-a para ver o que foi dado a um comprador, ou que pagamentos
ainda aguardam a euPago.
