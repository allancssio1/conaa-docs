# F1-E03 — Gestão de frequência

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E03 — Gestão de frequência |
| Fase | F1 (MVP) |
| Prioridade | Must |
| Estimativa | M |
| Bounded context(s) | `attendance` |
| Depende de | F1-E02, F1-E05 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F1-E03` |

## Objetivo

Registrar presença/falta por turma e aula, calcular automaticamente o percentual de frequência e permitir justificar faltas.

## Histórias cobertas

- `F1-E03-U01` — Professor faz a chamada de uma turma por aula.
- `F1-E03-U02` — Coordenação visualiza alunos com frequência abaixo do limite mínimo.
- `F1-E03-U03` — Secretaria justifica uma falta com base em atestado/documento.

## Escopo / fora de escopo

- **Dentro:** chamada por aula, cálculo de percentual de frequência, justificativa de falta.
- **Fora:** bloqueio de matrícula/notificação automática por baixa frequência (isso é regra de negócio de F1-E02/automação de F3-E03/F2-E02).

## Dependências e integrações

Depende de F1-E02 (vínculo aluno-turma) e F1-E05 (grade/horário define o que é "uma aula" para a chamada). Alimenta F1-E04 (frequência é um dos critérios de aprovação) e, mais adiante, F2-E02/F3-E03 (alertas automáticos de frequência baixa).

## Decisões em aberto / riscos

- Se este épico for implementado antes de F1-E05 estar pronto, definir um conceito simplificado de "aula do dia" (turma + disciplina + data) para não bloquear o desenvolvimento na dependência de horários.

## Próximo passo

Já especificado em [`spec-frequencia.md`](spec-frequencia.md). Ver status no [índice](../README.md).
