# F3-E01 — Analytics/BI avançado e IA

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F3-E01 — Analytics/BI avançado e IA |
| Fase | F3 (Analytics, IA & Diferenciação) |
| Prioridade | Must |
| Estimativa | GG |
| Bounded context(s) | `analytics` (novo) |
| Depende de | F1-E03, F1-E04, F1-E06 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F3-E01` |

## Objetivo

Dashboard de indicadores (aprovação, evasão, inadimplência, ocupação de vagas) e identificação de alunos em risco de evasão/reprovação.

## Histórias cobertas

- `F3-E01-U01` — Dashboard de indicadores para direção, filtrável por período/turma/unidade.
- `F3-E01-U02` — Lista de alunos em risco com fatores explicáveis e sugestão de ação.

## Escopo / fora de escopo

- **Dentro:** agregação/consolidação de dados existentes, modelo de risco explicável (baseado em regras ou score simples).
- **Fora:** recomendações personalizadas de estudo — mencionado no documento-fonte como fronteira internacional, não é critério de aceite.

## Dependências e integrações

Depende de F1-E03 (frequência), F1-E04 (notas) e F1-E06 (inadimplência) como fontes de dado. Se F2-E07 (multiunidade) já existir, o dashboard deve suportar filtro por unidade.

## Decisões em aberto / riscos

- O "modelo de IA" pode começar como heurística/score simples e evoluir depois. Uma heurística explicável atende melhor ao critério de aceite "resultado é explicável" com menos risco do que um modelo treinado desde o início — decidir a abordagem na spec.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
