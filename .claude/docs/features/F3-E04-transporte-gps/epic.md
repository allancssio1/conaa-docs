# F3-E04 — Transporte com GPS e segurança avançada

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F3-E04 — Transporte com GPS e segurança avançada |
| Fase | F3 (Analytics, IA & Diferenciação) |
| Prioridade | Could |
| Estimativa | G |
| Bounded context(s) | `transport` (extensão) |
| Depende de | F2-E05 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F3-E04` |

## Objetivo

Rastreamento em tempo real do veículo escolar em mapa, com notificação de embarque e desembarque do aluno.

## Histórias cobertas

- `F3-E04-U01` — Posição em tempo real do veículo + notificação de embarque/desembarque.

## Escopo / fora de escopo

- **Dentro:** rastreamento GPS, exibição em mapa, notificação de embarque/desembarque.
- **Fora:** rotas otimizadas automaticamente — mencionado no documento-fonte como direção futura, não é critério de aceite.

## Dependências e integrações

Depende de F2-E05 (rota/veículo já cadastrados). Precisa de um componente que emita a posição (app do motorista, dispositivo GPS dedicado, ou integração com rastreador de terceiros) — não coberto por nenhum épico anterior.

## Decisões em aberto / riscos

- Como a posição chega ao sistema é uma decisão de arquitetura/hardware a tomar antes de especificar (app do motorista vs. dispositivo dedicado vs. integração com rastreador de terceiros).

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
