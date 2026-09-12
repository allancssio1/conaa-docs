# F2-E05 — Transporte escolar

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F2-E05 — Transporte escolar |
| Fase | F2 (Eficiência & Comunicação) |
| Prioridade | Should |
| Estimativa | M |
| Bounded context(s) | `transport` (novo) |
| Depende de | F1-E01, F1-E02 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F2-E05` |

## Objetivo

Cadastro de veículos, motoristas e rotas com pontos de embarque, e vínculo de aluno matriculado a uma rota.

## Histórias cobertas

- `F2-E05-U01` — Cadastro de rota/veículo/motorista com pontos de embarque.
- `F2-E05-U02` — Vínculo aluno-rota, visível no portal do responsável.

## Escopo / fora de escopo

- **Dentro:** rota, veículo, motorista, vínculo aluno-rota, relatório de ocupação.
- **Fora:** rastreamento GPS em tempo real (isso é F3-E04 — aqui é só cadastro/organização, sem rastreamento).

## Dependências e integrações

Depende de F1-E01 (`Student`) e F1-E02 (matrícula ativa). O vínculo aluno-rota precisa aparecer no portal — toca F1-E08 (nova seção) ou o portal simplesmente consome o endpoint deste épico.

## Decisões em aberto / riscos

Nenhuma decisão maior em aberto.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
