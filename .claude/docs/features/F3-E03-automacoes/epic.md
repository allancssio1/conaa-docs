# F3-E03 — Motor de fluxos/automações configuráveis

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F3-E03 — Motor de fluxos/automações configuráveis |
| Fase | F3 (Analytics, IA & Diferenciação) |
| Prioridade | Must |
| Estimativa | G |
| Bounded context(s) | `automation` (novo) |
| Depende de | F1-E03, F1-E06, F2-E02 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F3-E03` |

## Objetivo

Permitir que usuários não técnicos configurem regras do tipo condição → ação (ex.: "se frequência < X%, notificar"; "se atraso > Y dias, bloquear rematrícula").

## Histórias cobertas

- `F3-E03-U01` — Regra configurável de notificação por frequência baixa.
- `F3-E03-U02` — Regra configurável de bloqueio de rematrícula por inadimplência.

## Escopo / fora de escopo

- **Dentro:** motor de regra (1 condição + 1 ação) configurável via UI, histórico de disparos.
- **Fora:** workflow engine genérico multi-etapa (BPMN completo) — as regras do BACKLOG são simples, não modelar fluxos complexos hipotéticos.

## Dependências e integrações

Depende de F1-E03/F1-E06 (fontes de condição), F2-E02 (canal de notificação como ação) e F1-E02 (ação de bloqueio de rematrícula). Generaliza o que `F2-E02-U02` já fazia de forma fixa — ao especificar, avaliar se vale reescrever `F2-E02-U02` para usar este motor.

## Decisões em aberto / riscos

- Este é o primeiro motor genérico do sistema — risco real de over-engineering. Manter o escopo restrito aos 2 casos citados no BACKLOG na primeira versão, sem generalizar para casos hipotéticos.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
