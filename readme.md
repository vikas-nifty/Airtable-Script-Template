# Airtable Scripting Workspace Template

This template is designed for Airtable scripting work where an AI assistant needs a structured, reusable workflow.

## Structure

- AGENTS.md: automatically discovered workspace instructions

- Methods/
  - prompt.md: global prompt instructions
  - codingStyle.md: coding conventions for Airtable scripts

- Context/
  - inputConfig.md: place to store the runtime input configuration and its output
  - baseSchema.md: place to store the compact base schema and its output

- output/
  - plan.md: current plan for the task
  - console_log.md: latest console output
  - script.js: final single script ready to paste into the Airtable script editor
  - script1.js, script2.js, ...: separate scripts when an Automation is split across multiple script actions

## How to Use

1. State the script goal.
2. Capture and verify a fresh base schema in Context/baseSchema.md.
3. Create the plan and table, view, and field dependency map.
4. Configure every dependency in Airtable and verify the logged input configuration in Context/inputConfig.md.
5. Write the script only after those gates pass.
6. Save runtime logs in output/console_log.md and iterate one verified step at a time.

## Notes

The files in Context are treated as authoritative. If a required input or field is missing, pause and ask the user to create or update the relevant context file before continuing.

The root `AGENTS.md` file is automatically discovered by compatible coding agents and directs them to the reusable instructions in `Methods/`.

An Automation may use multiple script actions. Scripts do not share runtime context, so pass values explicitly between actions or persist them in Airtable. Use numbered output files when a workflow is split.
