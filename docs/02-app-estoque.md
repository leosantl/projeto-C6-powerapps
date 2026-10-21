# App do Estoque (Power Apps), Etapa 1

São três telas:

- **Estoque:** lista com busca e leitor de código de barras;
- **Movimentar:** entrada ou saída, com o histórico do item;
- **Cadastro:** novo item ou edição.

Antes, crie as listas `Perifericos` e `Movimentacoes` ([01-listas.md](01-listas.md#etapa-1-estoque)).

## Sobre as fórmulas

As fórmulas estão no padrão do **Power Apps em português**:

- `;` separa os argumentos;
- `;;` separa um comando do outro.

Se o seu Power Apps estiver em inglês, troque `;` por `,` e `;;` por `;`. Os nomes das funções (`Filter`, `Patch`,
`Notify`...) são iguais nos dois.

Cada bloco diz **em qual controle** e **em qual propriedade** colar. A propriedade se escolhe na lista à esquerda
da barra de fórmulas.

---

## 1. Criar o app a partir da lista

1. Em **make.powerapps.com**, clique em **+ Criar → SharePoint**, escolha o site **Controle de Estoque** e a lista
   **Perifericos**.
2. O Power Apps gera três telas: navegação (lista), detalhes e edição. Renomeie as telas, clicando com o botão
   direito → Renomear:
   - `BrowseScreen1` → `telaEstoque`
   - `DetailScreen1` → `telaDetalhe`. Deixe como está: ela não será usada, porque a tela Movimentar faz esse papel
   - `EditScreen1` → `telaCadastro`
3. Adicione a outra lista: **Dados** (ícone de cilindro) → **+ Adicionar dados → SharePoint** → o mesmo site →
   **Movimentacoes**.
4. Salve: **Arquivo → Salvar como** → `Controle de Estoque`.

---

## 2. Tela Estoque (`telaEstoque`)

### Campo de busca

Use o campo de busca que já veio na tela (`TextSearchBox1`) e renomeie para `txtBusca`. Um **leitor USB** funciona
como teclado: ele digita o código e aperta Enter.

`txtBusca` → **HintText**:
```
"Buscar por modelo, marca ou código de barras"
```

`txtBusca` → **OnChange** (o leitor USB digita o código e abre o item direto):
```
If(
    !IsBlank(LookUp(Perifericos; CodigoBarras = txtBusca.Text));
    Set(varItem; LookUp(Perifericos; CodigoBarras = txtBusca.Text));;
    Reset(txtBusca);;
    Navigate(telaMovimentar)
)
```

### Lista de itens

Na galeria que já veio (`BrowseGallery1`):

**Items**:
```
SortByColumns(
    Filter(
        Perifericos;
        Ativo = true &&
        (IsBlank(txtBusca.Text) || StartsWith(Title; txtBusca.Text) || StartsWith(Marca; txtBusca.Text) || CodigoBarras = txtBusca.Text)
    );
    "Title";
    SortOrder.Ascending
)
```

Os rótulos da galeria (os textos de cada linha):

Primeiro rótulo → **Text**:
```
ThisItem.Categoria.Value & " · " & ThisItem.Marca & " " & ThisItem.Title
```

Segundo rótulo → **Text**:
```
ThisItem.Quantidade & " em estoque" & If(ThisItem.Minimo > 0; " · mínimo " & ThisItem.Minimo; "")
```

Segundo rótulo → **Color** (vermelho esgotado, laranja baixo):
```
If(ThisItem.Quantidade = 0; Color.Red; ThisItem.Quantidade <= ThisItem.Minimo; Color.DarkOrange; Color.Black)
```

Galeria → **OnSelect** (abre a movimentação do item tocado):
```
Set(varItem; ThisItem);;
Navigate(telaMovimentar)
```

### Leitor de código de barras (câmera do celular/tablet)

**Inserir → Mídia → Leitor de código de barras**. Renomeie para `leitorCodigo` e coloque no topo da tela.

`leitorCodigo` → **OnScan**:
```
Set(varCodigo; First(leitorCodigo.Barcodes).Value);;
Set(varItem; LookUp(Perifericos; CodigoBarras = varCodigo));;
If(
    IsBlank(varItem);
    Notify("Código " & varCodigo & " não cadastrado. Cadastre o item e informe esse código."; NotificationType.Warning);
    Navigate(telaMovimentar)
)
```

O leitor pela câmera funciona no app **Power Apps** do celular ou do tablet (baixe na loja e entre com a conta da
empresa). No computador, use o leitor USB no campo de busca.

O botão **+** que já veio na tela abre o cadastro de um item novo e não precisa mudar.

---

## 3. Tela Movimentar (`telaMovimentar`)

**Nova tela → Em branco**, com o nome `telaMovimentar`. Insira os controles abaixo, de cima para baixo.

| Controle | Tipo (Inserir →) | Nome |
|---|---|---|
| Voltar | Ícone → Seta para a esquerda | `icoVoltar` |
| Nome do item | Rótulo de texto | `lblItem` |
| Saldo | Rótulo de texto | `lblSaldo` |
| Entrada ou saída | Rádio | `rdTipo` |
| Quantidade | Entrada de texto | `txtQtd` |
| Responsável | Entrada de texto | `txtResponsavel` |
| Motivo | Entrada de texto | `txtMotivo` |
| Registrar | Botão | `btnRegistrar` |
| Editar cadastro | Botão | `btnEditar` |
| Histórico | Galeria vertical em branco | `galHistorico` |

### Fórmulas da tela

`icoVoltar` → **OnSelect**:
```
Back()
```

`lblItem` → **Text**:
```
varItem.Categoria.Value & " · " & varItem.Marca & " " & varItem.Title
```

`lblSaldo` → **Text**:
```
"Em estoque: " & varItem.Quantidade & If(varItem.Minimo > 0; " (mínimo " & varItem.Minimo & ")"; "")
```

`rdTipo` → **Items** e **Default**:
```
["Saída"; "Entrada"]
```
```
"Saída"
```

`txtQtd`:
- **Format**: `TextFormat.Number`
- **HintText**: `"Quantidade"`

`txtResponsavel` → **HintText**:
```
"Quem retirou / quem entregou"
```

`txtMotivo` → **HintText**:
```
"Destino ou origem (colaborador, compra, devolução...)"
```

### Botão Registrar

`btnRegistrar` → **Text**:
```
"Registrar " & Lower(rdTipo.Selected.Value)
```

`btnRegistrar` → **OnSelect**. Ele lê o saldo atual direto da lista antes de gravar e não deixa o saldo ficar
negativo:
```
Set(varAtual; LookUp(Perifericos; ID = varItem.ID));;
Set(varQtd; Value(txtQtd.Text));;
Set(varNovo; varAtual.Quantidade + If(rdTipo.Selected.Value = "Entrada"; varQtd; -varQtd));;
If(
    IsBlank(varQtd) || varQtd <= 0 || varQtd <> RoundDown(varQtd; 0);
        Notify("Informe uma quantidade inteira maior que zero."; NotificationType.Error);
    varNovo < 0;
        Notify("Saldo insuficiente: há " & varAtual.Quantidade & " em estoque."; NotificationType.Error);
    Patch(Perifericos; varAtual; {Quantidade: varNovo});;
    Patch(
        Movimentacoes;
        Defaults(Movimentacoes);
        {
            Title: varAtual.Categoria.Value & " · " & varAtual.Marca & " " & varAtual.Title;
            PerifericoID: varAtual.ID;
            Tipo: {Value: rdTipo.Selected.Value};
            Quantidade: varQtd;
            SaldoApos: varNovo;
            Responsavel: txtResponsavel.Text;
            Motivo: txtMotivo.Text;
            Data: Now()
        }
    );;
    Set(varItem; LookUp(Perifericos; ID = varAtual.ID));;
    If(
        varNovo <= varAtual.Minimo;
        Notify("Registrado. Atenção: estoque baixo (" & varNovo & ")."; NotificationType.Warning);
        Notify("Movimentação registrada."; NotificationType.Success)
    );;
    Reset(txtQtd);;
    Reset(txtResponsavel);;
    Reset(txtMotivo)
)
```

### Editar cadastro e histórico

`btnEditar` → **Text** e **OnSelect**:
```
"Editar cadastro"
```
```
EditForm(EditForm1);;
Navigate(telaCadastro)
```

`galHistorico` → **Items** (as movimentações deste item, da mais recente para a mais antiga):
```
SortByColumns(Filter(Movimentacoes; PerifericoID = varItem.ID); "Data"; SortOrder.Descending)
```

Dentro da galeria, insira dois rótulos.

Primeiro rótulo → **Text**:
```
Text(ThisItem.Data; "dd/mm/yyyy hh:mm") & " · " & ThisItem.Tipo.Value & " de " & ThisItem.Quantidade & " · saldo " & ThisItem.SaldoApos
```

Segundo rótulo → **Text**. Se o site do SharePoint estiver em inglês, troque `'Criado por'` por `'Created By'`:
```
ThisItem.Responsavel & If(IsBlank(ThisItem.Motivo); ""; " — " & ThisItem.Motivo) & " · lançado por " & ThisItem.'Criado por'.DisplayName
```

---

## 4. Tela Cadastro (`telaCadastro`)

É a tela de edição que o Power Apps gerou. Ajuste o formulário (`EditForm1`):

1. **Tire o cartão `Quantidade` do formulário.** Selecione o formulário → **Editar campos** → ⋯ ao lado de
   Quantidade → **Remover**. O saldo só muda pela tela Movimentar. Item novo começa com 0 (padrão da lista); depois
   registre uma **Entrada**.
2. Na mesma janela, deixe na ordem: Categoria, Marca, Modelo, Código de barras, Mínimo, Local, Descrição, Ativo.

`EditForm1` → **Item**. Para item novo, o formulário ignora este valor e começa em branco:
```
varItem
```

`EditForm1` → **OnSuccess** (depois de salvar, volta para o item):
```
Set(varItem; EditForm1.LastSubmit);;
Navigate(telaMovimentar)
```

O botão **+** da tela Estoque já chama `NewForm(EditForm1)`, e o ✓ da tela de cadastro já chama
`SubmitForm(EditForm1)`. Não precisa mudar.

---

## 5. Publicar e compartilhar

1. **Arquivo → Salvar → Publicar**.
2. **Compartilhar**: adicione as pessoas da equipe, ou o grupo do Teams. Elas também precisam ter acesso ao site do
   SharePoint, o que já acontece se forem membros da equipe do Teams.
3. Para abrir:
   - no **computador**, em **make.powerapps.com → Apps**, ou fixe o app como guia no Teams (**+ → Power Apps**);
   - no **celular/tablet**, pelo app **Power Apps**.

## Testar antes de usar

1. Cadastre um item de teste com mínimo 2 e código de barras `TESTE1`.
2. Registre uma **Entrada** de 5. O saldo vai para 5 e aparece no histórico.
3. Registre uma **Saída** de 10. O app recusa com *"Saldo insuficiente: há 5 em estoque"*.
4. Registre uma **Saída** de 3. O saldo vai para 2 e o app avisa de estoque baixo.
5. Digite `TESTE1` na busca e aperte Enter (como faz o leitor USB). O app abre direto o item.
6. Na lista `Movimentacoes` do SharePoint, confira as duas linhas, com "Criado por" preenchido.
7. Apague o item de teste e as movimentações dele.
