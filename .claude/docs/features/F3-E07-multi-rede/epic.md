# F3-E07 — Gestão multi-rede com governança

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F3-E07 — Gestão multi-rede com governança |
| Fase | F3 (Analytics, IA & Diferenciação) |
| Prioridade | Could |
| Estimativa | G |
| Bounded context(s) | `iam` (extensão) |
| Depende de | F2-E07 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F3-E07` |

## Objetivo

Permitir que mantenedoras definam metas de desempenho acadêmico/financeiro por unidade e acompanhem o cumprimento na rede.

## Histórias cobertas

- `F3-E07-U01` — Cadastro de metas por unidade e painel de progresso.

## Escopo / fora de escopo

- **Dentro:** metas (alvo + período) por unidade, painel de progresso reaproveitando indicadores de F2-E07/F3-E01.
- **Fora:** nada relevante identificado fora de escopo.

## Dependências e integrações

Depende de F2-E07 (dashboard consolidado por unidade) e F3-E01 (indicadores) — ambos já construídos
sobre a base multi-tenant fundacional de [`F1-E0A`](../F1-E0A-tenancy/spec-tenancy.md)/`F1-E09`
(isolamento por `Group`, escopo por `School`). Este épico não introduz nenhuma parte do isolamento
em si, só metas/governança sobre dados já escopados. É o topo da pirâmide de multiunidade — só faz
sentido especificar depois que F2-E07 e F3-E01 estiverem implementados.

## Decisões em aberto / riscos

Nenhuma decisão maior em aberto além das já herdadas de F2-E07.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
