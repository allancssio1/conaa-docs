# Arquitetura de referência: frontend (Next.js 16.3)

> **Proposta revisável.** Diferente de [arquitetura-ignite.md](arquitetura-ignite.md) (que documenta um projeto de referência existente), este documento é uma convenção proposta para o `apps/web` do CONAA, pensada para espelhar o mesmo nível de organização do backend. Ajuste livremente antes de considerá-la definitiva.
>
> **Nota de versão:** a API exata do Next.js 16.3 (nomes de APIs, comportamento padrão de cache, etc.) deve ser validada contra a documentação oficial no momento da implementação de cada ficha — não assuma este documento como fonte de verdade sobre a API do framework, só sobre a organização do projeto.

## 1. Visão geral

`apps/web` consome a API do `apps/api` via um client HTTP tipado. Organização por **feature** (não por tipo de arquivo), espelhando os bounded contexts do backend sempre que fizer sentido para a navegação do usuário (ex.: contexto `academic` → área "Turmas" no app).

```
app        →  camada de rotas (App Router) — fina, delega para features/
features   →  lógica de UI por domínio (componentes, hooks, schemas, api-client)
shared     →  design system, utilitários genéricos, client HTTP base
```

Regra de dependência: `app` depende de `features`; `features` pode depender de `shared`; `shared` não depende de nada de `app`/`features`.

## 2. Estrutura de pastas

```
apps/web/
  app/
    (public)/                 rotas sem autenticação (login, esqueci senha)
    (portal)/
      alunos/
      turmas/
      financeiro/
      ...                     uma pasta de rota por área, componente de página fino
    layout.tsx
    globals.css
  features/
    <bounded-context>/         ex.: academic, attendance, assessment, finance
      components/               componentes de UI específicos do contexto
      hooks/                     hooks de dados (data-fetching) do contexto
      api/                       funções tipadas que chamam a API (usam o client de shared/api)
      schemas/                   validação Zod dos formulários/DTOs do contexto
      types.ts
  shared/
    ui/                          design system (botão, input, tabela, dialog...)
    api/
      client.ts                  client HTTP base (fetch tipado, injeta auth token)
    auth/                        sessão, guards de rota, hook useSession
    lib/                         utilitários genéricos (formatação de data, etc.)
  middleware.ts                  proteção de rotas por sessão/perfil
  next.config.ts
```

Cada `features/<contexto>` é autocontido e opcionalmente mapeia 1:1 para um bounded context do backend (`apps/api/src/domain/<contexto>`), facilitando encontrar "onde mora a tela que fala com este use case".

## 3. Camada de rotas (`app/`)

- App Router, Server Components por padrão; `"use client"` só onde há interatividade/estado local.
- Componentes de página são **finos**: buscam dados (via `features/<contexto>/api` ou hooks) e compõem componentes de `features/<contexto>/components`. Nada de lógica de negócio dentro de `app/`.
- Agrupamento de rotas por área de acesso: `(public)` (sem sessão) e `(portal)` (autenticado, com `middleware.ts` validando sessão e perfil).
- Rotas dinâmicas (`[id]`) e paralelas/interceptadas apenas quando o caso de uso exigir (ex.: modal de detalhe de aluno sobre a lista).

## 4. Camada de dados (`features/<contexto>/api` + `shared/api`)

- `shared/api/client.ts`: client HTTP único (`fetch` tipado), injeta token de autenticação, trata erros de forma consistente (mapeia erro HTTP → tipo de erro de domínio conhecido pelo frontend).
- `features/<contexto>/api/`: uma função por operação (`getStudent`, `createStudent`, `listTurmas`...), tipada com os DTOs de resposta da API. Não expõe `fetch` cru para os componentes.
- Data-fetching em Server Components sempre que possível (menos JS no client); Client Components usam hooks de `features/<contexto>/hooks/` para mutações e dados que dependem de interação.
- Sem duplicar validação: os schemas Zod usados nos formulários (`features/<contexto>/schemas/`) devem espelhar os schemas de entrada do backend descritos na ficha da tarefa correspondente.

## 5. Formulários e validação

- React Hook Form + Zod resolver, usando os schemas de `features/<contexto>/schemas/`.
- Mensagens de erro client-side e as regras de negócio retornadas pela API (`Either` mapeado para erro HTTP) convergem para o mesmo componente de exibição de erro do design system (`shared/ui`).

## 6. Autenticação e autorização

- Sessão via cookie httpOnly (token emitido pelo backend `infra/auth`, JWT RS256 — ver arquitetura-ignite.md §6).
- `middleware.ts` bloqueia acesso a `(portal)/*` sem sessão válida e redireciona para login.
- Controle de visibilidade por perfil (secretaria, coordenação, professor, responsável, aluno) feito tanto no `middleware.ts` (rotas inteiras) quanto em componentes (`shared/auth` expõe hook/guard para esconder ações específicas).

## 7. Design system (`shared/ui`)

- Componentes de UI puros, sem chamada de API nem lógica de negócio — recebem dados via props.
- Base para os componentes específicos de `features/<contexto>/components`, que combinam componentes de `shared/ui` com dados/hooks do contexto.

## 8. Testes

- Componentes/hooks: Vitest + Testing Library, mesma stack de teste do backend por consistência de tooling no monorepo.
- Mocks de API: funções de `features/<contexto>/api` são mockadas nos testes de componente — nunca mocka-se `fetch` diretamente.
- E2E de fluxo crítico (login, matrícula, lançamento de nota): Playwright, rodando contra `apps/api` local.

## 9. Principais dependências (proposta)

| Categoria | Bibliotecas |
| --- | --- |
| Framework | `next` (16.3), `react`, `react-dom` |
| Formulários | `react-hook-form`, `@hookform/resolvers`, `zod` |
| Estado servidor/cache | recursos nativos do App Router (`fetch` cache, Server Actions) — evitar lib extra de data-fetching a menos que a necessidade apareça |
| UI | a definir (ex.: Tailwind + Radix/shadcn) — validar com o time antes da ficha de bootstrap |
| Testes | `vitest`, `@testing-library/react`, `playwright` |

## 10. O que aproveitar do padrão do backend

- Organização por domínio (`features/<contexto>` espelhando `domain/<contexto>`) mantém a navegação mental consistente entre as duas apps do monorepo.
- Reuso de Zod entre frontend e backend reduz duplicação de regras de validação (mesmo que os schemas não sejam literalmente compartilhados via pacote comum, a *fonte* de cada regra é a ficha da tarefa).
- Erros de negócio (`Either` no backend) devem ter um mapeamento único e prático no client HTTP, para que toda tela trate erro de negócio de forma consistente em vez de cada componente reinventar tratamento de erro.
