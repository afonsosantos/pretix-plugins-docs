# pretix-pt-invoicing

Emite **faturas-recibo** certificadas pela AT através de um fornecedor português de faturação
eletrónica, automaticamente assim que uma encomenda do pretix é paga — e uma **nota de crédito**
quando é totalmente reembolsada. Um painel no Control acompanha cada emissão e permite tentar de novo
as que falharam.

- PyPI: <https://pypi.org/project/pretix-pt-invoicing/>
- Código: <https://github.com/afonsosantos/pretix-pt-invoicing>
- Problemas: <https://github.com/afonsosantos/pretix-pt-invoicing/issues>
- Requer `pretix>=2024.1.0`, Python ≥ 3.11

## Fornecedores

Cada evento usa um fornecedor.

| Fornecedor | Identificador | Estado |
|---|---|---|
| [Fact.pt](factpt.md) | `factpt` | Verificado com uma conta sandbox real |
| [Moloni](moloni.md) | `moloni` | Feito a partir da documentação pública da API e do código do plugin oficial do Moloni; **ainda não testado com uma conta real** |

!!! warning "Antes de usar em produção"
    Emita alguns documentos de teste na sandbox do fornecedor e confirme os valores, o cliente e o
    IVA antes de o ativar num evento a sério. Veja as [Limitações conhecidas](limitations.md).

## Como funciona

1. Uma encomenda é paga → o pretix dispara `order_paid`.
2. O plugin põe uma tarefa na fila do Celery — o fornecedor nunca é chamado durante o checkout, por
   isso um fornecedor lento não atrasa o comprador.
3. A tarefa emite a fatura-recibo e guarda o resultado (número do documento, ligação para o PDF, ou
   o erro do fornecedor).
4. Opcionalmente, o PDF é enviado ao comprador por e-mail e fica disponível para descarregar na
   página da encomenda.

A emissão é idempotente: cada encomenda tem um identificador estável, e uma emissão bem-sucedida
nunca é repetida.

## Instalação

```bash
pip install pretix-pt-invoicing
python -m pretix migrate
```

Reinicie o pretix e ative **Faturação portuguesa** em **Configurações → Plugins** do evento. Depois
[configure-o](configuration.md).
