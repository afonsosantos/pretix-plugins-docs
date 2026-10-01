# Limitações conhecidas

**Uma taxa de IVA por evento.**
Todas as linhas são faturadas à taxa configurada. Um evento que venda artigos a taxas diferentes
falha a emissão (de forma visível) em vez de faturar mal. Ainda não existe um mapeamento por taxa. O
Moloni aceita também linhas a 0%, que fatura com o motivo de isenção.

**Categorias de artigos do Moloni: só as de topo.**
A lista **Categoria de artigos** mostra só as categorias de topo do Moloni.

**Reembolsos parciais não têm nota de crédito.**
Os dois fornecedores só conseguem creditar o valor total de um documento. Um reembolso parcial não
emite nada até os reembolsos atingirem o valor total; pode emitir à mão uma nota de crédito total.

**Clientes duplicados param a emissão (Fact.pt).**
Se um NIF corresponder a vários clientes no Fact.pt, limpe-os no Backoffice e tente novamente.

**Os dados de cliente no Fact.pt são substituídos.**
Quando um cliente é criado/atualizado a partir de uma encomenda, o endereço da encomenda substitui o
que estava na ficha desse NIF.

**Mudar de fornecedor a meio do evento não reemite nada.**
Os documentos antigos ficam no fornecedor que os emitiu e deixam de poder ser descarregados pelo
pretix.

**Sem emissão em massa.**
A emissão à mão é uma encomenda de cada vez, de propósito. Para um lote pendente, corra a tarefa numa
shell:

```python
# python -m pretix shell
from django_scopes import scopes_disabled
from pretix.base.models import Order
from pretix_ptinvoicing.tasks import issue_invoice

with scopes_disabled():
    for order in Order.objects.filter(event__slug="my-event", status=Order.STATUS_PAID):
        issue_invoice.apply(kwargs={"order_pk": order.pk, "event_pk": order.event.pk})
```

As encomendas já faturadas com sucesso são ignoradas.
