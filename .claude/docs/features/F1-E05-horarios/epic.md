# F1-E05 — Horários e grade de aulas

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E05 — Horários e grade de aulas |
| Fase | F1 (MVP) |
| Prioridade | Must |
| Estimativa | M |
| Bounded context(s) | `academic` |
| Depende de | F1-E02 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F1-E05` |

## Objetivo

Montar a grade de horários de cada turma (professor, disciplina, sala, dia/horário) evitando conflitos de alocação.

## Histórias cobertas

- `F1-E05-U01` — Coordenação monta grade de horários evitando conflito de professor/sala.
- `F1-E05-U02` — Professor visualiza sua própria grade consolidada da semana.

## Escopo / fora de escopo

- **Dentro:** grade por turma, alocação professor/disciplina/sala/horário, detecção de conflito, edição durante o ano com data de efeito.
- **Fora:** geração automática/otimizada de grade — citada no documento-fonte como diferencial de mercado, não é critério de aceite deste MVP.

## Dependências e integrações

Depende de F1-E01 (`Teacher`, `Sala`, `Disciplina`) e F1-E02 (`Turma`). É pré-requisito de F1-E03 e F1-E04, que precisam saber "quando" ocorre uma aula.

## Decisões em aberto / riscos

Nenhuma decisão maior em aberto — épico bem contido.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
