# Input Config

Use this file to capture the runtime input configuration for the script.

Every table, view, and field in the verified dependency map must be declared in the Airtable input configuration and captured here. Scripts must destructure these references from `input.config()` and must never hard-code their names or IDs.

For multiple Automation script actions, record and label the input configuration for each script separately. Scripts do not share variables or in-memory context.

Command to output the current input config:

```javascript
const inputConfig = input.config();
console.log(JSON.stringify(inputConfig, null, 2));
```

Paste the output below. This file is considered authoritative only after it has been checked against the dependency map in `output/plan.md`. If any required table, view, field, or other input is missing, ask the user to update the Airtable input configuration and this file before writing code.
