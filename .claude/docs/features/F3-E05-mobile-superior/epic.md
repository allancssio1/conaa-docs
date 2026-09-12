# F3-E05 — Experiência mobile superior

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F3-E05 — Experiência mobile superior |
| Fase | F3 (Analytics, IA & Diferenciação) |
| Prioridade | Should |
| Estimativa | G |
| Bounded context(s) | vários — extensão do app mobile |
| Depende de | F2-E01 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F3-E05` |

## Objetivo

Levar uma versão resumida do dashboard analítico (F3-E01) para o app mobile, otimizada para gestores em telas pequenas.

## Histórias cobertas

- `F3-E05-U01` — Dashboard resumido no app mobile para gestores, com metas de performance.

## Escopo / fora de escopo

- **Dentro:** adaptação do dashboard de F3-E01 para o app.
- **Fora:** reescrita de UX das telas já existentes no app — não está nos critérios de aceite, que tratam especificamente do dashboard.

## Dependências e integrações

Depende de F2-E01 (app já existente) e F3-E01 (dashboard já existente na web). É essencialmente "portar" o dashboard para o app.

## Decisões em aberto / riscos

- As "metas de performance definidas pelo time" citadas no BACKLOG ainda não existem — definir esses números (tempo de carregamento, etc.) antes ou durante a spec.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
