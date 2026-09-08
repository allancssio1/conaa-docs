<!--
  Template de ficha de tarefa por épico.
  Copie este arquivo para .claude/docs/features/<ID>-<slug>.md (ex.: F1-E02-matricula-turmas.md)
  e preencha cada seção. O objetivo desta ficha é ser AUTOCONTIDA: um agente deve
  conseguir implementar o épico lendo só CLAUDE.md + a(s) arquitetura(s) relevante(s)
  + esta ficha — sem precisar abrir BACKLOG.md/ROADMAP.md.
-->

# `<ID>` — `<Título do épico>`

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | `<ID>` — `<Título>` |
| Fase | `<F1 / F2 / F3>` |
| Prioridade | `<Must / Should / Could>` |
| Estimativa | `<P / M / G / GG>` |
| Bounded context(s) | `<ex.: academic, attendance>` |
| Depende de | `<IDs de outras fichas que precisam estar prontas antes>` |
| Rastreabilidade | [`BACKLOG.md`](../../BACKLOG.md) — buscar por `### <ID>` para histórias/critérios completos (referência, não leitura obrigatória) |

## Objetivo

`<1-2 frases: o que este épico entrega e por quê, em linguagem de produto>`

## Histórias cobertas

Lista resumida das histórias do épico (a versão completa com critérios de aceite está no BACKLOG, linkado acima). Resuma aqui só o suficiente para orientar a implementação:

- `<ID-U01>` — `<resumo de 1 linha>`
- `<ID-U02>` — `<resumo de 1 linha>`

## Modelo de domínio (`enterprise`)

`<Para cada entidade/aggregate/VO novo ou alterado:>`

### `<NomeDaEntidade>` — `apps/api/src/domain/<contexto>/enterprise/entities/<nome-kebab>.ts`

- Tipo: `<Entity | AggregateRoot>`
- Props principais: `<lista de campos e tipos>`
- Invariantes/regras: `<o que a entidade garante sobre si mesma>`
- Eventos de domínio disparados (se houver): `<NomeDoEvento>`

## Use cases (`application`)

`<Para cada use case:>`

### `<NomeUseCase>` — `apps/api/src/domain/<contexto>/application/useCases/<nome-kebab>.ts`

- Entrada: `<campos>`
- Saída (`Either`): `Either<<ErrosPossíveis>, { <sucesso> }>`
- Erros específicos: `<NomeDoErro>` em `useCases/errors/<nome-kebab>.ts`
- Ports usados: `<ex.: StudentsRepository>`
- Regras de negócio: `<passo a passo do que o use case valida/faz>`

## Persistência (`infra/database/prisma`)

- Alterações em `apps/api/prisma/schema.prisma`: `<novos models/campos/relations>`
- Repositórios: `apps/api/src/infra/database/prisma/repositories/<nome-kebab>.ts` (implementa o port de `application/repositories/`)
- Mappers: `apps/api/src/infra/database/prisma/mappers/<nome-kebab>.ts`

## HTTP (`infra/http`)

`<Para cada endpoint:>`

### `<MÉTODO> <rota>` — `apps/api/src/infra/http/controllers/<nome-kebab>.controller.ts`

- Schema Zod de entrada: `<campos e regras>`
- Presenter de saída: `apps/api/src/infra/http/presenters/<nome-kebab>.presenter.ts`
- Autenticação/perfis exigidos: `<ex.: secretaria, coordenação>`

## Frontend (`apps/web`)

- Rota(s): `apps/web/app/(portal)/<caminho>/page.tsx`
- Componentes: `apps/web/features/<contexto>/components/<Nome>.tsx`
- Chamadas à API: `apps/web/features/<contexto>/api/<nome>.ts`
- Schema de formulário: `apps/web/features/<contexto>/schemas/<nome>.ts`
- Estados de UI relevantes: `<loading, vazio, erro de negócio específico, etc.>`

## Eventos/subscribers (se houver)

- `<Evento>` disparado por `<use case>`, escutado por `<subscriber>` em `<caminho>` — efeito: `<o que acontece>`

## Arquivos a criar/editar (checklist)

- [ ] `apps/api/src/domain/<contexto>/enterprise/entities/...`
- [ ] `apps/api/src/domain/<contexto>/application/useCases/...`
- [ ] `apps/api/src/domain/<contexto>/application/repositories/...`
- [ ] `apps/api/src/infra/database/prisma/repositories/...`
- [ ] `apps/api/src/infra/database/prisma/mappers/...`
- [ ] `apps/api/prisma/schema.prisma`
- [ ] `apps/api/src/infra/http/controllers/...`
- [ ] `apps/api/src/infra/http/presenters/...`
- [ ] `apps/web/app/(portal)/...`
- [ ] `apps/web/features/<contexto>/...`

## Testes

- Repositório in-memory: `apps/api/test/repositories/in-memory-<nome>.ts`
- Factory: `apps/api/test/factories/make-<nome>.ts`
- Unit spec de use case: `<use-case>.spec.ts` (ao lado do use case)
- E2E do controller: `<controller>.e2e-spec.ts` (ao lado do controller)
- Frontend: teste de componente/hook relevante (se a tela tiver lógica não trivial)

## Definition of Done

- [ ] Todos os critérios de aceite das histórias no BACKLOG estão satisfeitos.
- [ ] Testes unitários e e2e listados acima passando.
- [ ] `pnpm build` (ou equivalente) sem erros em `apps/api` e `apps/web`.
- [ ] Checklist de "Arquivos a criar/editar" concluído.
