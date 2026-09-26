# SQL-to-Feature-Group decomposition method

Use this method for every SQL analysis. Its purpose is to make the design reproducible: another reviewer should be able to trace a proposed Feature Group back to the SQL and understand every assumption.

## 1. Frame the decision

Record, when available:

- business decision or prediction being supported;
- model or consumer name, as consumption metadata rather than a group boundary;
- final output row meaning;
- prediction or observation timestamp;
- target/label and its outcome window;
- required batch or online serving path;
- expected refresh, freshness, history, retention, and backfill behavior.

Do not stop merely because some context is absent. Continue with provisional assumptions and add them to the open-decisions register. Stop before claiming a publishable contract if an unresolved item can change entity, grain, target separation, point-in-time correctness, access policy, or physical implementation.

## 2. Parse the SQL by role

Read the query from source CTEs to the final projection, but inventory the final output first. For each final expression, record:

| Field | Meaning |
|---|---|
| output_name | Final alias or column name |
| role | identifier, time, feature, target, request-time, control, or other |
| expression | Normalized SQL expression |
| sources | Physical relations and source columns |
| filters | Status, exclusion, cohort, and quality predicates |
| joins | Join path and cardinality assumption |
| aggregation | Function, grouping keys, distinct behavior |
| window | Lookback or outcome window and boundary convention |
| entity | Subject described by the value |
| grain | Exact row meaning at this calculation stage |
| temporal fields | Event, availability, computation, and reference times |
| status | confirmed, inferred, or needs confirmation |

Do not assume every selected numeric expression is a feature. IDs, partition fields, labels, sample weights, training-only controls, and audit columns have different roles.

## 3. Trace lineage and semantic filters

For every candidate feature, follow aliases and dependencies through nested CTEs and subqueries until reaching source columns or unresolved external objects. Capture:

- all contributing sources and columns;
- join keys, join type, expected cardinality, and possible fan-out;
- `WHERE`, `ON`, `HAVING`, and conditional aggregation predicates;
- deduplication and record-selection logic;
- aggregations, window functions, and ordering;
- default values, casts, units, timezone conversions, and null treatment;
- dependencies on other derived expressions.

Treat a filter such as `status = 'approved'` or exclusion of reversals as part of the feature definition, not an incidental implementation detail.

## 4. Build the grain ledger before clustering

Write every grain as both a key and a sentence. Include at least the relevant source/CTE relations, the final model dataset, and each candidate Feature Group.

Example:

| Relation or candidate | Entity | Record key | One row means | Evidence |
|---|---|---|---|---|
| orders | order | `order_id` | one order | source key or deduplication |
| user_store_daily | user-store | `user_id + store_id + reference_date` | one user's behavior for one store on one date | `GROUP BY` |
| model_dataset | user | `user_id + prediction_timestamp` | one prediction opportunity for a user | final select |

Then make the fan-out explicit. A single source may produce, for example, user aggregates, user-store aggregates, order operational state, and store aggregates. Do not collapse these merely because the same table supplied them.

Block or flag the design when:

- the proposed key is not unique at the declared grain;
- a join can multiply rows before aggregation without deliberate handling;
- a dimension changes over time but the query always uses its current value;
- a composite entity is reduced to one component of its key;
- the timestamp is necessary to identify historical records but omitted from the record key.

## 5. Create feature cards

Create one card per candidate feature with:

- name and plain-language definition;
- role and feature kind: base, aggregate, derived, or request-time;
- entity, business identity, record grain, and lookup key;
- semantic family and the business question it answers;
- source columns, transformation, filters, and units;
- lookback window, inclusivity, timezone, and empty-window behavior;
- event/reference/availability/computation timestamps;
- update trigger, cadence, freshness requirement, and late-event policy;
- null/default policy and expected valid range;
- sensitivity or access classification;
- owner and known consumers;
- upstream dependencies and reuse hypothesis;
- evidence status and open questions.

If the SQL cannot establish a business definition, describe the expression precisely and ask for the domain meaning. Do not manufacture one.

## 6. Generate and split candidates

Before clustering, partition features into provisional semantic families. Give each family a nameable concept, the business question it answers, and evidence from the SQL or supplied requirements. A family may contain base, aggregate, and derived features when they describe the same concept.

Use this comparison before proposing any merge:

| Feature or family | Entity and grain | Semantic family | Time/window | Lifecycle evidence | Merge disposition |
|---|---|---|---|---|---|
| `customer_segment`, `account_age_days` | customer at reference time | account profile/lifecycle | current/reference time | unknown | candidate family |
| `transaction_count_30d`, `transaction_value_30d` | customer at reference time | recent transaction behavior | trailing 30 days | same source/window, cadence unknown | candidate family |

The example illustrates a decision pattern, not prescribed domain names. Equal entity, record key, source, or cadence makes families comparable; it is not proof that they belong in one Feature Group. Do not label a group with one concept while silently including a different concept.

Cluster each semantic family in this order:

1. entity and business identity;
2. record grain and key;
3. semantic concept;
4. temporal compatibility;
5. operational compatibility;
6. governance and ownership compatibility;
7. reuse and dependency shape.

Source proximity is not a required match. Use it later to plan pipelines and lineage.

For each pair or family of features, evaluate:

```text
same_entity?
same_grain?
same_semantic_context?
compatible_event_and_availability_time?
compatible_refresh_and_freshness?
compatible_governance_and_owner?
shared_computation_or_reuse?
```

Different entity or grain normally forces a split. `same_semantic_context?` must have evidence, not merely a positive inference from shared keys or sources. Semantic, temporal, operational, or governance differences force a split when sharing a contract would couple incompatible lifecycle or access requirements. When semantic evidence is absent, keep separate candidate groups and record the merge decision as unresolved rather than producing one broad group.

## 7. Handle temporal semantics

For each Feature Group, define:

- the timestamp to which the value is valid (`event_timestamp` or equivalent);
- the timestamp at which it became knowable (`available_at` or an explicit conservative lag policy) when late arrival matters;
- the observation window and exact start/end inclusivity;
- timezone and calendar rules;
- late-event, correction, and recomputation policy;
- history/backfill policy;
- online precedence rule for out-of-order writes, if applicable.

For a training row at prediction time `T`, the safe retrieval rule is conceptually:

```text
feature.event_timestamp <= T
and feature.available_at <= T  # when availability can lag validity
choose the latest permitted version for the entity and grain
```

Outcome windows belong to targets, not features. A value computed from events after `T` is a label or leakage unless the prediction is explicitly made after those events.

## 8. Place derived features

Use the following decision:

| Situation | Placement |
|---|---|
| Stable reusable domain rule with compatible owner and lifecycle | Same Feature Group or a domain-owned derived group |
| Cross-family derivation with independent meaning and multiple consumers | Derived Feature Group with explicit dependencies |
| Learned weights or experiment-specific formula | Model pipeline/artifact |
| Requires data known only at inference request | Request-time transformation reproduced consistently for training |

Record the dependency graph from source feature to aggregate to derived feature to consumer.

## 9. Validate the decomposition

Require evidence or a proposed test for:

- uniqueness at every declared grain;
- join cardinality and absence of accidental fan-out;
- schema, type, unit, null, and valid-range conformance;
- boundary cases for every window;
- no events or late-arriving knowledge from after prediction time;
- equivalence between the decomposed historical dataset and the original query on representative samples;
- idempotent backfill and handling of out-of-order events;
- offline/online parity when online serving is required;
- versioned semantic changes and known consumer impact.

Static analysis can propose these checks but cannot claim their results without execution against representative data.

## 10. Rate readiness

Use exactly one status for each proposed group:

- `draft`: critical assumptions or definitions remain open;
- `ready_for_review`: structure and contract are complete enough for domain and platform review, but data validation or approval remains;
- `publishable`: critical semantics are confirmed and the implementation has passed the declared data and temporal checks.

Never infer `publishable` from SQL structure alone.
