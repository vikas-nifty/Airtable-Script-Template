# Airtable Scripting Workflow

Use this workspace to design, build, review, and refine an Airtable scripting project.

## Authority

- Treat `Context/baseSchema.md` and `Context/inputConfig.md` as authoritative.
- Do not invent tables, views, fields, types, relationships, record IDs, inputs, secrets, or configuration values.
- Every table, view, and field used by a script must be mapped in Airtable input configuration and recorded in `Context/inputConfig.md`.
- Destructure these dependencies from `input.config()`. Never hard-code their names or IDs as literals, constants, fallbacks, or placeholders.
- Use configured Airtable models directly. If an input supplies an identifier, resolve it once after destructuring and reuse it.
- If required schema is missing, ask the user to update Airtable and refresh `Context/baseSchema.md`.
- If a required dependency is missing from input configuration, ask the user to add it and refresh `Context/inputConfig.md`.

## Files

- `Context/baseSchema.md`: latest compact schema for the target base.
- `Context/inputConfig.md`: logged input configuration, labelled per script when needed.
- `output/plan.md`: current goal, gates, dependency map, and inter-script handoffs.
- `output/console_log.md`: latest console output, labelled by script.
- `output/script.js`: single completed script.
- `output/script1.js`, `output/script2.js`, ...: separate Automation script actions.

## Gated workflow

Complete and verify one stage before moving to the next. Keep each response short and request only the next required action.

1. **Goal:** Establish the outcome, execution type, inputs, outputs, side effects, and expected scale. Do not write code.
2. **Schema:** Ask the user to run the command in `Context/baseSchema.md` against the target base and paste the fresh output there.
3. **Plan:** Verify the schema, then update `output/plan.md` with the approach, mutations, script split, and every required table, view, field, and runtime value.
4. **Configure:** Ask the user to map each dependency in the relevant Airtable script action and paste the logged `input.config()` output into `Context/inputConfig.md`.
5. **Verify:** Match every planned dependency to an exact input key. Do not write code while anything is missing or ambiguous.
6. **Code:** Update the plan, then write the complete script or numbered scripts following `Methods/codingStyle.md`.
7. **Test:** Store supplied runtime output in `output/console_log.md`, diagnose it, update the plan when needed, and make the smallest justified revision.

If a new table, view, or field becomes necessary at any later stage, return to Configure and Verify before continuing.

## Multiple Automation scripts

Split an Automation when separate script actions improve size, clarity, reliability, or runtime management. Each script is isolated:

- Give each script its own dependency map and verified input configuration.
- Record every value passed through `output.set()` and the later input key that consumes it.
- Persist shared or resumable state in Airtable when it should survive between actions or runs.

## Airtable reference

- Query one: `table.selectRecordAsync(recordId)`
- Query many: `table.selectRecordsAsync({ fields: [field], sorts: [{ field }] })`
- Raw value: `record.getCellValue(field)`
- Display value: `record.getCellValueAsString(field)`
- Create: `table.createRecordAsync({ fields: { [field.id]: value } })`
- Batch create: `table.createRecordsAsync(records)`
- Update: `table.updateRecordAsync(recordId, { [field.id]: value })`
- Batch update: `table.updateRecordsAsync(updates)`
- Extension HTTP: `remoteFetchAsync(url, options)`
- Automation HTTP: `fetch(url, options)`

`filterByFormula` is unavailable in `selectRecordsAsync`. Use a configured filtered view or filter loaded records in JavaScript. It is valid when calling Airtable's external REST API.

### Execution types

1. **Scripting Extension:** Manual run; `input.config()` or secrets unsupported.
2. **Record button:** Runs in Scripting Extension; `input.recordAsync()` may retrieve the clicked record.
3. **Automation:** Configure Input Variables and read them through `input.config()`. Never use `input.recordAsync()`.

Automation secrets: `input.secret('Secret Name')`

Automation logging: `console.log`, `console.warn`, `console.error`

Use `output.set(key, value)` only to pass data to later Automation actions.

### Authoritative limits

- Each Automation script action: 180 seconds. A Scripting Extension run has a 40-minute timeout.
- One Automation may contain multiple script actions and run for up to 40 minutes total.
- HTTP timeout: approximately 12 seconds.
- Memory: approximately 512 MB.
- Fetches: approximately 50.
- Queries: approximately 30.
- Record mutations: no more than 50 records per batch and 15 mutations per second.
- External packages such as `crypto` are unsupported.

For large datasets, batch work, track elapsed time, stop before timeout, and use status, cursor, checkpoint, or flag fields for resumption. Scripts should be safe to rerun and avoid duplicates.

### Field values

- Linked records: `[{ id: recordId }]`
- Multiple links: array of `{ id }` objects.
- Single select: `{ name: option }`
- Multiple select: `[{ name: option1 }, { name: option2 }]`
- Checkbox: boolean.
- Dates: use formats and timezones appropriate to the field.
- Attachments: use Airtable-supported attachment objects; scripts can read simple file types such as TXT and CSV.
- Formula, lookup, rollup, created-time, and other computed fields are generally read-only.
- Linked record fields are two-way; update the most efficient side.
- Confirm field types before writing.
- Field IDs do not survive base duplication; field names do not survive renaming.

## Safeguards

- Use Airtable JavaScript, not Python or unsupported libraries.
- Do not use `remoteFetchAsync` in Automations.
- Do not confuse scripting capabilities with the external REST API.
- Do not write linked record IDs as strings.
- Do not exceed mutation batch limits.
- Never log secrets, tokens, passwords, or sensitive full payloads.

When reviewing a script, cover its purpose, execution type, dependencies, inputs, outputs, secrets, APIs, mutations, batching, timeout and duplicate risks, handoffs, resume behaviour, likely bugs, and installation requirements.
