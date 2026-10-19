# pretix-pt-invoicing

Emite **faturas-recibo** certificadas pela AT através de um serviço português de faturação
eletrónica, automaticamente assim que uma encomenda do pretix é paga, e uma **nota de crédito**
quando é totalmente reembolsada. Um painel no Control acompanha cada emissão e permite tentar de novo
as que falharam.

![O painel Faturação PT da página da encomenda com uma fatura e uma nota de crédito emitidas](img/order-panel.png)

- PyPI: <https://pypi.org/project/pretix-pt-invoicing/>
- Código: <https://github.com/afonsosantos/pretix-pt-invoicing>
- Problemas: <https://github.com/afonsosantos/pretix-pt-invoicing/issues>
- Requer `pretix>=2024.1.0`, Python ≥ 3.11
- Versão documentada: **0.7.1**

É a primeira vez? Siga os [Primeiros passos](getting-started.md), um guia passo a passo com imagens.

## Serviços de faturação

Cada evento usa um serviço.

| Serviço | Identificador | Estado |
|---|---|---|
| [Fact.pt](factpt.md) | `factpt` | Verificado com uma conta sandbox real |
| [Moloni](moloni.md) | `moloni` | Verificado com uma conta real (faturas-recibo, notas de crédito, PDF) |

!!! warning "Antes de usar em produção"
    Emita alguns documentos de teste na sandbox do serviço e confira os valores, o cliente e o IVA
    antes de o ativar num evento a sério. Veja as [Limitações conhecidas](limitations.md).

## O que faz

- **Fatura-recibo ao pagar**, nota de crédito no reembolso total, as duas emitidas em segundo plano
  para um serviço lento nunca atrasar o comprador.
- **Todas as linhas da encomenda**: bilhetes, com os iguais agrupados, e todas as taxas;
  opcionalmente uma linha por participante. Os preços sem IVA são calculados para o total do
  documento ser exatamente o que foi pago.
- **O NIF do comprador** de toda a gente, não só das empresas, com um único campo NIF no checkout.
- **O comprador recebe o PDF** na página da encomenda e, se o ligar, por e-mail.
- **Um painel geral e um painel por encomenda** com estado, número do documento, erros, novas
  tentativas e páginas de confirmação antes de qualquer ação irreversível.
- **Modo só manual**, para eventos em que só algumas encomendas precisam de documento.
- **Avisos de alteração dos dados de faturação** quando um comprador edita a morada depois de
  faturado; nada é reemitido sem aprovação de quem gere o evento.
- **Moloni**: ligação por OAuth, associação de cada forma de pagamento do pretix a uma do Moloni,
  artigos de catálogo criados automaticamente.

## Como funciona

1. Uma encomenda é paga e o pretix dispara `order_paid`.
2. O plugin põe uma tarefa na fila do Celery. O serviço nunca é chamado durante o checkout.
3. A tarefa emite a fatura-recibo e guarda o resultado (número do documento, ligação para o PDF, ou
   o erro do serviço).
4. Opcionalmente, o PDF é enviado ao comprador por e-mail e fica disponível para descarregar na
   página da encomenda.

A emissão é idempotente: cada encomenda tem um identificador estável, e uma emissão bem-sucedida
nunca é repetida. Uma encomenda paga de novo depois de a fatura ter sido creditada (um reembolso,
seguido de um novo pagamento) recebe uma nova fatura; ver
[Paga de novo depois de um reembolso](usage.md#paga-de-novo-depois-de-um-reembolso).

## Instalação

```bash
pip install pretix-pt-invoicing
python -m pretix migrate
```

Reinicie o pretix e ative **Faturação portuguesa** em **Configurações → Plugins** do evento, no
separador **Integrações**. Depois [configure-o](getting-started.md).
