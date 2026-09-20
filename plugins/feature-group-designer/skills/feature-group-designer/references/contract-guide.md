# Guia de contrato de Feature Group

Leia esta referência quando o usuário solicitar uma especificação, contrato,
implementação ou revisão de um contrato existente.

## Princípios do contrato

O contrato é vendor-neutral, a menos que uma plataforma de destino seja
selecionada explicitamente. Ele deve tornar decisões semânticas e operacionais
inspecionáveis e testáveis. Uma definição física de tabela, sozinha, não é um
contrato de Feature Group.

Comece pelo [template de contrato de Feature Group](../assets/feature-group-contract.yaml)
e preserve suas seções principais. Remova um campo opcional somente quando ele
realmente não se aplicar; use `needs_confirmation` para decisões obrigatórias que
continuarem abertas.

## Decisões obrigatórias

### Identidade e grão

- `name` deve ser orientado a entidade e conceito, não ao modelo.
- `entity.name` identifica o sujeito.
- `entity.business_keys` define a identidade estável.
- `grain.description` declara o que uma linha representa.
- `grain.record_keys` deve ser suficiente para tornar um registro único,
  incluindo chaves temporais ou secundárias quando necessário.
- `lookup.keys` declara o que os consumidores fornecem para lookup e como o
  reference time histórico é informado.

### Tempo

- `event_timestamp` identifica quando o valor é válido.
- `available_at` identifica quando o valor se tornou disponível quando o
  conhecimento puder atrasar; caso contrário, documente por que ele é
  desnecessário e qual lag policy se aplica.
- `timezone`, limites da janela, refresh cadence, freshness SLO, late-event
  policy, backfill e retention devem ser explícitos.
- Um atributo estático de fonte ainda pode exigir historization. Uma idade
  calculada é temporal mesmo quando sua data de origem é estável.

### Features

Cada feature declara:

- tipo, definição, kind e referência à expressão ou transformação;
- fontes, filtros, grão, unidade, janela, comportamento de nulo/default;
- campos temporais e dependências;
- classificação de sensibilidade e regras de validação.

Não inclua entity keys, audit fields, targets ou sample controls em `features`.
Mantenha-os em suas seções próprias.

### Operação e serving

- Declare os requisitos offline e online de forma independente. Online é
  opcional.
- Defina update mode, idempotency key, correction policy e precedência para
  dados out-of-order.
- Defina owner, access class, lineage da fonte, versão da transformação,
  compatibilidade de schema, consumidores e regras de deprecation.

### Qualidade

No mínimo, especifique testes para:

- unicidade e não nulidade das chaves;
- expectativas de cobertura referencial ou de entidade;
- tipos, unidades, intervalos, proporções de nulo e valores permitidos;
- freshness e taxa de late-event;
- correção dos limites da janela;
- point-in-time leakage;
- paridade entre os cálculos original e decomposto;
- offline/online parity quando aplicável.

## Classificação de mudanças

| Mudança | Tratamento esperado |
|---|---|
| Adição de feature opcional | Versão minor compatível quando consumidores não forem afetados |
| Correção de lógica que altera valores históricos | Versão semântica e plano de impacto/backfill |
| Renomeação ou remoção de feature | Período de deprecation e migração dos consumidores |
| Mudança de tipo, unidade, filtro, grão ou janela | Breaking change; nunca silenciosa |
| Mudança de owner ou access classification | Revisão de governança antes da publicação |

## Gate de publicação

Um contrato não pode ser `publishable` enquanto qualquer item abaixo estiver
indefinido:

- identidade da entidade ou grão;
- classificação entre feature e target;
- semântica de event-time ou availability;
- definição de negócio ou filtros críticos;
- owner ou access classification;
- obrigação de refresh/freshness;
- validação de unicidade e correção temporal.

Use `ready_for_review` quando o contrato estiver completo, mas ainda aguardar
aprovação de domínio ou execução de dados. Use `draft` quando decisões críticas
ainda forem hipóteses.
