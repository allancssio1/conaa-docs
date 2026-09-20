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
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F1-E09` para histórias/critérios completos (referência, não leitura obrigatória) |

## Objetivo

Definir perfis de acesso com permissões por módulo, manter trilha de auditoria de ações sensíveis e gerenciar consentimento LGPD dos responsáveis.

## Histórias cobertas

- `F1-E09-U01` — Perfis de acesso configuráveis (secretaria, coordenação, professor, financeiro, direção, responsável, aluno).
- `F1-E09-U02` — Trilha de auditoria de ações sensíveis (notas, dados pessoais, baixas financeiras).
- `F1-E09-U03` — Registro/revogação de consentimento LGPD por responsável.

## Decisão resolvida nesta spec

O épico deixava em aberto se o guard fino de perfil seria enforced desde o início ou "fechado" só agora. Resolução: **fechado agora, com retrofit explícito**. Todos os controllers de `F1-E01` a `F1-E08` foram especificados usando só `JwtAuthGuard` padrão (autenticado, sem checagem de papel) e anotando qual perfil *deveria* ser exigido. Esta ficha:
1. Implementa o guard fino (`PermissionsGuard` + decorator `@RequirePermission(module, action)`).
2. Traz, como parte do checklist desta ficha (não uma spec separada), a tarefa de **adicionar o decorator em cada controller já implementado**, usando o perfil já documentado na seção HTTP de cada spec anterior como fonte de verdade — sem reabrir decisão de negócio, só aplicando o guard.

Da mesma forma, a auditoria (`F1-E09-U02`) é feita retroativamente: esta ficha expõe um port `AuditLogger` (`abstract class`, `core/audit/audit-logger.ts`) e o adiciona nos use cases sensíveis já existentes (`UpdateStudentStatusUseCase` de `F1-E01`, `LaunchGradesUseCase`/`PublishAssessmentUseCase` de `F1-E04`, `RegisterPaymentUseCase` de `F1-E06`) via injeção — 1 linha de chamada a `auditLogger.record(...)` no fim de cada `execute()`, sem mudar a assinatura pública do use case.

## Multi-tenancy

Mecanismo completo em [`arquitetura-ignite.md` §11](../../../arquitetura-ignite.md#11-multi-tenancy-fundacional-desde-o-dia-zero) e [`F1-E0A`](../F1-E0A-tenancy/spec-tenancy.md). Esta ficha é onde o **escopo de escola realmente é atribuído**, não só consumido:

- `Role`, `AuditLog`, `ConsentRecord` carregam `groupId` — perfis, auditoria e consentimento são configurados por group, não globalmente. `RoleName` é único **por group** (dois groups podem ter, cada um, seu próprio `'COORDENACAO'` com permissões diferentes).
- **`UserRole` carrega `schoolId?: string`** — é o campo que decide o escopo de cada atribuição de papel: `null`/ausente = papel **group-wide** (ex.: mantenedora, mantenedora-financeiro, vê todas as escolas do group); preenchido = papel **school-scoped** (ex.: diretor, secretário de uma unidade específica). Um mesmo usuário pode ter múltiplos `UserRole`, alguns group-wide e outros restritos a escolas diferentes.
- `GroupScopeGuard` (implementado em `F1-E0A`, com um stub `AllowAllSchoolsProvider` até esta ficha existir) passa a usar `UserRolesRepository` real: monta `allowedSchoolIds = null` se o usuário tiver qualquer `UserRole` sem `schoolId`; caso contrário, `allowedSchoolIds` = união dos `schoolId` de todos os seus `UserRole`. **Trocar o binding do provider no módulo por esta implementação real é parte do checklist desta ficha.**
- `AuditLogger.record(...)` grava `groupId` e, quando aplicável, `schoolId` da entidade auditada.

## Modelo de domínio (`enterprise`)

### Contexto `iam` — `conaa-api/src/domain/iam/enterprise/`

| Entidade | Tipo | Caminho (`entities/`) | Props principais |
| --- | --- | --- | --- |
| `Role` | `Entity` | `role.ts` | `name: RoleName`, `permissions: Permission[]` |
| `UserRole` | `Entity` | `user-role.ts` | `userId`, `roleId`, `schoolId?: string` (N:N — um usuário pode ter múltiplos papéis; `schoolId` ausente = papel group-wide, preenchido = restrito àquela escola — ver seção "Multi-tenancy") |
| `AuditLog` | `Entity` | `audit-log.ts` | `userId`, `module`, `action: AuditAction`, `entityId`, `previousValue: unknown`, `newValue: unknown`, `occurredAt` |
| `ConsentRecord` | `Entity` | `consent-record.ts` | `studentId`, `guardianId`, `status: ConsentStatus`, `termVersion: string`, `occurredAt` |

Value objects — `entities/valueObjects/`:
- `role-name.ts` — `RoleName = 'SECRETARIA' | 'COORDENACAO' | 'PROFESSOR' | 'FINANCEIRO' | 'DIRECAO' | 'RESPONSAVEL' | 'ALUNO'`.
- `permission.ts` — `Permission` (`module: string`, `actions: ('VIEW' | 'EDIT')[]`).
- `audit-action.ts` — `AuditAction = 'CREATE' | 'UPDATE' | 'DELETE'`.
- `consent-status.ts` — `ConsentStatus = 'GRANTED' | 'REVOKED'`.

Invariantes:
- Um `userId` pode ter mais de um `UserRole` (ex.: coordenador que também é professor) — não há unicidade de papel por usuário.
- `ConsentRecord` é sempre criado (append-only), nunca atualizado: revogar consentimento cria um novo registro com `status = 'REVOKED'`, preservando o histórico completo.
- Não existe entidade `ConsentTerm` separada nesta ficha — `termVersion` é só uma string livre gerenciada fora do sistema (ex.: `"2026-v1"`); versionar o texto do termo em si é responsabilidade de conteúdo/jurídico, não deste módulo.

## Use cases (`application`)

### `conaa-api/src/domain/iam/application/useCases/`

| Use case | Arquivo | Entrada | Saída (`Either`) | Erros específicos |
| --- | --- | --- | --- | --- |
| `ConfigureRolePermissionsUseCase` | `configure-role-permissions.ts` | `roleName`, `permissions` | `Either<never, { role: Role }>` | — |
| `AssignRoleToUserUseCase` | `assign-role-to-user.ts` | `userId`, `roleName`, `schoolId?` | `Either<ResourceNotFoundError, { userRole: UserRole }>` | — |
| `GetAuditTrailUseCase` | `get-audit-trail.ts` | `userId?`, `module?`, `startDate?`, `endDate?` | `Either<never, { logs: AuditLog[] }>` | — |
| `RegisterConsentUseCase` | `register-consent.ts` | `studentId`, `guardianId`, `termVersion` | `Either<ResourceNotFoundError, { consent: ConsentRecord }>` | — |
| `RevokeConsentUseCase` | `revoke-consent.ts` | `studentId`, `guardianId` | `Either<ResourceNotFoundError \| ConsentNotFoundError, { consent: ConsentRecord }>` | `consent-not-found-error.ts` |
| `GetConsentStatusUseCase` | `get-consent-status.ts` | `studentId` | `Either<ResourceNotFoundError, { records: ConsentRecord[]; currentStatus: ConsentStatus }>` | — |

Ports (`application/repositories/`): `RolesRepository`, `UserRolesRepository`, `AuditLogsRepository`, `ConsentRecordsRepository`.
Port de infraestrutura transversal (`core/audit/`): `AuditLogger` — `abstract record(entry: { userId, module, action, entityId, previousValue, newValue }): Promise<void>`. Implementado por `AuditLogsRepository` internamente (o port fica em `core` para poder ser injetado em use cases de qualquer contexto sem criar dependência de `iam/application` a partir de `people`/`assessment`/`finance`).

## Infra transversal — guard de permissões

- `conaa-api/src/infra/auth/permissions.guard.ts` — `PermissionsGuard`: lê o `role` do JWT (`@CurrentUser()`), busca `UserRole`s + `Role.permissions`, compara com o metadata setado pelo decorator.
- `conaa-api/src/infra/auth/require-permission.decorator.ts` — `@RequirePermission(module: string, action: 'VIEW' | 'EDIT')`: `SetMetadata` lido pelo `PermissionsGuard`.
- Registrar `PermissionsGuard` como segundo guard global (`APP_GUARD`), depois do `JwtAuthGuard` já existente — rotas sem `@RequirePermission` continuam exigindo só autenticação (comportamento atual preservado).

## Persistência (`infra/database/prisma`)

- `conaa-api/prisma/schema.prisma`: novos models `Role`, `UserRole`, `AuditLog`, `ConsentRecord`; enums `RoleName`, `AuditAction`, `ConsentStatus`. `Permission` como campo `Json` em `Role` (lista de `{ module, actions }`, não uma tabela própria — evita join extra para uma lista pequena e de baixa cardinalidade).
- Repositórios: `prisma-roles-repository.ts`, `prisma-user-roles-repository.ts`, `prisma-audit-logs-repository.ts`, `prisma-consent-records-repository.ts` em `infra/database/prisma/repositories/`.
- Mappers: `role-mapper.ts`, `user-role-mapper.ts`, `audit-log-mapper.ts`, `consent-record-mapper.ts`.

## HTTP (`infra/http`)

| Rota | Controller | Schema Zod (entrada) | Presenter |
| --- | --- | --- | --- |
| `PUT /roles/:roleName/permissions` | `configure-role-permissions.controller.ts` | `{ permissions }` | `role-presenter.ts` |
| `POST /users/:userId/roles` | `assign-role-to-user.controller.ts` | `{ roleName, schoolId? }` | `user-role-presenter.ts` |
| `GET /audit-logs` | `get-audit-trail.controller.ts` | query `{ userId?, module?, startDate?, endDate? }` | `audit-log-presenter.ts` (lista) |
| `POST /students/:studentId/consent` | `register-consent.controller.ts` | `{ guardianId, termVersion }` | `consent-record-presenter.ts` |
| `DELETE /students/:studentId/consent` | `revoke-consent.controller.ts` | `{ guardianId }` | `consent-record-presenter.ts` |
| `GET /students/:studentId/consent` | `get-consent-status.controller.ts` | — (param) | `consent-record-presenter.ts` (lista + `currentStatus`) |

Todas exigem `@RequirePermission('iam', 'EDIT'|'VIEW')`, perfil `direção`/`administrador` — exceto `register-consent`/`revoke-consent`, acessíveis também por `responsável` do próprio filho.

### Retrofit nos controllers existentes (checklist desta ficha)

Adicionar `@RequirePermission(<module>, <action>)` em cada controller de `F1-E01` a `F1-E08`, usando o mapeamento já anotado na seção "Perfis exigidos"/"HTTP" de cada spec:

| Épico | Módulo (`module`) | Controllers afetados |
| --- | --- | --- |
| F1-E01 | `people`, `academic` | todos os 12 controllers listados em `spec-cadastros-sis.md` |
| F1-E02 | `academic` | todos os 7 controllers de `spec-matricula.md` |
| F1-E03 | `attendance` | os 3 controllers de `spec-frequencia.md` |
| F1-E04 | `assessment` | os 6 controllers de `spec-avaliacoes-notas.md` |
| F1-E05 | `academic` | os 5 controllers de `spec-horarios.md` |
| F1-E06 | `finance` | os 4 controllers de `spec-financeiro.md` |
| F1-E07 | `reporting` | os 6 controllers de `spec-relatorios.md` |
| F1-E08 | `people`, `attendance` | os 2 controllers novos de `spec-portais-web.md` |

## Frontend (`conaa-web`)

- `app/(portal)/admin/perfis/page.tsx` — configuração de permissões por perfil (`F1-E09-U01`).
- `app/(portal)/admin/auditoria/page.tsx` — trilha de auditoria filtrável (`F1-E09-U02`).
- `app/(public)/consentimento/page.tsx` — termo de consentimento apresentado no primeiro acesso do responsável (`F1-E09-U03`), integrado ao fluxo de login existente.
- `features/iam/components/RolePermissionsForm.tsx`, `AuditLogTable.tsx`, `ConsentTermDialog.tsx`.
- `features/iam/api/iam.ts` (`configureRolePermissions`, `assignRoleToUser`, `getAuditTrail`, `registerConsent`, `revokeConsent`, `getConsentStatus`), `features/iam/schemas/iam.ts`.
- `shared/auth`: expor `role`/`permissions` do usuário logado para esconder ações na UI conforme permissão (complementa o guard do backend, não substitui).

## Arquivos a criar/editar (checklist)

- [ ] `conaa-api/src/domain/iam/enterprise/entities/role.ts`
- [ ] `conaa-api/src/domain/iam/enterprise/entities/user-role.ts`
- [ ] `conaa-api/src/domain/iam/enterprise/entities/audit-log.ts`
- [ ] `conaa-api/src/domain/iam/enterprise/entities/consent-record.ts`
- [ ] `conaa-api/src/domain/iam/enterprise/entities/valueObjects/role-name.ts`
- [ ] `conaa-api/src/domain/iam/enterprise/entities/valueObjects/permission.ts`
- [ ] `conaa-api/src/domain/iam/enterprise/entities/valueObjects/audit-action.ts`
- [ ] `conaa-api/src/domain/iam/enterprise/entities/valueObjects/consent-status.ts`
- [ ] `conaa-api/src/domain/iam/application/useCases/configure-role-permissions.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/iam/application/useCases/assign-role-to-user.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/iam/application/useCases/get-audit-trail.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/iam/application/useCases/register-consent.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/iam/application/useCases/revoke-consent.ts` (+ errors/, + .spec.ts)
- [ ] `conaa-api/src/domain/iam/application/useCases/get-consent-status.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/iam/application/repositories/roles-repository.ts`
- [ ] `conaa-api/src/domain/iam/application/repositories/user-roles-repository.ts`
- [ ] `conaa-api/src/domain/iam/application/repositories/audit-logs-repository.ts`
- [ ] `conaa-api/src/domain/iam/application/repositories/consent-records-repository.ts`
- [ ] `conaa-api/src/core/audit/audit-logger.ts`
- [ ] `conaa-api/prisma/schema.prisma` (editar)
- [ ] `conaa-api/src/infra/database/prisma/repositories/*.ts` (4 repositórios, ver seção Persistência)
- [ ] `conaa-api/src/infra/database/prisma/mappers/*.ts` (4 mappers, ver seção Persistência)
- [ ] `conaa-api/src/infra/auth/permissions.guard.ts`
- [ ] `conaa-api/src/infra/auth/require-permission.decorator.ts`
- [ ] `conaa-api/src/infra/http/controllers/*.ts` (6 controllers novos, ver seção HTTP)
- [ ] `conaa-api/src/infra/http/presenters/*.ts` (4 presenters, ver seção HTTP)
- [ ] Retrofit: adicionar `@RequirePermission` em todos os controllers de `F1-E01` a `F1-E08` (ver tabela acima)
- [ ] Retrofit: injetar `AuditLogger` em `UpdateStudentStatusUseCase`, `LaunchGradesUseCase`, `PublishAssessmentUseCase`, `RegisterPaymentUseCase`
- [ ] Retrofit: trocar o binding de `GroupScopeGuard` de `AllowAllSchoolsProvider` (stub de `F1-E0A`) para a implementação real baseada em `UserRolesRepository` (ver seção "Multi-tenancy")
- [ ] `conaa-web/app/(portal)/admin/perfis/page.tsx`
- [ ] `conaa-web/app/(portal)/admin/auditoria/page.tsx`
- [ ] `conaa-web/app/(public)/consentimento/page.tsx`
- [ ] `conaa-web/features/iam/components/*.tsx`
- [ ] `conaa-web/features/iam/api/iam.ts`
- [ ] `conaa-web/features/iam/schemas/iam.ts`

## Testes

- Repositórios in-memory: `in-memory-roles-repository.ts`, `in-memory-user-roles-repository.ts`, `in-memory-audit-logs-repository.ts`, `in-memory-consent-records-repository.ts`.
- Factories: `make-role.ts`, `make-user-role.ts`, `make-audit-log.ts`, `make-consent-record.ts`.
- Unit specs: múltiplos papéis por usuário (`AssignRoleToUserUseCase`), filtro de trilha de auditoria por usuário/período/módulo (`GetAuditTrailUseCase`), consentimento append-only e `currentStatus` sempre reflete o registro mais recente (`RegisterConsentUseCase`/`RevokeConsentUseCase`/`GetConsentStatusUseCase`).
- E2E: um `.e2e-spec.ts` por controller novo, mais um teste e2e específico verificando que uma rota com `@RequirePermission` retorna `403` para um usuário sem a permissão (usando qualquer controller retrofitado como exemplo, ex.: `POST /students` sem perfil `secretaria`).

## Definition of Done

- [ ] Critérios de aceite de `F1-E09-U01` a `F1-E09-U03` no BACKLOG satisfeitos.
- [ ] Todos os use cases e controllers novos implementados com testes unitários e e2e passando.
- [ ] Migração Prisma aplicada sem erro.
- [ ] Retrofit de `@RequirePermission` concluído em todos os controllers de `F1-E01` a `F1-E08` (checklist acima).
- [ ] Retrofit de `AuditLogger` concluído nos 4 use cases sensíveis listados.
- [ ] Tela de configuração de perfis, trilha de auditoria e termo de consentimento funcionando ponta a ponta.
- [ ] `.claude/docs/features/README.md` atualizado: status de `F1-E09` para 🟢 quando implementado.
