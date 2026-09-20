# F1-E07 — Relatórios administrativos e oficiais

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E07 — Relatórios administrativos e oficiais |
| Fase | F1 (MVP) |
| Prioridade | Must |
| Estimativa | M |
| Bounded context(s) | `reporting` |
| Depende de | F1-E01 a F1-E06 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F1-E07` |

## Objetivo

Gerar relatórios operacionais filtráveis (listas, frequência, notas, resultados finais, inadimplência) e exportar dados no formato exigido pelo Censo Escolar/INEP.

## Histórias cobertas

- `F1-E07-U01` — Relatórios filtráveis por turma/série/período/situação, exportáveis em PDF/planilha.
- `F1-E07-U02` — Exportação de dados no formato do Censo Escolar/INEP.

## Escopo / fora de escopo

- **Dentro:** consultas/relatórios sobre dados já existentes nos contextos anteriores, exportação PDF/planilha, exportação Censo/INEP.
- **Fora:** dashboards analíticos/preditivos (isso é F3-E01 — aqui são relatórios operacionais, não indicadores de BI/IA).

## Dependências e integrações

Depende de **todos** os épicos F1-E01 a F1-E06 — é uma camada de leitura sobre eles. Só faz sentido especificar em detalhe depois que os dados de origem (alunos, matrícula, frequência, notas, financeiro) já existirem.

## Decisões em aberto / riscos

- O layout oficial do Censo Escolar/INEP precisa ser validado contra a especificação vigente do ano corrente antes de especificar `F1-E07-U02` — já sinalizado como pendência no BACKLOG.

## Próximo passo

Já especificado em [`spec-relatorios.md`](spec-relatorios.md). Ver status no [índice](../README.md).
