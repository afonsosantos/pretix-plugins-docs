# Limitações conhecidas

**Uma taxa de IVA por evento.**
Todas as linhas com IVA são faturadas à taxa configurada. Um evento que venda artigos a taxas
diferentes falha a emissão (de forma visível, com um erro de *Divergência de IVA*) em vez de faturar
mal. Ainda não existe um mapeamento por taxa. O Moloni aceita também linhas a 0%, que fatura com o
motivo de isenção.

**Reembolsos parciais não têm nota de crédito.**
Os dois serviços só conseguem creditar o valor total de um documento. Um reembolso parcial não emite
nada até os reembolsos atingirem o valor total; pode emitir à mão uma nota de crédito total na
página da encomenda.

**O Fact.pt precisa de uma série com 8 casas decimais.**
Os preços unitários sem IVA são enviados com várias casas decimais para os documentos baterem certo
ao cêntimo. Uma série com 2 casas decimais recusa-os, e o arredondamento do lado do Fact.pt faria os
totais desviar-se por cêntimos. Associe uma série de 8 casas decimais à chave da API.

**O Moloni imprime os preços unitários com 5 casas decimais.**
Um preço com IVA de 35,00 € a 23% aparece no PDF com um preço unitário de 34,99999; a incidência, o
IVA e o total estão certos. Só preços cujo valor sem IVA são cêntimos inteiros ("preço inclui IVA"
desligado no pretix) saem limpos.

**Categorias de artigos do Moloni: só as de topo.**
A lista **Categoria de artigos** mostra só as categorias de topo do Moloni.

**Fichas de cliente duplicadas param a emissão.**
Se um NIF corresponder a vários clientes no Fact.pt, ou a vários clientes no Moloni, a emissão pára
com um erro explícito. Junte-os no back office do serviço e tente novamente.

**Os dados de cliente no Fact.pt são substituídos.**
Quando um cliente é criado ou atualizado a partir de uma encomenda, a morada da encomenda substitui
a que estava na ficha desse NIF. Os clientes do Moloni nunca são alterados; o documento leva uma
morada alternativa.

**Um documento do Fact.pt criado por um pedido que expirou não fica associado.**
A nova tentativa automática é recusada como duplicado, por isso não é emitida uma segunda fatura,
mas a linha fica em *Erro* sem número de documento. Procure o documento no Backoffice do Fact.pt.

**Os avisos de alteração dos dados de faturação só cobrem documentos emitidos desde a 0.7.**
As linhas mais antigas não têm os dados de faturação guardados, por isso uma edição posterior nunca
avisa.

**Mudar de serviço a meio do evento não reemite nada.**
Os documentos antigos ficam no serviço que os emitiu e deixam de poder ser descarregados pelo pretix,
porque as credenciais do serviço antigo não são guardadas.

**Exigir o Número de IVA estraga o campo único de NIF.**
Com **O campo personalizado da morada de faturação contém o NIF do comprador** ligado, o campo de
Número de IVA do pretix fica escondido em moradas portuguesas, por isso a opção **Exigir número de
IVA** do pretix recusaria as empresas portuguesas. Deixe-a desligada.

**Sem emissão em massa.**
A emissão à mão é uma encomenda de cada vez, de propósito. Para um lote pendente, corra a tarefa
numa shell; veja [Emissão em massa](usage.md#emissao-em-massa).
