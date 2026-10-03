# Configuração

## Definições partilhadas

Abra **Configurações → euPago** do evento. Estas definições são partilhadas por todos os métodos de
pagamento euPago, por isso só as define uma vez:

| Definição | Descrição |
|---|---|
| **Chave API** | A sua chave API da euPago, disponível em Backoffice → Canais → Listagem de Canais. |
| **Modo de sandbox** | Usa a sandbox da euPago em vez da produção. Precisa de uma chave API de sandbox, de sandbox.eupago.pt, porque as chaves de produção não funcionam lá. |
| **URL do webhook** | Só de leitura: o endereço a colar no Backoffice da euPago. Ver [Webhooks](webhooks.md). |
| **Chave do webhook** | A chave que a euPago gera para o webhook deste canal em Backoffice → Canais → Listagem de Canais → "Receber notificação para um URL". **Obrigatória** para o webhook 2.0: sem ela, os pagamentos não são confirmados por esse webhook. Também desencripta as notificações se ativar a encriptação do webhook na euPago. |

A chave API e a chave do webhook ficam ocultas depois de guardadas. Para mudar uma, escreva o novo
valor por cima.

A página avisa quando:

- não está definida nenhuma chave do webhook;
- o modo de sandbox está ativo mas o evento não está em modo de teste;
- a moeda do evento não é o euro.

!!! warning "O modo de sandbox só funciona em modo de teste"
    Os pagamentos em sandbox são fictícios. Para que uma sandbox esquecida não dê bilhetes reais de
    graça, o Multibanco e o MB WAY só são disponibilizados no checkout enquanto o evento está em
    **modo de teste**. Desligue o modo de sandbox antes de começar a vender.

## Métodos de pagamento

Ative e configure cada um individualmente em **Configurações → Pagamento**.

### Multibanco

| Definição | Descrição |
|---|---|
| **Validade da referência Multibanco (dias)** | Durante quantos dias uma referência gerada permanece válida (1–365). Nunca ultrapassa o prazo de pagamento do próprio pedido. |

A referência nunca ultrapassa o prazo de pagamento do pedido. Caso contrário, um comprador poderia
pagar uma referência antiga depois de o pedido ter expirado e os bilhetes terem sido vendidos a outra
pessoa.

A referência é enviada ao comprador por e-mail logo depois de ser gerada, num e-mail próprio no idioma
do comprador. Esse e-mail também inclui uma ligação para a página do pedido. O e-mail de "encomenda
efetuada" do próprio pretix sai *antes* de a referência existir, por isso não a pode incluir.

### MB WAY

Sem definições próprias. No checkout, o comprador indica um número de telemóvel português: 9 dígitos
começados por 9, com ou sem `+351`. Números estrangeiros são recusados. O comprador tem depois
**5 minutos** para aprovar o pagamento na aplicação MB WAY.

Se a euPago recusar o pedido, por exemplo porque o número não está registado no MB WAY, o comprador é
avisado de imediato e pode tentar outro número ou método. Se o pedido expirar ou for recusado na
aplicação, o comprador pode pagar novamente na página do pedido.

!!! note "Nota"
    O número de telemóvel do comprador é apagado do registo do pagamento quando o pretix elimina os
    dados pessoais.
