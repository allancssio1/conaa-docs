# F2-E02 — Comunicação multicanal e notificações

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F2-E02 — Comunicação multicanal e notificações |
| Fase | F2 (Eficiência & Comunicação) |
| Prioridade | Must |
| Estimativa | G |
| Bounded context(s) | `communication` |
| Depende de | F1-E01, F1-E03 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F2-E02` |

## Objetivo

Enviar comunicados por e-mail/SMS/WhatsApp para grupos de responsáveis, com disparo automático em dois cenários: frequência baixa e mensalidade vencida.

## Histórias cobertas

- `F2-E02-U01` — Envio de comunicado institucional multicanal por turma/série/escola.
- `F2-E02-U02` — Disparo automático de aviso por frequência baixa ou vencimento.

## Escopo / fora de escopo

- **Dentro:** envio multicanal, histórico de envios, os 2 gatilhos automáticos citados no BACKLOG.
- **Fora:** motor de automação genérico configurável pelo usuário (isso é F3-E03 — aqui os gatilhos são fixos, não configuráveis).

## Dependências e integrações

Depende de F1-E01 (destinatários) e F1-E03 (gatilho de frequência). É reutilizado por F2-E03 (lembrete de vencimento usa os mesmos canais).

## Decisões em aberto / riscos

- Escolha de provedores externos (e-mail transacional, SMS, WhatsApp Business API) é decisão de infraestrutura a tomar na spec — cada canal tem custo/complexidade de integração diferente.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
