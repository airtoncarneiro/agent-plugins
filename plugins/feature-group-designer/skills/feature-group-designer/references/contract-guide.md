# Feature Group contract guide

Read this reference when the user requests a specification, contract, implementation, or review of an existing contract.

## Contract principles

The contract is vendor-neutral unless a target platform is explicitly selected. It must make semantic and operational decisions inspectable and testable. A physical table definition alone is not a Feature Group contract.

Start from the [Feature Group contract template](../assets/feature-group-contract.yaml) and preserve its major sections. Remove an optional field only when it truly does not apply; use `needs_confirmation` for required decisions that remain open.

## Required decisions

### Identity and grain

- `name` is entity-and-concept oriented, not model-oriented.
- `entity.name` identifies the subject.
- `entity.business_keys` defines stable identity.
- `grain.description` states what one row means.
- `grain.record_keys` is sufficient to make a record unique, including temporal or secondary keys where necessary.
- `lookup.keys` states what consumers provide for lookup and how historical reference time is supplied.

### Time

- `event_timestamp` identifies when the value is valid.
- `available_at` identifies when it became available when late knowledge is possible; otherwise document why it is unnecessary and what lag policy applies.
- `timezone`, window boundaries, refresh cadence, freshness SLO, late-event policy, backfill, and retention are explicit.
- A static source attribute may still need historization. A computed age is temporal even if its source date is stable.

### Features

Each feature declares:

- type, definition, kind, expression or transformation reference;
- sources, filters, grain, unit, window, null/default behavior;
- temporal fields and dependencies;
- sensitivity classification and validation rules.

Do not include entity keys, audit fields, targets, or sample controls in `features`. Keep them in their dedicated sections.

### Operation and serving

- State offline and online requirements independently. Online is optional.
- Define update mode, idempotency key, correction policy, and out-of-order precedence.
- Define owner, access class, source lineage, transformation version, schema compatibility, consumers, and deprecation rules.

### Quality

At minimum, specify tests for:

- key uniqueness and non-nullness;
- referential or entity coverage expectations;
- types, units, ranges, null ratios, and allowed values;
- freshness and late-event rate;
- window boundary correctness;
- point-in-time leakage;
- parity between original and decomposed calculations;
- offline/online parity when applicable.

## Change classification

| Change | Expected treatment |
|---|---|
| Add an optional feature | Compatible minor version when consumers are unaffected |
| Fix logic that changes historical values | Semantic version and impact/backfill plan |
| Rename or remove a feature | Deprecation period and consumer migration |
| Change type, unit, filter, grain, or window | Breaking change; never silent |
| Change owner or access classification | Governance review before publication |

## Publication gate

A contract cannot be `publishable` while any of these are unresolved:

- entity identity or grain;
- feature versus target classification;
- event-time or availability semantics;
- business definition or critical filters;
- owner or access classification;
- refresh/freshness obligation;
- validation of uniqueness and temporal correctness.

Use `ready_for_review` when the contract is complete but still awaits domain approval or data execution. Use `draft` when critical decisions are still assumptions.
