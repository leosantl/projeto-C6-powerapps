# Estoque no Power Apps dentro da Field Services

Versão para o site compartilhado **Field Services** (`/sites/ControledeEstoque`): listas próprias com o prefixo `ControleEstoque`, sem tocar nas listas, páginas e permissões que já existem. Abaixo, o texto completo para colar no Claude (Cowork) ou seguir à mão.

```text
Responda sempre em português, de forma simples. Use o navegador que está aberto, com a MINHA conta da empresa já logada.
Vá passo a passo. Antes de criar ou salvar qualquer coisa, me diga o que vai fazer e espere eu responder "ok".

OBJETIVO
Montar o "Controle de Estoque" (só a parte de Estoque) no Power Apps, com os dados em duas listas NOVAS do site
SharePoint Field Services: https://futureinvest.sharepoint.com/sites/ControledeEstoque

REGRAS (obrigatórias)
- NÃO altere, renomeie, apague nem edite nada que já existe na Field Services: listas (como Compras, Estoque, Controle,
  Celulares), páginas, a página inicial, o menu, documentos ou permissões. Só crie as duas listas abaixo.
- A página inicial da Field Services tem um rascunho: NUNCA clique em "Publicar alterações" nela.
- Não mude configurações do site nem de permissões. Não compartilhe nada com ninguém sem eu pedir.
- Use só dados de teste inventados. Não digite senhas: se pedir login, pare e me chame.
- Se aparecer erro, "acesso negado" ou algo diferente do esperado, PARE, me mostre a tela e não tente contornar.

PARTE 1: LISTAS NA FIELD SERVICES (+ Novo → Lista → Lista em branco)
Lista 1: nome ControleEstoquePerifericos (exatamente assim). Renomeie a coluna Título para "Modelo". Colunas:
- Categoria: Escolha (Mouse, Teclado, Monitor, Headset, Celular, Adaptador, Cabo HDMI, Cabos diversos, Ferramentas,
  Material de escritório, Adaptador de rede, Tablet, Desktop)
- Marca: Texto
- CodigoBarras: Texto
- Local: Texto
- Quantidade: Número, valor padrão 0
- Minimo: Número, valor padrão 0
- Ativo: Sim/Não, padrão Sim
Lista 2: nome ControleEstoqueMovimentacoes. Renomeie Título para "Item". Colunas:
- PerifericoID: Número
- Tipo: Escolha (Entrada, Saída)
- Quantidade: Número
- SaldoApos: Número
- Responsavel: Texto
- Motivo: Texto
- Data: Data e hora, incluir hora
Use exatamente esses nomes de coluna (sem acento). Ao terminar, me mostre as duas listas.

PARTE 2: APP NO POWER APPS (make.powerapps.com)
1. + Criar → SharePoint → site Field Services → lista ControleEstoquePerifericos.
2. Renomeie as telas: BrowseScreen1 → telaEstoque; EditScreen1 → telaCadastro (DetailScreen1 fica sem uso).
3. Dados → + Adicionar dados → SharePoint → mesmo site → ControleEstoqueMovimentacoes.
4. Salve como "Controle de Estoque".
5. Crie a tela em branco telaMovimentar com os controles da tabela abaixo e cole as fórmulas, no controle e na
   propriedade indicados. As fórmulas estão no padrão do Power Apps em português (";" entre argumentos e ";;" entre
   comandos). Se o Power Apps estiver em inglês, troque ";" por "," e ";;" por ";".
6. Visual: fundo das telas RGBA(13;13;13;1); textos RGBA(242;242;242;1); cabeçalho preto RGBA(0;0;0;1);
   botões com fundo RGBA(242;242;242;1) e texto preto. Não escreva "C6" como texto em nenhum lugar.
7. Salve. NÃO publique nem compartilhe ainda: me avise que vou testar.

TESTE (me guie, eu clico)
- Cadastrar item de teste com mínimo 2 e código TESTE1; registrar Entrada de 5 (saldo 5); tentar Saída de 10 (deve
  recusar "Saldo insuficiente"); Saída de 3 (saldo 2 e aviso de estoque baixo); digitar TESTE1 na busca (abre o item).
- Depois apagar o item de teste e as movimentações dele.

FÓRMULAS
```

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
    !IsBlank(LookUp(ControleEstoquePerifericos; CodigoBarras = txtBusca.Text));
    Set(varItem; LookUp(ControleEstoquePerifericos; CodigoBarras = txtBusca.Text));;
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
        ControleEstoquePerifericos;
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

Segundo rótulo → **Color** (vermelho sem estoque, amarelo baixo, verde OK):
```
If(ThisItem.Quantidade = 0; RGBA(229;72;77;1); ThisItem.Quantidade <= ThisItem.Minimo; RGBA(245;165;36;1); RGBA(48;164;108;1))
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
Set(varItem; LookUp(ControleEstoquePerifericos; CodigoBarras = varCodigo));;
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
Set(varAtual; LookUp(ControleEstoquePerifericos; ID = varItem.ID));;
Set(varQtd; Value(txtQtd.Text));;
Set(varNovo; varAtual.Quantidade + If(rdTipo.Selected.Value = "Entrada"; varQtd; -varQtd));;
If(
    IsBlank(varQtd) || varQtd <= 0 || varQtd <> RoundDown(varQtd; 0);
        Notify("Informe uma quantidade inteira maior que zero."; NotificationType.Error);
    varNovo < 0;
        Notify("Saldo insuficiente: há " & varAtual.Quantidade & " em estoque."; NotificationType.Error);
    Patch(ControleEstoquePerifericos; varAtual; {Quantidade: varNovo});;
    Patch(
        ControleEstoqueMovimentacoes;
        Defaults(ControleEstoqueMovimentacoes);
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
    Set(varItem; LookUp(ControleEstoquePerifericos; ID = varAtual.ID));;
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
SortByColumns(Filter(ControleEstoqueMovimentacoes; PerifericoID = varItem.ID); "Data"; SortOrder.Descending)
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
2. Na mesma janela, deixe na ordem: Categoria, Marca, Modelo, Código de barras, Mínimo, Local, Ativo.

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

