# Formato das entregas

Use o menor conjunto de entregas que satisfaça a solicitação. Para um pedido
comum de Design, apresente as seções 1 a 8 abaixo. Para criação de artefatos,
escreva arquivos equivalentes e resuma seus caminhos.

## 1. Decisão executiva

Declare:

- quantos Feature Groups foram propostos;
- seus nomes e propósitos em uma linha;
- o grão do dataset final;
- se o resultado está `draft`, `ready_for_review` ou `publishable`;
- os bloqueios ou hipóteses mais importantes.

## 2. Inventário da query

Forneça uma linha para cada expressão da saída final:

| Output | Role | Expression/source | Entity | Grain | Window/time | Evidence status |
|---|---|---|---|---|---|---|

Targets, chaves, timestamps, controles e campos non-feature permanecem visíveis,
mesmo que não sejam publicados como features.

Para solicitações de Design, apresente também a matriz de compatibilidade dos
feature cards definida em [decomposition-method.md](decomposition-method.md),
com entidade, record grain, conceito semântico, janela, event timestamp,
refresh e fontes de cada feature candidata.

## 3. Contratos de grão e comparação

Forneça uma tabela explícita:

| Candidate | Entity | Record keys | Uma linha representa | Compatible with |
|---|---|---|---|---|

Mostre cada grão material produzido pela mesma fonte. Não esconda o grão em
prosa.

## 4. Feature Groups propostos

Para cada grupo, inclua:

- entidade, business key, record grain e lookup key;
- propósito semântico;
- features;
- event e availability timestamps;
- cadência, freshness, requisito offline/online;
- owner/governança, quando conhecidos;
- lineage da fonte;
- readiness status.

## 5. Registro de decisões de fronteira

Explique por que as features foram unidas ou separadas usando esta tabela:

| Decision | Features/groups | Evidence | Rule applied | Confidence |
|---|---|---|---|---|

Inclua alternativas rejeitadas, como um grupo por model, source table, CTE,
entidade ou janela, quando forem plausíveis a partir da entrada.

## 6. Separação de target e dependências

Liste:

- labels/outcomes excluídos dos Feature Groups;
- features base, agregadas, derivadas e request-time;
- dependências entre grupos e transformações específicas do modelo.

## 7. Plano de validação temporal e de qualidade

Especifique verificações executáveis ou testáveis para unicidade, fan-out, nulos,
limites de janela, conhecimento tardio, leakage, equivalência com a query
original, backfill e serving parity. Diferencie claramente testes propostos de
testes realmente executados.

## 8. Decisões em aberto

Priorize somente perguntas que possam alterar o contrato. Para cada uma, declare
a hipótese provisória e a consequência de uma resposta diferente.

## 9. Artefatos para solicitações de Create

A menos que o usuário especifique outro layout, crie:

```text
feature-groups/
├── design.md
├── contracts/
│   └── <feature-group-name>.yaml
├── transformations/
│   └── <feature-group-name>.sql
└── tests/
    └── <feature-group-name>-checks.md
```

A transformação SQL pode permanecer como um skeleton claramente marcado quando
schemas de origem ou semântica da plataforma de destino estiverem ausentes.
Nunca disfarce placeholders ou hipóteses como código deployable.

## 10. Handoff

Finalize com:

- artefatos criados ou revisados;
- validações realizadas e não realizadas;
- readiness status;
- próxima aprovação ou evidência necessária, de forma exata.
