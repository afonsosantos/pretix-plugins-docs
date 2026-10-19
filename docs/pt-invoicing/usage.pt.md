# Emissão e notas de crédito

## Emissão automática

Com o plugin configurado, cada encomenda que fica **paga** recebe uma fatura-recibo, emitida em
segundo plano pelos workers do Celery do pretix. Encomendas por pagar nunca são faturadas, seja qual
for a origem do pedido. Com **Emitir documentos automaticamente** desligado, nada é emitido sem
carregar num botão; veja [Emissão só manual](configuration.md#emissao-so-manual).

!!! tip "Dica"
    Se o Celery não estiver a correr, as tarefas ficam na fila e as linhas ficam em *A processar*.

## Painel

A entrada **Faturação PT** na barra lateral do evento lista cada tentativa de emissão: encomenda,
serviço, tipo (fatura ou nota de crédito), estado, número do documento, erro, número de tentativas e
última atualização. Filtre por estado ou procure pelo código da encomenda.

![O painel Faturação PT com a lista de faturas e notas de crédito](img/dashboard.png)

Em cada linha emitida, o número do documento descarrega o PDF; cada linha falhada tem **Tentar
novamente**. O número é o que aparece impresso no documento, por exemplo `FR M2026/20` no Moloni ou
`2025QG/9` no Fact.pt.

| Estado | Significado |
|---|---|
| **Emitida** | O serviço criou o documento. O número e o PDF estão disponíveis. |
| **A processar** | Um worker está a tratar dela, ou a tarefa ainda está na fila. |
| **Erro** | A tentativa falhou; a coluna de erro diz porquê. Carregue em **Tentar novamente** depois de corrigir a causa. |

## O painel na página da encomenda

Cada encomenda no Control ganha um painel **Faturação PT**, com uma linha por documento, da mais
antiga para a mais recente: todas as faturas e notas de crédito que a encomenda já teve, cada uma
com o estado, o número, a hora da última tentativa, se foi enviada por e-mail ao comprador, e um
botão **Descarregar PDF**.

![O painel da página da encomenda: uma fatura emitida, a sua nota de crédito e uma linha "Emitir fatura agora"](img/order-panel.png)

As ações disponíveis:

- **Emitir fatura agora** — para uma encomenda paga sem fatura emitida com sucesso. Use-o em
  encomendas pagas antes de o plugin estar configurado, encomendas marcadas como pagas à mão com a
  emissão automática desligada, ou numa encomenda
  [paga de novo depois de um reembolso](#paga-de-novo-depois-de-um-reembolso).
- **Tentar emitir novamente** — numa linha de fatura em *Erro*, depois de corrigir o que o serviço
  recusou.
- **Emitir nota de crédito** / **Tentar emitir a nota de crédito novamente** — disponível quando a
  fatura foi emitida com sucesso, haja ou não um reembolso registado no pretix.
- **Abrir no Fact.pt** — uma ligação para a página do documento no serviço, quando este a oferece.

Uma linha falhada mostra por baixo a mensagem do serviço, com os detalhes por campo quando o serviço
os devolveu:

![Um painel de encomenda com dois ciclos completos de fatura/nota de crédito e uma terceira tentativa em erro](img/order-panel-error.png)

Todas as ações passam primeiro por uma **página de confirmação**, porque um documento emitido é
comunicado à AT e não pode ser anulado. Indica o serviço, a encomenda e o valor:

![A página de confirmação mostrada antes de emitir uma fatura-recibo](img/confirm-issue.png)

A confirmação também recusa logo à partida, com uma mensagem, o que a tarefa de outro modo
ignoraria: nenhum serviço selecionado, serviço não configurado ou ligação expirada, encomenda por
pagar, fatura já emitida, nada por creditar, ou uma tentativa já em curso.

!!! note "Em curso"
    Enquanto um worker trata de um documento, a linha diz *Em curso — recarregue daqui a pouco* e não
    tem botão, para um segundo clique não emitir em duplicado. Uma linha presa em *A processar* há
    mais de dez minutos (um worker que morreu a meio) pode ser tentada de novo.

Se o serviço não estiver configurado, ou a ligação ao Moloni tiver expirado, o painel continua
visível, com um aviso e uma ligação para as definições em vez dos botões.

## Dois tipos de falha

| Falha | Exemplo | O que acontece |
|---|---|---|
| **Recusada pelo serviço** | NIF inválido, divergência de IVA, token errado, motivo de isenção em falta | Marcada como *Erro* com a mensagem do serviço. Nunca é repetida automaticamente: corrija os dados ou as definições e carregue em **Tentar novamente**. |
| **Rede / serviço em baixo** | timeout, sem ligação | Também marcada como *Erro*. Com o **Fact.pt**, a tarefa repete sozinha, até três vezes com uns minutos de intervalo (*nova tentativa automática dentro de alguns minutos*, depois *desistiu ao fim de várias tentativas*); o Fact.pt recusa duplicados, por isso é seguro. Com o **Moloni** nunca é repetida automaticamente: a linha diz *o documento pode ou não ter sido criado*, e é você que confirma no Moloni antes de carregar em **Tentar novamente**. |
| **Serviço não configurado** | ligação ao Moloni expirada, token em falta | Registada como uma linha em *Erro* com a mensagem *Não emitida: o … não está totalmente configurado, ou a ligação expirou*, para a encomenda paga nunca ficar esquecida em silêncio. Corrija as definições e carregue em **Tentar novamente**. |

## Notas de crédito

É emitida uma nota de crédito **automaticamente** quando os reembolsos de uma encomenda somam o valor
total pago, seja num só reembolso ou em vários, contando só os reembolsos que o pretix marca como
**concluídos**. Credita sempre a fatura original **na totalidade**.

Um reembolso **parcial** que não leve a encomenda a zero não emite nada, porque nenhum dos serviços
consegue creditar menos do que o documento inteiro. A decisão é sua: emitir à mão uma nota de
crédito total na página da encomenda, ou não emitir nada. Veja as
[Limitações conhecidas](limitations.md).

## Paga de novo depois de um reembolso

Uma encomenda pode ser paga, reembolsada e depois paga de novo. Depois de a primeira fatura ter sido
creditada, o novo pagamento recebe uma **nova fatura**, emitida automaticamente como a primeira. Um
reembolso total posterior credita essa nova fatura, nunca uma que já esteja creditada. O painel da
encomenda mantém todas, da mais antiga para a mais recente, e mostra *Paga novamente depois de a
última fatura ter sido creditada* com um botão **Emitir fatura agora** sempre que a encomenda está
paga e a sua última fatura está creditada.

Se o novo pagamento chegou antes de a nota de crédito ter sido emitida, nada é emitido sozinho para
ele. Use **Emitir fatura agora** depois de a nota de crédito existir.

## Alterações aos dados de faturação

Se o comprador editar a morada de faturação depois de a fatura ter sido emitida, o painel mostra um
aviso e o e-mail de contacto do evento recebe uma mensagem; nada é reemitido sem si. Veja
[Alterações aos dados de faturação depois de faturar](configuration.md#alteracoes-aos-dados-de-faturacao-depois-de-faturar).

## Do lado do comprador

![A página da encomenda do comprador com os botões para descarregar a fatura e a nota de crédito](img/buyer-order-page.png)

- **Página da encomenda** — um botão **Descarregar fatura** junto às descargas dos bilhetes quando o
  documento existe, e **Descarregar nota de crédito** quando essa também existe (se **Mostrar a
  fatura na página da encomenda do comprador** estiver ligado). Autenticado pela ligação secreta da
  encomenda, como as descargas do próprio pretix. Só são oferecidos os documentos emitidos pelo
  serviço atual do evento.
- **E-mail** — o PDF em anexo, se estiver ativo em [E-mail](configuration.md#e-mail). Emitir à mão
  no Control também envia o e-mail.

## Emissão em massa

A emissão à mão é uma encomenda de cada vez, de propósito. Para um lote pendente (encomendas pagas
antes de o plugin estar configurado), corra a tarefa numa shell; as encomendas que já têm uma fatura
emitida com sucesso são ignoradas:

```python
# python -m pretix shell
from django_scopes import scopes_disabled
from pretix.base.models import Order
from pretix_ptinvoicing.tasks import issue_invoice

with scopes_disabled():
    for order in Order.objects.filter(event__slug="my-event", status=Order.STATUS_PAID):
        issue_invoice.apply(kwargs={"order_pk": order.pk, "event_pk": order.event.pk})
```
