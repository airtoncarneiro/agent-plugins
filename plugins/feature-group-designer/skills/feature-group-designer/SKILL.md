---
name: feature-group-designer
description: Analisa uma query SQL de cientista de dados para desenhar, especificar, revisar ou implementar Feature Groups reutilizáveis e seus contratos. Use para decomposição SQL-to-Feature-Group, inventário de features, análise de grão e entidade, point-in-time safety, decisões de agrupamento, autoria de contratos e planejamento de implementação. Não trate um model dataset, CTE ou source table como uma fronteira automática de Feature Group.
---

# Feature Group Designer

Transforme uma query SQL orientada a modelo em um design auditável de Feature
Groups. Trabalhe a partir das expressões e da lineage real da query, diferencie
evidências de hipóteses e torne as fronteiras de grão visíveis antes de agrupar
features.

## Aceite da entrada

- Aceite SQL colado na conversa ou o caminho de um arquivo SQL local.
- Leia a query completa antes de propor grupos. Siga as dependências locais de
  SQL somente quando forem necessárias para resolver uma expressão ou fonte e
  estiverem dentro do escopo autorizado pelo usuário.
- Nunca exija conexão com banco para realizar análise estática de design.
- Se a plataforma de destino não for especificada, produza contratos
  vendor-neutral e um plano de implementação. Não invente sintaxe específica de
  produto.
- Responda no idioma do usuário; quando ele escrever em português, use PT-BR.

## Escolha a profundidade solicitada

- **Analyze:** inventarie a query e exponha grãos, lineage, riscos e grupos
  candidatos sem criar artefatos de implementação.
- **Design:** faça a análise e proponha Feature Groups com justificativas de
  fronteira e contratos preliminares. Este é o modo padrão.
- **Create:** além do Design, crie artefatos de contrato e transformação. Gere
  recursos específicos de plataforma somente quando a plataforma e as
  configurações operacionais necessárias forem conhecidas ou estiverem marcadas
  explicitamente como hipóteses.
- **Review:** avalie um Feature Group ou contrato existente segundo os mesmos
  critérios e reporte defeitos concretos e correções.

## Execute o fluxo de trabalho

Para análise ou design de SQL, leia e siga, na ordem, [references/decomposition-method.md](references/decomposition-method.md).

1. Estabeleça o objetivo da query, o grão final da saída, o timestamp de
   prediction/reference e o target. Quando necessário, faça uma inferência
   provisória e marque-a como tal.
2. Faça o inventário das expressões da saída final e classifique cada item como
   identifier, time, feature, target/label, request-time input, control field ou
   non-feature output.
3. Rastreie cada feature candidata pelos CTEs, joins, filtros, agregações,
   janelas e colunas de origem. Preserve filtros de negócio, como exclusões e
   predicados de status.
4. Escreva um contrato explícito de grão para cada relação candidata e cada
   Feature Group. Compare os grãos incompatíveis em uma tabela antes de agrupar.
5. Forme candidatos semânticos somente depois de conhecer entidade e grão. Em
   seguida, teste compatibilidade temporal, operacional, de governança, de
   ownership e de reuso.
6. Separe targets de features publicadas. Classifique features base, agregadas,
   derivadas e request-time e registre suas dependências.
7. Projete o comportamento de point-in-time usando validade do evento e,
   quando a chegada puder atrasar, availability time. Trate semântica temporal
   não resolvida como bloqueio para publicação.
8. Produza as entregas solicitadas usando [references/deliverables.md](references/deliverables.md).

Para uma solicitação de contrato ou criação, leia também
[references/contract-guide.md](references/contract-guide.md). Comece pelo
template [assets/feature-group-contract.yaml](assets/feature-group-contract.yaml)
em vez de inventar um contrato menor.

## Aplique as regras de fronteira

Um Feature Group é a menor família reutilizável de features que compartilha:

- a mesma entidade e uma business identity compatível;
- o mesmo row grain e record key;
- um semantic concept nomeável;
- semânticas compatíveis de event-time, availability, janela e late-data;
- refresh, freshness SLO, serving mode, retenção e ciclo de backfill
  compatíveis;
- ownership, classificação de acesso e política de evolução de schema
  compatíveis.

Use source tables e CTEs como evidência de lineage, não como fronteiras
automáticas. A mesma fonte pode produzir vários grãos e Feature Groups; um
Feature Group pode depender de várias fontes. Mantenha variantes de janela juntas
quando fatos, filtros, grão, cadência, ownership e governança forem compatíveis.
Separe-as quando esses contratos operacionais ou semânticos divergirem.

Trate DDD como uma lente complementar para linguagem, ownership e bounded
contexts. Não permita que DDD substitua os testes de entidade, grão, tempo,
operação e governança.

## Preserve as incertezas

- Nunca invente definições de negócio a partir apenas de um column alias.
- Classifique fatos como **confirmed by SQL**, **inferred** ou **needs
  confirmation**.
- Continue com um design provisório útil quando a informação ausente não for
  bloqueante.
- Marque um contrato como `draft` enquanto entidade, grão, separação de target,
  event time, semântica de availability, owner ou requisitos de freshness
  permanecerem indefinidos.
- Chame um contrato de `publishable` somente quando as decisões críticas forem
  explícitas e os testes de validação propostos puderem verificá-las.

## Segurança na criação

- Não execute o SQL de origem, crie tabelas, faça deploy de pipelines nem
  altere um feature store, a menos que o usuário solicite explicitamente a ação
  e identifique o ambiente de destino.
- Antes de uma criação específica de plataforma, mostre ou crie o contrato
  completo, liste as hipóteses e identifique implicações destrutivas ou de
  backfill.
- Mantenha transformações aprendidas e específicas do modelo no model pipeline,
  salvo quando tiverem significado de domínio reutilizável e ownership próprio.
- Nunca publique um outcome futuro ou label como feature.

## Critério de qualidade

Rejeite designs que escondam o grão, equacionem uma entidade a um único grupo
gigante, criem um grupo por modelo ou janela sem motivo operacional, omitam
regras de point-in-time ou apresentem hipóteses não resolvidas como fatos. A
entrega final deve permitir rastrear cada decisão de agrupamento ou separação
até uma evidência da SQL ou a um requisito identificado explicitamente.
