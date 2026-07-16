# Airtable Scripting Workspace Prompt

Use this workspace to design, build, review, and refine one Airtable script.

## Source of truth

- Treat `Context/inputConfig.md` and `Context/baseSchema.md` as authoritative for the given base and script you're about to build/are building.
- Do not invent or infer table names, view names, field names, field types, record IDs, input variables, secrets, or configuration values that are absent from those files.
- Every table, view, and field referenced by the script must be declared in `input.config()` and recorded in `Context/inputConfig.md`.
- Destructure every table, view, and field reference from the object returned by `input.config()`.
- Never declare a table name, table ID, view name, view ID, field name, or field ID in the script itself, whether as a string literal, constant, fallback, or placeholder.
- Use configured Airtable models directly when supplied. If an input supplies an identifier, resolve it once after destructuring and reuse the resolved model.
- Use information already supplied by the user or recorded in the workspace. Do not ask for it again.
- If any required table, view, field, or other input is missing, ask the user to add it to the Airtable input configuration and update `Context/inputConfig.md` before proceeding.
- If a required table, view, field, field type, option, or relationship is missing, ask the user to create or confirm it in Airtable and update `Context/baseSchema.md`.
- Do not work around missing schema or configuration with guessed names or temporary placeholders.

## Workspace files

- `Methods/prompt.md` defines the workspace workflow and Airtable reference information.
- `Methods/codingStyle.md` defines how the JavaScript should be written.
- `Context/inputConfig.md` contains the command and captured runtime input configuration.
- `Context/baseSchema.md` contains the command and captured compact base schema.
- `output/plan.md` contains the current task-specific plan.
- `output/console_log.md` contains the most recent runtime output supplied by the user.
- `output/script.js` contains a complete single script to copy into Airtable.
- `output/script1.js`, `output/script2.js`, and so on contain separate Automation scripts when splitting the work is clearer or safer.

## Required workflow

This is a gated workflow. Complete and verify one stage before moving to the next. Keep each response short and ask only for the next required action.

1. **Goal:** Read the user's requested outcome. Determine the likely execution type, inputs, outputs, and side effects without writing code.
2. **Fresh schema:** Ask the user to run the command in `Context/baseSchema.md` against the target base and paste the latest output into that same file. Do not rely on an older schema for a new goal.
3. **Schema verification:** Verify that the captured schema is present, readable, and sufficient to assess the goal. If Airtable schema changes are required, ask the user to make them and refresh `Context/baseSchema.md`.
4. **Initial plan:** Create a concise task-specific `output/plan.md`. Include the intended flow, execution type, mutations, and whether one script or multiple Automation scripts are appropriate.
5. **Dependency map:** In the plan, list every table, view, and field the script will reference. For multiple scripts, group dependencies by script. Each dependency must be mapped through Airtable input configuration.
6. **Input configuration:** Ask the user to add every listed dependency to the relevant script action's input configuration and paste the logged output into `Context/inputConfig.md`.
7. **Input verification:** Verify that the logged input configuration contains every planned table, view, field, and other required runtime value. Do not write code while any dependency is missing or ambiguous.
8. **Plan confirmation:** Update `output/plan.md` to reflect the verified mappings and resolve each dependency to the exact input key that will supply it.
9. **Code:** Only after schema and input verification, write or revise the complete JavaScript following `Methods/codingStyle.md`. Use `output/script.js` for one script or numbered files such as `output/script1.js` and `output/script2.js` for multiple Automation actions.
10. **Test and iterate:** When console output is supplied, store the latest relevant output in `output/console_log.md`, diagnose it, update the plan if needed, and make the smallest justified revision.
11. **New dependency gate:** If implementation or testing reveals another required table, view, or field, stop coding. Update the dependency map, ask the user to update Airtable input configuration and `Context/inputConfig.md`, verify it, then resume.
12. **Final verification:** Check every script against the authoritative schema and its verified input configuration. Reject hard-coded table, view, or field references.

Automation script actions may be split when one script becomes too large or when separate stages improve reliability and clarity. Each script runs in isolation and carries no JavaScript variables or in-memory context into another script. Pass required values explicitly through Automation outputs and inputs, or persist them in Airtable. Each script must have its own complete input dependency mapping.

## Airtable core reference

Tables/views: `base.getTable('Table')`, `table.getView('View')`

Query one: `table.selectRecordAsync('recXXX')`

Query many: `table.selectRecordsAsync({fields:['Field1'],sorts:[{field:'Field1'}]})`

Raw value: `record.getCellValue('Field')`

Display value: `record.getCellValueAsString('Field')`

Multi-select: `record.getCellValue('Field')?.[0]?.name`

Create: `table.createRecordAsync({fields:{Field:'Value'}})`

Batch create: `table.createRecordsAsync([{fields:{...}}])`

Update: `table.updateRecordAsync('recXXX',{Field:'Value'})`

Batch update: `table.updateRecordsAsync([{id:'recXXX',fields:{...}}])`

Extension HTTP: `remoteFetchAsync(url,options)`

Automation HTTP: `fetch(url,options)`

`filterByFormula` is unavailable in `selectRecordsAsync`. Use a filtered Airtable view or filter loaded records in JavaScript. `filterByFormula` is valid when calling Airtable's external REST API.

## Execution types

1. **Scripting Extension RUN button:** Manual run; `input.config()` or secrets unsupported.
2. **Record button configured to Run a Script:** Runs in Scripting Extension; `input.recordAsync()` optionally retrieves the clicked record.
3. **Automation Run a Script:** Create variables in Input Variables, then use:

   ```javascript
   const data = input.config();
   const value = data.inputValue;
   ```

Never use `input.recordAsync()` in automation scripts.

Automation secrets: `const key = input.secret('Secret Name');`

Automation logging: `console.log`, `console.warn`, `console.error`.

Use `output.set('key', value)` only to pass data to later automation actions. Do not use extension-only output methods in automations.

## Limits

Treat these limits as authoritative constraints:

- Each automation script action: 180 seconds. When run in an Extension, and not Automation, timeout is 40 minutes.
- One automation may contain multiple script actions and run for up to 40 minutes total.
- HTTP timeout: approximately 12 seconds.
- Memory: approximately 512 MB.
- Fetches: approximately 50.
- Queries: approximately 30.
- Record mutations: maximum 50 records per batch; mutations are rate-limited to 15 mutations per second.
- External packages such as `crypto` are not supported in Airtable.

For large datasets, batch work, track elapsed time, stop before timeout, and use status, cursor, checkpoint, or flag fields so later actions or runs can resume. Scripts should be safe to rerun and avoid duplicates.

## Field reference

- Linked records: `[{id:'recXXX'}]`
- Multiple links: an array of `{id}` objects.
- Single select: `{name:'Option'}`
- Multiple select: `[{name:'Option 1'},{name:'Option 2'}]`
- Checkbox: boolean.
- Dates: use formats and timezones appropriate to the field.
- Attachments: use Airtable-supported attachment objects. Scripts can read the contents of simple file types such as TXT and CSV.
- Formula, lookup, rollup, created-time, and other computed fields are generally read-only.
- Linked record fields are two-way; identify the most efficient side to update.
- Always confirm field types before writing.
- Field IDs do not survive base duplication; field names do not survive field renaming.

## Environment safeguards

- Use Airtable JavaScript, not Python.
- Do not use unsupported libraries.
- Do not use `remoteFetchAsync` in automations.
- Do not use `filterByFormula` in `selectRecordsAsync`.
- Do not confuse external REST API capabilities with scripting capabilities.
- Do not use `input.recordAsync()` in automations.
- Read automation variables through `input.config()`.
- Do not write linked record IDs as strings.
- Do not exceed 50 records per mutation batch.
- Do not assume field names, types, or relationships.
- Do not use unsupported output methods.
- Never log secrets, tokens, passwords, or sensitive full payloads.

## Script reviews

When asked to review or explain a script, identify its purpose, execution type, inputs, outputs, tables, views, fields, secrets, APIs, workflow, mutations, batching, timeout risks, duplicate risks, resume behaviour, likely bugs, and installation requirements. Explain at an undergraduate level unless the user requests more depth.
