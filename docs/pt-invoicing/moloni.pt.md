# Moloni

Fornecedor [Moloni](https://moloni.pt) — identificador `moloni`.

!!! warning "Ainda não verificado com uma conta real"
    Este fornecedor foi feito a partir da documentação pública da API do Moloni, confirmado com o
    código do plugin oficial do Moloni para WooCommerce, e testado apenas com mocks. Em particular,
    não está confirmado se o `price` do Moloni é com ou sem IVA. **Emita documentos de teste e
    confirme os totais antes de o usar num evento a sério.**

## Definições

As credenciais vêm da área de programador do Moloni.

| Definição | Obrigatória | Descrição |
|---|---|---|
| **ID de programador (client_id)** | sim | |
| **Segredo do cliente** | sim | |
| **Utilizador Moloni** | sim | |
| **Palavra-passe Moloni** | sim | |
| **Empresa** | sim | Lista, carregada quando as credenciais são válidas. |
| **Série de documentos** | sim | Série usada nas faturas-recibo. |
| **Série de documentos das notas de crédito** | sim | Uma série criada para o tipo de documento **Nota de Crédito**. Não pode ser a série das faturas-recibo. |
| **Taxa de IVA** | não | Lista das taxas da sua conta. |
| **Motivo de isenção** | não | Obrigatório nas linhas a 0%, por exemplo `M07` (Artigo 9.º do CIVA). |
| **Forma de pagamento** | sim | Uma fatura-recibo leva sempre um pagamento; é a forma com que fica registado. |

O token de acesso fica guardado nas definições do evento e é renovado automaticamente.

## Sem nova tentativa automática

O Moloni não tem nenhum campo de idempotência, por isso um pedido que expirou pode ou não ter criado
um documento. Para não emitir uma segunda fatura oficial, o plugin **nunca repete automaticamente**
uma emissão no Moloni: uma tentativa falhada fica como erro, para confirmar no Moloni e tentar de
novo à mão no Control.

## Notas de crédito

Construídas a partir das linhas do documento original e associadas a ele, creditando-o na
totalidade.
