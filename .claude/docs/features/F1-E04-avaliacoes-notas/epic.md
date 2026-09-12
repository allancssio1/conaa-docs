# F1-E04 — Avaliações, notas e boletins

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E04 — Avaliações, notas e boletins |
| Fase | F1 (MVP) |
| Prioridade | Must |
| Estimativa | G |
| Bounded context(s) | `assessment` |
| Depende de | F1-E02, F1-E05 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F1-E04` |

## Objetivo

Lançar notas/conceitos com pesos e médias configuráveis por segmento, gerar boletim para responsáveis/alunos e histórico escolar consolidado.

## Histórias cobertas

- `F1-E04-U01` — Coordenação configura fórmula de média por segmento (pesos, recuperação).
- `F1-E04-U02` — Professor lança notas de uma atividade para toda a turma de uma vez.
- `F1-E04-U03` — Responsável visualiza boletim do filho por disciplina/período.
- `F1-E04-U04` — Secretaria gera histórico escolar consolidado do aluno.

## Escopo / fora de escopo

- **Dentro:** lançamento de notas/conceitos, médias, recuperação, boletim, histórico escolar.
- **Fora:** exibição no portal em si (isso é F1-E08 — este épico expõe a API que o portal consome), mapeamento avançado por competência BNCC (F3-E02).

## Dependências e integrações

Depende de F1-E02 (turma/disciplina) e F1-E05 (grade, para relacionar lançamento a um período/aula). Usado por F1-E02 (critério de aprovação) e F1-E07 (relatórios de notas).

## Decisões em aberto / riscos

- Regras de recuperação e arredondamento variam muito por escola — decidir na spec se o motor de fórmula é totalmente configurável ou se há fórmulas fixas por segmento (Infantil/Fundamental/Médio) com poucos parâmetros.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
