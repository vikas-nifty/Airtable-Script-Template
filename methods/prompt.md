# Airtable Scripting Workflow

Use this workspace to design, build, review, and refine Airtable scripts from
user-owned requirements and fresh Airtable evidence.

This file is the immutable high-level workflow standard. Record project-specific
interpretation, implementation decisions, and evolving build notes in
`output/plan.md`; do not copy those details back into this file.

## Authority and ownership

1. `context/requirements.md` is the source request and may be modified only by
   the user. Never alter it, infer checked acceptance criteria into it, or repair
   it silently.
2. `context/baseSchema.md` is authoritative for tables, views, fields, types, options,
   and relationships only after the user has refreshed it for the target base.
3. `context/inputConfig.md` is authoritative for runtime mappings only after the
   user has refreshed it for the relevant script action.
4. `output/plan.md` is required. It is Codex's current interpretation of the
   requirements plus confirmed chat decisions and must identify assumptions,
   exclusions, dependencies, invariants, risks, and acceptance tests.
5. Chat provides runtime logs and test evidence. Do not create or maintain a
   repository console-log file.

Do not invent schema, options, relationships, record IDs, inputs, secret names,
API behaviour, business rules, or test results. Verify unstable external API
details against primary documentation when they affect implementation.

## Gated workflow

Complete and verify one stage before moving to the next.

### 1. Requirements and goal

- Read the requirements without editing them.
- Establish the outcome, execution type, trigger, record-selection method,
  inputs, outputs, side effects, expected scale, acceptance evidence, and explicit
  exclusions.
- Identify contradictions or decisions that materially change the solution.
- Do not write code.

### 2. Fresh schema

- Ask the user to run the extractor in `context/baseSchema.md` in the exact
  target base and replace stale output.
- Verify every required field exists with the expected type, select options,
  linked-table dependency, and writeability.
- If schema is missing or wrong, stop and ask the user to change Airtable and
  refresh the schema. Never fabricate IDs or proposed schema output.

### 3. Required plan

Write or update `output/plan.md` before input configuration. Include:

- requirement interpretation and out-of-scope items;
- Automation, Scripting Extension, or record-button execution model;
- trigger and record-selection strategy (record ID, configured view, or scan);
- field/API mapping and required/optional semantics;
- complete dependency map and input-key names;
- reads, mutations, outputs, secrets, and inter-action handoffs;
- create/update/delete ownership and source of truth;
- idempotency key, duplicate detection, concurrency behaviour, and rerun rules;
- validation, partial-failure, retry, checkpoint, and timeout behaviour;
- external API authentication, rate limits, response handling, and procurement
  implications where applicable;
- test matrix traced to the user requirements.

Do not treat “avoid duplicate external records” as sufficient. Also decide what
happens when multiple Airtable records claim the same external ID or business
key. Ambiguous ownership must fail visibly before external mutation.

### 4. Configure inputs and secrets

- Map every used table, view, field, record ID, and runtime setting in the
  relevant Airtable script action.
- Input keys use `table_field` with lower camel case within segments, such as
  `projects_startDate`. Non-schema settings use `scope_setting`, such as
  `api_baseUrl`.
- Record all non-secret values with the logger in `context/inputConfig.md`.
- Verify secret names and presence with the same helper. It may output only the
  documented masked preview; never print, copy, or persist an unmasked secret.
- For multiple script actions, capture and label each action independently.
  Actions may repeat identical mappings, but each action still has its own
  `input.config()` configuration in Airtable.

### 5. Verify inputs

- Match each planned dependency to one exact input key and one current schema
  object/value.
- Resolve all missing, stale, duplicated, ambiguously named, or unused mappings.
- Confirm computed fields used for eligibility/checkpoints are configured to
  watch the intended source fields; their dependency settings may require user
  confirmation if the schema extractor cannot expose them.
- Do not write code while a required dependency is unresolved.

### 6. Code

- Update the plan to reflect final verified keys.
- Write the complete script or numbered scripts following
  `methods/codingStyle.md`.
- Keep one `config = input.config()` object. Reference its verified values
  directly and resolve only Airtable objects needed for method calls.
- Build one `secrets` object from verified secret mappings listed once. Access
  secrets through that object rather than creating repeated standalone variables.
- Do not add a new dependency while coding. Return to Configure and Verify.
- Run static syntax and repository checks before handoff.

### 7. Test and revise

- Ask for the smallest test that proves the next unverified behaviour.
- Runtime evidence must be pasted in chat and must not contain credentials,
  authorization headers, unmasked secrets, or unnecessary sensitive payloads.
  The documented masked previews from the input-capture helper are permitted.
- Distinguish implementation from evidence: code presence does not prove a live
  create, update, duplicate, recovery, error, permission, or timeout path.
- Trace evidence to the plan's test matrix and mark only demonstrated cases.
- Diagnose before changing code and make the smallest justified revision.
- When a new dependency emerges, return to the relevant gate.

## Execution models

### Automation

- Read configured variables once with `const config = input.config()`.
- Read verified secrets once into a `secrets` object and reuse its properties.
- Use `fetch()` for HTTP; do not use `remoteFetchAsync()`.
- Never use `input.recordAsync()`.
- Do not assume the trigger supplies a record ID. If it does not, use a verified
  configured view or scan with explicit eligibility and checkpoint fields.
- Use `output.set()` only for a documented handoff to a later action.

### Multiple script actions in one Automation

- Name files `output/script1.js`, `output/script2.js`, and so on in the exact
  order their Airtable actions execute.
- Airtable executes actions sequentially, but each script starts with isolated
  memory and its own `input.config()`.
- Repeat the same input keys/values in multiple actions when they genuinely share
  dependencies; otherwise configure only what each action uses.
- Record every handoff in `output/plan.md`: producing script, `output.set()` key,
  consuming script/input key, value type, and failure behaviour.
- Use `output.set()` only for values consumed later in the same Automation run.
  Persist durable, resumable, or cross-run state in Airtable.
- Plan what happens when an earlier action fails: later actions must not assume a
  handoff exists unless the Automation path guarantees it.

### Scripting Extension and record button

- Secrets and Automation input variables are unavailable.
- Use interactive input methods appropriate to the extension.
- A record button may use `input.recordAsync()`.
- Use `remoteFetchAsync()` when Airtable's extension environment requires it.

## Airtable reference

- One known record: `table.selectRecordAsync(recordId)`.
- Many records: `table.selectRecordsAsync({ fields, sorts })`.
- Filtered set: configured view plus `view.selectRecordsAsync({ fields })`.
- `filterByFormula` is not available in `selectRecordsAsync`; filter in
  JavaScript or use a configured view.
- Raw value: `record.getCellValue(config.table_field)`.
- Display value: `record.getCellValueAsString(config.table_field)`.
- Create: `table.createRecordAsync({ fields: { [config.table_field]: value } })`.
- Update: `table.updateRecordAsync(recordId, { [config.table_field]: value } })`.
- Batch creates/updates/deletes: groups of no more than 50.
- Linked records: arrays of `{ id }`; never write record IDs as bare strings.
- Single select: `{ name: option }`; multiple select: arrays of `{ name }`.
- Formula, lookup, rollup, created-time, and similar computed fields are usually
  read-only.

## Operational constraints

- Automation script action: 180 seconds.
- Whole Automation: approximately 40 minutes.
- HTTP timeout: approximately 12 seconds.
- Memory: approximately 512 MB.
- Fetches: approximately 50.
- Queries: approximately 30.
- Airtable mutation batches: at most 50 records and approximately 15 mutations
  per second.
- External packages such as `crypto` are unsupported.

For larger workloads, use configured views, indexes, batches, elapsed-time
guards, checkpoints, resumable statuses, and safe rerun behaviour. Do not start
work that cannot be left in a coherent state before timeout.

## Safeguards

- Use Airtable JavaScript, not Python or unsupported packages.
- Never hard-code Airtable table, view, or field names/IDs in executable code.
- Never log unmasked secrets, tokens, passwords, authorization headers, or
  unnecessary sensitive payloads. Masked secret previews are permitted only in
  the input-capture helper.
- Validate all run-wide invariants before external calls or Airtable mutations.
- Validate ownership and duplicate claims before creating or updating external
  records.
- Do not delete, archive, deactivate, email, publish, or mutate external systems
  unless the requirements and plan explicitly authorize it.
- Surface record-level failures on the relevant Airtable record when the schema
  provides status/error fields. If writing the failure state also fails, preserve
  that fact in safe console output.

## Review checklist

Review purpose, execution type, trigger, record selection, requirements coverage,
schema freshness, dependencies, inputs, secrets, APIs, reads, mutations, field
types, writeability, idempotency, duplicate ownership, concurrency, batching,
limits, timeout, checkpoints, failure writes, handoffs, installation, and live
test evidence. Report findings before summaries and distinguish untested paths.
