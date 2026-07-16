# Airtable Scripting Workspace Template

A reusable, gated workspace for building Airtable scripts with verified schema
and input dependencies.

## Structure

- `AGENTS.md`: automatically discovered operating instructions.
- `Methods/prompt.md`: workflow and Airtable constraints.
- `Methods/codingStyle.md`: compact production coding standards.
- `Context/baseSchema.md`: fresh tables, views, and fields from the target base.
- `Context/inputConfig.md`: verified inputs for each script.
- `output/plan.md`: goal, gates, dependencies, and script handoffs.
- `output/console_log.md`: latest labelled runtime output.
- `output/script.js`: one final script.
- `output/script1.js`, `output/script2.js`, ...: separate Automation script actions.

## Workflow

1. State the goal.
2. Capture and verify a fresh base schema.
3. Plan the script and map every dependency.
4. Configure and verify all Airtable inputs.
5. Write code.
6. Test and revise one verified step at a time.

Context files are authoritative. Never hard-code table, view, or field names or
IDs. If a new dependency appears, update and verify the input configuration
before continuing.

Automation scripts share no runtime context. Pass values explicitly between
actions or persist them in Airtable.
