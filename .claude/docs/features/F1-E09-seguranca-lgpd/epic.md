# F1-E09 — Segurança, perfis de acesso e LGPD

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E09 — Segurança, perfis de acesso e LGPD |
| Fase | F1 (MVP) |
| Prioridade | Must |
| Estimativa | M |
| Bounded context(s) | `iam` |
| Depende de | F1-E01, F1-E10 (transversal) |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F1-E09` |

## Objetivo

Definir perfis de acesso com permissões por módulo, restringir dados de aluno a quem tem vínculo com ele (responsável/aluno), manter trilha de auditoria de ações sensíveis e dar aos responsáveis os direitos do titular previstos na LGPD (consentimento, exportação, canal do encarregado).

## Histórias cobertas

- `F1-E09-U01` — Perfis de acesso configuráveis (secretaria, coordenação, professor, financeiro, direção, responsável, aluno, admin), com escopo por escola quando aplicável.
- `F1-E09-U02` — Trilha de auditoria de ações sensíveis (notas, dados pessoais, baixas financeiras), com mascaramento de campos sensíveis.
- `F1-E09-U03` — Registro/revogação de consentimento LGPD por responsável.
- `F1-E09-U04` — Responsável exporta os dados do próprio filho (direito de acesso/portabilidade).
- `F1-E09-U05` — Canal de privacidade com contato do encarregado (DPO) do group, para corrigir/eliminar dados.

## Escopo / fora de escopo

- **Dentro:** guard de perfis por módulo, restrição por aluno (`allowedStudentIds`, contra IDOR), log de auditoria, registro de consentimento, exportação dos dados do titular, canal de contato do encarregado.
- **Fora:** gestão de perfis por unidade/rede consolidada — várias escolas numa sessão só (isso é extensão em F2-E07); automação de retenção/anonimização (proposta na spec, sem automação até a Fase 2); correção/eliminação de dados feitas manualmente pelo encarregado, não por um fluxo no sistema.

## Dependências e integrações

É transversal: vários épicos anteriores (F1-E04 notas, F1-E06 financeiro) já preveem auditoria das próprias alterações, que na prática é implementada por este épico. Depende também de `F1-E10` (identidade e acesso) — antes dela não existe `User` real de onde ler papéis. Idealmente entra em paralelo desde o início da Fase 1, não só depois de F1-E08.

## Decisões em aberto / riscos

Nenhuma em aberto — resolvidas ao especificar (ver `spec-seguranca-lgpd.md`): o guard de perfil fino é enforced desde `F1-E00`/`F1-E01` como `JwtAuthGuard` simples, e esta ficha faz o retrofit de `@RequirePermission` em todos os controllers anteriores usando o perfil já anotado em cada spec — não é um "fechar a casa" improvisado, é um checklist explícito desta ficha.

## Próximo passo

Já especificado em duas fichas nesta pasta:
- [`spec-seguranca-lgpd.md`](spec-seguranca-lgpd.md) — perfis, permissões, restrição por aluno e auditoria (`U01`, `U02`, `U03`).
- [`spec-direitos-titular.md`](spec-direitos-titular.md) — exportação de dados e canal de privacidade (`U04`, `U05`).

Ver status no [índice](../README.md).
