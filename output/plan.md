# Internal Build Notes

## Goal

Waiting for the user's script goal.

## Gates

- [ ] Fresh base schema verified.
- [ ] Approach and script split planned.
- [ ] Dependencies and handoffs mapped.
- [ ] Airtable input configuration captured.
- [ ] Input keys verified.
- [ ] Script implemented.
- [ ] Runtime output reviewed.
- [ ] Final verification complete.

## Requirements interpretation

- In scope:
- Out of scope:
- Assumptions and unresolved decisions:

## Execution and record selection

- Execution type and trigger:
- Record-selection strategy:
- Expected scale and timeout behaviour:

## Dependencies and mapping

- Tables, views, fields, record IDs, runtime settings, APIs, and secrets:
- Required and optional semantics:

## Ownership and safety

- Source of truth and create/update/delete ownership:
- Idempotency, duplicate detection, concurrency, and rerun behaviour:
- Validation, failure, retry, checkpoint, and recovery behaviour:

## Scripts

For each script, record:

- Purpose and execution type.
- Tables, views, fields, and runtime values with exact input keys.
- Reads, mutations, and expected outputs.
- Values passed to later actions or state persisted in Airtable.
- Objects that require resolution; all other dependencies should be referenced
  directly from the single input configuration object.

## Risks and tests

- Known risks and mitigations:
- Acceptance tests traced to requirements:
