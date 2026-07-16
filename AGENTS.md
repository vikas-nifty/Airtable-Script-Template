# Airtable Scripting Workspace

This repository is a reusable workspace for designing, building, testing, and
reviewing Airtable JavaScript.

## Read first

1. `context/requirements.md` — user-owned source requirements.
2. `methods/prompt.md` — immutable high-level workflow and Airtable constraints.
3. `methods/codingStyle.md` — implementation and review standards.
4. `context/baseSchema.md` and `context/inputConfig.md` — fresh, user-supplied
   facts from the target Airtable base and script action.

## Ownership

- Only the user may create or modify `context/requirements.md`. Never rewrite,
  normalize, complete, or check off requirements on the user's behalf.
- Codex maintains `output/plan.md` as the required interpretation of the
  requirements, conversation decisions, schema, dependencies, risks, and tests.
- The user refreshes `context/baseSchema.md` and `context/inputConfig.md` by
  running their embedded capture scripts in Airtable and pasting the latest
  output. Codex may improve the capture templates but must not fabricate output.
- Runtime logs and test evidence are supplied in chat, not stored in the
  repository.

## Mandatory gates

Work one verified stage at a time:

1. Requirements and goal.
2. Fresh base schema.
3. Plan, invariants, record-selection strategy, and dependency map.
4. Airtable input and secret configuration.
5. Exact input verification.
6. Code.
7. Test, evidence, and smallest justified revision.

Do not write code before schema and inputs are verified. If a new table, view,
field, secret, runtime value, handoff, or state field appears, return to the
appropriate configuration gate before continuing.

## Airtable dependencies

- Every table, view, and field used by a script must be supplied through
  `input.config()` and destructured at the start. Never hard-code their names or
  IDs in executable code.
- Resolve configured IDs once with `base.getTable()` / `table.getField()` and
  reuse the resulting models.
- Use input keys in `table_field` form, with lower camel case inside each segment,
  for example `projects_startDate`. Use `scope_setting` for non-schema runtime
  values, for example `api_baseUrl`.
- Decide explicitly how records are selected. Do not assume a triggering record
  ID exists. Choose a configured record ID, configured view, or a table scan with
  verified eligibility/checkpoint fields.

## Output files

- One script action: `output/script.js`.
- Multiple script actions in one Automation: `output/script1.js`,
  `output/script2.js`, and so on, matching their execution order.
- Airtable runs actions sequentially in the order configured in the Automation,
  but scripts share no runtime memory or common `input.config()` object.
- Each script action has its own input configuration. Actions may repeat the same
  keys and values when they share dependencies, or use separate action-specific
  inputs when their responsibilities differ.
- Use documented `output.set()` values as inputs to later actions for immediate
  handoffs. Persist state in Airtable when it must survive retries, later runs,
  or execution outside that Automation.

## Git publishing

When pushing the project to Git, include a one-sentence summary of the changes
compared with the previous commit.

Keep responses focused and code compact, but never skip a verification gate.
