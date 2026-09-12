# F2-E03 — Integração com meios de pagamento

## Cabeçalho

| Campo | Valor |
| --- | --- |
| Épico | F2-E03 — Integração com meios de pagamento |
| Fase | F2 (Eficiência & Comunicação) |
| Prioridade | Must |
| Estimativa | G |
| Bounded context(s) | `finance` (extensão) |
| Depende de | F1-E06 |
| Rastreabilidade | [`BACKLOG.md`](../../../BACKLOG.md) — buscar por `### F2-E03` |

## Objetivo

Permitir pagamento on-line de mensalidade (Pix/cartão/boleto) via gateway, com baixa automática, e lembretes automáticos com link de pagamento.

## Histórias cobertas

- `F2-E03-U01` — Pagamento on-line com baixa automática via gateway.
- `F2-E03-U02` — Lembrete automático de vencimento com link de pagamento.

## Escopo / fora de escopo

- **Dentro:** integração com gateway, baixa automática via webhook, lembrete com link de pagamento.
- **Fora:** emissão de nota fiscal / conciliação contábil avançada (não citada no BACKLOG).

## Dependências e integrações

Depende de F1-E06 (título/baixa já existem — aqui a "baixa manual" ganha um caminho automático via webhook do gateway). Reusa os canais de F2-E02 para o lembrete.

## Decisões em aberto / riscos

- Escolha do gateway (Pix direto via banco, ou PSP como Stripe/Pagar.me/Mercado Pago) é decisão de fornecedor a tomar antes da spec — afeta o modelo de dados de webhook/callback.

## Próximo passo

Quebrar este épico em specs dentro desta mesma pasta, usando [`_TEMPLATE-spec.md`](../_TEMPLATE-spec.md). Depois de criar a(s) spec(s), atualizar o status deste épico no [índice](../README.md).
