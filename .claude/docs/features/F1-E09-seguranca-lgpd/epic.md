# F1-E09 — Segurança, perfis de acesso e LGPD

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E09 — Segurança, perfis de acesso e LGPD |
| Fase | F1 (MVP) |
| Prioridade | Must |
| Estimativa | M |
| Bounded context(s) | `iam` |
| Depende de | F1-E01 (transversal) |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F1-E09` |

## Objetivo

Definir perfis de acesso com permissões por módulo, manter trilha de auditoria de ações sensíveis e gerenciar consentimento LGPD dos responsáveis.

## Histórias cobertas

- `F1-E09-U01` — Perfis de acesso configuráveis (secretaria, coordenação, professor, financeiro, direção, responsável, aluno).
- `F1-E09-U02` — Trilha de auditoria de ações sensíveis (notas, dados pessoais, baixas financeiras).
- `F1-E09-U03` — Registro/revogação de consentimento LGPD por responsável.

## Escopo / fora de escopo

- **Dentro:** guard de perfis por módulo, log de auditoria, tela/registro de consentimento.
- **Fora:** gestão de perfis por unidade/rede (isso é extensão em F2-E07 — aqui é só perfil por usuário, sem dimensão de unidade).

## Dependências e integrações

É transversal: vários épicos anteriores (F1-E04 notas, F1-E06 financeiro) já preveem auditoria das próprias alterações, que na prática é implementada por este épico. Idealmente entra em paralelo desde o início da Fase 1, não só depois de F1-E08.

## Decisões em aberto / riscos

- Decidir se o guard de perfil fino (por módulo) já é enforced desde `F1-E00`/`F1-E01` com um esqueleto mínimo, ou se cada épico anterior sobe sem checagem de papel e este épico "fecha a casa" depois — impacta o volume de retrabalho.

## Próximo passo

Já especificado em [`spec-seguranca-lgpd.md`](spec-seguranca-lgpd.md). Ver status no [índice](../README.md).
