# AGENTS.md — Sistema de Orçamentos (Treinamento MVC Protheus)

Projeto de treinamento MVC em TLPP no Protheus (tabelas ZB1–ZB7, rotinas ZORCA01–07 e ZORCR01). A fonte da verdade é `docs/especificacao-tecnica.md`. O `README.md` resume as rotinas, mas a seção "Estrutura" dele está desatualizada (cita `src/`, `dicionario/`, `testes/`): a estrutura real está abaixo.

## REGRA PRINCIPAL: NÃO MEXER NOS FONTES

O desenvolvedor usa a IA neste projeto **para consulta**. O objetivo é ele aprender MVC fazendo: quem escreve nos fontes é ele.

- NÃO criar, editar ou completar arquivos `.tlpp`, `.th`, `.prw` ou qualquer fonte do repositório (`fonts/`, `utils/` etc.).
- Exemplos de código **no chat** são permitidos e bem-vindos quando ajudam: trecho pronto, baseado no código real do projeto (funções do `utils`, namespaces), com a fonte TDN e o que precisa ser testado no ambiente. Não enrolar nem recusar: o dev adapta e cola.
- NÃO gerar scripts de dicionário (SX2/SX3/SIX/SXB/SX6); configuração de dicionário vai como passo a passo no SIGACFG, com os nomes de campo da tela (ex.: "Expressão", "Retorno"), não os códigos da SXB (tipo 2, tipo 5).

O que PODE fazer:
- Explicar conceitos MVC/TLPP (FWFormStruct, AddGrid, SetRelation, LoadValue x SetValue, FWSaveRows, FWLoadModel etc.) e indicar a página do TDN.
- Explicar uma regra da especificação (RN01–RN18) ou um critério de aceite.
- Ler código que o dev escreveu e apontar problemas, dizendo O QUE está errado e POR QUÊ — a correção ele faz.
- Interpretar mensagens de erro do compilador/AppServer.
- Responder em português, de forma direta.

## Estrutura real do repositório

```
fonts/     um .tlpp por rotina (ZORCA01–07, ZORCR01)
utils/     custom.orc.th (defines) e custom.orc.utils.tlpp (funções compartilhadas)
database/  arquivos do dicionário da empresa 99 (sx29901, sx39901, six9901, sxb9901) — referência, não editar
Tests/Imports/  CSVs de teste da Etapa 6 (válido, sem definição, linhas com erro, atividade zerada, só cabeçalho)
docs/      especificação, fluxo de status (img/) e o PDF "Code Review e Plano de Correção"
```

No Windows as pastas aparecem como `Fonts/` e `Utils/`; no git são `fonts/` e `utils/`.

## Arquitetura (o que exige ler vários arquivos)

- **Include `utils/custom.orc.th`**: além de status (`STATUS_*`), ações do histórico (`ACAO_*`) e situação (`ATIVO_*`), define o namespace de cada fonte (`NZORCA01`…`NZORCR01`, `NUTILS`). Chamadas entre fontes usam essas constantes em vez do namespace literal.
- **`utils/custom.orc.utils.tlpp`** (namespace `custom.orc.utils`): helpers reaproveitados pelos modelos — conversão de horas (`HoursToNum`, `NumToHours`, `VldHours`), recálculo pelo Horas/Dia (`OnHrDayChange`, `TotalHrsCharSet`), validações de cadastro (`VldDefin`, `VldArea`, `VldAprov`, `CodExiste`), ativação de modelo (`ModelActivate`) e montagem de query parametrizada (`QuerySet`). Antes de sugerir uma função nova, verificar se ela já existe aqui.
- **ZORCA04 (`custom.orc.orcamento.manutencao`)** é o modelo central: ZB4 (cabeçalho) → ZB5 (itens) → ZB6 (atividades), mais ZB7 (histórico, somente consulta). Gatilhos são criados no fonte (`TriggerSet`), e as regras de dias↔horas ficam nas validações desse modelo.
- **ZORCA05 (`aprovacao`)** troca status e grava ZB7 (`GravaHist`); **ZORCA06 (`importacao`)** grava orçamentos lendo o CSV e carregando o modelo do ZORCA04 sem tela — por isso qualquer regra que só esteja na view quebra a importação.
- **ZORCA07 / ZORCR01**: consulta histórica e relatório TReport, leitura via query.
- O ponto de entrada de cada fonte é `U_ZORCA0x` (ex.: `custom.orc.orcamento.manutencao.U_ZORCA04`), não `main` como diz a seção 7.1 da spec. Ao revisar, não apontar isso como erro sem o dev pedir.

## Compilação, execução e dados

- Não há build/test automatizado: o dev compila e testa no Protheus pelo TDS (VS Code). O linter do TDS usa os includes do Protheus e a pasta `Utils` (`.vscode/settings.json`).
- Execução via SmartClient Web Debug (`.vscode/launch.json`), programa inicial `sigabpm` com argumento `custom.orc.<...>.U_ZORCA0x`.
- Testes de importação: os CSVs em `Tests/Imports/` são CP1252, separador `;`, linhas tipo `I` (item) e `A` (atividade), decimal com vírgula.
- Dados reais: SQL Server local, banco `P12DEV`, empresa 99 (tabelas com sufixo `990`, ex. `ZB4990`). Consulta somente leitura: `sqlcmd -S localhost -E -C -d P12DEV -W -s "|" -Q "..."`, sempre com `D_E_L_E_T_ = ' '`. O MCP TOTVS não tem as tabelas ZB.

## Onde está cada coisa na especificação

- Seção 2: regras de negócio (RN01–RN18)
- Seção 3: dicionário de dados (campos, índices, `MV_XHRDIA`, F3)
- Seção 4: fluxo de status e aprovação (`docs/img/ciclo-de-vida-orcamento.png`)
- Seção 5: telas e o layout do CSV de importação
- Seção 6: etapas 1–7 e critérios de aceite
- Seção 7: padrões obrigatórios (7.1), armadilhas (7.2) e pontos dependentes da release (7.3)
- Seção 8: entregáveis e peso da avaliação

## Padrões do projeto (seção 7.1) — usar ao revisar código do dev

- Arquivo: `custom.orc.<indicativo>.<descricao>.tlpp`, minúsculas; namespace = nome do arquivo sem extensão.
- MenuDef/ModelDef/ViewDef como User Function dentro do namespace; em FWLoadModel/FWLoadView/`VIEWDEF.` usar o namespace completo.
- ID do modelo = código curto + `M` (ex.: `ZORCA01M`); submodelos `ZB4MASTER`, `ZB5DETAIL`.
- Fontes em CP1252 (UTF-8 quebra acentuação) — ler os arquivos convertendo de CP1252.
- Status/ações via `#Define` em `custom.orc.th`, nunca literais `"1"`, `"2"`.
- Queries filtram `xFilial` e `D_E_L_E_T_ = ' '`, com parâmetros (nunca concatenação), e fecham o alias.
- Mensagens com `Help()`; nada de MsgAlert/MsgYesNo em validação sem tela (`IsBlind()`).
- `GetArea`/`RestArea`, tipagem declarada, cabeçalho Protheus.doc.
- Toda regra de negócio no modelo, nunca na view (a importação da Etapa 6 grava sem tela).
- Armadilhas recorrentes (7.2): LoadValue no lado oposto de dias↔horas, `IsDeleted()` nas somas, nada de `GoLine` em validação (usar `FWSaveRows`/`FWRestRows`), troca de status com transação + lock + releitura, histórico com `SetOnlyQuery`.

## Pitfalls

- Itens da seção 7.3 variam conforme a release da lib — ao responder sobre eles, avisar que precisa ser validado no ambiente.
- Não editar `docs/especificacao-tecnica.md` nem o `README.md` sem pedido explícito: a spec é o escopo passado pelo gestor.
