# Listas do SharePoint

Todas as listas ficam no site do SharePoint da equipe **Controle de Estoque** (veja o
[Passo 0](../README.md#passo-0-conferir-o-acesso)).

## Como criar cada lista

1. No site, clique em **+ Novo → Lista → Lista em branco**. Dê o **nome exato** da tabela, sem acento (por exemplo,
   `Perifericos`). O nome sem acento evita problemas nas fórmulas. Depois você pode trocar o nome de exibição.
2. A coluna **Título** já existe. Renomeie conforme a tabela: clique no título da coluna → **Configurações da
   coluna → Editar**. O nome interno continua `Title`.
3. Para cada linha da tabela, clique em **+ Adicionar coluna**, escolha o tipo e use o **nome exato** da primeira
   coluna da tabela (sem acento). As opções (obrigatória, padrão, escolhas) ficam em **Mais opções**.
4. Onde houver **Validação da lista**, abra **Configurações da lista → Configurações de validação**, cole a fórmula
   e a mensagem.

As fórmulas do SharePoint (validação e colunas calculadas) estão escritas com **`;`** separando os argumentos, que é o
padrão de sites em português (Brasil). Se o SharePoint recusar a fórmula, troque `;` por `,`: isso acontece quando o
site está com as configurações regionais em inglês.

O **histórico de versões** já vem ligado em listas novas. Ele registra quem criou e quem alterou cada item, e o que
mudou. Para conferir: **Configurações da lista → Configurações de controle de versão**.

---

## Etapa 1: Estoque

### Lista `Perifericos`

| Coluna (nome interno) | Tipo | Opções |
|---|---|---|
| `Title` → renomear para **Modelo** | Linha de texto | Obrigatória |
| `Categoria` | Escolha | Obrigatória. Escolhas: Mouse, Teclado, Monitor, Headset, Celular, Adaptador, Cabo HDMI, Cabos diversos, Ferramentas, Material de escritório, Adaptador de rede, Tablet, Desktop |
| `Marca` | Linha de texto | Obrigatória |
| `CodigoBarras` → exibir como **Código de barras** | Linha de texto | |
| `Descricao` → **Descrição** | Várias linhas de texto | |
| `Local` | Linha de texto | |
| `Quantidade` | Número | Casas decimais: 0. Valor padrão: **0**. Mínimo: 0. **Só o app altera** (por movimentação) |
| `Minimo` → **Mínimo** | Número | Casas decimais: 0. Valor padrão: 0. Mínimo: 0. Alerta quando o saldo chega aqui |
| `Ativo` | Sim/Não | Padrão: Sim |

**Exibições sugeridas** (em **Todos os itens → Criar nova exibição**):
- **Estoque baixo:** filtro `Quantidade` menor ou igual a… Como o SharePoint não compara duas colunas no filtro, use
  a coluna calculada abaixo.
- Coluna calculada `Situacao` → **Situação**:
  - tipo **Calculada**, retorna **Linha de texto**;
  - fórmula: `=IF([Quantidade]=0;"Esgotado";IF([Quantidade]<=[Minimo];"Baixo";"OK"))`.

  Depois, filtre a exibição por `Situação` diferente de `OK`.

### Lista `Movimentacoes`

Cada entrada ou saída é uma linha. O app grava a movimentação e atualiza o saldo do periférico.

| Coluna (nome interno) | Tipo | Opções |
|---|---|---|
| `Title` → renomear para **Item** | Linha de texto | Obrigatória (o app grava "Categoria · Marca Modelo") |
| `PerifericoID` | Número | Obrigatória. Casas decimais: 0 (ID do item em `Perifericos`) |
| `Tipo` | Escolha | Obrigatória. Escolhas: Entrada, Saída |
| `Quantidade` | Número | Obrigatória. Casas decimais: 0. Mínimo: 1 |
| `SaldoApos` → **Saldo após** | Número | Casas decimais: 0 |
| `Responsavel` → **Responsável** | Linha de texto | Quem retirou ou entregou |
| `Motivo` | Linha de texto | Destino ou origem (colaborador, compra, devolução...) |
| `Data` | Data e hora | Inclui hora. Padrão: data de hoje |

A coluna automática **Criado por** registra quem lançou a movimentação.

**Movimentações não devem ser editadas nem apagadas. Opcional, recomendado:**
1. Em **Configurações da lista → Permissões para esta lista**, pare de herdar as permissões.
2. Dê aos membros da equipe um nível de permissão **"Adicionar sem editar"**. Para criar esse nível, use, nas
   configurações do site, **Permissões do site → Níveis de permissão**, copie *Colaborar* e desmarque
   *Editar itens* e *Excluir itens*.
3. Os proprietários continuam podendo corrigir.

---

## Etapa 2: Hubs (compras)

### Lista `Compras`

| Coluna | Tipo | Opções |
|---|---|---|
| `Title` → **Modelo** | Linha de texto | Obrigatória |
| `Hub` | Linha de texto | Hub/site que pediu |
| `Categoria` | Escolha | Mesmas escolhas de `Perifericos` |
| `Marca` | Linha de texto | Obrigatória |
| `Quantidade` | Número | Obrigatória. Casas decimais: 0. Mínimo: 1 |
| `ValorUnitario` → **Valor unitário** | Moeda | Real (R$) |
| `ValorTotal` → **Valor total** | Calculada (Moeda) | `=[Quantidade]*[ValorUnitario]` |
| `Fornecedor` | Linha de texto | |
| `Solicitante` | Linha de texto | Obrigatória |
| `DataSolicitacao` → **Data da solicitação** | Data | Padrão: hoje |
| `Status` | Escolha | Obrigatória. Escolhas na ordem: Cotação, Solicitado, Em aprovação, Aprovado, Aguardando entrega, Entregue, Cancelado. Padrão: Cotação |
| `ObsCancelamento` → **Motivo do cancelamento** | Várias linhas de texto | |
| `Observacoes` → **Observações** | Várias linhas de texto | |

**Validação da lista:**
- Fórmula: `=OR([Status]<>"Cancelado";LEN([ObsCancelamento])>0)`
- Mensagem: `Para cancelar, informe o motivo do cancelamento.`

O **nº da compra** é a coluna automática **ID**: mostre-a na exibição. A data de cada mudança de status fica no
**histórico de versões** do item.

---

## Etapa 3: Reparos (We Care)

### Lista `Reparos`

| Coluna | Tipo | Opções |
|---|---|---|
| `Title` → **Nº da OS** | Linha de texto | Obrigatória. **Impor valores exclusivos: Sim** |
| `Fornecedor` | Linha de texto | Padrão: We Care |
| `Patrimonio` → **Patrimônio** | Linha de texto | Obrigatória |
| `NumeroSerie` → **Nº de série** | Linha de texto | Obrigatória |
| `Orcamento` → **Orçamento** | Moeda | |
| `DataEnvio` → **Data de envio** | Data | Obrigatória. Padrão: hoje |
| `DataOrcamento` → **Data do orçamento** | Data | |
| `DataRetorno` → **Data do retorno** | Data | |
| `Observacoes` → **Observações** | Várias linhas de texto | |

**Validação da lista:**
- Fórmula: `=AND(OR(ISBLANK([DataOrcamento]);[DataOrcamento]>=[DataEnvio]);OR(ISBLANK([DataRetorno]);[DataRetorno]>=[DataEnvio]))`
- Mensagem: `As datas de orçamento e de retorno não podem ser antes da data de envio.`

---

## Etapa 4: Maneja

### Lista `Maneja`

| Coluna | Tipo | Opções |
|---|---|---|
| `Title` → **Patrimônio** | Linha de texto | Obrigatória |
| `ModeloMaquina` → **Modelo da máquina** | Linha de texto | |
| `Usuario` → **Usuário** | Linha de texto | Obrigatória |
| `DataRetirada` → **Data de retirada** | Data | Obrigatória. Padrão: hoje |
| `ItensRecebidos` → **Itens recebidos** | Escolha (**várias seleções**, permitir valores digitados) | Notebook, Carregador, Mouse, Teclado, Headset, Mochila, Monitor, Cabo HDMI, Adaptador, Celular, Crachá |
| `PossuiAvaria` → **Possui avaria** | Sim/Não | Padrão: Não |
| `TiposAvaria` → **Tipos de avaria** | Escolha (**várias seleções**) | Tela quebrada/trincada, Carcaça amassada ou riscada, Teclado danificado, Bateria estufada/ruim, Não liga, Dobradiça quebrada, Contato com líquido, Peças faltando, Outros |
| `Observacoes` → **Observações** | Várias linhas de texto | |

O SharePoint não valida colunas de várias seleções. A regra "tipos de avaria só com *possui avaria*" fica para a
tela do app. Enquanto isso, use uma exibição filtrada por `Possui avaria = Sim` para conferir.

---

## Etapa 5: Onboarding

### Lista `Onboarding`

| Coluna | Tipo | Opções |
|---|---|---|
| `Title` → **Nome completo** | Linha de texto | Obrigatória |
| `DataOnboarding` → **Data do onboarding** | Data | Obrigatória |
| `Contratacao` → **Contratação** | Linha de texto | Ex.: CLT, Efetivação, Estágio |
| `SistemaOperacional` → **Sistema operacional** | Escolha | Windows, macOS (permitir valores digitados) |
| `Status` | Escolha | Obrigatória. Pendente, Aguardando vinculação, Entregue, Declínio. Padrão: Pendente |
| `Login` | Linha de texto | Preenchido quando a máquina fica pronta |
| `Patrimonio` → **Patrimônio** | Linha de texto | Preenchido quando a máquina fica pronta |
| `Observacoes` → **Observações** | Várias linhas de texto | |

**Validação da lista:**
- Fórmula: `=AND(OR([Status]<>"Entregue";AND(LEN([Login])>0;LEN([Patrimonio])>0));OR(ISERROR(SEARCH("efetiva";[Contratacao]));[Status]="Pendente";[Status]="Declínio"))`
- Mensagem: `Para "Entregue", preencha login e patrimônio. Efetivação não recebe máquina (fica Pendente ou Declínio).`

**Para importar a planilha de admissões:** abra a lista em **Editar no modo de grade** e cole as colunas da
planilha (nome, data, contratação, sistema operacional).
