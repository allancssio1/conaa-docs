# Arquitetura de referência: nest-clean-ignite

> Documento gerado a partir da análise do projeto `nest-clean-ignite` (curso Rocketseat), para servir de referência arquitetural ao `conaa-controle-escolar`. Stack: NestJS + Prisma + TypeScript, seguindo Clean Architecture com DDD leve.

## 1. Visão geral

O projeto segue **Clean Architecture** com padrões táticos de **DDD**, organizado em duas "bounded contexts" (`forum` e `notification`), cada uma isolada em `domain/<contexto>`. A regra de dependência é sempre para dentro:

```
infra  →  domain/application  →  domain/enterprise
                    ↓
                  core
```

- `enterprise` (regras de negócio da empresa): entidades, value objects, eventos — zero dependência de framework.
- `application` (regras de negócio da aplicação): use cases + interfaces de repositório/serviço (ports).
- `infra`: tudo que é framework/IO — NestJS, Prisma, HTTP, auth, storage. Implementa os ports definidos em `application`.
- `core`: base compartilhada entre todos os contextos (Entity, Either, eventos de domínio, erros genéricos).

## 2. Estrutura de pastas

```
src/
  core/          Building blocks compartilhados (Entity, AggregateRoot, Either, DomainEvents, erros genéricos)
  domain/
    forum/
      application/    use cases, repositories (interfaces/ports), cryptography, storage
      enterprise/     entidades, value objects, eventos de domínio
    notification/
      application/    use cases, repository (interface), subscribers de eventos
      enterprise/     entidade Notification
  infra/
    database/prisma/  PrismaService, repositórios Prisma, mappers
    http/              controllers, pipes, presenters
    auth/              JWT strategy, guards, decorators
    cryptography/       adapters (bcrypt, jwt)
    storage/            adapter (Cloudflare R2 / S3)
    env/                validação e acesso tipado a variáveis de ambiente
test/
  repositories/    repositórios in-memory (para testes unitários)
  factories/       factories de entidades (unit + persistência via Prisma para e2e)
  cryptography/    fakes de hash/encrypter
  storage/         fake uploader
  setup-e2e.ts     isola schema de banco por execução de teste e2e
prisma/
  schema.prisma
  migrations/
```

Cada `domain/<contexto>` é autocontido (`application` + `enterprise`) e plugado em `infra` de forma independente — facilita adicionar novos contextos sem acoplar aos existentes.

## 3. Camada de domínio (`enterprise`)

### Classes base (`src/core/entities/`)

- **`Entity<Props>`**: guarda `protected props: Props` + `UniqueEntityId` privado; expõe `.id` e `.equals()`.
- **`AggregateRoot<Props>`**: estende `Entity`, adiciona lista de `domainEvents` e `addDomainEvent()` (marca o agregado para disparo de eventos).
- **`UniqueEntityId`**: wrapper sobre `randomUUID()` do Node, com `toString()/toValue()/equals()`.
- **`ValueObject<Props>`**: base para VOs com igualdade estrutural.
- **`WatchedList<T>`**: lista genérica que rastreia itens `current/new/removed` — usada para coleções que pertencem a um agregado (ex.: anexos de uma pergunta), permitindo saber o que inserir/remover ao persistir.

### Padrão `Optional` para criação de entidades

```ts
type Optional<T, K extends keyof T> = Pick<Partial<T>, K> & Omit<T, K>
```

Usado no `create()` de cada entidade para tornar opcionais os campos deriváveis/padrão (ex.: `createdAt`, `slug`), mantendo-os obrigatórios internamente:

```ts
static create(props: Optional<QuestionProps, 'createdAt' | 'slug' | 'attachments'>, id?: UniqueEntityId)
```

### Entidades e value objects

- Entidades em `src/domain/forum/enterprise/entities/` (`Question`, `Answer`, `Comment`, `Attachment`, `Student`, `Instructor`, etc.). `Question` e `Answer` são `AggregateRoot`; comentários/anexos são `Entity` simples.
- Construtor privado + getters/setters que mutam `this.props` e chamam um `touch()` privado para atualizar `updatedAt`.
- Value objects em `entities/valueObjects/` (ex.: `Slug` — gera slug a partir do título).

### Eventos de domínio

- Barramento estático `DomainEvents` (`src/core/events/domain-events.ts`): mapa de handlers por nome da classe do evento + lista de agregados marcados para disparo.
- Eventos concretos em `src/domain/forum/enterprise/events/` (ex.: `AnswerCreatedEvent`), disparados dentro de setters da entidade via `this.addDomainEvent(...)`.
- Repositórios chamam `DomainEvents.dispatchEventsForAggregate(id)` após persistir (`create`/`save`) — só aí os handlers são efetivamente executados.
- **Subscribers** (padrão Observer) em `src/domain/notification/application/subscribers/` — o contexto `notification` escuta eventos do contexto `forum` (ex.: `OnAnswerCreated`) e reage chamando o próprio use case de notificação. Não há broker externo, é tudo em processo.

## 4. Camada de aplicação (`application`)

- Um arquivo por use case em `application/useCases/`, nome kebab-case, classe `XxxUseCase` com `@Injectable()`.
- Injeta apenas **interfaces (ports)** de repositório/serviço no construtor — nunca implementações concretas.
- Retorno via monad **`Either<Error, Success>`** (`src/core/Either.ts`, classes `Left`/`Right` com `isLeft()/isRight()`), evitando lançar exceções para erros esperados de negócio:

```ts
type XxxUseCaseResponse = Either<ResourceNotFoundError | NotAllowedError, { question: Question }>

async execute(...): Promise<XxxUseCaseResponse> {
  if (!question) return left(new ResourceNotFoundError())
  return right({ question })
}
```

- Erros específicos de use case em `useCases/errors/` implementando `UseCaseError` (`{ message: string }`); erros genéricos reutilizáveis (`ResourceNotFoundError`, `NotAllowedError`) em `src/core/errors/errors/`.
- Interfaces de repositório são **abstract classes** (não `interface` do TS), para poderem ser usadas como token de injeção de dependência do Nest:

```ts
export abstract class QuestionsRepository {
  abstract create(question: Question): Promise<Question>
  abstract save(question: Question): Promise<void>
  abstract findBySlug(slug: string): Promise<Question | null>
  abstract findManyRecents(params: PaginationParams): Promise<Question[]>
}
```

- Ports adicionais nesse padrão: `cryptography/{hash-generator,hash-comparator,encrypter}.ts`, `storage/uploader.ts`.

## 5. Camada de infraestrutura — Prisma

- **`PrismaService`**: estende `PrismaClient`, implementa `OnModuleInit`/`OnModuleDestroy` (`$connect`/`$disconnect`).
- **Repositórios Prisma** (`infra/database/prisma/repositories/`): implementam os ports do domínio, injetam `PrismaService`, convertem entrada/saída via mapper.
- **Mappers** (`infra/database/prisma/mappers/`): classe estática por entidade com `toDomain(raw): Entity` e `toPrisma(entity): Prisma.CreateInput`, convertendo `string` ↔ `UniqueEntityId` e primitivos ↔ value objects.
- Coleções (ex.: anexos) usam o diff do `WatchedList` (`getNewItems()`/`getRemovedItems()`) dentro do repositório para gerar `createMany`/`deleteMany` corretos.
- **`DatabaseModule`**: registra `PrismaService` + cada repositório via `{ provide: AbstractRepository, useClass: PrismaRepository }`, exportando os tokens abstratos para outros módulos consumirem sem conhecer Prisma.

## 6. Camada HTTP

- Controllers em `infra/http/controllers/`, um por use case, nomeados pelo verbo/rota (ex.: `create-question.controller.ts`).
- **Validação com Zod** (sem `class-validator`/`class-transformer`): schema `z.object({...})` no escopo do módulo, aplicado via `ZodValidationPipe` custom (`infra/http/pipes/zod-validation-pipe.ts`), erros formatados com `zod-validation-error`.
- Controller injeta a **classe concreta** do use case (não uma interface — use cases são providers diretos), chama `.execute()`, verifica `result.isLeft()` e lança `HttpException` apropriada.
- **Presenters** (`infra/http/presenters/`): `toHTTP(entity)` estático, converte entidade de domínio em JSON de resposta.
- **Autenticação**: JWT RS256 via Passport (`infra/auth/`):
  - `JwtStrategy` valida payload com schema Zod, chave pública lida do env (base64).
  - `JwtAuthGuard` registrado globalmente (`APP_GUARD`) — toda rota exige autenticação por padrão, exceto as marcadas com decorator `@Public()`.
  - `@CurrentUser()` decorator extrai o payload do usuário autenticado (`{ sub: string }`).

## 7. Organização de módulos NestJS

- `AppModule`: composition root — importa `ConfigModule.forRoot({ validate })`, `AuthModule`, `HttpModule`, `EnvModule`. Não declara controllers/providers próprios.
- `HttpModule`: importa `DatabaseModule`, `CryptographyModule`, `StorageModule`; declara todos os controllers e todos os use cases como providers.
- `DatabaseModule`, `CryptographyModule`, `StorageModule`: seguem o padrão `{ provide: AbstractPort, useClass: ConcreteAdapter }`, exportando os tokens abstratos.
- `EnvModule`/`EnvService`: wrapper tipado sobre `ConfigService` do `@nestjs/config`.

## 8. Testes

- **Unitários** (`*.spec.ts` ao lado do use case): usam **repositórios in-memory** (`test/repositories/`) que implementam os mesmos ports do domínio, guardando entidades em array e replicando paginação/filtros/diff de anexos.
- **Factories** (`test/factories/make-*.ts`): função `makeX(override?, id?)` com dados padrão via `@faker-js/faker`; versão `@Injectable() XFactory` com `makePrismaX()` persiste no banco real via `PrismaService` — usada em testes e2e.
- **Fakes de ports**: `test/cryptography/fake-hasher.ts`, `fake-encrypter.ts`, `test/storage/faker-uploader.ts`.
- **E2E** (`*.controller.e2e-spec.ts` ao lado do controller): `Test.createTestingModule({ imports: [AppModule, DatabaseModule] })` + `supertest`. `test/setup-e2e.ts` cria um schema Postgres único por execução (`randomUUID()`), roda `prisma migrate deploy` e derruba o schema no `afterAll`.
- Dois configs do Vitest: `vitest.config.ts` (unitário) e `vitest.config.e2e.ts` (e2e, com `setupFiles`).

## 9. Principais dependências

| Categoria | Bibliotecas |
|---|---|
| Framework | `@nestjs/common`, `@nestjs/core`, `@nestjs/platform-express`, `@nestjs/config` |
| Auth | `@nestjs/jwt`, `@nestjs/passport`, `passport-jwt` (RS256) |
| Persistência | `@prisma/client`, `prisma` (Postgres) |
| Validação | `zod`, `zod-validation-error` |
| Hash | `bcryptjs` |
| Storage | `@aws-sdk/client-s3`, `multer` (Cloudflare R2, API compatível com S3) |
| Testes | `vitest`, `@nestjs/testing`, `supertest`, `@faker-js/faker` |

## 10. Configuração de ambiente

- Schema Zod em `infra/env/env.ts` valida `DATABASE_URL`, `JWT_PRIVATE_KEY`/`JWT_PUBLIC_KEY` (base64, decodificadas com `Buffer.from(key, 'base64')`), `PORT` e variáveis do R2/S3.
- Validação plugada no `ConfigModule.forRoot({ validate: (env) => envSchema.parse(env), isGlobal: true })` — a aplicação falha ao subir se o `.env` estiver inválido.
- `EnvService` expõe acesso tipado (`.get('CHAVE')`) em vez de `process.env` direto.

---

## 11. Multi-tenancy (fundacional desde o dia zero)

O CONAA é **multi-tenant desde o início** — não um retrofit de fase posterior. O tenant é o
**`Group`** (grupo): uma organização que pode ser uma secretaria, um município, uma rede ou uma
escola isolada (o campo `type` é só um rótulo informativo, não muda o comportamento). Um `Group`
tem uma ou mais `School` (escolas). Bounded context `tenancy` — ver
[`F1-E0A`](docs/features/F1-E0A-tenancy/spec-tenancy.md) para o detalhe de implementação.

### Duas dimensões, dois papéis diferentes

- **`groupId` — isolamento (boundary de segurança).** Todo model de negócio carrega `groupId`.
  Nenhum usuário de um `Group` pode ler ou escrever dado de outro `Group`, sem exceção.
- **`schoolId` — escopo de permissão (visibilidade dentro do tenant).** Nas tabelas operacionais
  (`Student`, `Teacher`, `Turma`, `Invoice`, etc.), `schoolId` decide **quais escolas do próprio
  Group** o usuário autenticado enxerga. Um papel *group-wide* (ex.: mantenedora/admin do Group)
  vê todas as escolas; um papel *school-scoped* (ex.: diretor de uma unidade) só vê a(s) escola(s)
  atribuída(s) a ele. Essa distinção é decidida por `UserRole.schoolId` (`null` = group-wide;
  preenchido = restrito), modelado em `F1-E09`.

### `GroupContext` (`AsyncLocalStorage`)

Contexto de request populado a partir do usuário autenticado, nunca a partir do corpo/query da
requisição:

```ts
type GroupContext = {
  groupId: string
  allowedSchoolIds: string[] | null // null = acesso a todas as escolas do group
}
```

- `GroupScopeGuard` (`infra/auth/`) roda **depois** do `JwtAuthGuard`: lê `groupId` do payload do
  JWT, busca os `UserRole` do usuário para montar `allowedSchoolIds`, e popula o ALS para a duração
  da requisição.
- Rotas de onboarding (`RegisterGroupUseCase`, `RegisterSchoolUseCase`, criação do primeiro usuário
  admin) rodam **fora** deste contexto (equivalentes a `@Public()` ou um guard próprio de signup).

### Enforcement via Prisma Client Extension

Uma única `$extends` (`infra/database/prisma/extensions/tenant-scope.extension.ts`) intercepta
toda query dos models de negócio:

- **Leitura:** injeta `where: { groupId }`; se o model tem `schoolId` e `allowedSchoolIds != null`,
  injeta também `where: { schoolId: { in: allowedSchoolIds } }`.
- **Escrita (`create`/`createMany`):** injeta `data: { groupId }` automaticamente. `schoolId` **não**
  é auto-injetado em escrita — cada use case recebe/valida explicitamente qual escola está sendo
  usada (deve pertencer ao `groupId` corrente e, se `allowedSchoolIds != null`, estar contida nele).

Isso garante que nenhum repositório "esqueça" de filtrar — o isolamento e o escopo são uma
propriedade da camada de infra, não uma disciplina que cada repositório precisa lembrar de aplicar.

### Convenções derivadas

- **Prisma schema:** todo model de negócio tem `groupId String` (+ índice); models operacionais
  também têm `schoolId String` (+ índice). `Group` e `School` (em `tenancy`) não têm essas colunas
  (são elas próprias a raiz da hierarquia).
- **Domínio (`enterprise`):** entidades recebem `groupId` (e `schoolId`, quando aplicável) como
  props normais, setadas a partir do `GroupContext` no `execute()` do use case — nunca hardcoded,
  nunca aceitas cruas de um DTO de entrada não confiável.
- **Unicidade:** índices únicos que hoje seriam globais (CPF, `RoleName`) passam a ser compostos
  com `groupId` (único **por group**, não globalmente). Índices que fazem sentido por escola (ex.:
  `SchoolYear.year`) são compostos com `schoolId`.
- **Testes:** repositórios in-memory replicam o filtro de `groupId`/`allowedSchoolIds` manualmente
  (sem a Prisma extension) para os unit specs continuarem provando isolamento/escopo sem banco.
- **JWT:** payload passa a incluir `groupId` (e a lista de `UserRole` é resolvida via banco no
  `GroupScopeGuard`, não embutida no token, para revogação de acesso ser imediata).

## O que aproveitar no `conaa-controle-escolar`

- Separar `domain` (regras de negócio puras) de `infra` (Nest/Prisma/HTTP) desde o início evita acoplamento e facilita testes unitários rápidos (sem banco).
- Usar `Either` para erros de negócio esperados (aluno já matriculado, turma lotada, etc.) e reservar exceções para erros realmente excepcionais.
- Repositórios como abstract class = um único padrão de DI, fácil trocar implementação (ex.: in-memory nos testes, Prisma em produção).
- Zod para validação de entrada HTTP e de env evita duplicar regras de validação com DTOs de classe.
- Eventos de domínio + subscribers são úteis se houver múltiplos módulos que precisam reagir a ações (ex.: matrícula concluída → gerar boleto, enviar notificação) sem acoplar os módulos diretamente.
