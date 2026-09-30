# Configuração

Abra **Configurações → Faturação PT** do evento. (Chama-se "PT" para se distinguir da entrada
"Facturação" do próprio pretix.)

## Geral

| Definição | Por omissão | Descrição |
|---|---|---|
| **Fornecedor de faturação** | nenhum | Que fornecedor emite os documentos. Sem nenhum, nada é emitido. Só as definições do fornecedor selecionado são guardadas. |
| **O campo personalizado da morada de faturação contém o NIF do comprador** | desligado | Veja [NIF](#nif). |
| **Mostrar a fatura na página da encomenda do comprador** | ligado | Acrescenta um botão de descarga junto às descargas dos bilhetes. O PDF passa pelo pretix, por isso as credenciais do fornecedor nunca chegam ao browser. |

## E-mail

Os e-mails de encomenda do próprio pretix não conseguem anexar estes documentos (só anexam as faturas
do próprio pretix), por isso o plugin envia os seus.

| Definição | Por omissão | Descrição |
|---|---|---|
| **Enviar a fatura-recibo por e-mail ao comprador** | desligado | Envia o PDF assim que é emitido. |
| **Enviar a nota de crédito por e-mail ao comprador** | desligado | Envia o PDF da nota de crédito assim que é emitida. |

Um e-mail que falhe fica registado no log e nunca marca a emissão como falhada — o documento já
existe.

## Definições do fornecedor

Abaixo das definições gerais, preencha os campos do fornecedor selecionado:

- [Fact.pt](factpt.md#definicoes)
- [Moloni](moloni.md#definicoes)

Alguns campos (taxa de IVA, empresa, série de documentos…) passam a listas preenchidas em direto a
partir da sua conta no fornecedor, assim que as credenciais são válidas — não é preciso guardar
primeiro.

## NIF

Por omissão, o NIF do comprador é lido do campo **Número de IVA** do endereço de faturação do pretix.
O pretix só mostra esse campo a clientes empresa, por isso os particulares não têm onde indicar o
NIF.

Para o pedir a toda a gente:

1. Em **Configurações → Facturação** do pretix, crie um campo personalizado de destinatário com o
   rótulo, por exemplo, "NIF".
2. Ligue aqui **O campo personalizado da morada de faturação contém o NIF do comprador**.

O campo personalizado passa a ser usado quando o Número de IVA está vazio. Os NIF portugueses cujo
dígito de controlo não confira são ignorados, não são enviados.

Sem NIF, o documento é emitido a um **consumidor final** (`999999990`).
