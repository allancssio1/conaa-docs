# F1-E00 — Bootstrap do monorepo

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E00 — Bootstrap do monorepo |
| Fase | F1 (pré-requisito de todas as demais fichas) |
| Prioridade | Must |
| Estimativa | M |
| Bounded context(s) | — (infraestrutura de projeto, não domínio de negócio) |
| Depende de | — |
| Rastreabilidade | Não existe no BACKLOG — é um *enabler* técnico necessário antes de qualquer épico de produto poder ser implementado. |

## Objetivo

Criar o esqueleto do monorepo (`apps/api` + `apps/web`) com toda a fundação técnica descrita em [arquitetura-ignite.md](../../arquitetura-ignite.md) (backend) e [arquitetura-frontend.md](../../arquitetura-frontend.md) (frontend), **sem nenhuma regra de negócio ainda**. Ao final desta ficha, `F1-E01` (e as demais) só precisam adicionar código de domínio sobre uma base já funcional.

## Escopo

### Monorepo

- [ ] Ferramenta de monorepo (ex.: pnpm workspaces) configurada na raiz: `pnpm-workspace.yaml`, `package.json` raiz com scripts agregados (`dev`, `build`, `test`, `lint`).
- [ ] `apps/api` e `apps/web` como pacotes independentes do workspace.
- [ ] Lint/format compartilhado na raiz (ESLint + Prettier), configs específicas herdadas por cada app quando necessário.
- [ ] `.gitignore` cobrindo `node_modules`, `.next`, `dist`, `.env`.

### `apps/api` (NestJS + Prisma)

Estrutura conforme arquitetura-ignite.md §2:

- [ ] Projeto NestJS inicializado (`@nestjs/core`, `@nestjs/common`, `@nestjs/platform-express`, `@nestjs/config`).
- [ ] `src/core/`:
  - [ ] `entities/entity.ts` — `Entity<Props>`
  - [ ] `entities/aggregate-root.ts` — `AggregateRoot<Props>`
  - [ ] `entities/unique-entity-id.ts` — `UniqueEntityId`
  - [ ] `entities/value-object.ts` — `ValueObject<Props>`
  - [ ] `entities/watched-list.ts` — `WatchedList<T>`
  - [ ] `either.ts` — `Either<L, R>`, `left()`, `right()`, `isLeft()`, `isRight()`
  - [ ] `events/domain-events.ts` — barramento estático de eventos de domínio
  - [ ] `errors/errors/resource-not-found-error.ts`, `not-allowed-error.ts` (implementam `UseCaseError`)
- [ ] `src/infra/env/env.ts` — schema Zod (`DATABASE_URL`, `JWT_PRIVATE_KEY`, `JWT_PUBLIC_KEY`, `PORT`), `env.module.ts`, `env.service.ts` (wrapper tipado sobre `ConfigService`).
- [ ] `src/infra/database/prisma/prisma.service.ts` — estende `PrismaClient`, `OnModuleInit`/`OnModuleDestroy`.
- [ ] `src/infra/database/database.module.ts` — registra `PrismaService` (sem repositórios ainda — cada ficha de domínio adiciona os seus).
- [ ] `src/infra/auth/`:
  - [ ] `jwt.strategy.ts` — valida payload com Zod, chave pública via env (base64).
  - [ ] `jwt-auth.guard.ts` — registrado globalmente via `APP_GUARD`.
  - [ ] `public.decorator.ts` — `@Public()`.
  - [ ] `current-user.decorator.ts` — `@CurrentUser()`.
  - [ ] `auth.module.ts`.
- [ ] `src/infra/http/pipes/zod-validation-pipe.ts`.
- [ ] `src/infra/http/http.module.ts` — importa `DatabaseModule`; sem controllers ainda.
- [ ] `src/app.module.ts` — composition root: `ConfigModule.forRoot({ validate, isGlobal: true })`, `AuthModule`, `HttpModule`, `EnvModule`.
- [ ] `prisma/schema.prisma` — apenas datasource/generator configurados, sem models de negócio.
- [ ] `test/setup-e2e.ts` — cria schema Postgres isolado por execução (`randomUUID()`), roda `prisma migrate deploy`, derruba schema no `afterAll`.
- [ ] `vitest.config.ts` (unitário) e `vitest.config.e2e.ts` (e2e, com `setupFiles`).
- [ ] `.env.example` com todas as variáveis exigidas pelo schema de env.

### `apps/web` (Next.js 16.3)

Estrutura conforme [arquitetura-frontend.md](../../arquitetura-frontend.md) §2:

- [ ] Projeto Next.js 16.3 inicializado (App Router).
- [ ] Pastas base: `app/(public)/`, `app/(portal)/`, `features/`, `shared/ui/`, `shared/api/client.ts`, `shared/auth/`, `shared/lib/`.
- [ ] `shared/api/client.ts` — client HTTP tipado apontando para `apps/api` (usa variável de ambiente para a base URL).
- [ ] `middleware.ts` — placeholder de proteção de rota (sem lógica de perfil ainda, só estrutura).
- [ ] Página pública mínima (`app/(public)/login/page.tsx`) e página protegida mínima (`app/(portal)/page.tsx`) para provar que o roteamento e o middleware funcionam.
- [ ] `.env.example` com a URL da API.

### Tooling raiz

- [ ] `README.md` do projeto (raiz) com instruções de setup local (`pnpm install`, subir Postgres, `pnpm --filter api prisma migrate dev`, `pnpm dev`).
- [ ] Docker Compose (ou instrução equivalente) para subir Postgres local de desenvolvimento.
- [ ] Atualizar a seção "Comandos" de [`CLAUDE.md`](../../../CLAUDE.md) com os comandos reais (dev, test unit, test e2e, migrate, build) assim que definidos.

## Arquivos a criar/editar (checklist)

```
pnpm-workspace.yaml
package.json                              (raiz)
apps/api/package.json
apps/api/src/core/**
apps/api/src/infra/env/**
apps/api/src/infra/database/prisma/prisma.service.ts
apps/api/src/infra/database/database.module.ts
apps/api/src/infra/auth/**
apps/api/src/infra/http/pipes/zod-validation-pipe.ts
apps/api/src/infra/http/http.module.ts
apps/api/src/app.module.ts
apps/api/prisma/schema.prisma
apps/api/test/setup-e2e.ts
apps/api/vitest.config.ts
apps/api/vitest.config.e2e.ts
apps/api/.env.example
apps/web/package.json
apps/web/app/(public)/login/page.tsx
apps/web/app/(portal)/page.tsx
apps/web/shared/api/client.ts
apps/web/middleware.ts
apps/web/.env.example
README.md                                  (raiz)
docker-compose.yml                         (Postgres local)
```

## Testes

- [ ] `apps/api`: um spec trivial (ex.: `either.spec.ts`) só para validar que o harness Vitest unitário roda.
- [ ] `apps/api`: um e2e trivial (ex.: health check endpoint público) para validar que `setup-e2e.ts` sobe o schema isolado corretamente.
- [ ] `apps/web`: build (`next build`) sem erros.

## Definition of Done

- [ ] `pnpm install` na raiz resolve as duas apps.
- [ ] `apps/api` sobe localmente (`pnpm --filter api dev`) e conecta ao Postgres.
- [ ] `apps/web` sobe localmente (`pnpm --filter web dev`) e a página protegida redireciona para login sem sessão.
- [ ] `prisma migrate dev` aplica a migração inicial (vazia) sem erro.
- [ ] Suítes de teste (unit e e2e) do `apps/api` rodam e passam (mesmo que triviais).
- [ ] `CLAUDE.md` atualizado com os comandos reais.
- [ ] Nenhum código de domínio de negócio incluído nesta ficha — isso é responsabilidade de `F1-E01` em diante.
