# Emissão e notas de crédito

## Emissão automática

Com o plugin configurado, cada encomenda que fica **paga** recebe uma fatura-recibo, emitida em
segundo plano. Encomendas por pagar nunca são faturadas, seja qual for a origem do pedido.

!!! tip "Dica"
    A emissão corre nos workers do Celery do pretix. Se o Celery não estiver a correr, as tarefas
    ficam apenas na fila.

## Painel

A entrada **Faturação PT** na barra lateral do evento lista cada tentativa de emissão com o tipo
(fatura / nota de crédito), fornecedor, estado, número do documento e erro, filtrável por estado.
Cada linha bem-sucedida tem a descarga do PDF; cada linha falhada tem **Tentar novamente**.

## O painel na página da encomenda

Cada encomenda no Control ganha um painel **Faturação PT**, com uma linha por documento:

- **Emitir fatura agora / Reemitir** — para uma encomenda paga sem fatura emitida com sucesso. Use-o
  em encomendas pagas antes de o plugin estar configurado, encomendas marcadas como pagas à mão, ou
  depois de corrigir o que o fornecedor recusou.
- **Emitir nota de crédito / Repetir nota de crédito** — disponível quando a fatura foi emitida com
  sucesso.

## Dois tipos de falha

| Falha | Exemplo | O que acontece |
|---|---|---|
| **Recusada pelo fornecedor** | NIF inválido, divergência de IVA, token errado | Marcada como *Erro* com a mensagem do fornecedor. Não é repetida — corrija os dados ou as definições e carregue em Tentar novamente. |
| **Rede / fornecedor em baixo** | timeout, sem ligação | Também marcada como *Erro*. Os fornecedores incluídos tratam-nas como recusas, por isso também não são repetidas automaticamente — carregue em Tentar novamente quando o fornecedor voltar. |

## Notas de crédito

É emitida uma nota de crédito **automaticamente** quando os reembolsos de uma encomenda somam o valor
total pago — seja num só reembolso ou em vários. Credita sempre a fatura original **na totalidade**.

Um reembolso **parcial** que não leve a encomenda a zero não emite nada, porque nenhum dos
fornecedores consegue creditar menos do que o documento inteiro. A decisão é sua: emitir à mão uma
nota de crédito total na página da encomenda, ou não emitir nada. Veja as
[Limitações conhecidas](limitations.md).

## Do lado do comprador

- **Página da encomenda** — um botão de descarga junto às descargas dos bilhetes, quando o documento
  existe (se estiver ativo). Autenticado pela ligação secreta da encomenda, como as descargas do
  próprio pretix.
- **E-mail** — o PDF em anexo, se estiver ativo nas [definições](configuration.md#e-mail). Emitir à
  mão no Control também envia o e-mail.
