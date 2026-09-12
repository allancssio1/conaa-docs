<!--
  Template de EPIC (nível de planejamento) — leve, de propósito.
  Copie para .claude/docs/features/<ID>-<slug>/epic.md (crie a pasta).
  Esta ficha não é para implementar direto: ela orienta o próximo passo,
  que é quebrar o épico em uma ou mais SPECs (_TEMPLATE-spec.md) dentro
  da mesma pasta. Não repita aqui o detalhe técnico que pertence à spec
  (entidades, use cases, rotas, arquivos) — isso é decidido na hora de
  especificar, com o contexto de implementação mais fresco.

  Como o arquivo passa a viver 1 nível mais profundo que este template
  (que está em .claude/docs/features/), ajuste os links relativos em +1
  nível ao copiar (ex.: ../../BACKLOG.md -> ../../../BACKLOG.md;
  _TEMPLATE-spec.md -> ../_TEMPLATE-spec.md; README.md -> ../README.md),
  seguindo o mesmo padrão usado nos épicos já existentes.
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
| Depende de | `<IDs de outros épicos que precisam estar especificados/implementados antes>` |
| Rastreabilidade | [`BACKLOG.md`](../../BACKLOG.md) — buscar por `### <ID>` para histórias/critérios completos |

## Objetivo

`<1-2 frases: o que este épico entrega e por quê, em linguagem de produto>`

## Histórias cobertas

- `<ID-U01>` — `<resumo de 1 linha>`
- `<ID-U02>` — `<resumo de 1 linha>`

(Lista completa de critérios de aceite está no BACKLOG — aqui é só o suficiente para dar contexto ao quebrar em specs.)

## Escopo / fora de escopo

- **Dentro:** `<o que este épico resolve>`
- **Fora:** `<o que parece relacionado mas pertence a outro épico — evita escopo inchado>`

## Dependências e integrações

`<Com quais outros épicos/contextos este épico conversa, e por quê. Ex.: "depende de F1-E01 porque precisa de Student/Turma já existentes".>`

## Decisões em aberto / riscos

`<O que ainda precisa ser decidido antes ou durante a especificação — escolha de lib, formato de integração externa, regra de negócio ambígua no BACKLOG, etc. Vazio ("nenhuma no momento") é uma resposta válida.>`

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](_TEMPLATE-spec.md). Um épico grande pode gerar mais de uma spec (fatias verticais implementáveis); um épico pequeno pode virar uma spec só. Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](README.md).
