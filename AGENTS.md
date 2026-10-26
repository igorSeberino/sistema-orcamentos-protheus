# AGENTS.md — Sistema de Orçamentos (Treinamento MVC Protheus)

Projeto de treinamento MVC em TLPP no Protheus (tabelas ZB1–ZB7, rotinas ZORCA01–07 e ZORCR01). A fonte da verdade é `docs/especificacao-tecnica.md`; o `README.md` resume estrutura e rotinas.

## REGRA PRINCIPAL: NÃO MEXER NOS FONTES

O desenvolvedor usa a IA neste projeto **para consulta**. O objetivo é ele aprender MVC fazendo: quem escreve nos fontes é ele.

- NÃO criar, editar ou completar arquivos `.tlpp`, `.th`, `.prw` ou qualquer fonte do repositório (`Fonts/`, `utils/` etc.).
- Exemplos de código **no chat** são permitidos e bem-vindos quando ajudam: trecho pronto, baseado no código real do projeto (funções do `utils`, namespaces), com a fonte TDN e o que precisa ser testado no ambiente. Não enrolar nem recusar: o dev adapta e cola.
- NÃO gerar scripts de dicionário (SX2/SX3/SIX/SXB/SX6); configuração de dicionário vai como passo a passo no SIGACFG, com os nomes de campo da tela (ex.: "Expressão", "Retorno"), não os códigos da SXB (tipo 2, tipo 5).

O que PODE fazer:
- Explicar conceitos MVC/TLPP (FWFormStruct, AddGrid, SetRelation, LoadValue x SetValue, FWSaveRows, FWLoadModel etc.) e indicar a página do TDN.
- Explicar uma regra da especificação (RN01–RN18) ou um critério de aceite.
- Ler código que o dev escreveu e apontar problemas, dizendo O QUE está errado e POR QUÊ — a correção ele faz.
- Interpretar mensagens de erro do compilador/AppServer.
- Responder em português, de forma direta.

## Onde está cada coisa na especificação

- Seção 2: regras de negócio (RN01–RN18)
- Seção 3: dicionário de dados (campos, índices, `MV_XHRDIA`, F3)
- Seção 4: fluxo de status e aprovação (`docs/img/ciclo-de-vida-orcamento.png`)
- Seção 5: telas e o layout do CSV de importação
- Seção 6: etapas 1–7 e critérios de aceite
- Seção 7: padrões obrigatórios (7.1) e armadilhas (7.2)
- Seção 8: entregáveis e peso da avaliação

## Padrões do projeto (seção 7.1) — usar ao revisar código do dev

- Arquivo: `custom.orc.<indicativo>.<descricao>.tlpp`, minúsculas; namespace = nome do arquivo sem extensão.
- Função principal `main`; MenuDef/ModelDef/ViewDef como User Function dentro do namespace.
- ID do modelo = código curto + `M` (ex.: `ZORCA01M`); submodelos `ZB4MASTER`, `ZB5DETAIL`.
- Fontes em CP1252 (UTF-8 quebra acentuação).
- Status/ações via `#Define` em `custom.orc.th`, nunca literais `"1"`, `"2"`.
- Queries filtram `xFilial` e `D_E_L_E_T_ = ' '`, com parâmetros (nunca concatenação).
- Mensagens com `Help()`; nada de MsgAlert/MsgYesNo em validação sem tela (`IsBlind()`).
- `GetArea`/`RestArea`, tipagem declarada, cabeçalho Protheus.doc.
- Toda regra de negócio no modelo, nunca na view (a importação da Etapa 6 grava sem tela).

## Pitfalls

- Não há build/test automatizado neste repositório: compilação e testes são feitos no Protheus (base de testes), pelo dev.
- Itens da seção 7.3 variam conforme a release da lib — ao responder sobre eles, avisar que precisa ser validado no ambiente.
- Não editar `docs/especificacao-tecnica.md` sem pedido explícito: é o escopo passado pelo gestor.
