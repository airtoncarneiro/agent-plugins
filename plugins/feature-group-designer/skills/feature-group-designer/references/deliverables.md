# Deliverable format

Use the smallest set of deliverables that satisfies the request. For an ordinary design request, present sections 1 through 8 below. For artifact creation, write equivalent files and summarize their paths.

## 1. Executive decision

State:

- how many Feature Groups are proposed;
- their names and one-line purposes;
- the final dataset grain;
- whether the result is `draft`, `ready_for_review`, or `publishable`;
- the most important blockers or assumptions.

## 2. Query inventory

Provide one row for every final output expression:

| Output | Role | Expression/source | Entity | Grain | Window/time | Evidence status |
|---|---|---|---|---|---|---|

Targets, keys, timestamps, controls, and non-feature fields remain visible even though they are not published features.

## 3. Grain contracts and comparison

Provide an explicit table:

| Candidate | Entity | Record keys | One row means | Compatible with |
|---|---|---|---|---|

Show every material grain produced from the same source. Do not bury grain inside prose.

## 4. Proposed Feature Groups

For each group, include:

- entity, business key, record grain, lookup key;
- semantic purpose;
- features;
- event and availability timestamps;
- cadence, freshness, offline/online requirement;
- owner/governance if known;
- source lineage;
- readiness status.

## 5. Boundary decision log

Explain why features were joined or split using this table:

| Decision | Features/groups | Evidence | Rule applied | Confidence |
|---|---|---|---|---|

Include rejected alternatives such as one group per model, source table, CTE, entity, or window when they were plausible from the input.

## 6. Target and dependency separation

List:

- labels/outcomes excluded from Feature Groups;
- base, aggregated, derived, and request-time features;
- cross-group dependencies and model-specific transformations.

## 7. Temporal and data-quality validation plan

Specify executable or testable checks for uniqueness, fan-out, nulls, windows, late knowledge, leakage, equivalence to the original query, backfill, and serving parity. Clearly distinguish proposed tests from tests actually executed.

## 8. Open decisions

Prioritize only questions that can change the contract. For each, state the provisional assumption and the consequence of a different answer.

## 9. Artifacts for creation requests

Unless the user specifies another layout, create:

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

The SQL transformation may remain a clearly marked skeleton when source schemas or target-platform semantics are missing. Never disguise placeholders or assumptions as deployable code.

## 10. Handoff

End with:

- artifacts created or reviewed;
- validation performed and not performed;
- readiness status;
- exact next approval or evidence needed.
