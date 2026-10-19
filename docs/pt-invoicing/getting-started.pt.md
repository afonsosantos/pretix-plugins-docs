# Primeiros passos

Da instalação à primeira fatura-recibo, passo a passo. Conte com cerca de meia hora, a maior parte
do lado do serviço de faturação.

**Antes de começar**, tenha à mão:

- uma instalação do **pretix** (≥ 2024.1) onde possa instalar pacotes, com os workers do Celery a
  correr;
- uma conta num serviço de faturação: um token da API do [Fact.pt](https://fact.pt) associado a uma
  série com **8 casas decimais**, ou uma conta [Moloni](https://moloni.pt) com uma aplicação de
  programador (a página de cada serviço diz exatamente o que criar);
- a taxa de IVA a que os bilhetes são vendidos, tal como está em **Configurações → Impostos** do
  evento.

## 1. Instalar o plugin

Instale-o no mesmo ambiente Python de onde o pretix corre, aplique a migração e reinicie o servidor
web **e** os workers do Celery (os plugins só são descobertos no arranque):

```bash
pip install pretix-pt-invoicing
python -m pretix migrate
```

## 2. Ativar no evento

Abra **Configurações → Plugins** do evento, mude para o separador **Integrações** e carregue em
**Ativar** em **Faturação portuguesa**.

![O separador Integrações da página de plugins, com a Faturação portuguesa ativa](img/plugins.png)

Aparecem duas entradas novas na barra lateral do evento: **Configurações → Faturação PT**, onde o
plugin se configura, e **Faturação PT** mais abaixo, o painel com todos os documentos emitidos. As
duas chamam-se "PT" para se distinguirem das entradas "Facturação" do próprio pretix, que dizem
respeito às faturas internas do pretix, não a estas.

## 3. Escolher o serviço de faturação

Abra **Configurações → Faturação PT** e clique no cartão do serviço por onde vai emitir. Só as
definições do serviço selecionado são mostradas, guardadas e usadas.

![A página de definições Faturação PT: os cartões dos serviços e as opções gerais](img/settings-general.png)

Por agora deixe as opções gerais como estão. Estão explicadas em [Configuração](configuration.md); a
que provavelmente vai querer mais tarde é **O campo personalizado da morada de faturação contém o
NIF do comprador**, configurada no passo 6.

!!! info "Ainda não está em uso"
    Escolher um cartão só mostra as definições desse serviço. Até carregar em **Guardar** com tudo o
    que o serviço precisa preenchido, um aviso azul no topo da sua secção diz *Ainda não está em
    uso*, e não é emitido nenhum documento por ele.

## 4. Ligar ao serviço

Preencha a secção do serviço no fundo da mesma página e carregue em **Guardar**. Cada serviço tem o
seu guia:

- [Configurar o Fact.pt](factpt.md#configurar-o-factpt): cole o token da API, marque **Usar
  ambiente de testes (sandbox)** enquanto testa, escolha a taxa de IVA na lista que aparece.
- [Configurar o Moloni](moloni.md#configurar-o-moloni): registe o redirect URI numa aplicação de
  programador do Moloni, carregue em **Ligar ao Moloni** e depois escolha a empresa, as séries de
  documentos e o resto nas listas.

Listas como a taxa de IVA ou a empresa são preenchidas em direto a partir da sua conta assim que as
credenciais escritas forem válidas, por isso não é preciso guardar primeiro para as ver. Se um
campo ficar uma caixa numérica simples, a linha de estado por baixo da secção diz porquê (token
errado, serviço inacessível).

## 5. Acertar a taxa de IVA

Os dois serviços recebem o preço **sem IVA** de cada linha e acrescentam-lhe a taxa que têm
configurada. Por isso o plugin recusa emitir quando a taxa do serviço é diferente da que o pretix
cobrou, em vez de emitir um documento com um valor que o comprador nunca pagou.

Confirme que a taxa escolhida no passo 4 é a da regra de IVA do evento (**Configurações →
Impostos**). Um evento que venda bilhetes a várias taxas ainda não pode ser faturado; veja as
[Limitações conhecidas](limitations.md).

## 6. Pedir o NIF ao comprador

O pretix só pede o Número de IVA a clientes empresa, por isso um particular não tem onde escrever o
NIF e seria faturado como consumidor final. Para o pedir a toda a gente:

1. Em **Configurações → Facturação** do pretix, abra o separador **Formulário de endereço** e
   preencha **Custom recipient field label** com `NIF`. Confirme que é pedido um endereço de
   faturação (**Pedir endereço de facturação**).

    ![As definições de faturação do pretix com o campo personalizado de destinatário chamado NIF](img/pretix-invoice-custom-field.png)

2. De volta a **Configurações → Faturação PT**, marque **O campo personalizado da morada de
   faturação contém o NIF do comprador** e guarde.

No checkout, o comprador passa a ver um único campo **NIF** no formulário da morada de faturação,
quer compre como particular quer como empresa. Em moradas portuguesas o campo de Número de IVA do
próprio pretix fica escondido, por isso nunca há um segundo sítio para escrever o número.

![O formulário da morada de faturação no checkout, com um único campo NIF](img/checkout-nif.png)

Veja [NIF](configuration.md#nif) para saber como o número é validado e o que acontece sem ele.

## 7. Escolher o que o comprador recebe

Ainda em **Configurações → Faturação PT**:

- **Enviar a fatura-recibo por e-mail ao comprador** e **Enviar a nota de crédito por e-mail ao
  comprador** enviam cada documento em PDF assim que é emitido. As duas opções estão desligadas por
  omissão; os e-mails de encomenda do próprio pretix não conseguem levar estes documentos.
- **Mostrar a fatura na página da encomenda do comprador** (ligada por omissão) acrescenta botões
  de descarga junto às descargas dos bilhetes.

![A página da encomenda do comprador com os botões para descarregar a fatura e a nota de crédito](img/buyer-order-page.png)

## 8. Emitir um documento de teste

Mantenha o evento em **modo de teste** e o serviço na sandbox (Fact.pt) ou numa empresa de testes
(Moloni) para isto.

1. Compre um bilhete na loja e marque a encomenda como paga no Control, ou marque como paga uma
   encomenda de teste já existente.
2. Abra a encomenda. O painel **Faturação PT** mostra a linha da fatura passados poucos segundos,
   com o número do documento e um botão **Descarregar PDF**.

    ![O painel Faturação PT da página da encomenda com uma fatura emitida](img/order-panel.png)

3. Abra o PDF e confira o **total**, a divisão do **IVA** e o **cliente** (nome e NIF). Se a
   encomenda foi paga antes de o plugin estar configurado, use **Emitir fatura agora** no mesmo
   painel.

O painel **Faturação PT** da barra lateral lista todos os documentos do evento, com o estado e
eventuais erros. [Emissão e notas de crédito](usage.md) explica o resto: notas de crédito, novas
tentativas e o que significa cada falha.

## 9. Passar a produção

- Fact.pt: cole o token de **produção** e desmarque **Usar ambiente de testes (sandbox)**. Os tokens
  são diferentes.
- Moloni: ligue à empresa real e escolha as suas séries de documentos reais.
- Tire o evento do modo de teste.

!!! warning "Antes de usar em produção"
    Emita primeiro alguns documentos de teste na sandbox do serviço e confira os valores, o cliente e
    o IVA nos PDF. Um documento emitido é comunicado à AT e só pode ser anulado com uma nota de
    crédito.
