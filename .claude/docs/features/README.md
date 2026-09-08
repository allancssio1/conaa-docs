# Índice de fichas de tarefa

Cada linha é uma unidade implementável (1 ficha = 1 épico, granularidade acordada). **Para implementar, siga o protocolo em [`CLAUDE.md`](../../../CLAUDE.md)** — não é necessário ler `BACKLOG.md`/`ROADMAP.md` para codar, só para planejar/priorizar.

**Status:**
- 🟢 `pronta` — ficha escrita, pode ser implementada.
- ⚪ `planejada` — existe no ROADMAP/BACKLOG, ficha ainda não foi detalhada. Peça para criá-la antes de implementar.

## Fundação

| ID | Título | Camadas | Status | Depende de |
| --- | --- | --- | --- | --- |
| [F1-E00](F1-E00-bootstrap.md) | Bootstrap do monorepo (scaffold api + web) | backend, frontend | 🟢 pronta | — |

## Fase 1 — MVP

| ID | Título | Bounded context | Camadas | Status | Depende de |
| --- | --- | --- | --- | --- | --- |
| [F1-E01](F1-E01-cadastros-sis.md) | Cadastros / SIS básico | `people`, `academic` | backend | 🟢 pronta | F1-E00 |
| F1-E02 | Matrícula, rematrícula e turmas | `academic` | backend | ⚪ planejada | F1-E01 |
| F1-E03 | Gestão de frequência | `attendance` | backend | ⚪ planejada | F1-E02, F1-E05 |
| F1-E04 | Avaliações, notas e boletins | `assessment` | backend | ⚪ planejada | F1-E02, F1-E05 |
| F1-E05 | Horários e grade de aulas | `academic` | backend | ⚪ planejada | F1-E02 |
| F1-E06 | Financeiro essencial | `finance` | backend | ⚪ planejada | F1-E01, F1-E02 |
| F1-E07 | Relatórios administrativos e oficiais | `reporting` | backend | ⚪ planejada | F1-E01…F1-E06 |
| F1-E08 | Portais web (pais, alunos, professores) | vários | frontend | ⚪ planejada | F1-E01…F1-E06 |
| F1-E09 | Segurança, perfis de acesso e LGPD | `iam` | backend | ⚪ planejada | F1-E01 (transversal) |

## Fase 2 — Eficiência & Comunicação

| ID | Título | Bounded context | Camadas | Status | Depende de |
| --- | --- | --- | --- | --- | --- |
| F2-E01 | App mobile | vários | mobile | ⚪ planejada | F1-E08 |
| F2-E02 | Comunicação multicanal e notificações | `communication` | backend, frontend | ⚪ planejada | F1-E01, F1-E03 |
| F2-E03 | Integração com meios de pagamento | `finance` | backend, frontend | ⚪ planejada | F1-E06 |
| F2-E04 | Biblioteca e patrimônio | `library` (novo) | backend, frontend | ⚪ planejada | F1-E01 |
| F2-E05 | Transporte escolar | `transport` (novo) | backend, frontend | ⚪ planejada | F1-E01, F1-E02 |
| F2-E06 | RH e folha de pagamento | `hr` (novo) | backend, frontend | ⚪ planejada | F1-E01 |
| F2-E07 | Multiunidade / rede escolar | `iam` (extensão) | backend, frontend | ⚪ planejada | F1-E01…F1-E09 |
| F2-E08 | LMS/EAD integrado básico | `lms` (novo) | backend, frontend | ⚪ planejada | F1-E02, F1-E05 |

## Fase 3 — Analytics, IA & Diferenciação

| ID | Título | Bounded context | Camadas | Status | Depende de |
| --- | --- | --- | --- | --- | --- |
| F3-E01 | Analytics/BI avançado e IA | `analytics` (novo) | backend, frontend | ⚪ planejada | F1-E03, F1-E04, F1-E06 |
| F3-E02 | Aderência avançada à BNCC | `assessment` (extensão) | backend, frontend | ⚪ planejada | F1-E04 |
| F3-E03 | Motor de fluxos/automações configuráveis | `automation` (novo) | backend, frontend | ⚪ planejada | F1-E03, F1-E06, F2-E02 |
| F3-E04 | Transporte com GPS e segurança avançada | `transport` (extensão) | backend, frontend | ⚪ planejada | F2-E05 |
| F3-E05 | Experiência mobile superior | vários | mobile | ⚪ planejada | F2-E01 |
| F3-E06 | Ecossistema de integrações | `integrations` (novo) | backend | ⚪ planejada | F1-E09 |
| F3-E07 | Gestão multi-rede com governança | `iam` (extensão) | backend, frontend | ⚪ planejada | F2-E07 |

## Como adicionar uma nova ficha

1. Copie [`_TEMPLATE.md`](_TEMPLATE.md) para `.claude/docs/features/<ID>-<slug>.md`.
2. Preencha usando o épico correspondente em [`BACKLOG.md`](../../BACKLOG.md) como fonte das histórias/critérios.
3. Atualize a linha do épico nesta tabela: link no ID e status para 🟢 `pronta`.
