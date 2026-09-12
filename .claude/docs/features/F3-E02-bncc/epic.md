# F3-E02 — Aderência avançada à BNCC e avaliação por competências

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F3-E02 — Aderência avançada à BNCC e avaliação por competências |
| Fase | F3 (Analytics, IA & Diferenciação) |
| Prioridade | Should |
| Estimativa | G |
| Bounded context(s) | `assessment` (extensão) |
| Depende de | F1-E04 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F3-E02` |

## Objetivo

Mapear avaliações e conteúdos para habilidades/competências da BNCC e relatar o domínio de cada aluno por competência, não só por nota.

## Histórias cobertas

- `F3-E02-U01` — Associar avaliação a habilidades BNCC e relatar domínio por competência.

## Escopo / fora de escopo

- **Dentro:** catálogo de habilidades BNCC, associação com avaliação, relatório de domínio por competência.
- **Fora:** nada relevante identificado fora de escopo — épico pequeno e focado.

## Dependências e integrações

Depende de F1-E04 (estende o modelo de avaliação existente). Precisa de uma base/catálogo oficial da BNCC como dado de referência.

## Decisões em aberto / riscos

- O catálogo de habilidades BNCC (milhares de códigos oficiais) precisa de fonte de dados — decidir se é importado de uma base pública oficial ou cadastrado manualmente pela escola.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
