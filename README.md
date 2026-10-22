# Sistema de Orçamentos (Treinamento MVC Protheus)

Substitui a planilha `template.xlsx` de orçamentos: o orçamento é montado em dias por fase, convertido em horas pelo fator Horas/Dia, aprovado por um usuário autorizado e guardado como base histórica de consulta.

É também a atividade prática de MVC para consultores júnior: cada etapa introduz um padrão MVC sobre o mesmo sistema (Modelo 1 → pai/filho → pai/filho/neto → pastas → ações customizadas → rotina automática → consulta/TReport).

Especificação completa: [docs/especificacao-tecnica.md](docs/especificacao-tecnica.md)

## Stack

- TOTVS Protheus, release 12.1.2410 ou superior (MVC em TLPP)
- Fontes `.tlpp` com namespace, encoding CP1252
- Tabelas ZB1 a ZB7, parâmetro `MV_XHRDIA`

## Estrutura

```
docs/        especificação técnica e imagens
src/         fontes .tlpp e o include custom.orc.th
dicionario/  exportações/prints do dicionário (SX2, SX3, SIX, SXB, SX6)
testes/      roteiros de teste e evidências por etapa
```

## Rotinas

| Código  | Fonte                                  | Etapa   |
| :------ | :------------------------------------- | :------ |
| ZORCA01 | custom.orc.cadastro.areas.tlpp         | 1       |
| ZORCA02 | custom.orc.cadastro.definicoes.tlpp    | 1       |
| ZORCA03 | custom.orc.cadastro.aprovadores.tlpp   | 1       |
| ZORCA04 | custom.orc.orcamento.manutencao.tlpp   | 2, 3, 4 |
| ZORCA05 | custom.orc.orcamento.aprovacao.tlpp    | 5       |
| ZORCA06 | custom.orc.orcamento.importacao.tlpp   | 6       |
| ZORCA07 | custom.orc.consulta.historico.tlpp     | 7       |
| ZORCR01 | custom.orc.relatorio.orcamento.tlpp    | 7       |
| —       | custom.orc.util.funcoes.tlpp           | 1       |
| —       | custom.orc.th (include)                | 1       |

## Entregas por etapa

Fonte compilando sem warnings, dicionário criado na base de testes, roteiro com no mínimo 3 casos positivos e 3 negativos, evidências dos critérios de aceite e autoavaliação (seção 8 da especificação).
