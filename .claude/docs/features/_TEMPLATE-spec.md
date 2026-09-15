<!--
  Template de SPEC (nível de implementação) para um épico já com epic.md pronto.
  Use este template no próximo passo do fluxo: depois que o épico tem sua pasta
  (.claude/docs/features/<ID>-<slug>/epic.md), copie este arquivo para dentro
  dela, ex.: .claude/docs/features/F1-E02-matricula-rematricula/spec-matricula.md
  (uma spec por fatia implementável do épico, se ele for grande demais para 1 spec).

  Como o arquivo passa a viver 1 nível mais profundo que este template (que está
  em .claude/docs/features/), ajuste os links relativos em +1 nível ao copiar
  (ex.: ../../BACKLOG.md -> ../../../BACKLOG.md), seguindo o mesmo padrão usado em
  F1-E00-bootstrap/epic.md e F1-E01-cadastros-sis/epic.md.

  O objetivo desta spec é ser AUTOCONTIDA: um agente deve conseguir implementar
  lendo só CLAUDE.md + a(s) arquitetura(s) relevante(s) + esta spec — sem precisar
  abrir BACKLOG.md/ROADMAP.md.
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

### `<NomeDaEntidade>` — `conaa-api/src/domain/<contexto>/enterprise/entities/<nome-kebab>.ts`

- Tipo: `<Entity | AggregateRoot>`
- Props principais: `<lista de campos e tipos>`
- Invariantes/regras: `<o que a entidade garante sobre si mesma>`
- Eventos de domínio disparados (se houver): `<NomeDoEvento>`

## Use cases (`application`)

`<Para cada use case:>`

### `<NomeUseCase>` — `conaa-api/src/domain/<contexto>/application/useCases/<nome-kebab>.ts`

- Entrada: `<campos>`
- Saída (`Either`): `Either<<ErrosPossíveis>, { <sucesso> }>`
- Erros específicos: `<NomeDoErro>` em `useCases/errors/<nome-kebab>.ts`
- Ports usados: `<ex.: StudentsRepository>`
- Regras de negócio: `<passo a passo do que o use case valida/faz>`

## Persistência (`infra/database/prisma`)

- Alterações em `conaa-api/prisma/schema.prisma`: `<novos models/campos/relations>`
- Repositórios: `conaa-api/src/infra/database/prisma/repositories/<nome-kebab>.ts` (implementa o port de `application/repositories/`)
- Mappers: `conaa-api/src/infra/database/prisma/mappers/<nome-kebab>.ts`

## HTTP (`infra/http`)

`<Para cada endpoint:>`

### `<MÉTODO> <rota>` — `conaa-api/src/infra/http/controllers/<nome-kebab>.controller.ts`

- Schema Zod de entrada: `<campos e regras>`
- Presenter de saída: `conaa-api/src/infra/http/presenters/<nome-kebab>.presenter.ts`
- Autenticação/perfis exigidos: `<ex.: secretaria, coordenação>`

## Frontend (`conaa-web`)

- Rota(s): `conaa-web/app/(portal)/<caminho>/page.tsx`
- Componentes: `conaa-web/features/<contexto>/components/<Nome>.tsx`
- Chamadas à API: `conaa-web/features/<contexto>/api/<nome>.ts`
- Schema de formulário: `conaa-web/features/<contexto>/schemas/<nome>.ts`
- Estados de UI relevantes: `<loading, vazio, erro de negócio específico, etc.>`

## Eventos/subscribers (se houver)

- `<Evento>` disparado por `<use case>`, escutado por `<subscriber>` em `<caminho>` — efeito: `<o que acontece>`

## Arquivos a criar/editar (checklist)

- [ ] `conaa-api/src/domain/<contexto>/enterprise/entities/...`
- [ ] `conaa-api/src/domain/<contexto>/application/useCases/...`
- [ ] `conaa-api/src/domain/<contexto>/application/repositories/...`
- [ ] `conaa-api/src/infra/database/prisma/repositories/...`
- [ ] `conaa-api/src/infra/database/prisma/mappers/...`
- [ ] `conaa-api/prisma/schema.prisma`
- [ ] `conaa-api/src/infra/http/controllers/...`
- [ ] `conaa-api/src/infra/http/presenters/...`
- [ ] `conaa-web/app/(portal)/...`
- [ ] `conaa-web/features/<contexto>/...`

## Testes

- Repositório in-memory: `conaa-api/test/repositories/in-memory-<nome>.ts`
- Factory: `conaa-api/test/factories/make-<nome>.ts`
- Unit spec de use case: `<use-case>.spec.ts` (ao lado do use case)
- E2E do controller: `<controller>.e2e-spec.ts` (ao lado do controller)
- Frontend: teste de componente/hook relevante (se a tela tiver lógica não trivial)

## Definition of Done

- [ ] Todos os critérios de aceite das histórias no BACKLOG estão satisfeitos.
- [ ] Testes unitários e e2e listados acima passando.
- [ ] `pnpm build` (ou equivalente) sem erros em `conaa-api` e `conaa-web`.
- [ ] Checklist de "Arquivos a criar/editar" concluído.
