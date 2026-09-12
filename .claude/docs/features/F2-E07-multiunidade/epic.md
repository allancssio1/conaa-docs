# F2-E07 — Multiunidade / rede escolar

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F2-E07 — Multiunidade / rede escolar |
| Fase | F2 (Eficiência & Comunicação) |
| Prioridade | Should |
| Estimativa | G |
| Bounded context(s) | `iam` (extensão) |
| Depende de | F1-E01 a F1-E09 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F2-E07` |

## Objetivo

Gerenciar múltiplas unidades (escolas/campi) no mesmo ambiente, com indicadores consolidados e permissões segmentadas por nível de rede.

## Histórias cobertas

- `F2-E07-U01` — Dashboard consolidado por unidade (matrículas, inadimplência, frequência).
- `F2-E07-U02` — Permissões por nível: mantenedora, direção de unidade, coordenação.

## Escopo / fora de escopo

- **Dentro:** conceito de "unidade" atravessando os contextos existentes, dashboard consolidado, extensão de perfis de F1-E09.
- **Fora:** metas e comparativos avançados entre unidades (isso é F3-E07 — aqui é só consolidação/visualização, sem metas).

## Dependências e integrações

Depende de **todos** os épicos F1-E01 a F1-E09 — introduz "unidade" como dimensão transversal, provavelmente exigindo revisitar entidades/queries de vários contextos para adicionar uma referência de unidade. É o épico de maior risco arquitetural da Fase 2.

## Decisões em aberto / riscos

- A estratégia de multiunidade (schema único com coluna de unidade em cada tabela vs. schema por unidade) é uma decisão de arquitetura que deveria ser tomada e documentada em `arquitetura-ignite.md`/spec **antes** de tocar qualquer contexto existente — retrabalho alto se decidido tarde.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
