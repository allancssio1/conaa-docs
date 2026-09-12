# F2-E06 — RH e folha de pagamento

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F2-E06 — RH e folha de pagamento |
| Fase | F2 (Eficiência & Comunicação) |
| Prioridade | Could |
| Estimativa | G |
| Bounded context(s) | `hr` (novo) |
| Depende de | F1-E01 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F2-E06` |

## Objetivo

Cadastro de colaboradores (jornada, férias, afastamentos) e cálculo de folha de pagamento básico, exportável para escritórios de contabilidade.

## Histórias cobertas

- `F2-E06-U01` — Cadastro de colaborador com jornada/férias/afastamentos.
- `F2-E06-U02` — Cálculo de folha integrado a jornada/afastamentos, exportável.

## Escopo / fora de escopo

- **Dentro:** colaborador, jornada, férias, afastamento, cálculo de folha, exportação.
- **Fora:** integração automática direta com sistemas contábeis externos (aqui é só exportação em formato aceito).

## Dependências e integrações

Depende de F1-E01 (reaproveita ou estende `Teacher`/`Staff` como base de colaborador). Contexto relativamente isolado do resto da Fase 2.

## Decisões em aberto / riscos

- Regras de cálculo de folha (CLT, encargos, décimo terceiro) variam por legislação trabalhista e podem exigir validação especializada antes de especificar em detalhe — não assumir que é "só uma conta".

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
