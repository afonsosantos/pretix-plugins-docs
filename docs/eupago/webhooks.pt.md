# Webhooks e reembolsos

## URL do webhook

Os pagamentos são confirmados pela euPago ao chamar o pretix. Configure **um** URL no Backoffice da
euPago (Canais → Listagem de Canais → "Receber notificação para um URL") para cada canal que usar, e
escolha o **webhook 2.0**:

```
https://<o-seu-domínio-pretix>/eupago/webhook/
```

O URL exato aparece na página **Configurações → euPago** do evento. É global, não por evento, e aceita
os dois formatos de webhook da euPago:

| Formato | Verificado com |
|---|---|
| v1.0 (GET) | a chave API configurada |
| v2.0 (POST) | a **chave do webhook** (assinatura, ou desencriptação se a encriptação estiver ativa) |

Copie a chave do webhook do canal no Backoffice para a definição **Chave do webhook**. Sem ela, as
notificações v2.0 de pagamento e de reembolso são **recusadas**. Caso contrário, qualquer pessoa que
conheça o código do seu próprio pedido poderia marcá-lo como pago. Os cancelamentos e expirações
continuam a ser aceites sem a chave, porque no pior caso falham um pagamento que o comprador pode
repetir.

A opção "encriptar" do webhook no Backoffice também é suportada. O conteúdo é desencriptado com a
mesma chave, e só é aceite para pagamentos do evento a que essa chave pertence.

!!! warning "Atenção"
    Sem um webhook a funcionar, os pagamentos nunca são confirmados e as encomendas ficam pendentes
    até expirarem.

## Pagamentos feitos depois de um cancelamento

Se um comprador mudar de método de pagamento, o pretix cancela o pagamento Multibanco pendente. A
referência em si continua a poder ser paga na euPago até expirar. Se o comprador a pagar na mesma, o
pagamento é confirmado no pretix, para que o dinheiro nunca se perca de vista.

Se entretanto o pedido expirou e os bilhetes foram vendidos, o pretix não o consegue marcar como pago.
O pagamento fica registado e o log do pretix mostra um erro `quota is exceeded`, para que possa
resolver a situação com o comprador.

## Cancelamentos, erros e expiração

Uma notificação v2.0 `Cancel`, `Error` ou `Expired` dá o pagamento pendente como falhado. Por exemplo,
o comprador recusou o pedido MB WAY, o pedido falhou, ou uma referência Multibanco expirou. O comprador
pode então pagar novamente na página do pedido. Uma notificação que chegue *depois* de o pagamento
estar confirmado é ignorada.

## Reembolsos

Os reembolsos **não podem ser iniciados a partir do pretix**. Faça-os no Backoffice da euPago. Quando
a euPago envia a notificação `Refund`/`Refunded`, o plugin regista um reembolso externo no pagamento
do pretix.

A notificação da euPago não inclui o valor do reembolso, por isso é sempre registado como reembolso
**total**, e apenas uma vez por pagamento. Reenvios do webhook não reembolsam duas vezes. Uma
notificação de reembolso que chegue antes da confirmação do próprio pagamento recebe um pedido para
voltar a tentar, para que a euPago a reenvie mais tarde.

Se também usar o [pt-invoicing](../pt-invoicing/index.md), esse reembolso total emite
automaticamente uma nota de crédito.
