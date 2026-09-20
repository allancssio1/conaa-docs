# F1-E06 — Financeiro essencial (mensalidades)

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E06 — Financeiro essencial (mensalidades) |
| Fase | F1 (MVP) |
| Prioridade | Must |
| Estimativa | G |
| Bounded context(s) | `finance` |
| Depende de | F1-E01, F1-E02 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F1-E06` |

## Objetivo

Cadastrar planos de mensalidade, gerar títulos a receber, registrar baixa de pagamento (manual) e reportar inadimplência.

## Histórias cobertas

- `F1-E06-U01` — Cadastrar plano de mensalidade e gerar títulos automaticamente ao vincular a um aluno.
- `F1-E06-U02` — Emitir boleto e registrar baixa de pagamento (manual).
- `F1-E06-U03` — Consultar situação financeira por aluno, turma ou série.

## Escopo / fora de escopo

- **Dentro:** plano de mensalidade, título, baixa manual, relatório de inadimplência.
- **Fora:** integração real com gateway de pagamento/baixa automática (isso é F2-E03 — aqui a baixa é registrada manualmente).

## Dependências e integrações

Depende de F1-E01 (responsável financeiro) e F1-E02 (matrícula ativa). Consumido por F1-E02 (checar pendência na rematrícula), F1-E07 (relatório de inadimplência) e, mais adiante, F2-E03 (pagamento on-line) e F3-E03 (bloqueio automático).

## Decisões em aberto / riscos

- Vale decidir já o modelo de dados de "título/boleto" pensando em não retrabalhar quando F2-E03 (gateway) for especificado — ex.: campo para identificador externo do gateway, mesmo que não usado ainda.

## Próximo passo

Já especificado em [`spec-financeiro.md`](spec-financeiro.md). Ver status no [índice](../README.md).
