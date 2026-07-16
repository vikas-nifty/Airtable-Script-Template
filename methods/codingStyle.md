# Airtable JavaScript Coding Standard

Write production Airtable JavaScript that is direct, verifiable, safe to rerun,
and readable in one pass.

## Design before code

- Implement only the verified `output/plan.md`.
- Define source of truth, ownership, invariants, side effects, scale, eligibility,
  checkpoint, rerun behaviour, and realistic failure modes.
- Trace the flow: config → model resolution → query → validation → indexing →
  transformation → external operations → Airtable writes → summary.
- Choose the simplest execution model and script split that satisfies the goal.
- Treat every Automation script action as an isolated process, even though
  Airtable executes actions sequentially in the Automation's configured order.
- For multiple actions, document each action's purpose, inputs, outputs, side
  effects, execution order, and stop/failure behaviour.

## Configuration and schema

- Destructure every used table, view, field, record ID, and runtime setting from
  `input.config()` at the start.
- Never hard-code Airtable table, view, or field names/IDs as literals, constants,
  fallbacks, comments used as configuration, or placeholders.
- Resolve each configured table/view/field once and reuse the model.
- Use the verified field model in reads and computed `[field.id]` keys in writes.
- Do not add speculative dependencies. Stop when code requires an unmapped one.
- Secret names must be documented in the plan. Secret values stay in Airtable
  and must never be logged unmasked or copied into files. Only the input-capture
  helper may emit the documented masked preview.
- Each script action has its own `input.config()`. Repeated keys across actions
  are valid when deliberately configured in each action; never assume one
  action's config or variables are visible to another.

## Record selection

- Use `selectRecordAsync()` only when a verified record ID is available.
- Otherwise use a configured view or scan only required fields and filter with
  explicit eligibility/checkpoint rules.
- Do not infer “the current record” in an Automation.
- Ensure script-written status/error/checkpoint fields do not accidentally
  retrigger the Automation or perpetually requalify records.
- If scanning, make the expected scale and timeout behaviour explicit.

## Validation and ownership

- Validate run-wide configuration and invariants before mutations or API calls.
- Validate record-level required values before marking work as in progress.
- For create/update integrations, define a stable external ID and, where useful,
  a unique business/recovery key.
- Index external IDs and business keys with `Map`/`Set` before writes.
- Detect both directions of duplication:
  - one Airtable record accidentally creating multiple external records;
  - multiple Airtable records claiming one external ID or business key.
- Treat ambiguous ownership as a visible failure; do not choose a winner based on
  record order unless requirements explicitly define that rule.
- Make concurrent and partial runs safe. Unique external keys are useful guards,
  but a rejected duplicate request still needs recovery behaviour.

## Main flow and functions

- Keep the main flow readable from top to bottom.
- Prefer guard clauses and `continue` over deep nesting.
- Extract a function only when it names meaningful business logic, isolates an
  external/failure boundary, or is reused.
- Avoid classes, generic frameworks, speculative abstractions, nested ternaries,
  dense chains, and clever one-liners.
- Use precise business names and one representation for IDs, blanks, statuses,
  dates, and payload values.
- Comment decisions and non-obvious Airtable/API behaviour, not syntax.

## Queries and indexes

- Query only fields the script reads.
- Prefer a configured filtered view when it materially reduces a large set.
- Build `Map`/`Set` indexes for uniqueness, joins, repeated lookup, grouping, or
  deduplication; avoid `.find()`/`.filter()` inside record loops.
- Compute stable values once and avoid unnecessary passes or cell conversions.
- Release query results when useful in memory-sensitive scripts.

## Mutations

- Prepare and validate mutation payloads before writing.
- Write only effective changes.
- Batch Airtable creates, updates, and deletes in groups of at most 50 when
  record-level sequencing is unnecessary.
- Do not parallelize Airtable mutations blindly.
- Sequential `await` inside a loop is acceptable when external API ordering,
  rate limits, per-record failure isolation, or returned IDs require it; state
  the scale and timeout tradeoff in the plan.
- Use configured field models and correct Airtable value shapes.

## External APIs

- Verify endpoint, method, authentication, required headers, field semantics,
  response shape, pagination, rate limits, and error bodies from primary docs.
- Centralize request handling when multiple calls need consistent headers,
  parsing, status checks, and sanitized errors.
- Check `response.ok`; do not treat parsed JSON as proof of success.
- URL-encode query values and JSON-encode request bodies.
- Never log authorization headers, tokens, or unnecessarily complete responses.
- Respect Airtable's approximate HTTP timeout and fetch count. Add retries only
  for documented transient conditions and only when idempotent/safe.

## Failures and observability

- Fail early with plain-English context when the entire run is unsafe.
- Use record-level error boundaries only when partial completion is useful and
  consistent.
- When status/error fields exist, write failures to the relevant source record.
- Keep failure-state writes outside the operation that failed where necessary,
  and handle a failure to write the failure state without hiding the root cause.
- Sanitize and bound stored error text.
- Log concise counters and record identifiers needed for diagnosis; avoid noisy
  payload dumps and sensitive business data.
- Do not claim a path works until live evidence demonstrates it.

## Timeouts and resumption

- For non-trivial workloads, track elapsed time and stop before the Automation
  timeout with a coherent checkpoint.
- Persist state that must survive retries or later actions in Airtable.
- Ensure incomplete records remain eligible for safe recovery.
- Avoid status transitions that strand records in `Syncing` after interruption;
  define stale-work recovery when scale/risk warrants it.

## Multi-action handoffs

- Use `output.set(key, value)` only for a documented later action in the same
  Automation run.
- Configure the later action to receive that output under its verified input key.
- Validate handoff values in the consuming script; do not assume presence, type,
  or successful production.
- Persist state in Airtable instead of an output when it must survive failed
  actions, retries, separate Automations, or later runs.
- Keep numbered script files independently executable with explicit dependencies;
  no script may rely on variables, functions, or imports from another file.

## JavaScript style

- Prefer `const`; use `let` only for reassignment.
- Use destructuring, optional chaining, and nullish coalescing when they improve
  clarity.
- Use array methods for one clear transformation/question and `for...of` for
  branching, accumulation, sequencing, early exits, or error handling.
- Avoid large spread copies, unclear shared mutation, and implicit coercion at
  business-rule boundaries.
- Keep lines and functions readable in Airtable's editor.

## Final verification

- Requirements remain user-owned and unchanged.
- Plan matches the implemented behaviour and dependencies.
- Schema and input captures are current.
- Every used Airtable model comes from verified input configuration.
- Record selection, field types, and computed-field dependencies are valid.
- Ownership, duplicates, concurrency, reruns, and partial failures are handled.
- Queries, API calls, and writes fit expected scale and platform limits.
- Unmasked secrets and sensitive payloads are absent from logs and repository
  files; masked previews appear only in input-capture output.
- Static syntax checks pass.
- Tests cover create/update/error/recovery and every requirement-specific path;
  untested behaviour is labelled as such.

Prefer boring, explicit, measurable correctness over sophisticated-looking code.
