# E-mails automáticos (Power Automate), Etapa 1

São dois fluxos, só com conectores padrão (SharePoint, Office 365 Outlook e OneDrive for Business):

1. **Alerta de estoque baixo:** a cada saída que deixa o item no mínimo ou abaixo dele.
2. **Relatório quinzenal:** a cada 15 dias, um e-mail com a situação do estoque e as movimentações do período, com
   o relatório em PDF anexado.

Para criar, abra **make.powerautomate.com → + Criar**.

Os campos aparecem com o **nome de exibição** das colunas (por exemplo, *Mínimo*). Nas expressões, use o **nome
interno** (`Minimo`), como está abaixo.

> **Dono do fluxo:** o fluxo roda com a conta de quem o criou. Em **Detalhes → Proprietários**, adicione outro
> gestor como coproprietário, para o fluxo não parar se você trocar de área ou sair da empresa.

---

## 1. Alerta de estoque baixo

**+ Criar → Fluxo de nuvem automatizado** → nome `Estoque - alerta de estoque baixo`.

1. **Gatilho:** SharePoint → **Quando um item é criado**.
   - Endereço do site: o site Controle de Estoque.
   - Nome da lista: `Movimentacoes`.
2. **+ Nova etapa → SharePoint → Obter item**.
   - Site: o mesmo.
   - Lista: `Perifericos`.
   - Id: o campo dinâmico **PerifericoID** (do gatilho).
3. **+ Nova etapa → Condição**. Clique em **Editar no modo avançado** e cole:
   ```
   @and(equals(triggerBody()?['Tipo']?['Value'], 'Saída'), greater(body('Obter_item')?['Minimo'], 0), lessOrEquals(body('Obter_item')?['Quantidade'], body('Obter_item')?['Minimo']))
   ```
   Se a etapa 2 tiver outro nome, troque `Obter_item` pelo nome dela, com `_` no lugar dos espaços.
4. **Se sim → Office 365 Outlook → Enviar um email (V2)**:
   - **Para:** os e-mails dos gestores, separados por `;`.
   - **Assunto:**
     ```
     Estoque baixo: @{triggerBody()?['Title']}
     ```
   - **Corpo:**
     ```
     O item @{triggerBody()?['Title']} ficou com @{body('Obter_item')?['Quantidade']} unidade(s) em estoque (mínimo: @{body('Obter_item')?['Minimo']}).

     Última saída: @{triggerBody()?['Quantidade']} para @{triggerBody()?['Responsavel']} (@{triggerBody()?['Motivo']}), lançada por @{triggerBody()?['Author']?['DisplayName']}.
     ```
5. **Salvar** e testar: registre no app uma saída que deixe um item no mínimo.

---

## 2. Relatório quinzenal

**+ Criar → Fluxo de nuvem agendado** → nome `Estoque - relatório quinzenal`. Configure:
- Início: o próximo dia 1º às 08:00;
- Repetir a cada: **15 Dia**.

Em **Recorrência → Mostrar opções avançadas**, escolha o fuso **(UTC-03:00) Brasília**.

### Buscar os dados

1. **SharePoint → Obter itens** (renomeie a etapa para `Itens`):
   - Lista: `Perifericos`.
   - Consulta de filtro: `Ativo eq 1`.
   - Ordenar por: `Title`.
   - Contagem superior: `5000`.
2. **SharePoint → Obter itens** (renomeie para `Movimentos`):
   - Lista: `Movimentacoes`.
   - Consulta de filtro:
     ```
     Data ge '@{addDays(utcNow(), -15, 'yyyy-MM-dd')}'
     ```
   - Ordenar por: `Data desc`.
   - Contagem superior: `5000`.

### Montar as tabelas

3. **Operação de Dados → Selecionar** (renomeie para `TabelaItens`).
   - **De:** `value` da etapa `Itens`.
   - **Mapear:** clique no ícone **T** para trocar para o modo texto e cole:
     ```
     {
       "Categoria": "@{item()?['Categoria']?['Value']}",
       "Marca": "@{item()?['Marca']}",
       "Modelo": "@{item()?['Title']}",
       "Local": "@{item()?['Local']}",
       "Em estoque": "@{item()?['Quantidade']}",
       "Mínimo": "@{item()?['Minimo']}",
       "Situação": "@{if(equals(item()?['Quantidade'], 0), 'Esgotado', if(lessOrEquals(item()?['Quantidade'], item()?['Minimo']), 'Baixo', 'OK'))}"
     }
     ```
4. **Operação de Dados → Selecionar** (renomeie para `TabelaMovimentos`).
   - **De:** `value` da etapa `Movimentos`.
   - **Mapear:**
     ```
     {
       "Data": "@{formatDateTime(convertFromUtc(item()?['Data'], 'E. South America Standard Time'), 'dd/MM/yyyy HH:mm')}",
       "Item": "@{item()?['Title']}",
       "Tipo": "@{item()?['Tipo']?['Value']}",
       "Quantidade": "@{item()?['Quantidade']}",
       "Saldo após": "@{item()?['SaldoApos']}",
       "Responsável": "@{item()?['Responsavel']}",
       "Motivo": "@{item()?['Motivo']}",
       "Lançado por": "@{item()?['Author']?['DisplayName']}"
     }
     ```
5. **Operação de Dados → Criar tabela HTML** (renomeie para `HtmlItens`).
   - **De:** saída de `TabelaItens`.
6. **Operação de Dados → Criar tabela HTML** (renomeie para `HtmlMovimentos`).
   - **De:** saída de `TabelaMovimentos`.
7. **Operação de Dados → Compor** (renomeie para `Relatorio`) e cole:
   ```
   <html><head><meta charset="utf-8"><style>
     body { font-family: Segoe UI, Arial, sans-serif; font-size: 12px; color: #1a1a1a; }
     h1 { font-size: 18px; } h2 { font-size: 14px; margin-top: 20px; }
     table { border-collapse: collapse; width: 100%; }
     th { background: #121212; color: #fff; text-align: left; padding: 5px; }
     td { border: 1px solid #ddd; padding: 5px; }
   </style></head><body>
   <h1>Relatório do estoque</h1>
   <p>Gerado em @{formatDateTime(convertFromUtc(utcNow(), 'E. South America Standard Time'), 'dd/MM/yyyy HH:mm')} · movimentações dos últimos 15 dias</p>
   <h2>Itens do estoque</h2>
   @{body('HtmlItens')}
   <h2>Movimentações do período</h2>
   @{body('HtmlMovimentos')}
   </body></html>
   ```

### Gerar o PDF e enviar

8. **OneDrive for Business → Criar arquivo**:
   - Caminho da pasta: `/Relatorios Estoque`.
   - Nome do arquivo:
     ```
     Estoque_@{formatDateTime(convertFromUtc(utcNow(), 'E. South America Standard Time'), 'yyyy-MM-dd')}.html
     ```
   - Conteúdo: saída de `Relatorio`.
9. **OneDrive for Business → Converter arquivo**:
   - Arquivo: **Id** da etapa anterior.
   - Tipo de destino: **PDF**.
10. **Office 365 Outlook → Enviar um email (V2)**:
    - **Para:** os gestores.
    - **Assunto:** `Relatório quinzenal do estoque`.
    - **Corpo:** saída de `Relatorio`.
    - **Anexos:**
      - Nome:
        ```
        Estoque_@{formatDateTime(convertFromUtc(utcNow(), 'E. South America Standard Time'), 'yyyy-MM-dd')}.pdf
        ```
      - Conteúdo: **Conteúdo do arquivo** da etapa *Converter arquivo*.
11. **Salvar** e, para testar, clique em **Testar → Manualmente**.

Para guardar os PDFs no SharePoint em vez de só no e-mail, acrescente a etapa **SharePoint → Criar arquivo** na
biblioteca **Documentos** do site, com o mesmo nome e o mesmo conteúdo do anexo.
