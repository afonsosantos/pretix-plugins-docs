# Resolução de problemas

## Ambos os plugins

**O plugin não aparece em Configurações → Plugins.**
O pretix só lê os plugins instalados no arranque — reinicie-o (servidor web *e* workers do Celery)
depois do `pip install`. Confirme que instalou no mesmo ambiente Python em que o pretix corre.

## euPago

**Os pagamentos ficam pendentes para sempre.**
Os pagamentos só são confirmados pelo webhook da euPago. Confirme que:

- o URL do webhook `https://<o-seu-domínio-pretix>/eupago/webhook/` está definido no Backoffice da
  euPago para **todos** os canais que usa ([Webhooks](eupago/webhooks.md));
- a sua instância do pretix está acessível a partir da internet nesse endereço;
- a **Chave do webhook** está definida em **Configurações → euPago** — sem ela, as notificações de
  pagamento v2.0 são recusadas;
- o log do pretix mostra linhas `euPago webhook` quando é feito um pagamento — se não houver
  nenhuma, a euPago não está a chegar até si.

**O webhook responde `webhook key not configured`.**
Chegou uma notificação v2.0 de pagamento ou reembolso para um evento sem **Chave do webhook**. Copie a
chave do webhook do canal no Backoffice para **Configurações → euPago**. A euPago volta a tentar as
notificações falhadas durante algum tempo, por isso os pagamentos recentes costumam confirmar-se
sozinhos depois de a definir; confirme os mais antigos à mão.

**Uma notificação v1.0 é ignorada (`chave_api mismatch` no log).**
A notificação trouxe uma chave API diferente da configurada no pretix. Copie de novo a chave API desse
canal em Backoffice → Canais → Listagem de Canais.

**O webhook responde `invalid signature`.**
A assinatura de uma notificação v2.0 não corresponde à **Chave do webhook**. Copie de novo a chave
nas definições de webhook do canal no Backoffice.

**O webhook responde `invalid credentials` (webhooks encriptados).**
A notificação foi encriptada com a chave do webhook de outro evento, e não com a do evento a que o
pagamento pertence. Confirme que cada evento tem a chave do seu próprio canal.

**As notificações chegam, mas nada acontece (webhooks encriptados).**
Com a opção "encriptar" da euPago ligada, o log do pretix mostra
`could not decrypt payload with any configured webhook secret`. A notificação é aceite mas ignorada,
por isso a euPago não a volta a enviar. Defina a chave do canal como **Chave do webhook** no pretix e
confirme à mão os pagamentos afetados.

**O Multibanco e o MB WAY não aparecem no checkout.**
Ficam ocultos quando a moeda do evento não é o euro, ou quando o **Modo de sandbox** está ativo mas o
evento não está em modo de teste. A página **Configurações → euPago** mostra um aviso em ambos os
casos.

**O checkout diz "Não foi possível contactar o fornecedor de pagamentos" ou "O fornecedor de pagamentos devolveu um erro".**
O pedido à euPago falhou. Normalmente é uma chave API errada, ou o **Modo de sandbox** não corresponde
à chave: as chaves de sandbox e de produção são diferentes.

**O checkout diz "Não foi possível enviar o pedido MB WAY para este número".**
A euPago recusou o pedido MB WAY — normalmente o número não está registado no MB WAY. O comprador pode
tentar outro número ou método de pagamento. A resposta da euPago fica no log do pretix.

**Aparece "O modo de sandbox da euPago está ativo" no checkout.**
Desligue o **Modo de sandbox** antes de vender a sério — em sandbox não é cobrado nenhum pagamento
real.

**O comprador não aprovou o MB WAY a tempo.**
O pedido expira ao fim de 5 minutos. O comprador pode pagar novamente na página da encomenda.

**Foi feito um pagamento, mas o pedido continua expirado (`quota is exceeded` no log).**
O comprador pagou depois de o pedido expirar e de os bilhetes terem sido vendidos. O pagamento fica
registado no pedido; reembolse-o ou liberte um bilhete à mão.

**Um reembolso feito na euPago não aparece no pretix.**
É registado quando a euPago envia uma notificação `Refund` — confirme que o webhook está a funcionar.
Os reembolsos não podem ser iniciados a partir do pretix.

## Faturação PT

A maioria dos problemas aparece no painel **Faturação PT** ou no painel **Faturação PT** da
encomenda, com a mensagem do fornecedor. Depois de corrigir a causa, carregue em **Tentar
novamente**.

**Uma encomenda paga não tem fatura, nem linha no painel.**
A emissão é ignorada sem aviso quando:

- não há nenhum **Fornecedor de faturação** selecionado;
- o fornecedor selecionado não está configurado: o Fact.pt não tem token da API, ou falta alguma
  definição obrigatória do Moloni;
- o plugin não está ativo no evento, ou a encomenda não está paga.

Corrija as definições e use **Emitir fatura agora** na página da encomenda. As encomendas pagas antes
de o plugin estar configurado também precisam disto.

**Uma linha fica em "A processar".**
A tarefa foi posta na fila mas nunca correu — os workers do Celery do pretix não estão a correr, ou
não estão a apanhar tarefas. Verifique-os e carregue em **Tentar novamente**.

**`Divergência de IVA: o pretix cobrou X% nesta encomenda mas a taxa configurada no Fact.pt é Y%`**
O pretix e o Fact.pt não concordam na taxa de IVA, e a fatura ficaria com o valor errado. Corrija a
regra fiscal do evento no pretix, ou escolha a **taxa de IVA** correspondente nas definições do
plugin. Veja [Fact.pt](pt-invoicing/factpt.md#a-taxa-de-iva-tem-de-coincidir-com-a-do-pretix).

**`A taxa de IVA N não existe nesta conta Fact.pt.`**
A taxa configurada foi apagada no Fact.pt, ou o token é de outra conta (ou do outro ambiente —
confirme **Usar ambiente de testes (sandbox)**). Escolha de novo a taxa.

**`Multiple clients with same tin. Specify an ID.`**
O Fact.pt tem vários clientes com o NIF do comprador, e o plugin não adivinha a qual faturar. Junte
ou apague os duplicados no Backoffice do Fact.pt e tente novamente.

**`Falha de ligação ao Fact.pt: …` / `Não foi possível contactar o Moloni: …`**
O fornecedor estava em baixo ou inacessível. Estas falhas não são repetidas automaticamente —
carregue em **Tentar novamente** quando voltar. No Moloni, confirme primeiro que não foi criado
nenhum documento para a encomenda.

**`Resposta inválida do Fact.pt (HTTP …)`**
Muitas vezes é um token de produção usado na sandbox, ou o contrário. Confirme **Usar ambiente de
testes (sandbox)**.

**`O Moloni recusou as credenciais.`**
Confirme o ID de programador, o segredo do cliente, o utilizador e a palavra-passe. São verificados
em direto quando os preenche: os erros aparecem por baixo das definições do Moloni.

**`Sem ligação ao Moloni, ou a ligação expirou.`**
A ligação ao Moloni não existe ou expirou, e não há utilizador/palavra-passe definidos como
alternativa. Carregue em **Ligar ao Moloni** (ou **Voltar a ligar**) nas definições e tente
novamente. O plugin renova a ligação todos os dias, por isso isto costuma significar que o Moloni a
revogou. Veja [Ligar ao Moloni](pt-invoicing/moloni.md#ligar-ao-moloni).

**`Divergência de IVA: o pretix cobrou X% nesta encomenda mas a taxa configurada no Moloni é Y%`**
Tal como no Fact.pt: corrija a regra fiscal do evento, ou escolha a **taxa de IVA** correspondente.
Nada foi criado no Moloni. Veja [IVA](pt-invoicing/moloni.md#iva).

**`Esta encomenda tem linhas com IVA a 0%, que o Moloni só aceita com um motivo de isenção.`**
Defina um **Motivo de isenção** (por exemplo `M07`) nas definições do Moloni e tente novamente.

**As listas de taxa de IVA / empresa / série de documentos não aparecem.**
São preenchidas a partir da sua conta no fornecedor quando as credenciais são válidas. Se ficarem
como campos numéricos, a linha por baixo das definições do fornecedor diz porquê (token errado,
fornecedor inacessível).

**A fatura saiu para consumidor final, mas o comprador indicou um NIF.**
O pretix só pede Número de IVA a clientes empresa. Peça-o a toda a gente com um campo personalizado
no endereço de faturação — veja [NIF](pt-invoicing/configuration.md#nif). Os NIF que não passam no
dígito de controlo português são ignorados.

**Uma encomenda foi reembolsada, mas não foi emitida nota de crédito.**
Só é emitida automaticamente quando:

- o reembolso está marcado como **concluído** no pretix;
- os reembolsos somam o valor total pago — um reembolso parcial não emite nada;
- a fatura da encomenda foi emitida com sucesso.

Caso contrário, use **Emitir nota de crédito** na página da encomenda. Veja
[Notas de crédito](pt-invoicing/usage.md#notas-de-credito).

**O comprador não recebeu o documento por e-mail.**
Os dois e-mails estão **desligados** por omissão — ligue-os em
[E-mail](pt-invoicing/configuration.md#e-mail). Um e-mail que falhe não marca a emissão como falhada;
consulte o log do pretix.

**A descarga do PDF devolve "não encontrado".**
O documento foi emitido por outro fornecedor que não o atualmente selecionado, e as credenciais do
fornecedor antigo não são guardadas. Descarregue-o no back office desse fornecedor.
