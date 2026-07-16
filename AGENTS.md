# Airtable Scripting Workspace

This repository is a reusable starting point for Airtable scripting projects.

Work in small, verified stages. Do not complete the whole workflow in one step:

1. Read and follow `Methods/prompt.md`.
2. Read and follow `Methods/codingStyle.md`.
3. After the user states the goal, ask them to fetch the latest base schema and store its output in `Context/baseSchema.md`.
4. Verify the schema, create `output/plan.md`, and list every table, view, and field dependency that must be mapped in Airtable input configuration.
5. Ask the user to configure those dependencies and paste the logged `input.config()` output into `Context/inputConfig.md`.
6. Verify the input configuration, update the plan, and only then write code.

Every table, view, and field used by a script must be declared through
`input.config()` and destructured from its returned object. Never hard-code a
table name, table ID, view name, view ID, field name, or field ID in the script.

If another table, view, or field becomes necessary, stop and repeat the input
configuration verification before outputting more code.

Write the current plan to `output/plan.md` and the latest runtime output to
`output/console_log.md`. Write a single paste-ready script to `output/script.js`.
When an Automation is better split across multiple script actions, use
`output/script1.js`, `output/script2.js`, and so on. Scripts do not share runtime
context; each script must receive its own inputs, and required values must be
passed explicitly between Automation actions or persisted in Airtable.

Keep responses small and code compact.
