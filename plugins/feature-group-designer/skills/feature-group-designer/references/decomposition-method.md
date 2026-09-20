# Método de decomposição SQL-to-Feature-Group

Use este método em toda análise de SQL. O objetivo é tornar o design
reproduzível: outro revisor deve conseguir rastrear um Feature Group proposto
até a query e entender cada hipótese.

## 1. Enquadre a decisão

Registre, quando disponível:

- decisão de negócio ou previsão apoiada;
- nome do modelo ou consumidor, como metadata de consumo e não como fronteira;
- significado de uma linha da saída final;
- timestamp de prediction ou observation;
- target/label e sua outcome window;
- caminho de serving batch ou online necessário;
- refresh, freshness, histórico, retenção e backfill esperados.

Não pare apenas porque algum contexto está ausente. Continue com hipóteses
provisórias e inclua-as no registro de decisões em aberto. Pare antes de chamar
um contrato de `publishable` se um item não resolvido puder mudar entidade,
grão, separação do target, point-in-time correctness, política de acesso ou
implementação física.

## 2. Analise o SQL por papel

Leia a query das CTEs de origem até a projeção final, mas faça primeiro o
inventário da saída final. Para cada expressão final, registre:

| Campo | Significado |
|---|---|
| `output_name` | Alias ou coluna final |
| `role` | identifier, time, feature, target, request-time, control ou outro |
| `expression` | Expressão SQL normalizada |
| `sources` | Relações físicas e colunas de origem |
| `filters` | Predicados de status, exclusão, coorte e qualidade |
| `joins` | Caminho do join e hipótese de cardinalidade |
| `aggregation` | Função, chaves de agrupamento e uso de distinct |
| `window` | Lookback ou outcome window e seus limites |
| `entity` | Sujeito descrito pelo valor |
| `grain` | Significado exato da linha nessa etapa |
| `temporal fields` | Event, availability, computation e reference times |
| `status` | confirmed, inferred ou needs confirmation |

Não presuma que toda expressão numérica selecionada seja uma feature. IDs,
campos de partição, labels, sample weights, controles de treinamento e audit
columns têm papéis diferentes.

## 3. Rastreie lineage e filtros semânticos

Para cada feature candidata, siga aliases e dependências por CTEs aninhadas e
subqueries até chegar às colunas de origem ou a objetos externos não resolvidos.
Capture:

- todas as fontes e colunas contribuintes;
- join keys, tipo de join, cardinalidade esperada e possível fan-out;
- predicados em `WHERE`, `ON`, `HAVING` e agregações condicionais;
- deduplicação e lógica de seleção de registro;
- agregações, window functions e ordenação;
- defaults, casts, unidades, conversões de timezone e tratamento de nulo;
- dependências de outras expressões derivadas.

Trate um filtro como `status = 'approved'` ou a exclusão de reversões como parte
da definição da feature, não como detalhe incidental de implementação.

## 4. Construa o grain ledger antes de agrupar

Escreva cada grão como uma chave e como uma frase. Inclua pelo menos as relações
de origem/CTE relevantes, o model dataset final e cada Feature Group candidato.

Exemplo:

| Relação ou candidato | Entidade | Record key | Uma linha representa | Evidência |
|---|---|---|---|---|
| `orders` | order | `order_id` | um pedido | chave de origem ou deduplicação |
| `user_store_daily` | user-store | `user_id + store_id + reference_date` | o comportamento de um usuário em uma loja em uma data | `GROUP BY` |
| `model_dataset` | user | `user_id + prediction_timestamp` | uma oportunidade de previsão para um usuário | select final |

Torne o fan-out explícito. Uma mesma fonte pode produzir, por exemplo,
agregados de usuário, agregados user-store, estado operacional do pedido e
agregados de loja. Não os colapse apenas porque vieram da mesma tabela.

Bloqueie ou sinalize o design quando:

- a chave proposta não for única no grão declarado;
- um join puder multiplicar linhas antes da agregação sem tratamento deliberado;
- uma dimensão mudar ao longo do tempo, mas a query sempre usar seu valor atual;
- uma entidade composta for reduzida a apenas um componente da chave;
- o timestamp for necessário para identificar registros históricos, mas estiver
  ausente da record key.

## 5. Crie feature cards

Crie um card para cada feature candidata com:

- nome e definição em linguagem simples;
- papel e tipo: base, aggregate, derived ou request-time;
- entidade, business identity, record grain e lookup key;
- colunas de origem, transformação, filtros e unidades;
- lookback window, inclusividade, timezone e comportamento de janela vazia;
- event/reference/availability/computation timestamps;
- trigger de atualização, cadência, requisito de freshness e late-event policy;
- política de nulo/default e intervalo válido esperado;
- classificação de sensibilidade ou acesso;
- owner e consumidores conhecidos;
- dependências upstream e hipótese de reuso;
- status da evidência e perguntas em aberto.

Consolide os cards em uma matriz de compatibilidade antes de formar os grupos.
Use o grão como significado da linha, não apenas como o nome da entidade, e
registre o campo temporal em vez de classificá-lo genericamente como snapshot:

| Feature | Entity | Record grain | Semantic concept | Window | Event timestamp | Refresh | Sources |
|---|---|---|---|---|---|---|---|
| `dias_desde_ativacao` | usuario | um snapshot por usuário e reference time | ciclo de vida | — | `snapshot_timestamp` | diário | `contas` |
| `quantidade_pedidos_30d` | usuario | um snapshot por usuário e reference time | comportamento de compras | 30d | `snapshot_timestamp` | horário | `pedidos` |
| `quantidade_pedidos_90d` | usuario | um snapshot por usuário e reference time | comportamento de compras | 90d | `snapshot_timestamp` | horário | `pedidos` |
| `valor_medio_pedido_30d` | usuario | um snapshot por usuário e reference time | comportamento de compras | 30d | `snapshot_timestamp` | horário | `pedidos` |
| `quantidade_transferencias_7d` | usuario | um snapshot por usuário e reference time | transferências | 7d | `snapshot_timestamp` | 5 min | `transferencias` |
| `valor_transferencias_7d` | usuario | um snapshot por usuário e reference time | transferências | 7d | `snapshot_timestamp` | 5 min | `transferencias` |

Use a matriz para tornar candidatos de agrupamento e incompatibilidades
visíveis. Ela não substitui os feature cards nem implica separação apenas porque
as fontes são diferentes.

Se o SQL não permitir estabelecer uma definição de negócio, descreva a expressão
com precisão e peça o significado de domínio. Não o fabrique.

## 6. Gere e separe candidatos

Agrupe nesta ordem:

1. entidade e business identity;
2. record grain e key;
3. semantic concept;
4. compatibilidade temporal;
5. compatibilidade operacional;
6. governança e ownership;
7. formato de reuso e dependências.

A proximidade na fonte não é requisito de compatibilidade. Use-a depois para
planejar pipelines e lineage.

### Identifique a coesão semântica

Depois de estabelecer entidade e grão, pergunte qual aspecto do sujeito cada
feature descreve.

Por exemplo, no mesmo grão de usuário, `dias_desde_cadastro`,
`dias_desde_primeiro_pedido` e `fez_pedido_primeiros_14d` podem formar o
candidato `usuario_ciclo_vida`. Já `quantidade_pedidos_28d`,
`valor_medio_cesta_28d` e `quantidade_itens_28d` podem formar o candidato
`usuario_comportamento_compras`.

Um Feature Group deve possuir coesão semântica, não ser apenas um agrupamento
físico de colunas. Uma decomposição semântica candidata pode ser:

```text
usuario_id
│
├── Perfil
│     ├── regiao_cadastro
│     └── data_cadastro
│
├── Comportamento de compras
│     ├── quantidade_pedidos_14d
│     ├── valor_compras_14d
│     ├── quantidade_pedidos_84d
│     └── valor_compras_84d
│
├── Engajamento no aplicativo
│     ├── categoria_loja_preferida
│     └── duracao_media_sessao_14d
│
└── Preferências de entrega
      └── quantidade_enderecos_salvos
```

Essa decomposição identifica clusters semânticos candidatos; ela ainda não
afirma que cada cluster será um Feature Group. Como entidade e grão já foram
estabelecidos, teste em seguida a compatibilidade temporal, operacional, de
governança, de ownership e de reuso. Use a origem para lineage e planejamento
da transformação, não como fronteira automática.

Para cada par ou família de features, avalie:

```text
same_entity?
same_grain?
same_semantic_context?
compatible_event_and_availability_time?
compatible_refresh_and_freshness?
compatible_governance_and_owner?
shared_computation_or_reuse?
```

Entidade ou grão diferentes normalmente obrigam uma separação. Diferenças
semânticas, temporais, operacionais ou de governança também obrigam a separação
quando compartilhar um contrato acoplar ciclos de vida ou acesso incompatíveis.

## 7. Trate as semânticas temporais

Para cada Feature Group, defina:

- timestamp ao qual o valor é válido (`event_timestamp` ou equivalente);
- timestamp em que se tornou conhecível (`available_at` ou uma lag policy
  conservadora explícita) quando a chegada puder atrasar;
- observation window e inclusão/exclusão exata dos limites;
- regras de timezone e calendário;
- política de late-event, correção e recomputação;
- política de histórico/backfill;
- precedência online para writes out-of-order, se aplicável.

Para uma training row no prediction time `T`, a regra segura de recuperação é
conceitualmente:

```text
feature.event_timestamp <= T
and feature.available_at <= T  # quando availability pode atrasar a validade
choose the latest permitted version for the entity and grain
```

Outcome windows pertencem a targets, não a features. Um valor calculado a partir
de eventos posteriores a `T` é label ou leakage, salvo quando a previsão for
explicitamente feita depois desses eventos.

## 8. Posicione features derivadas

Use esta decisão:

| Situação | Posicionamento |
|---|---|
| Regra de domínio estável e reutilizável, com owner e ciclo compatíveis | Mesmo Feature Group ou derived group do domínio |
| Derivação entre famílias, com significado próprio e vários consumidores | Derived Feature Group com dependências explícitas |
| Pesos aprendidos ou fórmula específica de experimento | Model pipeline/artifact |
| Requer dados conhecidos apenas no inference request | Request-time transformation reproduzida de forma consistente no treinamento |

Registre o grafo de dependências da feature de origem ao agregado, à feature
derivada e ao consumidor.

### Visualize o catálogo e as dependências

Use uma visão arquitetural compacta para situar os grupos no domínio e tornar
as dependências explícitas, sem repetir o inventário completo de features:

```text
Domínio: experiencia_usuario
Entidade: usuario (`usuario_id`)
│
├── FG: usuario_perfil
│
├── FG: usuario_comportamento_compras
│
├── FG: usuario_engajamento_aplicativo
│
└── FG derivado: usuario_pontuacao_atividade
      ├── depende de: usuario_perfil
      └── depende de: usuario_comportamento_compras
```

Essa visão organiza o catálogo e explicita dependências. Ela não substitui os
contratos individuais de grão, tempo, operação e governança, nem transforma
domínio, entidade ou fonte em fronteiras automáticas de Feature Group.

## 9. Valide a decomposição

Exija evidência ou um teste proposto para:

- unicidade em cada grão declarado;
- cardinalidade dos joins e ausência de fan-out acidental;
- conformidade de schema, tipo, unidade, nulo e intervalo válido;
- casos de limite de cada janela;
- ausência de eventos ou conhecimento tardio posterior ao prediction time;
- equivalência entre o dataset histórico decomposto e a query original em
  amostras representativas;
- backfill idempotente e tratamento de eventos out-of-order;
- offline/online parity quando houver serving online;
- mudanças semânticas versionadas e impacto conhecido nos consumidores.

A análise estática pode propor esses testes, mas não pode afirmar seus resultados
sem execução contra dados representativos.

## 10. Classifique o readiness

Use exatamente um status para cada grupo proposto:

- `draft`: hipóteses ou definições críticas continuam abertas;
- `ready_for_review`: estrutura e contrato estão completos o suficiente para
  revisão de domínio e plataforma, mas ainda falta validação de dados ou
  aprovação;
- `publishable`: semânticas críticas estão confirmadas e a implementação passou
  pelos testes de dados e temporais declarados.

Nunca infira `publishable` apenas pela estrutura da SQL.
