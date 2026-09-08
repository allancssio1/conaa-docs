# CLAUDE.md — Router do CONAA Controle Escolar

> Este arquivo é o **ponto de entrada mínimo** para qualquer agente que for implementar código neste projeto. Leia-o primeiro, sempre.

## Regra de ouro (protocolo de leitura)

Para **implementar uma tarefa**, leia **apenas**:

1. Este arquivo (`CLAUDE.md`).
2. A arquitetura relevante:
   - Backend/API → [.claude/arquitetura-ignite.md](.claude/arquitetura-ignite.md)
   - Frontend/web → [.claude/arquitetura-frontend.md](.claude/arquitetura-frontend.md)
3. A ficha da tarefa: `.claude/docs/features/<ID>.md` (ex.: `.claude/docs/features/F1-E01-cadastros-sis.md`)

**NÃO leia `BACKLOG.md`, `ROADMAP.md` ou o documento-fonte de pesquisa para implementar.** Esses arquivos existem para quem está planejando o produto, não para quem está codando — a ficha da tarefa já destila tudo que é necessário (histórias, critérios de aceite, arquivos a tocar, contratos). Se a ficha referenciar algo do backlog, é só um link de rastreabilidade, não leitura obrigatória.

Se a ficha da tarefa não existir ainda para o que você precisa implementar, **pare e peça para criá-la** em vez de improvisar a partir do backlog — assim ela também fica registrada para a próxima vez.

## O que é este projeto

CONAA é um Sistema de Gestão Escolar (SGE) brasileiro. O escopo completo (o "o quê" e o "por quê") está em `.claude/ROADMAP.md`/`.claude/BACKLOG.md`; este arquivo e as fichas cobrem o "como implementar".

## Stack e regra de dependência

- **Backend:** NestJS + Prisma + TypeScript, **Clean Architecture + DDD leve**. Regra de dependência (sempre para dentro):

  ```
  infra  →  domain/application  →  domain/enterprise
                      ↓
                    core
  ```

- **Frontend:** Next.js 16.3.
- Monorepo: `apps/api` (backend), `apps/web` (frontend). Criados pela ficha `.claude/docs/features/F1-E00-bootstrap.md`.

## Mapa de pastas

```
apps/
  api/
    src/
      core/                    building blocks compartilhados (Entity, Either, DomainEvents, erros genéricos)
      domain/
        <bounded-context>/
          application/         use cases + ports (repositories/services como abstract class)
          enterprise/          entidades, value objects, eventos de domínio — zero dependência de framework
      infra/
        database/prisma/       PrismaService, repositórios Prisma, mappers
        http/                  controllers, pipes, presenters
        auth/                  JWT strategy, guards, decorators
        env/                   validação Zod de variáveis de ambiente
    test/
      repositories/            repositórios in-memory (testes unitários)
      factories/                factories de entidades
    prisma/
      schema.prisma
  web/
    ...                        ver .claude/arquitetura-frontend.md
.claude/
  docs/
    features/                  fichas de tarefa (1 por épico) — comece por aqui para implementar
```

## Bounded contexts do CONAA (backend)

Cada contexto vive em `apps/api/src/domain/<contexto>/`. Mapeamento épico → contexto (ver `.claude/docs/features/README.md` para a lista completa e status):

| Contexto | Épicos | Conteúdo |
| --- | --- | --- |
| `people` | F1-E01 | Alunos, responsáveis, professores, funcionários |
| `academic` | F1-E01, F1-E02, F1-E05 | Séries, turmas, disciplinas, salas, calendário, matrícula, horários |
| `attendance` | F1-E03 | Frequência |
| `assessment` | F1-E04 | Notas, boletins |
| `finance` | F1-E06 | Mensalidades, cobranças, inadimplência |
| `reporting` | F1-E07 | Relatórios administrativos e oficiais |
| `iam` | F1-E09 | Perfis de acesso, auditoria, consentimento LGPD |
| `communication` | F2-E02 | Comunicação multicanal e notificações |

Novos contextos são adicionados conforme novas fichas forem criadas nas fases seguintes.

## Cheatsheet de convenções do backend

(Resumo — a fonte de verdade é [.claude/arquitetura-ignite.md](.claude/arquitetura-ignite.md), leia-a antes de implementar.)

- Erros de negócio esperados → `Either<Error, Success>` (`left`/`right`), nunca `throw`.
- Repositórios são `abstract class` em `application/repositories/` (não `interface`) — servem de token de DI.
- 1 arquivo por use case em `application/useCases/`, nome kebab-case, classe `XxxUseCase`.
- Validação de entrada HTTP e de env → Zod (nunca `class-validator`).
- Controllers injetam a classe concreta do use case, chamam `.execute()`, tratam `isLeft()/isRight()`.
- Persistência: mapper estático por entidade (`toDomain`/`toPrisma`) em `infra/database/prisma/mappers/`.
- Testes unitários usam repositórios in-memory (`test/repositories/`); e2e usa Prisma real com schema isolado por execução.

## Comandos

> Ficam vazios até `.claude/docs/features/F1-E00-bootstrap.md` ser executado — depois desta ficha, atualize esta seção com os comandos reais (dev, test unit, test e2e, migrate).

## Índice de referência (produto, não implementação)

- [.claude/ROADMAP.md](.claude/ROADMAP.md) — fases, épicos, sequenciamento
- [.claude/BACKLOG.md](.claude/BACKLOG.md) — histórias de usuário e critérios de aceite por épico
- [.claude/arquitetura-ignite.md](.claude/arquitetura-ignite.md) — arquitetura de referência do backend
- [.claude/arquitetura-frontend.md](.claude/arquitetura-frontend.md) — arquitetura de referência do frontend
- [.claude/docs/features/README.md](.claude/docs/features/README.md) — índice de fichas de tarefa e status
