# F1-E00 — Bootstrap dos repositórios (conaa-api + conaa-web)

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F1-E00 — Bootstrap dos repositórios (conaa-api + conaa-web) |
| Fase | F1 (pré-requisito de todas as demais fichas) |
| Prioridade | Must |
| Estimativa | M |
| Bounded context(s) | — (infraestrutura de projeto, não domínio de negócio) |
| Depende de | — |
| Rastreabilidade | Não existe no BACKLOG — é um *enabler* técnico necessário antes de qualquer épico de produto poder ser implementado. |

## Objetivo

Criar **dois repositórios git independentes** — `conaa-api` (backend) e `conaa-web` (frontend) — com toda a fundação técnica descrita em [arquitetura-ignite.md](../../../arquitetura-ignite.md) (backend) e [arquitetura-frontend.md](../../../arquitetura-frontend.md) (frontend), **sem nenhuma regra de negócio ainda**. Ao final desta ficha, `F1-E01` (e as demais) só precisam adicionar código de domínio sobre uma base já funcional em cada repositório.

Este repositório (`conaa-controle-escolar`) não recebe código nesta ficha — ele continua sendo só o hub de documentação/planejamento.

## Escopo

### Repositórios

- [ ] Criar os 2 repositórios git: `conaa-api` e `conaa-web`, cada um independente (histórico, versionamento e deploy próprios).
- [ ] Em cada repositório: `package.json` próprio com scripts (`dev`, `build`, `test`, `lint`), lint/format próprio (ESLint + Prettier) e `.gitignore` cobrindo `node_modules`, `dist`/`.next`, `.env`.
- [ ] Não há ferramenta de workspace/monorepo (pnpm workspaces, turborepo etc.) — cada repositório é standalone e resolve suas próprias dependências.

### `conaa-api` (NestJS + Prisma)

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
- [ ] `docker-compose.yml` — Postgres local de desenvolvimento (fica neste repositório, junto do que o consome).
- [ ] `README.md` do repositório com instruções de setup local (`pnpm install`, subir Postgres via `docker compose up`, `pnpm prisma migrate dev`, `pnpm dev`).

### `conaa-web` (Next.js 16.3)

Estrutura conforme [arquitetura-frontend.md](../../../arquitetura-frontend.md) §2:

- [ ] Projeto Next.js 16.3 inicializado (App Router).
- [ ] Pastas base: `app/(public)/`, `app/(portal)/`, `features/`, `shared/ui/`, `shared/api/client.ts`, `shared/auth/`, `shared/lib/`.
- [ ] `shared/api/client.ts` — client HTTP tipado apontando para a API (usa variável de ambiente para a base URL do `conaa-api`, já que são repositórios/deploys separados).
- [ ] `middleware.ts` — placeholder de proteção de rota (sem lógica de perfil ainda, só estrutura).
- [ ] Página pública mínima (`app/(public)/login/page.tsx`) e página protegida mínima (`app/(portal)/page.tsx`) para provar que o roteamento e o middleware funcionam.
- [ ] `.env.example` com a URL da API (`conaa-api`, rodando localmente em outra porta/processo).
- [ ] `README.md` do repositório com instruções de setup local (`pnpm install`, `pnpm dev`, variável de ambiente apontando para o `conaa-api` local já rodando).

### Atualizar documentação (neste hub)

- [ ] Atualizar a seção "Comandos" de [`CLAUDE.md`](../../../../CLAUDE.md) com os comandos reais de cada repositório (dev, test unit, test e2e, migrate, build) assim que definidos.

## Arquivos a criar/editar (checklist)

```
conaa-api/package.json
conaa-api/src/core/**
conaa-api/src/infra/env/**
conaa-api/src/infra/database/prisma/prisma.service.ts
conaa-api/src/infra/database/database.module.ts
conaa-api/src/infra/auth/**
conaa-api/src/infra/http/pipes/zod-validation-pipe.ts
conaa-api/src/infra/http/http.module.ts
conaa-api/src/app.module.ts
conaa-api/prisma/schema.prisma
conaa-api/test/setup-e2e.ts
conaa-api/vitest.config.ts
conaa-api/vitest.config.e2e.ts
conaa-api/.env.example
conaa-api/docker-compose.yml
conaa-api/README.md

conaa-web/package.json
conaa-web/app/(public)/login/page.tsx
conaa-web/app/(portal)/page.tsx
conaa-web/shared/api/client.ts
conaa-web/middleware.ts
conaa-web/.env.example
conaa-web/README.md
```

## Testes

- [ ] `conaa-api`: um spec trivial (ex.: `either.spec.ts`) só para validar que o harness Vitest unitário roda.
- [ ] `conaa-api`: um e2e trivial (ex.: health check endpoint público) para validar que `setup-e2e.ts` sobe o schema isolado corretamente.
- [ ] `conaa-web`: build (`next build`) sem erros.

## Definition of Done

- [ ] `pnpm install` funciona de forma independente em cada repositório (`conaa-api` e `conaa-web`).
- [ ] `conaa-api` sobe localmente (`pnpm dev` dentro do repositório) e conecta ao Postgres do `docker-compose.yml`.
- [ ] `conaa-web` sobe localmente (`pnpm dev` dentro do repositório, apontando via env para o `conaa-api` local) e a página protegida redireciona para login sem sessão.
- [ ] `prisma migrate dev` aplica a migração inicial (vazia) sem erro.
- [ ] Suítes de teste (unit e e2e) do `conaa-api` rodam e passam (mesmo que triviais).
- [ ] `CLAUDE.md` (neste hub) atualizado com os comandos reais de cada repositório.
- [ ] Nenhum código de domínio de negócio incluído nesta ficha — isso é responsabilidade de `F1-E01` em diante.
