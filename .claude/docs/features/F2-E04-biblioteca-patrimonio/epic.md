# F2-E04 — Biblioteca e patrimônio

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F2-E04 — Biblioteca e patrimônio |
| Fase | F2 (Eficiência & Comunicação) |
| Prioridade | Should |
| Estimativa | M |
| Bounded context(s) | `library` (novo) |
| Depende de | F1-E01 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F2-E04` |

## Objetivo

Cadastro de acervo com empréstimo/devolução/multa, e controle de patrimônio escolar (equipamentos/mobiliário).

## Histórias cobertas

- `F2-E04-U01` — Empréstimo de livro reaproveitando cadastro de aluno existente.
- `F2-E04-U02` — Cadastro de item de patrimônio com localização e status.

## Escopo / fora de escopo

- **Dentro:** acervo, empréstimo, multa por atraso, cadastro/relatório de patrimônio.
- **Fora:** RFID/código de barras físico — citado no documento-fonte como "implementação mais avançada", não é critério de aceite deste épico.

## Dependências e integrações

Depende de F1-E01 (`Student` como tomador do empréstimo). Sem dependência de outros épicos de negócio — é o contexto mais independente da Fase 2.

## Decisões em aberto / riscos

Nenhuma decisão maior em aberto.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
