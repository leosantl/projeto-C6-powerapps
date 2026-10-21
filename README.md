# Controle de Estoque · Power Apps

Esta é a versão do Controle de Estoque feita no **Microsoft 365 da empresa**:

- **listas do SharePoint** guardam os dados;
- um **app do Power Apps** traz as telas e o código de barras;
- o **Power Automate** manda os e-mails de alerta e os relatórios.

Os dados ficam no SharePoint da empresa. O login é a conta Microsoft corporativa. Não é preciso servidor, Azure nem
instalar nada.

> Projetos separados:
> - **projeto-C6** é o sistema local, instalado no computador;
> - **projeto-C6-azure** é a versão na nuvem (Azure App Service).
>
> Este projeto **não reaproveita código** deles: é o mesmo controle, refeito com as ferramentas do Microsoft 365.

## Licença: só o que o Microsoft 365 já inclui

Tudo aqui usa **conectores padrão**: SharePoint, Office 365 Outlook e OneDrive for Business. Eles fazem parte das
licenças comuns do Microsoft 365 (E3/E5). **Não** é preciso Dataverse, conector premium nem licença extra do Power
Apps. Se a empresa tiver bloqueado algum conector (política de DLP), a TI informa qual.

## Passo 0: conferir o acesso

Antes de começar, confira duas coisas. Nenhuma delas cria nada.

1. **Power Apps:** abra **make.powerapps.com** com a conta da empresa e clique em **+ Criar**.
   - Se a tela de criar app aparecer, você pode criar.
   - Se aparecer bloqueio ou "sem licença", peça à TI acesso ao Power Apps, no ambiente padrão.
2. **SharePoint:** você precisa ser **proprietário** de um site.
   - O jeito mais simples: no **Teams**, crie uma equipe privada **Controle de Estoque**. Isso cria um site do
     SharePoint seu. Para abrir o site, use, na equipe, **Arquivos → Abrir no SharePoint**.
   - No site, clique em **+ Novo → Lista**. Se a janela de criar lista abrir, está tudo certo.

## Etapas

| Etapa | O que entrega | Guia |
|---|---|---|
| **1. Estoque** | Periféricos com código de barras, entrada e saída com saldo, histórico por item, alerta de estoque baixo e relatório quinzenal por e-mail | [Listas](docs/01-listas.md#etapa-1-estoque) → [App](docs/02-app-estoque.md) → [Fluxos](docs/03-fluxos.md) → [Dados atuais](docs/04-migrar-dados.md) |
| 2. Hubs (compras) | Compras com status (cotação → … → entregue/cancelado) e histórico de mudanças | [Listas](docs/01-listas.md#etapa-2-hubs-compras) |
| 3. Reparos (We Care) | Ordens de serviço com datas e orçamento | [Listas](docs/01-listas.md#etapa-3-reparos-we-care) |
| 4. Maneja | Máquinas recebidas, itens e avarias | [Listas](docs/01-listas.md#etapa-4-maneja) |
| 5. Onboarding | Admissões, máquina pronta → vinculada → entregue | [Listas](docs/01-listas.md#etapa-5-onboarding) |

Comece pela **Etapa 1**. Nas etapas 2 a 5, as listas já funcionam sozinhas no SharePoint: dá para cadastrar, filtrar
e exportar para o Excel. As telas no app entram depois, no mesmo modelo da Etapa 1.

## O que muda em relação ao sistema local

| Função | Aqui |
|---|---|
| Cadastros, status, filtros, exportar para Excel | ✅ Nativo do SharePoint |
| Quem alterou o quê | ✅ **Histórico de versões** do SharePoint (item → ⋯ → Histórico de versões) |
| Código de barras | ✅ Câmera no celular/tablet (app Power Apps). No computador, com leitor USB |
| Alerta de estoque baixo e relatório quinzenal por e-mail | ✅ Power Automate |
| Saldo nunca negativo | ⚠️ O app confere antes de gravar. Se duas pessoas movimentarem **o mesmo item no mesmo segundo**, uma pode sobrescrever a outra. O histórico de movimentações mostra e permite corrigir |
| Relatórios em Word com modelo | ❌ Exige licença premium. O relatório sai por e-mail e em PDF |
| Cada setor ver só os próprios dados | ⚠️ Possível com permissões do SharePoint, mas trabalhoso. O mais simples é um site por setor |
| Muitos registros | ⚠️ O Power Apps busca no máximo 2.000 itens por consulta. As fórmulas deste guia já filtram na lista (sem esse limite) |

## Estrutura

```
docs/01-listas.md         listas do SharePoint, coluna por coluna (todas as etapas)
docs/02-app-estoque.md    app do Power Apps da Etapa 1: telas e fórmulas para colar
docs/03-fluxos.md         Power Automate: alerta de estoque baixo e relatório quinzenal
docs/04-migrar-dados.md   como trazer os dados do sistema local para as listas
```
