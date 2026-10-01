# Moloni

Fornecedor [Moloni](https://moloni.pt) — identificador `moloni`.

A emissão de faturas-recibo e notas de crédito, e a descarga dos respetivos PDF, foram verificadas
com uma conta real do Moloni.

!!! warning "Confirme primeiro um documento com IVA"
    As linhas são enviadas **sem IVA**, e o Moloni acrescenta a taxa de IVA configurada. Os testes com
    a conta real usaram linhas a 0%, por isso emita um documento de teste com um bilhete com IVA e
    confirme o total antes de um evento a sério.

## Ligar ao Moloni

No topo das definições do Moloni, **Ligar ao Moloni** leva-o ao Moloni para autorizar o plugin e
depois volta à página de definições. Não é guardada nenhuma palavra-passe do Moloni, só os tokens de
acesso.

1. Na área de programador do Moloni, registe o **redirect URI** que aparece debaixo do botão. É o
   mesmo para todos os eventos: `https://<o seu pretix>/control/ptinvoicing/moloni/callback/`.
2. Preencha o **ID de programador** e o **Segredo do cliente** e carregue em **Ligar ao Moloni**. Não
   é preciso guardar primeiro.
3. Depois de ligado, o painel mostra **Ligado** e oferece **Voltar a ligar** e **Desligar**.

O refresh token do Moloni expira ao fim de **14 dias sem uso**. O plugin renova-o todos os dias em
cada evento ligado ao Moloni, por isso um evento sem vendas nunca perde a ligação. Se o Moloni
recusar a renovação, a ligação fica **expirada**: a página de definições indica-o, e o **e-mail de
contacto** do evento (pretix **Configurações → Geral**) recebe uma mensagem com uma ligação para voltar a
ligar.

!!! note "Utilizador e palavra-passe"
    Continuam a ser aceites como alternativa a **Ligar ao Moloni**, e ficam escondidos enquanto
    estiver ligado. Se estiverem preenchidos, uma ligação expirada volta a entrar sozinha, sem esperar
    que volte a ligar.

## Definições

A maioria dos campos passa a lista, preenchida a partir da sua conta Moloni, depois de ligado. Os
campos de cada empresa carregam depois de escolher a **Empresa**.

| Definição | Obrigatória | Descrição |
|---|---|---|
| **ID de programador (client_id)** | sim | Da área de programador do Moloni. |
| **Segredo do cliente** | sim | Da área de programador do Moloni. |
| **Utilizador / palavra-passe Moloni** | não | Só sem **Ligar ao Moloni** — ver acima. |
| **Empresa** | sim | Com várias empresas na conta, tem de escolher uma explicitamente. Cada opção mostra o nome e o NIF. |
| **Série de documentos** | sim | Série usada nas faturas-recibo. |
| **Série de documentos das notas de crédito** | sim | Uma série criada para o tipo de documento **Nota de Crédito**. Não pode ser a série das faturas-recibo. |
| **Taxa de IVA** | em bilhetes com IVA | Tem de ser a mesma taxa que o pretix cobra — ver [IVA](#iva). |
| **Motivo de isenção** | em bilhetes a 0% | Os códigos de isenção do Moloni, por exemplo `M07` (Artigo 9.º do CIVA). |
| **Forma de pagamento** | sim | Uma fatura-recibo leva sempre um pagamento; é a forma com que fica registado. |
| **Prazo de vencimento** | sim | Usado em cada novo cliente no Moloni, por exemplo "Pronto pagamento". |
| **Categoria de artigos** | sim | Onde o plugin cria os artigos do catálogo — ver [Artigos](#artigos). Só aparecem as categorias de topo. |
| **Tipo de artigo** | sim | *Serviço* (por omissão) ou *Produto*. |
| **Unidade** | sim | A unidade de medida dos artigos. |

## Artigos

O Moloni só fatura artigos do seu catálogo, por isso cada produto do pretix tem um, criado
automaticamente na primeira venda. É criado na categoria, tipo e unidade escolhidos, com a referência
`pretix-item-<id>`, e reutilizado daí em diante, por isso os relatórios de vendas do Moloni continuam
separados por tipo de bilhete. Cada linha do documento continua a mostrar o nome do produto no pretix
e o preço efetivamente pago.

## IVA

Cada linha segue o IVA que o pretix efetivamente cobrou:

- **Linha com IVA** — enviada com a **Taxa de IVA** configurada. Se essa taxa não for a que o pretix
  cobrou, a emissão pára com `Divergência de IVA: …` *antes* de criar o que quer que seja no Moloni.
  Caso contrário, o documento teria um valor diferente do que o comprador pagou.
- **Linha a 0%** — enviada sem IVA, com o **Motivo de isenção**. O Moloni exige um, por isso uma
  encomenda com linhas a 0% e sem motivo de isenção definido pára com um erro explícito.

## Clientes

Um comprador com NIF é associado a um cliente existente no Moloni quando há exatamente um com esse
NIF. Caso contrário, é criado um cliente a partir da morada de faturação da encomenda. Compradores
sem NIF são criados como consumidor final (`999999990`).

## Sem nova tentativa automática

O Moloni não tem nenhum campo de idempotência, por isso um pedido que expirou pode ou não ter criado
um documento. Para não emitir uma segunda fatura oficial, o plugin **nunca repete automaticamente**
uma emissão no Moloni: uma tentativa falhada fica como erro, para confirmar no Moloni e tentar de
novo à mão no Control.

## Notas de crédito

Construídas a partir das linhas do documento original e associadas a ele, creditando-o na
totalidade.
