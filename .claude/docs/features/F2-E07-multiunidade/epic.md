# F2-E07 — Multiunidade / rede escolar

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F2-E07 — Multiunidade / rede escolar |
| Fase | F2 (Eficiência & Comunicação) |
| Prioridade | Should |
| Estimativa | G |
| Bounded context(s) | `iam` (extensão), `reporting` (extensão) |
| Depende de | F1-E0A, F1-E01 a F1-E09 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F2-E07` |

## Objetivo

Entregar visão consolidada e governança de rede sobre a base **já multi-tenant desde o MVP**:
dashboard comparando indicadores entre as escolas de um `Group` e permissões de nível de rede
(mantenedora) mais refinadas do que o binário group-wide/school-scoped já existente desde `F1-E09`.

## Nota de reescopo (importante)

Versões anteriores deste épico tratavam a estratégia de isolamento (banco compartilhado vs. schema
por unidade) e o próprio conceito de "escola como escopo de permissão" como decisões em aberto a
serem tomadas **aqui**. Isso foi corrigido: a multi-tenancy é fundacional desde o primeiro épico de
domínio —
- Isolamento por `Group` (`groupId`) e escopo de permissão por `School` (`schoolId` em `UserRole`,
  `null` = group-wide) já existem desde [`F1-E0A`](../F1-E0A-tenancy/spec-tenancy.md) e
  [`F1-E09`](../F1-E09-seguranca-lgpd/spec-seguranca-lgpd.md).
- **Este épico não introduz nenhuma parte do isolamento/escopo** — ele só constrói *sobre* essa
  base: dashboards comparativos e refinamentos de governança (ex.: papéis intermediários entre
  "vê uma escola" e "vê todas", relatórios consolidados por rede).

## Histórias cobertas

- `F2-E07-U01` — Dashboard consolidado por unidade (matrículas, inadimplência, frequência).
- `F2-E07-U02` — Permissões por nível: mantenedora, direção de unidade, coordenação.

## Escopo / fora de escopo

- **Dentro:** dashboard comparativo entre escolas do mesmo `Group` (agregações de `reporting` já
  filtradas por escola, ver `F1-E07`); eventuais papéis de granularidade intermediária, se o
  binário group-wide/school-scoped de `F1-E09` se mostrar insuficiente na prática.
- **Fora:** isolamento (`groupId`) e escopo básico de escola (`schoolId` em `UserRole`) — já
  entregues em `F1-E0A`/`F1-E09`. Metas e comparativos avançados entre unidades (isso é `F3-E07`).

## Dependências e integrações

Depende de `F1-E0A` (Group/School) e de todos os épicos `F1-E01` a `F1-E09` (a base multi-tenant e
os dados que serão consolidados no dashboard). Como o isolamento já é fundacional, o risco
arquitetural deste épico caiu bastante em relação às versões anteriores do plano — o que resta é
principalmente um trabalho de agregação/visualização sobre dados que já são naturalmente
escopados por escola.

## Decisões em aberto / riscos

- Definir se algum papel de granularidade intermediária (ex.: "coordenador pedagógico de 3 escolas
  específicas dentro de um group de 10") é necessário, ou se o modelo atual de `F1-E09`
  (`UserRole` com múltiplas entradas, uma por escola) já cobre esse caso sem extensão de schema.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
