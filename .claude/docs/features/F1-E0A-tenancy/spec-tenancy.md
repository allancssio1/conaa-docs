# F1-E0A — Multi-tenancy (Group e School)

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E0A — Multi-tenancy (Group e School) |
| Fase | F1 (pré-requisito de todas as demais fichas de domínio) |
| Prioridade | Must |
| Estimativa | M |
| Bounded context(s) | `tenancy` |
| Depende de | F1-E00 (bootstrap) |
| Rastreabilidade | Decisão de arquitetura — ver [`arquitetura-ignite.md` §11](../../../arquitetura-ignite.md#11-multi-tenancy-fundacional-desde-o-dia-zero) para o modelo completo (esta ficha implementa esse modelo). |

## Objetivo

Implementar a fundação multi-tenant: entidades `Group` (tenant) e `School`, o onboarding de ambas,
e o mecanismo de isolamento (`groupId`) + escopo de permissão (`schoolId`) que todo épico de
domínio subsequente (F1-E01 em diante) vai consumir. **Leia primeiro
[`arquitetura-ignite.md` §11](../../../arquitetura-ignite.md#11-multi-tenancy-fundacional-desde-o-dia-zero)**
— esta spec implementa o modelo descrito lá, não repete a explicação conceitual.

## Modelo de domínio (`enterprise`)

### Contexto `tenancy` — `conaa-api/src/domain/tenancy/enterprise/`

| Entidade | Tipo | Caminho (`entities/`) | Props principais |
| --- | --- | --- | --- |
| `Group` | `AggregateRoot` | `group.ts` | `name`, `type: GroupType`, `document?` (CNPJ), `status: GroupStatus`, `createdAt`, `updatedAt` |
| `School` | `Entity` | `school.ts` | `groupId`, `name`, `inepCode?`, `address: Address` (reusa VO de `people`, `F1-E01` — se `F1-E0A` for implementado antes, declarar o VO aqui e `F1-E01` reusa de `tenancy`; a ordem de quem declara não importa, o importante é não duplicar), `status: SchoolStatus`, `createdAt`, `updatedAt` |

Value objects — `entities/valueObjects/`:
- `group-type.ts` — `GroupType = 'SECRETARIA' | 'MUNICIPIO' | 'REDE' | 'ESCOLA'` (rótulo informativo, não muda nenhum comportamento de isolamento).
- `group-status.ts` — `GroupStatus = 'ACTIVE' | 'SUSPENDED'`.
- `school-status.ts` — `SchoolStatus = 'ACTIVE' | 'INACTIVE'`.

Invariantes:
- `Group` e `School` **não** carregam `groupId`/`schoolId` como as demais entidades do sistema — são elas a raiz da hierarquia de tenancy.
- `School.groupId` deve referenciar um `Group` existente com `status = 'ACTIVE'`.
- Um `Group` com `status = 'SUSPENDED'` bloqueia login de qualquer usuário vinculado a ele (verificado no `GroupScopeGuard`, não nesta entidade).

## Use cases (`application`)

### `conaa-api/src/domain/tenancy/application/useCases/`

| Use case | Arquivo | Entrada | Saída (`Either`) | Erros específicos |
| --- | --- | --- | --- | --- |
| `RegisterGroupUseCase` | `register-group.ts` | `name`, `type`, `document?` | `Either<InvalidDocumentError, { group: Group }>` | `invalid-document-error.ts` |
| `RegisterSchoolUseCase` | `register-school.ts` | `groupId`, `name`, `inepCode?`, `address` | `Either<ResourceNotFoundError, { school: School }>` | — |
| `ListGroupSchoolsUseCase` | `list-group-schools.ts` | `groupId` | `Either<ResourceNotFoundError, { schools: School[] }>` | — |

Ports (`application/repositories/`): `GroupsRepository`, `SchoolsRepository`.

Regras de negócio principais:
- `RegisterGroupUseCase`/`RegisterSchoolUseCase` rodam **fora** do `GroupContext` (onboarding —
  ver seção HTTP): não há `groupId` de sessão ainda quando um `Group` está sendo criado pela
  primeira vez.
- `RegisterSchoolUseCase`: valida que `groupId` existe e está `ACTIVE` antes de criar a escola.
- Nenhum destes use cases decide quem pode chamá-los — isso é resolvido no controller (ver HTTP:
  rotas de onboarding exigem um fluxo de signup/super-admin próprio, fora do escopo desta ficha
  detalhar o processo comercial, só a mecânica técnica).

## Infra transversal de escopo (`core`/`infra`)

Esta é a parte mais importante da ficha — o contrato que **todos** os outros contextos vão usar.

### `GroupContext` — `conaa-api/src/core/tenancy/group-context.ts`

```ts
export type GroupContext = {
  groupId: string
  allowedSchoolIds: string[] | null // null = acesso a todas as escolas do group
}
```

Implementado com `AsyncLocalStorage<GroupContext>`, expondo `run(context, callback)` (popula) e
`get(): GroupContext` (lê o contexto corrente; lança erro de programação — não de negócio — se
chamado fora de uma requisição autenticada, já que isso indica um bug de wiring, não um caso de erro esperado).

### `GroupScopeGuard` — `conaa-api/src/infra/auth/group-scope.guard.ts`

- Roda **depois** do `JwtAuthGuard` já existente (`F1-E00`).
- Lê `groupId` do payload do JWT (`@CurrentUser()`).
- Busca os `UserRole` do usuário (port `UserRolesRepository`, de `F1-E09` — nesta ficha, se
  `F1-E09` ainda não estiver implementado, usar um provider stub `AllowAllSchoolsProvider` que
  sempre retorna `allowedSchoolIds: null`, substituído pela implementação real quando `F1-E09`
  existir, análogo ao padrão de stub já usado em `F1-E02`).
- Popula `GroupContext.run({ groupId, allowedSchoolIds }, () => next())`.
- Se `Group.status = 'SUSPENDED'`, retorna `403` antes de prosseguir.

### Prisma Client Extension — `conaa-api/src/infra/database/prisma/extensions/tenant-scope.extension.ts`

- Aplicada ao `PrismaService` (`$extends`) para todo model que tiver `groupId` no schema.
- **Leitura** (`findMany`, `findFirst`, `findUnique`→`findFirst` quando precisa filtrar, `count`,
  etc.): mescla `where: { groupId: GroupContext.get().groupId }`; se o model tiver `schoolId` e
  `allowedSchoolIds !== null`, mescla também `where: { schoolId: { in: allowedSchoolIds } }`.
- **Escrita** (`create`, `createMany`): mescla `data: { groupId: GroupContext.get().groupId }`.
  **Não** injeta `schoolId` automaticamente — cada repositório passa o `schoolId` explicitamente,
  vindo do use case (que já validou o valor contra o escopo do usuário antes de chegar ao repositório).
- Models `Group`/`School` ficam **fora** desta extension (são a raiz, não têm `groupId`).

### Validação de `schoolId` em escrita (helper reutilizável)

`conaa-api/src/core/tenancy/assert-school-in-scope.ts` — função pura `assertSchoolInScope(schoolId, context)`
usada pelos use cases de outros contextos antes de gravar uma entidade com `schoolId`: lança
`SchoolNotInScopeError` (`core/errors/errors/`) se `allowedSchoolIds !== null` e `schoolId` não
estiver na lista. Documentado aqui porque é consumido por praticamente todo use case de escrita
dos épicos seguintes — evita reimplementar a checagem em cada um.

## Persistência (`infra/database/prisma`)

- `conaa-api/prisma/schema.prisma`: novos models `Group`, `School` (sem `groupId`/`schoolId`
  próprios); enums `GroupType`, `GroupStatus`, `SchoolStatus`. A partir desta ficha, **todo novo
  model de negócio criado em qualquer épico seguinte inclui `groupId String` indexado** (e
  `schoolId String` indexado, se operacional) — convenção registrada em `arquitetura-ignite.md §11`.
- Repositórios: `prisma-groups-repository.ts`, `prisma-schools-repository.ts` em `infra/database/prisma/repositories/`.
- Extension: `infra/database/prisma/extensions/tenant-scope.extension.ts` (ver seção acima).
- Mappers: `group-mapper.ts`, `school-mapper.ts`.

## HTTP (`infra/http`)

| Rota | Controller | Schema Zod (entrada) | Presenter | Contexto |
| --- | --- | --- | --- | --- |
| `POST /groups` | `register-group.controller.ts` | dados de `RegisterGroupUseCase` | `group-presenter.ts` | `@Public()` ou guard de signup próprio — fora do `GroupScopeGuard` |
| `POST /groups/:groupId/schools` | `register-school.controller.ts` | dados de `RegisterSchoolUseCase` (exceto `groupId`) | `school-presenter.ts` | idem — onboarding |
| `GET /schools` | `list-group-schools.controller.ts` | — | `school-presenter.ts` (lista) | dentro do `GroupScopeGuard` normal (usa `groupId` da sessão, não de param) |

O onboarding (`POST /groups`, `POST /groups/:groupId/schools`) roda sem `GroupScopeGuard` porque
ainda não existe usuário/sessão vinculada ao `Group` que está sendo criado. O processo de negócio
completo de onboarding (quem pode criar um Group, criação do primeiro usuário admin) pertence a
`F1-E09`/fluxo de signup — esta ficha só entrega a mecânica dos dois use cases e das rotas.

## Frontend (`conaa-web`)

- `app/(public)/onboarding/grupo/page.tsx` — formulário de registro de `Group` (`RegisterGroupUseCase`).
- `app/(public)/onboarding/grupo/[groupId]/escolas/page.tsx` — cadastro de `School`(s) do group recém-criado.
- `app/(portal)/admin/escolas/page.tsx` — lista de escolas do group logado (`GET /schools`), usada como base do seletor de escola referenciado em `arquitetura-frontend.md §10`.
- `features/tenancy/components/RegisterGroupForm.tsx`, `RegisterSchoolForm.tsx`, `SchoolSelector.tsx` (componente compartilhado — outras telas de outros contextos importam este componente para o seletor de escola, não recriam um próprio).
- `features/tenancy/api/tenancy.ts` (`registerGroup`, `registerSchool`, `listGroupSchools`), `features/tenancy/schemas/tenancy.ts`.
- `shared/auth/useSession.ts`: passa a expor `groupId` e `allowedSchoolIds` (ver `arquitetura-frontend.md §10`).

## Arquivos a criar/editar (checklist)

- [ ] `conaa-api/src/domain/tenancy/enterprise/entities/group.ts`
- [ ] `conaa-api/src/domain/tenancy/enterprise/entities/school.ts`
- [ ] `conaa-api/src/domain/tenancy/enterprise/entities/valueObjects/group-type.ts`
- [ ] `conaa-api/src/domain/tenancy/enterprise/entities/valueObjects/group-status.ts`
- [ ] `conaa-api/src/domain/tenancy/enterprise/entities/valueObjects/school-status.ts`
- [ ] `conaa-api/src/domain/tenancy/application/useCases/register-group.ts` (+ errors/, + .spec.ts)
- [ ] `conaa-api/src/domain/tenancy/application/useCases/register-school.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/tenancy/application/useCases/list-group-schools.ts` (+ .spec.ts)
- [ ] `conaa-api/src/domain/tenancy/application/repositories/groups-repository.ts`
- [ ] `conaa-api/src/domain/tenancy/application/repositories/schools-repository.ts`
- [ ] `conaa-api/src/core/tenancy/group-context.ts`
- [ ] `conaa-api/src/core/tenancy/assert-school-in-scope.ts`
- [ ] `conaa-api/src/core/errors/errors/school-not-in-scope-error.ts`
- [ ] `conaa-api/src/infra/auth/group-scope.guard.ts`
- [ ] `conaa-api/src/infra/auth/allow-all-schools-provider.ts` (stub, até `F1-E09` existir)
- [ ] `conaa-api/prisma/schema.prisma` (editar — models `Group`/`School` + convenção `groupId`/`schoolId` para todo model futuro)
- [ ] `conaa-api/src/infra/database/prisma/repositories/prisma-groups-repository.ts`
- [ ] `conaa-api/src/infra/database/prisma/repositories/prisma-schools-repository.ts`
- [ ] `conaa-api/src/infra/database/prisma/extensions/tenant-scope.extension.ts`
- [ ] `conaa-api/src/infra/database/prisma/mappers/group-mapper.ts`
- [ ] `conaa-api/src/infra/database/prisma/mappers/school-mapper.ts`
- [ ] `conaa-api/src/infra/http/controllers/register-group.controller.ts`
- [ ] `conaa-api/src/infra/http/controllers/register-school.controller.ts`
- [ ] `conaa-api/src/infra/http/controllers/list-group-schools.controller.ts`
- [ ] `conaa-api/src/infra/http/presenters/group-presenter.ts`, `school-presenter.ts`
- [ ] `conaa-web/app/(public)/onboarding/grupo/page.tsx`
- [ ] `conaa-web/app/(public)/onboarding/grupo/[groupId]/escolas/page.tsx`
- [ ] `conaa-web/app/(portal)/admin/escolas/page.tsx`
- [ ] `conaa-web/features/tenancy/components/RegisterGroupForm.tsx`, `RegisterSchoolForm.tsx`, `SchoolSelector.tsx`
- [ ] `conaa-web/features/tenancy/api/tenancy.ts`
- [ ] `conaa-web/features/tenancy/schemas/tenancy.ts`
- [ ] `conaa-web/shared/auth/useSession.ts` (editar)

## Testes

- Repositórios in-memory: `in-memory-groups-repository.ts`, `in-memory-schools-repository.ts`
  (e, a partir desta ficha, **todo** repositório in-memory de qualquer contexto passa a filtrar
  por `groupId`/`schoolId` manualmente, replicando a extension — documentar isso no arquivo de
  cada repositório in-memory subsequente).
- Factories: `make-group.ts`, `make-school.ts`.
- Unit specs: `RegisterSchoolUseCase` rejeita `groupId` inexistente ou `SUSPENDED`;
  `assertSchoolInScope` aceita quando `allowedSchoolIds = null` e quando `schoolId` está na lista,
  rejeita quando não está.
- Teste de integração dedicado da extension: criar duas entidades de teste em `Group`s diferentes
  via Prisma real, provar que uma query sem filtro explícito (usando o client já estendido) só
  retorna a do `Group` do contexto corrente — este teste é a prova de que o isolamento funciona
  antes de qualquer contexto de domínio ser implementado sobre ele.
- E2E: um `.e2e-spec.ts` por controller listado na seção HTTP.

## Definition of Done

- [ ] `Group`/`School` implementados com use cases de onboarding testados.
- [ ] `GroupContext`, `GroupScopeGuard` e a Prisma extension implementados e cobertos pelo teste de
      integração de isolamento descrito em Testes.
- [ ] `assertSchoolInScope` implementado e exportado para uso pelos épicos seguintes.
- [ ] Migração Prisma aplicada sem erro.
- [ ] Onboarding de group + escolas funcionando ponta a ponta.
- [ ] `.claude/docs/features/README.md` atualizado: status de `F1-E0A` para 🟢 quando implementado — **antes** de qualquer épico de domínio (F1-E01 em diante) ser iniciado.
