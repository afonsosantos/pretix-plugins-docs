# Webhooks e reembolsos

## URL do webhook

Os pagamentos são confirmados pela euPago ao chamar o pretix. Configure **um** URL no Backoffice da
euPago (Canais → Listagem de Canais → "Receber notificação para um URL"), para cada canal que usar:

```
https://<o-seu-domínio-pretix>/eupago/webhook/
```

Este URL é global — não é por evento — e aceita os dois formatos de webhook da euPago:

| Formato | Verificado com |
|---|---|
| v1.0 (GET) | a chave API configurada |
| v2.0 (POST) | o segredo de assinatura do webhook, se estiver definido; caso contrário, apenas uma verificação cruzada mais fraca |

A opção "encriptar" do webhook no Backoffice também é suportada — o conteúdo é desencriptado com o
mesmo segredo.

!!! warning "Atenção"
    Sem webhook, os pagamentos nunca são confirmados e as encomendas ficam pendentes até expirarem.

## Cancelamentos e expiração

Uma notificação v2.0 `Cancel` ou `Expired` (por exemplo, o comprador recusou o pedido MB WAY, ou uma
referência Multibanco expirou) dá o pagamento pendente como falhado. Uma notificação que chegue
*depois* de o pagamento estar confirmado é ignorada.

## Reembolsos

Os reembolsos **não podem ser iniciados a partir do pretix** — faça-os no Backoffice da euPago.
Quando a euPago envia a notificação `Refund`/`Refunded`, o plugin regista um reembolso externo no
pagamento do pretix.

A notificação da euPago não inclui o valor do reembolso, por isso é sempre registado como reembolso
**total**, e apenas uma vez por pagamento — reenvios do webhook não reembolsam duas vezes.

Se também usar o [pt-invoicing](../pt-invoicing/index.md), esse reembolso total emite
automaticamente uma nota de crédito.
