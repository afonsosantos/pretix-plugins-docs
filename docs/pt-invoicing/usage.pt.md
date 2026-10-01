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

O número do documento é o que aparece impresso nele, por exemplo `FR M2026/20` no Moloni ou
`2025QG/9` no Fact.pt (tal como o Fact.pt o devolve).

## O painel na página da encomenda

Cada encomenda no Control ganha um painel **Faturação PT**, com uma linha por documento — todas as
faturas e notas de crédito que a encomenda já teve, da mais antiga para a mais recente:

- **Emitir fatura agora / Reemitir** — para uma encomenda paga sem fatura emitida com sucesso. Use-o
  em encomendas pagas antes de o plugin estar configurado, encomendas marcadas como pagas à mão,
  depois de corrigir o que o fornecedor recusou, ou numa encomenda
  [paga de novo depois de um reembolso](#paga-de-novo-depois-de-um-reembolso).
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

## Paga de novo depois de um reembolso

Uma encomenda pode ser paga, reembolsada e depois paga de novo. Depois de a primeira fatura ter sido
creditada, o novo pagamento recebe uma **nova fatura**, emitida automaticamente como a primeira. Um
reembolso total posterior credita essa nova fatura, nunca uma que já esteja creditada. O painel da
encomenda mantém todas, da mais antiga para a mais recente.

Se o novo pagamento chegou antes de a nota de crédito ter sido emitida, nada é emitido sozinho para
ele. Use **Emitir fatura agora** na página da encomenda depois de a nota de crédito existir.

## Do lado do comprador

- **Página da encomenda** — um botão de descarga junto às descargas dos bilhetes, quando o documento
  existe (se estiver ativo). Autenticado pela ligação secreta da encomenda, como as descargas do
  próprio pretix.
- **E-mail** — o PDF em anexo, se estiver ativo nas [definições](configuration.md#e-mail). Emitir à
  mão no Control também envia o e-mail.
