# Configuração

## Definições partilhadas

Em **Configurações → Pagamento → euPago**. São partilhadas por todos os métodos de pagamento euPago,
por isso só as define uma vez:

| Definição | Descrição |
|---|---|
| **Chave API** | A sua chave API da euPago, disponível em Backoffice → Canais → Listagem de Canais. |
| **Sandbox / Modo de teste** | Usa o ambiente de sandbox da euPago em vez do de produção. |
| **Segredo de assinatura do webhook** | Opcional, fortemente recomendado. A chave de encriptação gerada para o webhook deste canal em Backoffice → Canais → Listagem de Canais → "Receber notificação para um URL". Quando definida, as notificações v2.0 só são aceites se a assinatura corresponder. |

## Métodos de pagamento

Ative e configure cada um individualmente em **Configurações → Pagamento**.

### Multibanco

| Definição | Descrição |
|---|---|
| **Validade da referência Multibanco (dias)** | Durante quantos dias uma referência gerada permanece válida (1–365). |

A referência é enviada ao comprador por e-mail logo depois de ser gerada, num e-mail próprio. O e-mail
de "encomenda efetuada" do próprio pretix sai *antes* de a referência existir, por isso não a pode
incluir.

### MB WAY

Sem definições próprias. O comprador indica o número de telemóvel no checkout e tem **5 minutos**
para aprovar o pagamento na aplicação MB WAY. Se o pedido expirar, pode voltar a tentar o pagamento
na página da encomenda.

!!! note "Nota"
    O número de telemóvel do comprador é apagado do registo do pagamento quando o pretix elimina os
    dados pessoais.
