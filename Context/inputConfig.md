# Input Config

Every planned table, view, field, and runtime value must be configured in
Airtable and captured here. Label each script separately when an Automation uses
multiple script actions.

Run inside each script action:

```javascript
const inputConfig = input.config();
console.log(JSON.stringify(inputConfig, null, 2));
```

Paste the output below. It becomes authoritative only after it matches the
dependency map in `output/plan.md`. Do not write code while a dependency is
missing or ambiguous.

Scripts do not share variables or in-memory context.
