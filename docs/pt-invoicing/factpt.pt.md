# Fact.pt

Fornecedor [Fact.pt](https://fact.pt) — identificador `factpt`.

## Definições

| Definição | Descrição |
|---|---|
| **Token da API (x-auth-token)** | O seu token da API do Fact.pt. |
| **Usar ambiente de testes (sandbox)** | Emite em `api.sandbox.fact.pt`. Os tokens de sandbox e de produção são diferentes. |
| **ID da taxa de IVA** | Lista das taxas ativas da sua conta, carregada em direto quando o token é válido. Aplicada a todas as linhas. |
| **Unidade** | Unidades / Metros / Caixas / Kilos / Litros. |
| **Tipo de item** | Serviço ou Produto. |
| **Enviar o e-mail do comprador para o Fact.pt** | Desligado por omissão. Guarda o e-mail na ficha de cliente do Fact.pt, o que permite ao próprio Fact.pt enviar o documento. |

## A taxa de IVA tem de coincidir com a do pretix

O Fact.pt recebe o preço **sem IVA** de cada linha e acrescenta-lhe a taxa configurada. Se essa taxa
for diferente da que o pretix cobrou, o documento ficaria com um valor que o comprador nunca pagou
(por exemplo, um bilhete de 15,00 € sem regra fiscal no pretix, faturado a 23%, passa a 18,45 €).

Por isso, antes de cada emissão, o plugin compara a taxa configurada com a taxa do pretix em cada
linha e **recusa com um erro explícito** se não coincidirem. Corrija a regra fiscal ou a definição e
tente novamente.

!!! note "Nota"
    Uma taxa de IVA por evento: um evento que venda artigos com taxas diferentes ainda não pode ser
    faturado. Veja as [Limitações conhecidas](limitations.md).

## Clientes

- **Com NIF** — é reutilizado um cliente do Fact.pt com exatamente esse NIF. Caso contrário, é criado
  um (ou atualizado, se o Fact.pt já tiver esse NIF) a partir do endereço de faturação da encomenda.
- **Sem NIF** — é reutilizado um cliente consumidor final com o mesmo nome; caso contrário, o
  Fact.pt cria um com o NIF `999999990`.

!!! warning "Clientes duplicados"
    Se vários clientes do Fact.pt tiverem o NIF do comprador, a emissão para com
    `Multiple clients with same tin. Specify an ID.` O plugin não adivinha a qual deles faturar —
    junte os duplicados no Backoffice do Fact.pt e tente novamente.

Um cliente criado ou atualizado a partir do pretix substitui a ficha desse NIF no Fact.pt com os
dados do endereço da encomenda.

## Proteção contra duplicados

Cada documento leva um `identifierId` (`pretix-<evento>-<código da encomenda>`), e o Fact.pt recusa
um segundo documento com o mesmo. Por isso, carregar em "Tentar novamente" depois de uma falha
ambígua nunca emite duas vezes.

Uma encomenda [paga de novo depois de um reembolso](usage.md#paga-de-novo-depois-de-um-reembolso)
recebe um novo por cada nova fatura (`…-r1`, `…-r2`), e cada nota de crédito usa o da sua fatura
seguido de `-credit`.

## Notas de crédito

Emitidas com o endpoint de crédito do Fact.pt, creditando o documento original na totalidade. O
Fact.pt só permite uma nota de crédito por documento, pelo valor total.
