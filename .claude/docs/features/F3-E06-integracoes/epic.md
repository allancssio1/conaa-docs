# F3-E06 — Ecossistema de integrações

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F3-E06 — Ecossistema de integrações |
| Fase | F3 (Analytics, IA & Diferenciação) |
| Prioridade | Should |
| Estimativa | GG |
| Bounded context(s) | `integrations` (novo) |
| Depende de | F1-E09 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F3-E06` |

## Objetivo

Oferecer API pública documentada com autenticação por token e webhooks, e sincronizar contas com Google Workspace/Microsoft 365.

## Histórias cobertas

- `F3-E06-U01` — API pública documentada, autenticada, com webhooks.
- `F3-E06-U02` — Sincronização de contas de alunos/professores com Google/Microsoft.

## Escopo / fora de escopo

- **Dentro:** camada de API pública, webhooks, provisionamento externo de conta.
- **Fora:** marketplace de integrações de terceiros / SDKs — mencionado no documento-fonte como visão de longo prazo, não é critério de aceite.

## Dependências e integrações

Depende de F1-E09 (perfis/permissões que a API pública deve respeitar). Decisão pendente: expor a mesma API interna via gateway de API keys, ou criar uma API pública separada/versionada.

## Decisões em aberto / riscos

- Expor dados de alunos/responsáveis via API pública tem implicação direta de LGPD — qualquer decisão de escopo de dados exposto deve ser revisada à luz de F1-E09 antes de especificar.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
