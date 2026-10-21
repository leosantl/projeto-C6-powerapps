# Trazer os dados do sistema local

Faça isso **uma vez**, antes de a equipe começar a usar o app. O sistema local continua funcionando até lá.

## Estoque (Etapa 1)

1. **No sistema local:**
   1. Abra **Estoque → Relatório quinzenal → Gerar agora**.
   2. Na linha do relatório, clique em **Excel**.
   3. A planilha traz Categoria, Marca, Modelo, Local, Em estoque, Mínimo e Situação.
2. **No Excel:** deixe as colunas na ordem da lista `Perifericos` no modo de grade. Clique na lista para ver a ordem;
   a que vale é a da exibição **Todos os itens**:

   | Modelo | Categoria | Marca | Código de barras | Local | Quantidade | Mínimo |
   |---|---|---|---|---|---|---|

   - A **categoria** precisa estar escrita igual às escolhas da lista (Mouse, Teclado, Monitor...). O sistema local
     já exporta assim.
   - O **código de barras** não vem no relatório. Deixe vazio e preencha depois, de um destes jeitos:
     - no modo de grade, clicando na célula e lendo a etiqueta com o leitor USB;
     - no app, em **Editar cadastro**.
3. **No SharePoint:**
   1. Abra a lista `Perifericos` e clique em **Editar no modo de grade**.
   2. Selecione a primeira célula da coluna Modelo e **cole** (Ctrl+V) as linhas do Excel.
   3. Clique em **Sair do modo de grade**.
4. **Saldo inicial:** na importação, a coluna Quantidade recebe o saldo atual direto. Essa é a única vez em que o
   saldo é preenchido fora do app.
   - Para deixar registrado, crie em `Movimentacoes` uma linha por item: Tipo **Entrada**, Quantidade = saldo,
     Motivo `Saldo inicial (sistema local)`.
   - Ou simplesmente anote a data da migração.
   - O histórico antigo de movimentações continua consultável no sistema local e no backup dele.

## Hubs, Reparos, Maneja e Onboarding (etapas 2 a 5)

O caminho é o mesmo:
1. No sistema local, use **Exportar → Excel** na aba.
2. No Excel, coloque as colunas na ordem da lista.
3. Cole no **modo de grade**.

Pontos de atenção:
- **Status** e escolhas precisam estar escritos igual às opções da lista. Por exemplo: *Em aprovação*, *Aguardando
  entrega*, *Aguardando vinculação*.
- **Datas** no formato `dd/mm/aaaa`.
- **Itens recebidos** e **tipos de avaria** da Maneja, com várias opções, devem ser preenchidos no próprio
  SharePoint, porque a colagem não preenche colunas de várias seleções. Para os registros antigos, você pode colar o
  texto em **Observações**.
- Em **Hubs**, o número da compra vira o **ID** do SharePoint, que recomeça do 1. Se precisar do número antigo, cole
  em **Observações** (ex.: `Nº antigo: 57`).

## Conferir

- Compare a **quantidade de itens** e o **total em estoque** do relatório do sistema local com a lista:
  1. no SharePoint, crie uma exibição com **Totais → Quantidade: Soma**;
  2. exporte a lista para o Excel e some lá.
- Escolha 3 itens e confira saldo, mínimo e local um a um.
