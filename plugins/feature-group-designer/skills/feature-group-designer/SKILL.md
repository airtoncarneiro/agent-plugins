---
name: feature-group-designer
description: Analyze a data scientist's SQL query to design, specify, review, or implement reusable Feature Groups and their contracts. Use for SQL-to-Feature-Group decomposition, feature inventories, grain and entity analysis, point-in-time safety, grouping decisions, contract authoring, and implementation planning. Do not use to treat a model dataset, CTE, or source table as an automatic Feature Group boundary.
---

# Feature Group Designer

Transform a model-oriented SQL query into an auditable Feature Group design. Work from the query's actual expressions and lineage, distinguish evidence from assumptions, and make grain boundaries visible before grouping features.

## Accept the input

- Accept SQL pasted in the conversation or a local SQL file path.
- Read the complete query before proposing groups. Follow local SQL dependencies only when they are needed to resolve an expression or source and are available within the user's authorized scope.
- Never require a database connection merely to perform static design analysis.
- If the target platform is not specified, produce vendor-neutral contracts and an implementation plan. Do not invent product-specific syntax.
- Match the user's language; default to Brazilian Portuguese when the user writes in Portuguese.

## Choose the requested depth

- **Analyze:** inventory the query, expose grains, lineage, risks, and candidate groups without creating implementation artifacts.
- **Design:** perform the analysis and propose Feature Groups with boundary rationales and draft contracts. This is the default.
- **Create:** additionally create contract and transformation artifacts. Only generate platform-specific resources when the platform and required operational settings are known or explicitly marked as assumptions.
- **Review:** evaluate an existing Feature Group or contract against the same criteria and report concrete defects and corrections.

## Execute the workflow

For SQL analysis or design, read [references/decomposition-method.md](references/decomposition-method.md) and follow it in order.

1. Establish the query's purpose, final output grain, prediction/reference timestamp, and target. Infer provisionally when necessary and mark each inference.
2. Inventory final output expressions and classify every item as identifier, time, feature, target/label, request-time input, control field, or non-feature output.
3. Trace each candidate feature through CTEs, joins, filters, aggregations, windows, and source columns. Preserve business filters such as exclusions and status predicates.
4. Write an explicit grain contract for every candidate relation and Feature Group. Compare incompatible grains in a table before clustering.
5. Form semantic candidates only after entity and grain are known. Then test temporal, operational, governance, ownership, and reuse compatibility.
6. Separate targets from published features. Classify base, aggregated, derived, and request-time features and record their dependencies.
7. Design point-in-time behavior using both event validity and, when arrival can be delayed, availability time. Treat unresolved temporal semantics as a publication blocker.
8. Produce the requested deliverables using [references/deliverables.md](references/deliverables.md).

For a contract or creation request, also read [references/contract-guide.md](references/contract-guide.md). Start from [assets/feature-group-contract.yaml](assets/feature-group-contract.yaml) rather than inventing a smaller contract.

## Apply boundary rules

A Feature Group is the smallest reusable family of features that shares:

- the same entity and compatible business identity;
- the same row grain and record key;
- one nameable semantic concept;
- compatible event-time, availability, window, and late-data semantics;
- compatible refresh, freshness SLO, serving mode, retention, and backfill lifecycle;
- compatible ownership, access classification, and schema evolution policy.

Use source tables and CTEs as lineage evidence, not automatic boundaries. The same source may fan out into several grains and Feature Groups; one Feature Group may depend on several sources. Keep window variants together when the facts, filters, grain, cadence, ownership, and governance are compatible. Split them when those operational or semantic contracts diverge.

Treat DDD as a supporting lens for language, ownership, and bounded contexts. Do not let DDD replace the entity, grain, time, operation, and governance tests.

## Preserve uncertainty

- Never invent business definitions from a column alias alone.
- Label facts as **confirmed by SQL**, **inferred**, or **needs confirmation**.
- Continue with a useful provisional design when missing information is non-blocking.
- Mark a contract `draft` while entity, grain, target separation, event time, availability semantics, owner, or freshness requirements remain unresolved.
- Call a contract `publishable` only when the critical decisions are explicit and the proposed validation checks can enforce them.

## Creation safety

- Do not execute source SQL, create tables, deploy pipelines, or mutate a feature store unless the user explicitly requests that action and the target environment is identified.
- Before platform-specific creation, show or create the complete proposed contract, list assumptions, and identify destructive or backfill implications.
- Keep model-specific learned transformations in the model pipeline unless they have an independently owned, reusable domain meaning.
- Never publish a future outcome or label as a feature.

## Quality bar

Reject designs that hide grain, equate one entity with one giant group, create one group per model or time window without an operational reason, omit point-in-time rules, or present unresolved assumptions as facts. The final result must make every grouping and split decision traceable to evidence in the SQL or to an explicitly identified requirement.
