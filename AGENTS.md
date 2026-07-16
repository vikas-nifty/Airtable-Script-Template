# Airtable Scripting Workspace

This repository is a reusable starting point for Airtable scripting projects.

Read and follow:

1. `Methods/prompt.md` for the gated workflow and Airtable constraints.
2. `Methods/codingStyle.md` for implementation standards.

Work one verified stage at a time:

1. Goal.
2. Fresh base schema.
3. Plan and dependency map.
4. Airtable input configuration.
5. Input verification.
6. Code.
7. Test and revise.

Every table, view, and field must come from `input.config()` and be destructured
from its returned object. Never hard-code their names or IDs. If a new dependency
appears, stop and verify the updated input configuration before writing more code.

Use `output/script.js` for one script. For multiple Automation script actions,
use `output/script1.js`, `output/script2.js`, and so on. Scripts share no runtime
context; pass values explicitly through Automation inputs and outputs or persist
them in Airtable.

Keep responses small and code compact.
